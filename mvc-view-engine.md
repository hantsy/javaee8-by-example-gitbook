# View engine

Spring MVC provides `View` and `ViewResolver` to find view and render view. MVC provides `ViewEngine` to do the same job.


## View Engine 

By default, **ozark** support tow built-in view engines to render **JSP** and **Facelets**, respectively.

In the `org.glassfish.ozark.ext` (check [org.glassfish.ozark.ext](http://mvnrepository.com/search?q=org.glassfish.ozark.ext)) package, **ozark** provides several `ViewEngine` implementations for the popular template.

* Thymeleaf
* Freemarker
* Velocity
* Handlebars
* Mustache
* Asciidoc
* Stringtemplate
* Jade 
* JSR223(Script engine)

## Facelets

Originally, **Facelets** is part of JSF specification, now it can be worked as a standard template engine in MVC.

1. Activate **Facelets** in web.xml.

	```xml
	<servlet>
		<servlet-name>Faces Servlet</servlet-name>
		<servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>Faces Servlet</servlet-name>
		<url-pattern>*.xhtml</url-pattern>
	</servlet-mapping>
	```	

   We do not need **face-config.xml** file to activate JSF here.
   
2. Convert all jsp files in the **mvc** sample to facelets template. Facelets supports **Composite View** pattern, we can define a template for all views.

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE html>
	<html xmlns="http://www.w3.org/1999/xhtml"
		xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
		xmlns:f="http://xmlns.jcp.org/jsf/core"
		xmlns:h="http://xmlns.jcp.org/jsf/html"
		xmlns:c="http://java.sun.com/jsp/jstl/core">
		<f:view contentType="text/html" encoding="UTF-8">
			<ui:insert name="metadata"></ui:insert>
			<h:head>
				<meta charset="utf-8" />
				<title>Task List</title>
				<meta name="viewport" content="width=device-width, initial-scale=1.0" />
				<meta name="description" content="" />
				<meta name="author" content="" />

				<!-- styles -->
				<link href="#{request.contextPath}/webjars/bootstrap/3.3.5/css/bootstrap.min.css" rel="stylesheet" />
				<link href="#{request.contextPath}/webjars/font-awesome/4.3.0/css/font-awesome.min.css" rel="stylesheet" />
				<link href="#{request.contextPath}/webjars/bootstrap-material-design/0.3.0/css/material.min.css" rel="stylesheet"/>
				<link href="#{request.contextPath}/webjars/bootstrap-material-design/0.3.0/css/ripples.min.css" rel="stylesheet"/>
				<link href="#{request.contextPath}/webjars/bootstrap-material-design/0.3.0/css/roboto.min.css" rel="stylesheet"/>

				<link href="#{request.contextPath}/resources/css/main.css" rel="stylesheet" />
				<!-- HTML5 shim, for IE6-8 support of HTML5 elements -->
				<![CDATA[
				<!--[if lt IE 9]>
				<script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
				<script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
				<![endif]-->
				]]>

				<ui:insert name="headIncludes"></ui:insert>
			</h:head>

			<h:body>
				<ui:include src="/WEB-INF/layout/header.xhtml" />
				<h:panelGroup layout="block" styleClass="container">
					<ui:include src="/WEB-INF/layout/alert.xhtml" />
					<h:panelGroup layout="block" styleClass="page-header">
						<h1><ui:insert name="pageTitle"></ui:insert></h1>
					</h:panelGroup>
					<ui:insert name="content"/>
				</h:panelGroup>

				<ui:include src="/WEB-INF/layout/footer.xhtml"/>
				<!-- Placed at the end of the document so the pages load faster -->
				<script type="text/javascript" src="#{request.contextPath}/webjars/jquery/2.1.3/jquery.min.js">
					/** stop autoclosing **/
				</script>
				<script type="text/javascript" src="#{request.contextPath}/webjars/bootstrap/3.3.5/js/bootstrap.min.js">

				</script>
				<script type="text/javascript" src="#{request.contextPath}/webjars/bootstrap-material-design/0.3.0/js/material.min.js">

				</script>
				<script type="text/javascript" src="#{request.contextPath}/webjars/bootstrap-material-design/0.3.0/js/ripples.min.js">

				</script>
				<script>
					$.material.init();
				</script>
			</h:body>
		</f:view>
	</html>
	```	

   The template includes **header** and **footer** fragments directly, it also define a **content** placeholder, in the extended views the content will be inserted in the **content** definition. 

   If you are familiar with JSF before, it is easy to understand the codes.
   
   Check out the codes and explore them yourself.
  
3. Do not forget to change the view postfix to **.xhtml** in the `TaskController`. eg. the view in  `allTasks` method.

	```java
	@GET
	@View("tasks.xhtml")
	public void allTasks() {
		log.log(Level.INFO, "fetching all tasks");

		List<Task> todotasks = taskRepository.findByStatus(Task.Status.TODO);
		List<Task> doingtasks = taskRepository.findByStatus(Task.Status.DOING);
		List<Task> donetasks = taskRepository.findByStatus(Task.Status.DONE);

		log.log(Level.INFO, "got all tasks: todotasks@{0}, doingtasks@{1}, donetasks@{2}", new Object[]{todotasks.size(), doingtasks.size(), donetasks.size()});

		models.put("todotasks", todotasks);
		models.put("doingtasks", doingtasks);
		models.put("donetasks", donetasks);

	} 
	```	
  
## Source Codes

1. Clone the codes from my github.com account.

    [https://github.com/hantsy/ee8-sandbox/](https://github.com/hantsy/ee8-sandbox/)
	
2. Open the **mvc-facelets** project in NetBeans IDE.
3. Run it on Glassfish.
4. After it is deployed and runging on Glassfish application server, navigate [http://localhost:8080/ee8-mvc/mvc/tasks](http://localhost:8080/ee8-mvc/mvc/tasks) in browser.

