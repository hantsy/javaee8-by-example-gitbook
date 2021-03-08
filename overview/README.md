# Overview

Java EE 7 was born in 2013, after a long-awaited 5 years, we finally got the release news of Java EE 8 and Glassfish v5. For more details, please read the official announcement [Java EE 8 and GlassFish 5.0 Released!](https://blogs.oracle.com/theaquarium/java-ee-8-is-final-and-glassfish-50-is-released) from Oracle blog portal.

## A brief intro of Java EE 8

Unfortunately, most of original proposed specifications are not included in the final Java EE 8. And in a long period, the development were paused for some unknown reasons.

To save Java EE, the [Java community](https://javaee-guardians.io/) created a [petition](https://www.change.org/p/larry-ellison-tell-oracle-to-move-forward-java-ee-as-a-critical-part-of-the-global-it-industry) and wished Oracle can speed up Java EE development progress.

At the same time, IBM worked together with Redhat and other Java communities and launched a new [MicroProfile](http://microprofile.io) which targets lightweight Java EE and cloud native applications. 

> Currently, Microprofile is also part of Eclipse EE4j project.

After Java EE was released,  Oracle decided to [open up Java EE progress](https://blogs.oracle.com/theaquarium/opening-up-java-ee) and [move it to Eclipse foundation](https://blogs.oracle.com/theaquarium/opening-up-ee-update).

Java EE 8 should be the last version released by Oracle\(and Sun\).

## What is new in Java EE 8

Including me, many developers are a little disappointed about Java EE 8\(JSR 366\) , it did not cover all points of the original design and also delayed again and again. But no doubt there are still lots of new features and improvements which are valuable to update ourselves.

There tow new specifications were newly introduced in Java EE 8.

* JSR 375 – Java EE Security API 1.0
* JSR 367 – The Java API for JSON Binding \(JSON-B\) 1.0

Some specifications have been updated to align with Java 8 and CDI or involved as a maintenance release.

* JSR 365 – Contexts and Dependency Injection \(CDI\) 2.0
* JSR 369 – Java Servlet 4.0
* JSR 370 – Java API for RESTful Web Services \(JAX-RS\) 2.1
* JSR 372 – JavaServer Faces \(JSF\) 2.3
* JSR 374 – Java API for JSON Processing \(JSON-P\)1.1
* JSR 380 – Bean Validation 2.0
* JSR 250 – Common Annotations 1.3
* JSR 338 – Java Persistence 2.2
* JSR 356 – Java API for WebSocket 1.1
* JSR 919 – JavaMail 1.6

The other specifications such as JMS, Batch have no updates in this version.

Unfortunately, some proposed specifications are not included in Java EE 8, including:

* JSR 371 - MVC is vetoed in the final stage, but it is still existed as a community driven specification. 
* JSR 107 - JCache had missed the last train of Java EE 7, and also be absent from Java EE 8.

