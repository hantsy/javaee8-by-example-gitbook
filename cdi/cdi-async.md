# Async Events

CDI 2.0 add the capability of firing an CDI event asynchronously, this will free the client from long-awaited blocking request.

Create a simple backing bean.

```java
@ViewScoped
@Named("eventBean")
public class EventBean implements Serializable {

    private static final Logger LOG = Logger.getLogger(EventBean.class.getName());

    @Inject
    Event<Message> event;

    private String message;

    private String notification;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getNotification() {
        return notification;
    }

    public void setNotification(String notification) {
        this.notification = notification;
    }


    public void fireEvent() {
        LOG.log(Level.INFO, "fire event async...");
        event.fireAsync(new Message(this.message))
                .thenAccept((m) -> this.setNotification(m+ " was sent")); 
    }

    public void fireEventOptions() {
        LOG.log(Level.INFO, "fire event async...");
        event.fireAsync(
                new Message(this.message),
                NotificationOptions.builder()
                        .set("weld.async.notification.timeout", 1000)
                        .build()
        );
    }
}
```

CDI 2.0 added two methods to fire a CDI event asynchronously, `fireAsync` will return a `CompletionStage`, it could be used in further steps, `fireAsync` has a variant which an extra `NotificationOptions` argument.

```java
@ApplicationScoped
public class EventHandler implements Serializable {

    public static final Logger LOG = Logger.getLogger(EventHandler.class.getName());

    public void onMessage(@ObservesAsync Message message) {
        LOG.log(Level.INFO, "observes event:{0}", message);
    }
}
```

In the `EventHandler`, use `@ObservesAsync` to observe the incoming event.

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.

