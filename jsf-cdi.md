# More CDI alignments

JSF 2.2 starts CDI integration, provides built-in CDI compatible `ViewScoped` and `@FlowScoped`, and all CDI scopes are compatible with JSF. 

In JSF 2.3, more CDI alignments are added.

A lots of JSF built-in facilites are exposed as CDI beans, and can be injected as general CDI beans.

In JSF 2.2, to get JSF facilites.

```java
FacesContext context= FacesContext.getCurrentInstance();
ExternalContext externalContext = FacesContext.getCurrentInstance().getExternalContext();
Map<String, Object> cookieMap = FacesContext.getCurrentInstance().getExternalContext().getRequestCookieMap();
Map<String, Object> viewMap = FacesContext.getCurrentInstance().getViewRoot().getViewMap();
```

In JSF 2.3, you can inject them directly in your backing bean.

```java
@Inject
FacesContext facesContext;

@Inject
ExternalContext externalContext;

@Inject
@RequestCookieMap
Map<String, Object> cookieMap;

@Inject
@ViewMap
Map<String, Object> viewMap;
```

And all of these can be resolved via EL in facelets template.

There is a table listed all supported facilites in Arjan Tijms's blog entry, [What's new in JSF 2.3? ](http://arjan-tijms.omnifaces.org/p/jsf-23.html), a must-read article when upgrading to JSF 2.3.

Artefact |	EL name |	Qualifier |	Type
---|---|---|---
Application 	|#{application} |	- 	|java.lang.Object (javax.servlet.ServletContext)
ApplicationMap |	#{applicationScope} 	|@ApplicationMap 	|java.util.Map<String, Object>
CompositeComponent |	#{cc} |	(Not injectable) |	javax.faces.component.UIComponent
Component 	|#{component} 	|(Not injectable) |	javax.faces.component.UIComponent
RequestCookieMap |	#{cookie} |	@RequestCookieMap |	java.util.Map<String, Object>
FacesContext 	|#{facesContext} |	- |	javax.faces.context.FacesContext
Flash 	|#{flash} 	|- 	|javax.faces.context.Flash
FlowMap 	|#{flowScope} 	|@FlowMap |	java.util.Map<Object, Object>
HeaderMap |	#{header} |	@HeaderMap 	|java.util.Map<String, String>
HeaderValuesMap 	|#{headerValues} |	@HeaderValuesMap |	java.util.Map<String, String[]>
InitParameterMap 	|#{initParam} |	@InitParameterMap |	java.util.Map<String, String>
RequestParameterMap |	#{param} |	@RequestParameterMap |	java.util.Map<String, String>
RequestParameterValuesMap |	#{paramValues} |	@RequestParameterValuesMap |	java.util.Map<String, String[]>
Request 	|#{request} |	(Not injectable) |	java.lang.Object (javax.servlet.http.HttpServletRequest)
RequestMap 	|#{requestScope} |	@RequestMap |	java.util.Map<String, Object>
ResourceHandler |	#{"resource"} |	- 	|javax.faces.application.ResourceHandler
Session 	|#{session} 	|(Not injectable) |	java.lang.Object (javax.servlet.http.HttpSession)
SessionMap |	#{sessionScope} |	@SessionMap |	java.util.Map<String, Object>
View 	|#{view} 	|- 	|javax.faces.component.UIViewRoot
ViewMap 	|#{viewScope} 	|@ViewMap |	java.util.Map<String, Object>
ExternalContext 	|#{externalContext} (new) |	- 	|javax.faces.context.ExternalContext

