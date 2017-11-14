#MVC and CDI

In before posts, we have already use `@Inject` to inject CDI beans into the Controller class. In fact, MVC embraced CDI internally. 


##RedirectScoped

There is a new CDI compatible scope named `RedirectScoped` introduced in MVC 1.0. 

A bean annotated with `@RedirectScoped` annotation can be transfered between two requests, conceptly, it is similar with JSF Flash.

The `AlertMessage` is an example.

1. Declare a bean with `@RedirectScoped` annotation.

		@RedirectScoped
		@Named("flashMessage")
		public class AlertMessage implements Serializable {}

2. Inject it into a Controller class.

		@Inject
		AlertMessage flashMessage;

3. Access the bean properties in the view via EL.

		<c:if test="${not empty flashMessage and not empty flashMessage.text}">
			<div class="alert alert-${flashMessage.type} alert-dismissible"
				 role="alert">
				<button type="button" class="close" data-dismiss="alert"
						aria-label="Close">
					<span aria-hidden="true"><![CDATA[&times;]]></span>
				</button>
				<p>${flashMessage.text}</p>
			</div>
		</c:if>
	
It is easy to understand the code snippets.

`AlertMessage` is a `RedirectScoped` bean, which means it is can be access in current request and the next request. It is named with CDI `@Named` which indicates it can be accessed in view via EL by name **flashMessage**.

##MVC Event

MVC defined a series of CDI compatible events, with which you can track the MVC request lifecycle.

MVC provides a `MvcEvent` interface as the base of all events in MVC, there are several specific event implementations.

* AfterControllerEvent
* AfterProcessViewEvent
* BeforeControllerEvent
* BeforeProcessViewEvent
* ControllerRedirectEvent

MVC itself will fire these events in the request processing progress, you can observe the events via CDI `@Observes`.

	@ApplicationScoped
	public class MvcEventListener {

		@Inject Logger LOGGER;

		private void onControllerMatched(@Observes BeforeControllerEvent event) {
			LOGGER.info(() -> "Controller matched for " + event.getUriInfo().getRequestUri());
		}

		private void onViewEngineSelected(@Observes BeforeProcessViewEvent event) {
			LOGGER.info(() -> "View engine: " + event.getEngine());
		}

		@PostConstruct
		private void init() {
			LOGGER.config(() -> this.getClass().getSimpleName() + " created");
		}
	}

## Source codes

1. Clone the codes from my github.com account.

    [https://github.com/hantsy/ee8-sandbox/](https://github.com/hantsy/ee8-sandbox/)
	
2. Open the **mvc** project in NetBeans IDE.
3. Run it on Glassfish.
4. After it is deployed and runging on Glassfish application server, navigate [http://localhost:8080/ee8-mvc/mvc/tasks](http://localhost:8080/ee8-mvc/mvc/tasks) in browser.

