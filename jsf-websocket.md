# Websocket support

One of the most attractive features is JSF 2.3 added native websocket support, it means you can write real-time applications with JSF and no need extra effort.

To enable websocket support, you have to add `javax.faces.ENABLE_WEBSOCKET_ENDPOINT` in *web.xml*.

```xml
<context-param>
	<param-name>javax.faces.ENABLE_WEBSOCKET_ENDPOINT</param-name>
	<param-value>true</param-value>
</context-param> 
```

## Hello Websocket

Let's start with a simple example.

```java
@ViewScoped
@Named("helloBean")
public class HelloBean implements Serializable {
    
    private static final Logger LOG = Logger.getLogger(HelloBean.class.getName());
    
    @Inject
    @Push
    PushContext helloChannel;
    
    String message;
    
    public void sendMessage() {
        LOG.log(Level.INFO, "send push message");
        this.sendPushMessage("hello");
    }
    
    private void sendPushMessage(Object message) {
        helloChannel.send("" + message + " at " + LocalDateTime.now());
    }
    
    public String getMessage() {
        return message;
    }
    
    public void setMessage(String message) {
        this.message = message;
    }
    
    public void sendMessage2() {
       // log.log(Level.INFO, "send push message from input box::" + this.message);
        this.sendPushMessage(this.message);
    }
    
}
```

In the backing bean, inject a `PushContext` with `@Push` qualifier.

```xml
<!DOCTYPE html>
<html lang="en" 
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:jsf="http://xmlns.jcp.org/jsf"
      xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
      xmlns:f="http://xmlns.jcp.org/jsf/core"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      >
    <h:head>
        <title>JSF 2.3: Websocket Sample</title>
        <script>
            function onMessage(message, channel, event) {
                console.log('jsf push message::' + message + ", channel ::" + channel + ", event::" + event);
                document.getElementById("message").innerHTML = message;
            }
        </script>    
    </h:head>
    <h:body>
        <h1>JSF 2.3: Hello Websocket </h1>
        <div id="message" />
        <hr />
        <h:form id="form">
            <div>
                <h:commandButton 
                    id="sendMessage" 
                    type="submit"
                    action="#{helloBean.sendMessage()}" value="Send Message">
                    <f:ajax />
                </h:commandButton>
            </div>
            <div>
                <h:outputLabel for="messageInput" value="Say hi to JSF Websocket" />
                <h:inputText 
                    id="messageInput" 
                    value="#{helloBean.message}"/>

            </div>
            <h:panelGroup layout="block" id="messageFromInputBox">
                Input text is :: #{helloBean.message}
            </h:panelGroup>
            <div>
                <h:commandButton 
                    id="sendMessage2" 
                    action="#{helloBean.sendMessage2()}" 
                    value="Send Message from Input Box">
                    <f:ajax execute="@form" render="messageFromInputBox" />
                </h:commandButton>
            </div>
            <div>
                <button 
                    jsf:id="sendMessage3" 
                    jsf:action="#{helloBean.sendMessage2()}" >
                    Send Message from Input Box(HTML 5 Button)
                    <f:ajax execute="@form" render="messageFromInputBox" />
                </button>
            </div>
        </h:form>
        <f:websocket channel="helloChannel" onmessage="onMessage" />
    </h:body> 
</html>
```

The first button sends a fixed **hello** string, and the second button accepts user custom message. 

`sendMessage` and `sendMessage2` will call `sendPushMessage` which utilizes the injected `pushContext` to send messages to the defined channel. 

In the facelets template, `onmessage` indicates which handlers(`onMessage` method) will be used when messages is comming from the specified channel(`helloChannel`).

`onMessage` method will add the message body from websocket channel and put it into the content of **message** block.

Run this application on Glassfish, open your browser and navigate to [http://localhost:8080/jsf-websocket/hello.faces](http://localhost:8080/jsf-websocket/hello.faces).

![jsf websocket hello](jsf-websocket-hello.png)

Try click first button or input some messages and click the second button, the response message from server side will be displayed.

## Scope 

`f:websocket` has an attribute `scope` which accepts **application**, **session**, or **view** as its value, it is not difficult to understand.

Another attribute `user` can be used for identifying different client users, it can be a user id or serializable object that stands for a user. It is useful to communicate with a specific user.

Create a backing bean.

```java
@ViewScoped
@Named("scopeBean")
public class ScopeBean implements Serializable {

    private static final Logger LOG = Logger.getLogger(ScopeBean.class.getName());

    @Inject
    @Push
    PushContext applicationChannel;

    @Inject
    @Push
    PushContext sessionChannel;

    @Inject
    @Push
    PushContext viewChannel;

    @Inject
    @Push
    PushContext userChannel;

    public void pushToApplicationChannel() {
        applicationChannel.send("sent to applicationChannel at ::" + LocalDateTime.now());
    }

    public void pushToSessionChannel() {
        sessionChannel.send("sent to sessionChannel at ::" + LocalDateTime.now());
    }

    public void pushToViewChannel() {
        viewChannel.send("sent to viewChannel at ::" + LocalDateTime.now());
    }

    public void pushToUserChannel() {
        userChannel.send("sent to userChannel at ::" + LocalDateTime.now(), "user");
    }

     public void pushToMultiUsersChannel() {
        userChannel.send("sent to userChannel at ::" + LocalDateTime.now(), Arrays.asList("user", "hantsy"));
    }
}
```

To demonstrate different cases, we defined a series of `PushContext` in the backing bean.

```xml
<!DOCTYPE html>
<html lang="en" 
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:jsf="http://xmlns.jcp.org/jsf"
      xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
      xmlns:f="http://xmlns.jcp.org/jsf/core"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      >
    <h:head>
        <title>JSF 2.3: Websocket Sample</title>
        <script>
            function onMessage(message, channel, event) {
                var m = "message:" + message + ", channel:" + channel + ", event:" + event;
                console.log(m);
                var ul = document.getElementById("messages");
                var li = document.createElement("li");
                li.appendChild(document.createTextNode(m));
                ul.appendChild(li);
            }
        </script>    
    </h:head>
    <h:body>
        <h1>JSF 2.3: Websocket Scopes </h1>
        <ul id="messages" >
        </ul>
        <hr />
        <h:form id="form">
            <div>
                <h:commandButton 
                    id="pushToApplicationChannel" 
                    action="#{scopeBean.pushToApplicationChannel()}" value="pushToApplicationChannel">
                    <f:ajax />
                </h:commandButton>
            </div>

            <div>
                <h:commandButton 
                    id="pushToSessionChannel" 
                    action="#{scopeBean.pushToSessionChannel()}" value="pushToSessionChannel">
                    <f:ajax />
                </h:commandButton>
            </div>

            <div>
                <h:commandButton 
                    id="pushToViewChannel" 
                    action="#{scopeBean.pushToViewChannel()}" value="pushToViewChannel">
                    <f:ajax />
                </h:commandButton>
            </div>

            <div>
                <h:commandButton 
                    id="pushToUserChannel" 
                    action="#{scopeBean.pushToUserChannel()}" value="pushToUserChannel">
                    <f:ajax />
                </h:commandButton>
            </div>

            <div>
                <h:commandButton 
                    id="pushToMultiUsersChannel" 
                    action="#{scopeBean.pushToMultiUsersChannel()}" value="pushToMultiUsersChannel">
                    <f:ajax />
                </h:commandButton>
            </div>

        </h:form>

        <f:websocket channel="applicationChannel" scope="application" onmessage="onMessage" />
        <f:websocket channel="sessionChannel" scope="session" onmessage="onMessage" />
        <f:websocket channel="viewChannel" scope="view" onmessage="onMessage" />
        <f:websocket channel="userChannel" user="hantsy" onmessage="onMessage" />
        <f:websocket channel="userChannel" user="user" onmessage="onMessage" />
    </h:body> 
</html>
```

Run the application on Glassfish, open your browser and navigate to [http://localhost:8080/jsf-websocket/scope.faces](http://localhost:8080/jsf-websocket/scope.faces).

## Event 

JSF 2.3 provides a `WebsocketEvent`, in the backend codes, you can observe it when it is opened (via CDI `@Opened` qualifier) or closed(via CDI `@Closed` qualifier).

```java
@ApplicationScoped
public class WebsocketObserver {

    @Inject
    Logger LOG;

    public void onOpen(@Observes @Opened WebsocketEvent opened) {
        LOG.log(Level.INFO, "event opend::{0}", opened);
    }

    public void onClose(@Observes @Closed WebsocketEvent closed) {
        LOG.log(Level.INFO, "event closed::{0}", closed);
    }

}
```

Besides `onmessage` attribute of `f:websocket`, it also provides other two attributes `onopen` and `onclose` to listen the websocket connection when it is opened or closed. `connected` allow you set it disconnected by default, and use JSF built-in `jsf.push.open()` to connect to server side manually.

Create a simple backing bean.

```java
@ViewScoped
@Named("eventBean")
public class EventBean implements Serializable {

    private static final Logger LOG = Logger.getLogger(EventBean.class.getName());

    @Inject
    @Push
    PushContext eventChannel;

    public void sendMessage() {
        eventChannel.send("event message was sent at::" + LocalDateTime.now());
    }
}
```

The facelets template file:

```xml
<!DOCTYPE html>
<html lang="en" 
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:jsf="http://xmlns.jcp.org/jsf"
      xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
      xmlns:f="http://xmlns.jcp.org/jsf/core"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      >
    <h:head>
        <title>JSF 2.3: Websocket Sample</title>
        <script>
            function onMessage(message, channel, event) {
                console.log('Client onMessage listener: message: ' + message + ', channel:' + channel + ", event:" + event);
                document.getElementById("message").innerHTML = message;
            }
            function onOpen(channel) {
                console.log('Client onOpen listener:  channel:' + channel);
            }
            function onClose(code, channel, event) {
                console.log('Client onClose listener: code: ' + code + ', channel:' + channel + ", event:" + event);

                if (code === -1) {
                    // Web sockets not supported by client.
                } else if (code === 1000) {
                    // Normal close (as result of expired session or view).
                } else {
                    // Abnormal close reason (as result of an error).
                }
            }
        </script>    
    </h:head>
    <h:body>
        <h1>JSF 2.3: Websocket Events </h1>
        <div id="message" />
        <hr />
        <h:form id="form">
            <div>
                <h:commandButton onclick="jsf.push.open('eventChannel')" value="Open Event Channel">
                    <f:ajax />
                </h:commandButton>
                <h:commandButton onclick="jsf.push.close('eventChannel')" value="Close Event Channel">
                    <f:ajax />
                </h:commandButton>
            </div>
            <h:commandButton 
                id="sendMessage" 
                type="submit"
                action="#{eventBean.sendMessage}" value="Send Message">
                <f:ajax />
            </h:commandButton>
        </h:form>

        <f:websocket id="eventChannel" 
                     channel="eventChannel" 
                     onopen="onOpen" 
                     onclose="onClose" 
                     onmessage="onMessage" 
                     connected="false"/>
    </h:body> 
</html>
```

When you run this application on Glassfish, open your browser and navigate to [http://localhost:8080/jsf-websocket/event.faces](http://localhost:8080/jsf-websocket/event.faces).

You can see the logs in your browser console.

![console log](jsf-websocket-event.png)

And the IDE console, you can see the log from `WebsocketObserver`.

```
Info:   event opend::WebsocketEvent[channel=eventChannel, user=null, closeCode=null]
...
Info:   event closed::WebsocketEvent[channel=eventChannel, user=null, closeCode=NORMAL_CLOSURE]
```

## Ajax 

`f:websocket` can be bridged with `f:ajax` event attribute, and allow you trigger ajax event from websocket.


```java
@ViewScoped
@Named("ajaxBean")
public class AjaxBean implements Serializable {

    private static final Logger LOG = Logger.getLogger(AjaxBean.class.getName());
    
    @Inject
    @Push
    PushContext ajaxChannel;

    @Inject
    @Push
    PushContext ajaxListenerChannel;

    @Inject
    @Push
    PushContext commandScriptChannel;

    private List<String> messages = new ArrayList<>();

    public void ajaxPushed(AjaxBehaviorEvent e) throws AbortProcessingException{
        LOG.log(Level.INFO, "ajax pushed: " + e.toString());  
        messages.add("ajaxListenerEvent is sent at: " + LocalDateTime.now());
    }

    public void commandScriptExecuted() {
        LOG.log(Level.INFO, "commandScriptExecuted pushed.");
        messages.add("commandScriptExecuted message is sent at: " + LocalDateTime.now());
    }
    
    public void pushToAjaxChannel() {  
        messages.add("ajaxEvent is sent at: " + LocalDateTime.now());    
        ajaxChannel.send("ajaxEvent");
    }
    
    public void pushToAjaxListenerChannel(){ 
        ajaxListenerChannel.send("ajaxListenerEvent");
    }
    
     public void pushToCommandScriptChannel() {
        commandScriptChannel.send("onCommandScript");
    }

    public List<String> getMessages() {
        return messages;
    }

    public void setMessages(List<String> messages) {
        this.messages = messages;
    }

}
```

And facelets template.

```xml
<!DOCTYPE html>
<html lang="en" 
      xmlns="http://www.w3.org/1999/xhtml"
      xmlns:jsf="http://xmlns.jcp.org/jsf"
      xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
      xmlns:f="http://xmlns.jcp.org/jsf/core"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      >
    <h:head>
        <title>JSF 2.3: Websocket Sample</title> 
    </h:head>
    <h:body>
        <h1>JSF 2.3: Websocket and Ajax </h1>
        <h:panelGroup id="messagePanel" layout="block">
            <ul>
                <ui:repeat value="#{ajaxBean.messages}" var="m">
                    <li>#{m}</li>
                </ui:repeat>
            </ul>
        </h:panelGroup>

        <h:form id="form">
            <h:commandButton 
                id="pushToAjaxChannel" 
                action="#{ajaxBean.pushToAjaxChannel()}" 
                value="pushToAjaxChannel">
                <f:ajax/>
            </h:commandButton>
            <h:commandButton 
                id="pushToAjaxListenerChannel" 
                action="#{ajaxBean.pushToAjaxListenerChannel()}" 
                value="pushToAjaxListenerChannel">
                <f:ajax/>
            </h:commandButton>
            <h:commandButton 
                id="pushToCommandScriptChannel" 
                action="#{ajaxBean.pushToCommandScriptChannel()}" 
                value="pushToCommandScriptChannel">
                <f:ajax/>
            </h:commandButton>
        </h:form>
        <h:form>
            <f:websocket channel="ajaxChannel" scope="view">
                <f:ajax event="ajaxEvent" render=":messagePanel" />
            </f:websocket>
        </h:form>
        <h:form>
            <f:websocket channel="ajaxListenerChannel" scope="view">
                <f:ajax event="ajaxListenerEvent" listener="#{ajaxBean.ajaxPushed}" render=":messagePanel" />
            </f:websocket>
        </h:form>

        <f:websocket channel="commandScriptChannel" scope="view" onmessage="onCommandScript"/>
        <h:form>
            <h:commandScript name="onCommandScript" action="#{ajaxBean.commandScriptExecuted()}" render=":messagePanel"/>
        </h:form>
    </h:body> 
</html>
```

In the backend codes, use pushContext send the `f:ajax` event name as content.

Another alternative here, use `h:commandScript`(newly added in JSF 2.3) with `f:websocket` instead, set`onmessage` handler as the name attribute of `h:commandScript`.

Run this application on Glassfish, open your browser and navigate to [http://localhost:8080/jsf-websocket/ajax.faces](http://localhost:8080/jsf-websocket/ajax.faces).

![jsf websocket and ajax](jsf-websocket-ajax.png)

