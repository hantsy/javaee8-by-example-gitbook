# Server Sent Event

Jersey itself supports SSE for years, now it is standardized as a part of JAXRS 2.1.

A simple SSE example.

```java
@Path("events")
@RequestScoped
public class SseResource {

    @GET
    @Produces(MediaType.SERVER_SENT_EVENTS)
    public void eventStream(@Context Sse sse, @Context SseEventSink eventSink) {
        // Resource method is invoked when a client subscribes to an event stream.
        // That implies that sending events will most likely happen from different
        // context - thread / event handler / etc, so common implementation of the
        // resource method will store the eventSink instance and the application 
        // logic will retrieve it when an event should be emitted to the client.

        // sending events:
        eventSink.send(sse.newEvent("event1"));
    }    
}
```

Notice, you should declare `@Produces` value as _text/event-stream_\( via `MediaType.SERVER_SENT_EVENTS`\). `SseEventSink` should be request aware, `Sse` is helpful to build event payloads etc.

An example of SSE client.

```java
WebTarget target = ClientBuilder.newClient().target("http://localhost:8080/jaxrs-sse/rest/events");

try (SseEventSource eventSource = SseEventSource.target(target).build()) {

    // EventSource#register(Consumer<InboundSseEvent>)
    // Registered event handler will print the received message.
    eventSource.register(System.out::println);

    // Subscribe to the event stream.
    eventSource.open();

    // Consume events for just 500 ms
    Thread.sleep(500);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

The following is an example demonstrates the usage of `SseBroadcaster`, which allow you send messages to all registered clients.

```java
@Path("broadcastEvents")
@Singleton
public class BroadcastResource {

    @Context
    private Sse sse;

    private volatile SseBroadcaster sseBroadcaster;

    @PostConstruct
    public void init() {
        this.sseBroadcaster = sse.newBroadcaster();
    }

    @GET
    //@Path("register")
    @Produces(MediaType.SERVER_SENT_EVENTS)
    public void register(@Context SseEventSink eventSink) {
        eventSink.send(sse.newEvent("welcome!"));
        sseBroadcaster.register(eventSink);
    }

    @POST
    //@Path("broadcast")
    @Consumes(MediaType.APPLICATION_FORM_URLENCODED)
    public void broadcast(@FormParam("event") String event) {
        sseBroadcaster.broadcast(sse.newEvent(event));
    }

    public void eventStreamCdi(@Observes Message msg) {
        sseBroadcaster.broadcast(
                sse.newEventBuilder()
                        .mediaType(MediaType.APPLICATION_JSON_TYPE)
                        .id(UUID.randomUUID().toString())
                        .name("message from cdi")
                        .data(msg)
                        .build()
        );
    }
}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.

