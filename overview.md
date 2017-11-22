#  A quick glance at Java EE 8

The past September was a busy month, the most exciting news is Java 9 reaches GA, as well as the release of the long-waiting Java EE 8 and Glassfish v5. For more details, please read the official announcement [Java EE 8 and GlassFish 5.0 Released!](https://blogs.oracle.com/theaquarium/java-ee-8-is-final-and-glassfish-50-is-released) from Oracle blog portal.

## A brief intro of Java EE 8

The world changes so quickly, after Java EE 7 was born in 2013, cloud service and microservice became more and more popular. Java EE had to embrace the changes, so a lot of perspectives are proposed to be brought into Java EE 8, including Configuration, Load Balance, Circuit breaker, Service Registry and Discovery, programnatic Security API, MVC etc. 

But the road to Java EE 8 is not straight, most of proposed specfications are moved out of Java EE 8 finally. And in a long period, the development of some specifications were paused for some reasons. 

To save Java EE, the [Java community](https://javaee-guardians.io/) created a [petition](https://www.change.org/p/larry-ellison-tell-oracle-to-move-forward-java-ee-as-a-critical-part-of-the-global-it-industry) and wished Oracle can move forward Java EE more quickly.

At the same time, IBM, Redhat and other Java communities launched a new [MicroProfile](http://microprofile.io) which targets ligthweight Java EE and cloud computing service. Now it is a project under Eclipse foundation.

Although the Java EE 8 way is a little hard, finally it is released to the public. 

And surprisingly Oracle decided to [open up Java EE progress](https://blogs.oracle.com/theaquarium/opening-up-java-ee) and [move it to Eclipse foundation](https://blogs.oracle.com/theaquarium/opening-up-ee-update). [An updated petition](https://www.change.org/p/larry-ellison-tell-oracle-to-move-forward-java-ee-as-a-critical-part-of-the-global-it-industry/u/21473794?utm_medium=email&utm_source=petition_update&utm_campaign=146669&sfmc_tk=xZ%2f6z4TGoQ02piKnRtK%2bejNgWC%2bWD6nr3P%2bcjkRrgGJqXJLLTSlXDQ6alq40O5pe&j=146669&sfmc_sub=46994739&l=32_HTML&u=27789648&mid=7259882&jb=1) was created to help moving Java EE to Eclipse more smoothly.

Java EE 8 should be the last version released by Oracle(and Sun).

## What is new in Java EE 8

Like me, some developers are a little disappointed about Java EE 8(JSR 366) , even complain it comes a little late. But no doubt there are still lots of new features and improvements which are valuable to update ourselves.

There tow new specifications were newly introduced in Java EE 8.

* JSR 375 – Java EE Security API 1.0
* JSR 367 – The Java API for JSON Binding (JSON-B) 1.0

Some specifications have been updated to align with Java 8 and CDI or involved as a maintainance release.

* JSR 365 – Contexts and Dependency Injection (CDI) 2.0
* JSR 369 – Java Servlet 4.0
* JSR 370 – Java API for RESTful Web Services (JAX-RS) 2.1
* JSR 372 – JavaServer Faces (JSF) 2.3
* JSR 374 – Java API for JSON Processing (JSON-P)1.1
* JSR 380 – Bean Validation 2.0
* JSR 250 – Common Annotations 1.3
* JSR 338 – Java Persistence 2.2
* JSR 356 – Java API for WebSocket 1.1
* JSR 919 – JavaMail 1.6

The other specifications such as JMS, Batch have no updates in this version.

Unfortunately, some proposed specfications are not included in Java EE 8, including:

* JSR 371 - MVC is vetoed in the final stage, but it is still existed as a community based specification. 
* JSR 107 - JCache had missed the last train of Java EE 7, and also lost its attractiveness in Java EE 8.
  
## Example codes 

You can get the source codes from my github account, and install JDK 8, NetBeans 8.2 or 9, Glassfish v5, and play the sample codes locally.

### Get source codes

The source codes of this book is available on [https://github.com/hantsy/ee8-sandbox](https://github.com/hantsy/ee8-sandbox).

Clone it into your local disk.

	git clone https://github.com/hantsy/ee8-sandbox
	

### Install Oracle JDK 8 

1. Get the latest Oracle Java 8, http://java.oracle.com
2. Follow [the offical installation guide](https://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html) to install it into your local system.
3. Verify the installation, open terminal and type `java --version`, it should print java version info.

### Install NetBeans IDE

Currently no IDEs have built-in Java EE 8 support. 

NetBeans is being migrated to Apache Foundation as [an incubating project](http://netbeans.apache.org), and NetBeans 9 is still under active development(but NetBeans 9 targets Java 9, not Java EE 8).

1. Download NetBeans from http://www.netbeans.org,  NetBeans 9 nightly is highly recommended.
2. Install it into your local system.

### Install Glassfish v5

1. Download Glassfish v5 from http://github.com/glassfish
2. Exract the archive into your local disc.
3. Open NetBeans IDE, click **Services** tab. If it is not opend, open it from **Window** menu.
4. Right click the **Servers** ndoe, in the popup context menu, click **Add Server** and follow the wizard to add Glassfish v5 into your IDE.


### Import sample codes

1. Starts up NetBeans IDE.
2. Click **Open** icon or **File**/**Open** menu to select a project. Maven projects are recognised by NetBeans IDE automaticially.
3. Select the above cloned project, open it in NetBeans IDE, it looks like.
	
![nb](nb-javaee8.png)	

Click any subproject node under **Modules** to open the certain project, and run it on Glassfish application server by click **Run** in the context menu.


## References

I create a repository to collect all useful Java EE 8 resources, check the latest [Awesome Java EE 8](https://github.com/hantsy/awesome-javaee8) checklist.













 

