#Exception Handling and form validation

When submitting a form, it should validate the form data before it is stored in the backend database.

## Form binding and validation

Like  Spring MVC, Struts, Stripes, JSF etc. MVC provides the similiar progress to process form submission.

1. Gather user input form data.
2. Convert form data to the target form bean. If there are some conversion failure, it is possbile to stop the progress and notify user.
3. Bind the converted value to the form bean.
4. Validate the form bean via **Bean Validation**.  If there are some constraint voilations, it is possbile to stop the progress and notify user.
5. Continue to process form.

MVC provides a `BindingResult` to gather all of the binding errors and constraint voilations in the request.

## Handling form validation

Inject it in the controller class.

    @Inject
    private BindingResult validationResult;

In the controller method, add `@Valid` annotation on the methed parameters.

    @ValidateOnExecution(type = ExecutableType.NONE)
    public Response save(@Valid @BeanParam TaskForm form) {
        log.log(Level.INFO, "saving new task @{0}", form);

        if (validationResult.isFailed()) {
            AlertMessage alert = AlertMessage.danger("Validation voilations!");
            validationResult.getAllViolations()
                    .stream()
                    .forEach((ConstraintViolation t) -> {
                        alert.addError(t.getPropertyPath().toString(), "", t.getMessage());
                    });
            models.put("errors", alert);
            return Response.status(BAD_REQUEST).entity("add.jspx").build();
        }
    }

If the validation is failed, the `isFailed` method should return `true`. 

You can iterate all voilations(`validationResult.getAllViolations()`) and gather the voilation details for each properties.

Then display the error messages in the JSP pages.

	<c:if test="${not empty errors}">
		 <c:forEach items="${errors.errors}" var="error">
		<div class="alert alert-danger alert-dismissible"
			 role="alert">
			<button type="button" class="close" data-dismiss="alert"
					aria-label="Close">
				<span aria-hidden="true"><![CDATA[&times;]]></span>
			</button>
			<p>${error.field}: ${error.message}</p>
		</div>
		</c:forEach>
	</c:if>
    
## Handling exception

Like JAX-RS exception handling, you can handle exception via `ExceptionMapper` and display errors in the certain view.

Create a custom `ExceptionMapper` and add annotation `@Provider`.

    @Provider
    public class TaskNotFoundExceptionMapper implements ExceptionMapper<TaskNotFoundException>{
        
        @Inject Logger log;
        
        @Inject Models models;

        @Override
        public Response toResponse(TaskNotFoundException exception) {
            log.log(Level.INFO, "handling exception : TaskNotFoundException");
            models.put("error", exception.getMessage());
            return Response.status(Response.Status.NOT_FOUND).entity("error.jspx").build();
        }     
    }

Different from JAX-RS, the entity value is the view that will be returned. In the *error.jspx* file, display the `error` model via EL directly.

    <div class="container">
        <div class="page-header">
            <h1>Psst...something was wrong!</h1>
        </div>
        <div class="row">
            <div class="col-md-12">
                <p class="text-danger">${error}</p>
            </div>
        </div>
    </div>

When the `TaskNotFoundException` is caught, it will display the erorr like the following.

![mvc error](mvc-error.png)

## Source codes

1. Clone the codes from my github.com account.

    [https://github.com/hantsy/ee8-sandbox/](https://github.com/hantsy/ee8-sandbox/)
	
2. Open the **mvc** project in NetBeans IDE.
3. Run it on Glassfish.
4. After it is deployed and runging on Glassfish application server, navigate [http://localhost:8080/ee8-mvc/mvc/tasks](http://localhost:8080/ee8-mvc/mvc/tasks) in browser.

