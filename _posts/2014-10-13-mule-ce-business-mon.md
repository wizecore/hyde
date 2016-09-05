---
published: false
layout: post
---
## Mule business event logging via ServerNotificationListener

Mule CE 3.5.0 does not have any sort of business monitoring. Only logs with lots of technical information.

So you must setup your own infrastructure, for example using Graylog2 or Logstash + Kibana.

But first of all how to get high level information about flows and events in your Mule ESB?

1) Do not init listeners via registry-bootstrap.properties as described in Start Me Oh So Gently

The MuleContext instance and internal NotificationManager and it`s Policy field will be overwritten later in process with new instance (this only applies to running in standalone mode, somehow running flow in IDE does not produce such behaviour)

2) Use standard syntax in your flow XML file:

```
<spring:bean name="logger1" class="class implementing MessageProcessorNotificationListener"/>
<spring:bean name="logger1" class="class implementing EndpointMessageNotification"/>

<notifications dynamic="true">
  <notification event="MESSAGE-PROCESSOR"/>
  <notification event="ENDPOINT-MESSAGE"/>
  <notification-listener ref="logger1"/>
  <notification-listener ref="logger1"/>
</notifications>
```

I recommend listening at least on these two events (MessageProcessorNotification and EndpointMessageNotification). The first one produces information about message being processed and sent to outbound endpoints. The second one generate event with information about message received from inbound endpoint.

3) Do the code to extract information from events


```java
public class EMLogger implements EndpointMessageNotificationListener<EndpointMessageNotification> {
  private Logger log = Logger.getLogger(getClass().getName());
  
  public void onNotification(EndpointMessageNotification m) {
    MuleMessage msg = m.getSource();  
    Object payload = msg.getPayload();
    String ref = payload != null ? payload.toString() : "null";
  
    boolean inc = true;
    if (m.getAction() != EndpointMessageNotification.MESSAGE_RECEIVED) {
      inc = false;
    }
  
    if (inc) {
      log.warn(msg.getMessageRootId() + " [INBOUND] Message " + ref + " <- " + m.getEndpoint());
    }
  }
}

public class MPLogger implements MessageProcessorNotificationListener<MessageProcessorNotification> {
  private Logger log = Logger.getLogger(getClass().getName());
  
  public void onNotification(MessageProcessorNotification m) {
    MuleEvent ev = m.getSource();
    MuleMessage msg = ev.getMessage();
    Object payload = msg.getPayload();
    String ref = payload != null ? payload.toString() : "null";

    MessageProcessor proc = m.getProcessor();
    boolean inc = true;
    inc = inc && (proc instanceof LoggerMessageProcessor ? false : true); 
    inc = inc && (proc instanceof AsyncDelegateMessageProcessor ? false : true);

    if (m.getAction() != MessageProcessorNotification.MESSAGE_PROCESSOR_PRE_INVOKE) {
      inc = false;
    }

    boolean outgoing = proc instanceof DefaultOutboundEndpoint;

    if (inc) {
      if (outgoing) {
        AbstractEndpoint ep = (AbstractEndpoint) proc;
        log.warn(msg.getMessageRootId() + " [OUTBOUND] Message " + ref + " -> " + ep.getEndpointURI());
      } else {
        log.warn(msg.getMessageRootId() + " [PROCESSING] Message " + ref + " <> " + proc.getClass().getSimpleName());
      }
    }
  }
}
```

4) Write it to different Log4j logger and separate file (mule-standalone-3.5.0/conf/log4j.properties)

```
log4j.appender.my = org.apache.log4j.FileAppender
log4j.appender.my.layout = org.apache.log4j.PatternLayout
log4j.appender.my.layout.ConversionPattern = %d %m%n
log4j.appender.my.Append = true
log4j.appender.gpb.File = ../logs/my.log

log4j.logger.package.with.listeners = INFO, my
log4j.additivity.package.with.listeners = false
```
