# SecurityContext

In Java EE 7 or earlier versions, other specifications, such as Servelt, EJB, JAX-RS, JAX-WS, etc. have their own specific APIs to query current security context.

* Servlet - HttpServletRequest\#getUserPrincipal, HttpServletRequest\#isUserInRole
* EJB - EJBContext\#getCallerPrincipal, EJBContext\#isCallerInRole
* JAX-WS - WebServiceContext\#getUserPrincipal, WebServiceContext\#isUserInRole
* JAX-RS - SecurityContext\#getUserPrincipal, SecurityContext\#isUserInRole
* JSF - ExternalContext\#getUserPrincipal, ExternalContext\#isUserInRole
* CDI - @Inject Principal
* WebSockets - Session\#getUserPrincipal

In Java EE 8, you can use the new `SecurityContext` introduced in Java EE Security 1.0 instead.

A default implementation should be available at runtime, you can inject it in CDI beans.

```java
@Inject SecurityContext securityContext;
```

The new `SecurityContext` provides similar methods with the one in other specifications.

```text
Principal getCallerPrincipal();
<T extends Principal> Set<T> getPrincipalsByType(Class<T> pType);
boolean isCallerInRole(String role);
```

The new `SecurityContext` allow you create own `Principal` instead of the default one, `getPrincipalsByType` can be used to fetch it.

Beside these methods.

It also provides,

* `boolean hasAccessToWebResource(String resource, String... methods)` to check the caller has permission to access some web resources.
* `AuthenticationStatus authenticate(HttpServletRequest request, HttpServletResponse response, AuthenticationParameters parameters);` perform a manual authentication flow.

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.

