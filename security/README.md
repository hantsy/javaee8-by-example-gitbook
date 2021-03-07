# Java EE Security API 1.0

Java EE Security API provides portable interfaces for HTTP authentication and identity stores, and a context aware `SecurityContext` which can be injected into other beans and allow you customize security programmatically.

`HttpAuthenticationMechanism` is used for authenticating users from applications.

`IdentityStore` is used for validating user credentials and retrieve its group information.

`SecurityContext` is used for querying the current security context in any context, eg. Servlet, JAX-RS, EJB etc.

Beside these, such as `Authenticaiton`, `Authorization`, `UserPricinpal`, `Realm` etc are very similar with [the terminology from Apache Shiro](https://shiro.apache.org/terminology.html).

