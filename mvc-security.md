# MVC Security

MVC has built-in some security features to protect pages, eg. CSRF protection.

## CSRF protection

MVC has built-in CSRF protection, there is a `Csrf` interface.



1. Configure `Csrf` in the `Application` class. Override the `getProperties` method.

	```java
	@Override
	public Map<String, Object> getProperties() {
		Map<String, Object> props = new HashMap<>();
		
		props.put(Csrf.CSRF_PROTECTION, Csrf.CsrfOptions.EXPLICIT);
		
		//view folder
		//props.put(ViewEngine.DEFAULT_VIEW_FOLDER, ViewEngine.VIEW_FOLDER);
		return super.getProperties();
	}
	```	
		
	And there are some options to configure CSRF via `Csrf.CsrfOptions`.

	* **OFF** to disable Csrf.
	* **EXPLICIT** to enable Csrf wtih annotation `@CsrfValid` on the Controller method.
	* **IMPLICIT** to enable Csrf autmaticially. No need `@CsrfValid`.

2. Add annotation `@CsrfValid` on the Controller method.

	```java
	@POST
	@CsrfValid
	@ValidateOnExecution(type = ExecutableType.NONE)
	public Response save(@Valid @BeanParam TaskForm form) {
	
	}
	```	

4. In the view, add hidden field to insert the Csrf value.

	```xml
	<input type="hidden" name="${mvc.csrf.name}" value="${mvc.csrf.token}"/>
	```


When you run the codes on Glassfish, in the view, the Csrf field looks like:

```xml
<input value="f3ca389f-efba-4f28-afe7-2a1e7231a238" name="X-Requested-By" type="hidden" />
```	
		
Every request will generate a unique **X-Requested-By** value. 

When the form is submitted, and it will be validated by MVC provider.

		
## MvcContext

`MvcContext` interface includes the contextual data of MVC, such as context path, application path, etc. And also includes 
MVC security, such as `Csrf` and `Encoders`.

In the above section, we have used `Csrf`.

At the runtime environment, `MvcContext` is exposed by EL ${mvc} in the view.

* `${mvc.contextPath}` will get context path.
* `${mvc.applicationPath}` will get the application path declared in the `Application` class.
* `${mvc.csrf.name}` generate the Csrf token name.
* `${mvc.csrf.token}` generate the Csrf token value.
* `${mvc.encoders.js(jsValue)}` will escape the js scripts.
* `${mvc.encoders.html(htmlValue)}` will escape the html snippets.

## Source Codes

1. Clone the codes from my github.com account.

    [https://github.com/hantsy/ee8-sandbox/](https://github.com/hantsy/ee8-sandbox/)
	
2. Open the **mvc** project in NetBeans IDE.
3. Run it on Glassfish.
4. After it is deployed and runging on Glassfish application server, navigate [http://localhost:8080/ee8-mvc/mvc/tasks](http://localhost:8080/ee8-mvc/mvc/tasks) in browser.

