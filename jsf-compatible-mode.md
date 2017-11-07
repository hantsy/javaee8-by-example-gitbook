# JSF 2.2 Compatible mode

For those not ready for new CDI activation, JSF 2.3 provides a compatible mode to make your codes working seamlessly with a JSF 2.2 *faces-config.xml*.

In contrast with the former JSF 2.3 configration, we should perform some modificaiton here.


1. Use a 2.2 versioned *faces-config.xml* instead.
   ​ 
	```xml
	<?xml version='1.0' encoding='UTF-8'?>
	<faces-config xmlns="http://xmlns.jcp.org/xml/ns/javaee"
		  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-facesconfig_2_2.xsd"
		  version="2.2">
	</faces-config>
	```	

2. Declare a *beans.xml*, CDI is enabled by default in Java EE 7, so it is **optional**.

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
		   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		   xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/beans_2_0.xsd"
		   bean-discovery-mode="all" 
		   version="2.0">
	</beans>
	```

3. Add 3.1 versioned *web.xml*. It is **optional**, but the compatible mode only works with 3.1, does not work with a 4.0 versioned *web.xml*.

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
			 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
			 version="3.1">
	</web-app>
	```
4. Remove the `@FacesConfig` annotated class, it is only for JSF 2.3.

Done.

Now create a simple sample to try it.

Firstly create a simple backing bean.

```java
@Named
@RequestScoped
public class HelloBean {

    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public void sayHi() {
        this.message = "Greeting at " + LocalDateTime.now();
    }

    @Override
    public int hashCode() {
        int hash = 7;
        hash = 13 * hash + Objects.hashCode(this.message);
        return hash;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null) {
            return false;
        }
        if (getClass() != obj.getClass()) {
            return false;
        }
        final HelloBean other = (HelloBean) obj;
        if (!Objects.equals(this.message, other.message)) {
            return false;
        }
        return true;
    }

}
```

And a simple Facelets template.

```xml
<!DOCTYPE html>
<html lang="en"
    xmlns="http://www.w3.org/1999/xhtml"
    xmlns:h="http://xmlns.jcp.org/jsf/html"
    xmlns:jsf="http://xmlns.jcp.org/jsf">
    <h:head/>
    
    <h:messages />
    
    <body>
        <p>
            Hello JSF 2.3(JSF 2.2 Compatible Mode)
        </p>   
         <form jsf:id="form">
            <p>
                <strong>Say Hi to JSF 2.3: </strong> 
                <div>#{helloBean.message}</div>
            </p>
            <p>
                <input type="submit" value="Login" jsf:action="#{helloBean.sayHi()}" />
            </p>
        </form>   
    </body>
	
</html>
```

In NetBeans IDE, run it on Glassfish directly, finally it will open the home page.

Get the [source codes](https://github.com/hantsy/ee8-sandbox/tree/master/jsf-compatible-mode-jsf22) from my github account.
