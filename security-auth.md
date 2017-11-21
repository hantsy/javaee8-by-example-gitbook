# HTTP authentication

`HttpAuthenticationMechanism` allow customsize your own HTTP authentication mechanism.

An examples for custom `HttpAuthenticationMechanism`.

```java
@ApplicationScoped
public class TestAuthenticationMechanism implements HttpAuthenticationMechanism {

    @Inject
    private IdentityStoreHandler identityStoreHandler;

    @Override
    public AuthenticationStatus validateRequest(HttpServletRequest request, HttpServletResponse response, HttpMessageContext httpMessageContext) throws AuthenticationException {
        final String name = request.getParameter("name");
        final String pwd = request.getParameter("password");

        if (name != null && pwd != null ) {

            // Get the (caller) name and password from the request
            // NOTE: This is for the smallest possible example only. In practice
            // putting the password in a request query parameter is highly
            // insecure
            
            Password password = new Password(pwd);

            // Delegate the {credentials in -> identity data out} function to
            // the Identity Store
            CredentialValidationResult result = identityStoreHandler.validate(
                    new UsernamePasswordCredential(name, password));

            if (result.getStatus() == VALID) {
                // Communicate the details of the authenticated user to the
                // container. In many cases the underlying handler will just store the details 
                // and the container will actually handle the login after we return from 
                // this method.
                return httpMessageContext.notifyContainerAboutLogin(
                        result.getCallerPrincipal(), result.getCallerGroups());
            }

            return httpMessageContext.responseUnauthorized();
        }

        return httpMessageContext.doNothing();
    }

}
```

`validate` of `IdentityStoreHandler` will transport the validation to an application scoped `IdentityStore` or container built-in approache to handle it.

Java EE Security API provides three built-in annotations('@BasicAuthenticationMechanismDefinition', 'FormAuthenticationMechanismDefinition',  '@CustomFormAuthenticationMechanismDefinition') to handle HTTP Basic, Form, a custom form authentication.

An example of '@BasicAuthenticationMechanismDefinition'.

```java
@BasicAuthenticationMechanismDefinition(
    realmName="${'test realm'}" // Doesn't need to be expression, just for example
)

@WebServlet("/servlet")
@DeclareRoles({ "foo", "bar", "kaz"})
@ServletSecurity(@HttpConstraint(rolesAllowed = "foo"))
public class TestServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        response.getWriter().write("This is a servlet \n");

        String webName = null;
        if (request.getUserPrincipal() != null) {
            webName = request.getUserPrincipal().getName();
        }

        response.getWriter().write("web username: " + webName + "\n");

        response.getWriter().write("web user has role \"foo\": " + request.isUserInRole("foo") + "\n");
        response.getWriter().write("web user has role \"bar\": " + request.isUserInRole("bar") + "\n");
        response.getWriter().write("web user has role \"kaz\": " + request.isUserInRole("kaz") + "\n");
    }

}
```

You can add `@BasicAuthenticationMechanismDefinition` on a Servlet class or a CDI `ApplicationScoped` bean.

Here is an example of `@FormAuthenticationMechanismDefinition`. 

```java
@FormAuthenticationMechanismDefinition(
    loginToContinue = @LoginToContinue(
        loginPage="/login-servlet",
        errorPage="/error-servlet"
    )
)
@WebServlet("/servlet")
@DeclareRoles({ "foo", "bar", "kaz" })
@ServletSecurity(@HttpConstraint(rolesAllowed = "foo"))
public class TestServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String webName = null;
        if (request.getUserPrincipal() != null) {
            webName = request.getUserPrincipal().getName();
        }
        
        response.getWriter().write(
                "<html><body> This is a servlet <br><br>\n" +
        
                    "web username: " + webName + "<br><br>\n" +
                            
                    "web user has role \"foo\": " + request.isUserInRole("foo") + "<br>\n" +
                    "web user has role \"bar\": " + request.isUserInRole("bar") + "<br>\n" +
                    "web user has role \"kaz\": " + request.isUserInRole("kaz") + "<br><br>\n" + 

                        
                    "<form method=\"POST\">" +
                        "<input type=\"hidden\" name=\"logout\" value=\"true\"  >" +
                        "<input type=\"submit\" value=\"Logout\">" +
                    "</form>" +
                "</body></html>");
    }
    
    @Override
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        if ("true".equals(request.getParameter("logout"))) {
            request.logout();
            request.getSession().invalidate();
        }
        
        doGet(request, response);
    }

}
```

Declare a `@FormAuthenticationMechanismDefinition`, set `LoginToContinue` properties.

Create a servlet for login.

```java
@WebServlet({"/login-servlet"})
public class LoginServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().write(
            "<html><body> Login to continue \n" +
                "<form method=\"POST\" action=\"j_security_check\">" +
                    "<p><strong>Username </strong>" +
                    "<input type=\"text\" name=\"j_username\">" +
                    
                    "<p><strong>Password </strong>" +
                    "<input type=\"password\" name=\"j_password\">" +
                    "<p>" +
                    "<input type=\"submit\" value=\"Submit\">" +
                "</form>" +
            "</body></html>");
    }

}
```

**Note**, for a `@FormAuthenticationMechanismDefinition`, in the login page, form submit action should `j_security_check`, username field name should be `j_username`, password field name should be `j_password`.

This seems a little unreasoneable, to free you from these fixed settings, there is a `@CustomFormAuthenticationMechanismDefinition`, this annoation accept the same properties as `@FormAuthenticationMechanismDefinition`.

Create a login page.

```xml
<!DOCTYPE html>
<html lang="en"
    xmlns="http://www.w3.org/1999/xhtml"
    xmlns:f="http://xmlns.jcp.org/jsf/core"
    xmlns:h="http://xmlns.jcp.org/jsf/html"
    xmlns:jsf="http://xmlns.jcp.org/jsf"
>
    <h:head/>
    
    <h:messages />
    
    <body>
        <p>
            Login to continue
        </p>
    
         <form jsf:id="form">
            <p>
                <strong>Username </strong> 
                <input jsf:id="username" type="text" jsf:value="#{loginBean.username}" />
            </p>
            <p>
                <strong>Password </strong> 
                <input jsf:id="password" type="password" jsf:value="#{loginBean.password}" />
            </p>
            <p>
                <input type="submit" value="Login" jsf:action="#{loginBean.login}" />
            </p>
        </form>
    
    </body>

</html>
```

And create a backing bean to handle the login manually.

```java
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package com.hantsylabs.example.ee8.security;

import java.util.logging.Logger;
import javax.enterprise.context.RequestScoped;
import javax.faces.application.FacesMessage;
import static javax.faces.application.FacesMessage.SEVERITY_ERROR;
import javax.faces.context.FacesContext;
import javax.inject.Inject;
import javax.inject.Named;
import javax.security.enterprise.AuthenticationStatus;
import static javax.security.enterprise.AuthenticationStatus.SEND_CONTINUE;
import static javax.security.enterprise.AuthenticationStatus.SEND_FAILURE;
import javax.security.enterprise.SecurityContext;
import static javax.security.enterprise.authentication.mechanism.http.AuthenticationParameters.withParams;
import javax.security.enterprise.credential.Credential;
import javax.security.enterprise.credential.Password;
import javax.security.enterprise.credential.UsernamePasswordCredential;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

/**
 *
 * @author hantsy
 */
@Named
@RequestScoped
public class LoginBean {

    @Inject
    private SecurityContext securityContext;

    @NotNull
    @Size(min = 3, max = 15, message = "Username must be between 3 and 15 characters")
    private String username;

    @NotNull
    @Size(min = 5, max = 50, message = "Password must be between 5 and 50 characters")
    private String password;

    @Inject
    Logger LOG;

    public void login() {

        FacesContext context = FacesContext.getCurrentInstance();
        Credential credential = new UsernamePasswordCredential(username, new Password(password));

        AuthenticationStatus status = securityContext.authenticate(
                getRequest(context),
                getResponse(context),
                withParams()
                        .credential(credential));

        LOG.info("authentication result:" + status);

        if (status.equals(SEND_CONTINUE)) {
            // Authentication mechanism has send a redirect, should not
            // send anything to response from JSF now.
            context.responseComplete();
        } else if (status.equals(SEND_FAILURE)) {
            addError(context, "Authentication failed");
        }

    }

    private static HttpServletResponse getResponse(FacesContext context) {
        return (HttpServletResponse) context
                .getExternalContext()
                .getResponse();
    }

    private static HttpServletRequest getRequest(FacesContext context) {
        return (HttpServletRequest) context
                .getExternalContext()
                .getRequest();
    }

    private static void addError(FacesContext context, String message) {
        context
                .addMessage(
                        null,
                        new FacesMessage(SEVERITY_ERROR, message, null));
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

}
```

In `loginBean`, use injected `SecurityContext` to handle authentication, it will delegate the process to the application server internally.

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.
