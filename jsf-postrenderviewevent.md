# PostRenderViewEvent 

JSF 2.3 added a new `ComponentSystemEvent`, the so-called `PostRenderViewEvent`, it will be fired after the view is rendered in the `RENDER_RESPONSE` phase.

Create a sample to taste it.

Firstly create a backing bean.

```java
@Model
public class PostRenderViewBean {

    @Inject
    Logger LOG;

    public void init(PreRenderViewEvent e) {
        LOG.log(Level.INFO, "fire PreRenderViewEvent:" + e);
    }

    public void execute() {
        LOG.log(Level.INFO, "execute viewAction");
    }

    public void initialized(PostRenderViewEvent e) {
        LOG.log(Level.INFO, "fire PostRenderViewEvent:" + e);
    }
}	
```

In the facelets template, use `f:event` to declare events, and set type to `postRenderView` and listener to `#{postRenderViewBean.initialized}`.

```xml
<f:metadata>
	<f:event type="preRenderView" listener="#{postRenderViewBean.init}" />
	<f:viewAction action="#{postRenderViewBean.execute}" />
	<f:event type="postRenderView" listener="#{postRenderViewBean.initialized}"/>
</f:metadata>
```

`viewAction` was added in Java EE 7, check my [Java EE 7 sample](https://github.com/hantsy/ee7-sandbox/wiki/jsf-view-action) for details.

Run this application on Glassfish v5, open browser and navigate to [http://localhost:8080/jsf-postrenderview-event/postRenderView.faces](http://localhost:8080/jsf-postrenderview-event/postRenderView.faces);

You will see the info in the IDE console.

```
Info:   before phase:RESTORE_VIEW 1
Info:   after phase:RESTORE_VIEW 1
Info:   invoking custom ExceptionHandlder...
Info:   before phase:APPLY_REQUEST_VALUES 2
Info:   after phase:APPLY_REQUEST_VALUES 2
Info:   invoking custom ExceptionHandlder...
Info:   before phase:PROCESS_VALIDATIONS 3
Info:   after phase:PROCESS_VALIDATIONS 3
Info:   invoking custom ExceptionHandlder...
Info:   before phase:UPDATE_MODEL_VALUES 4
Info:   after phase:UPDATE_MODEL_VALUES 4
Info:   invoking custom ExceptionHandlder...
Info:   before phase:INVOKE_APPLICATION 5
Info:   execute viewAction
Info:   after phase:INVOKE_APPLICATION 5
Info:   invoking custom ExceptionHandlder...
Info:   before phase:RENDER_RESPONSE 6
Info:   fire PreRenderViewEvent:javax.faces.event.PreRenderViewEvent[source=javax.faces.component.UIViewRoot@248724dd]
Info:   fire PostRenderViewEvent:javax.faces.event.PostRenderViewEvent[source=javax.faces.component.UIViewRoot@248724dd]
Info:   after phase:RENDER_RESPONSE 6
Info:   invoking custom ExceptionHandlder...
```

The `viewAction` is executed in `INVOKE_APPLICATION` phase, and `PreRenderViewEvent` and `PostRenderViewEvent` are fired in `RENDER_RESPONSE` phase.

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.
