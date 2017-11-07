# MVC 1.0(JSR371)

**Update**: Unfortunately the Java EE export group decided to move MVC out of Java EE 8, and now it becomes a community based specification, more details please check the [JSR 371 specification page](https://jcp.org/en/jsr/detail?id=371).

~~MVC is a new specification which will be introduced in Java EE 8~~ (MVC is not part of Java EE 8 ) MVC is based on the existing JAXRS specification.

In the [Spring4 Sandbox](https://github.com/hantsy/spring4-sandbox/), I provided a simple **task board** sample to demonstrate Spring MVC with Apache Tiles, Thymeleaf, Freemarker etc.

In order to introduce the features of MVC 1.0, I will try to port this sample and use Java EE 8 MVC to reimplement it.

* [Getting started with MVC](https://github.com/hantsy/ee8-sandbox/wiki/mvc-getting-start)
* [Handling form submission](https://github.com/hantsy/ee8-sandbox/wiki/mvc-handle-form)
* [Exception handling and form validation](https://github.com/hantsy/ee8-sandbox/wiki/mvc-validation)
* [Processing PUT and DELETE methods](https://github.com/hantsy/ee8-sandbox/wiki/mvc-hidden-method)
* [Page navigation](https://github.com/hantsy/ee8-sandbox/wiki/mvc-page-nav)
* [MVC and CDI](https://github.com/hantsy/ee8-sandbox/wiki/mvc-cdi)
* [Security](https://github.com/hantsy/ee8-sandbox/wiki/mvc-security)
* [Bean parameter conversion](https://github.com/hantsy/ee8-sandbox/wiki/mvc-bean-conversion)
* [View engine](https://github.com/hantsy/ee8-sandbox/wiki/mvc-view-engine)