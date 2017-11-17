# MVC 1.0(JSR371)

**Update**: Unfortunately the Java EE export group decided to move MVC out of Java EE 8, and now it becomes a community based specification, more details please check the [JSR 371 specification page](https://jcp.org/en/jsr/detail?id=371).

~~MVC is a new specification which will be introduced in Java EE 8~~ (MVC is not part of Java EE 8 ).

MVC is based on the existing JAXRS specification.

In the [Spring4 Sandbox](https://github.com/hantsy/spring4-sandbox/), I provided a simple **task board** sample to demonstrate Spring MVC with Apache Tiles, Thymeleaf, Freemarker etc.

In order to introduce the features of MVC 1.0, I will try to port this sample and use Java EE 8 MVC to reimplement it.

* [Getting started with MVC](mvc-getting-start.md)
* [Handling form submission](mvc-handle-form.md)
* [Exception handling and form validation](mvc-validation.md)
* [Processing PUT and DELETE methods](mvc-hidden-method.md)
* [Page navigation](mvc-page-nav.md)
* [MVC and CDI](mvc-cdi.md)
* [Security](mvc-security.md)
* [Bean parameter conversion](mvc-bean-conversion.md)
* [View engine](mvc-view-engine.md)
