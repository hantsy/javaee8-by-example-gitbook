# Page navigation(Routing)

As introducing the feature pieces of the MVC specification, we have visited most of views in this demo. In fact, when you desgin a website, you should have a map for the page navigation among all views.

## View navigation summary

1. When the appliation starts up, it will display the task list view.
2. User can create a new task by clicking the **New Task** button, the task creating view is displayed.
3. When user input the task title and description, submit the form.
	* If the task is submitted successfully, return the main task list view. The new created task should be in the list.
	* If the task form data is invalid, show the validation errors.
4. User can edit the new created task by click edit icon.
    * It will read the task from database and fill the existing data into form fields
	* The submission progress is similiar with the new task subflow described in above 3.
5. User can update the task status(TODO to DOING, or DOING to DONE). When the status is changed successfully, return to the task list.
6. User can delete the DONE task. When it is deleted, return to the task list.

//todo add a graph to display the navigations
![]()

## View navigation path definition

We have used GET, POST, PUT, DELETE http methods to process basic CRUD operations, listed in the following table.

|Navigation path|HTTP method|View page|Descritption|
|---------------|-----------|---------|------------|
|/tasks|GET|tasks.jspx| Task list view|
|/tasks/new| GET| add.jspx| Display new task view|
|/tasks| POST| redirect:tasks| Submit task form, save task, and redirect to task list view|
|/tasks/{id}/edit| GET| edit.jspx| Display edit task view|
|/tasks/{id}| PUT| redirect:tasks| Submit task form, update task, and redirect to task list view|
|/tasks/{id}| DELETE| redirect:tasks| Submit task form, delete task, and redirect to task list view|
|/tasks/{id}/status| PUT| redirect:tasks| Submit task form, update task status, and redirect to task list view|

It is easy to understand. As you see, the navigation rules are very similar with the paths for RESTful APIs.

## Display views

In the above paths, the task list view, new task view and edit task view can display the views directly via inputing the url path in browser locaiton bar. Others are POST submission, they will redirect to the task list view.

There are several approaches provided in MVC specification to navigate the view path. 

1. Add `@View` annotation to a `void` controller method.
2. Return a `Viewable` object to a controller method.
3. Return a `Response` entity which body content includes the view path.

	```java
	public Response someAction(){
		 return Response.ok("redirect:tasks").build();
	}
	```	
		
4. Return a `String` type value directly.	

	```java
	public String someAction(){
		 return "redirect:tasks";
	}	
	```	

The 1 and 2 are new added to the MVC specification, and the 3rd is reusing the existing JAXRS specification.		
		
## POST-REDIRECT-GET pattern

The new task form submission and edit task form submission follow the **POST-REDIRECT-GET** pattern.

1. When the form is submitted successfully, a form **POST** request is sent to the server.
2. After the server received the request, and processed it, it will **REDIRECT** to the target view.
3. And perform a new **GET** request on the target view.
 

## Source codes

1. Clone the codes from my github.com account.

    [https://github.com/hantsy/ee8-sandbox/](https://github.com/hantsy/ee8-sandbox/)
	
2. Open the **mvc** project in NetBeans IDE.
3. Run it on Glassfish.
4. After it is deployed and runging on Glassfish application server, navigate [http://localhost:8080/ee8-mvc/mvc/tasks](http://localhost:8080/ee8-mvc/mvc/tasks) in browser.

