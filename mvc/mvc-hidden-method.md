# Processing PUT and DELETE methods

When you try to submit a `PUT` or `DELETE` HTTP request via form submission, and you want to invoke the methods annotated with `@PUT` or `@DELETE` in the Controller to handle the request.

Unfortunately it does not work.

There is a simple solution which can help us to overcome this issue.

If you have used Spring MVC before and I think you could know there is a `HiddenMethodFilter` in the Spring MVC to fix this issue. Java EE 8 MVC does not ship the similar Filter, we can create one.

## Solution

1. Create a Filter to convert the request method to the target HTTP method.

   ```java
    @WebFilter(filterName = "HiddenHttpMethodFilter", urlPatterns = {"/*"}, dispatcherTypes = {DispatcherType.REQUEST})
    public class HiddenHttpMethodFilter implements Filter {

        @Inject
        Logger log;

        /**
        * Default method parameter: {@code _method}
        */
        public static final String DEFAULT_METHOD_PARAM = "_method";

        private String methodParam = DEFAULT_METHOD_PARAM;

        /**
        * Set the parameter name to look for HTTP methods.
        *
        * @see #DEFAULT_METHOD_PARAM
        */
        public void setMethodParam(String methodParam) {
            this.methodParam = methodParam;
        }

        @Override
        public void doFilter(ServletRequest req, ServletResponse res, FilterChain filterChain)
                throws ServletException, IOException {
            log.log(Level.INFO, "entering HttpHiddenFilter...");
            HttpServletRequest request = (HttpServletRequest) req;
            HttpServletResponse response = (HttpServletResponse) res;

            String paramValue = request.getParameter(this.methodParam);
            log.log(Level.INFO, "paramValue @" + paramValue);
            log.log(Level.INFO, "request method @" + request.getMethod());

            if ("POST".equals(request.getMethod()) && paramValue != null && paramValue.trim().length() > 0) {
                String method = paramValue.toUpperCase(Locale.ENGLISH);
                HttpServletRequest wrapper = new HttpMethodRequestWrapper(request, method);
                filterChain.doFilter(wrapper, response);
            } else {
                filterChain.doFilter(request, response);
            }
        }

        @Override
        public void init(FilterConfig filterConfig) throws ServletException {

        }

        @Override
        public void destroy() {

        }

        /**
        * Simple {@link HttpServletRequest} wrapper that returns the supplied
        * method for {@link HttpServletRequest#getMethod()}.
        */
        private static class HttpMethodRequestWrapper extends HttpServletRequestWrapper {

            private final String method;

            public HttpMethodRequestWrapper(HttpServletRequest request, String method) {
                super(request);
                this.method = method;
            }

            @Override
            public String getMethod() {
                return this.method;
            }
        }

    }
   ```

2. Add a hidden field named `_method` in the form, which value is the target HTTP method.

   ```markup
    <form action="${markDoingUrl}" method="post">
        <input type="hidden" name="_method" value="PUT"><jsp:text /></input>
        <input type="hidden" name="status" value="DOING"><jsp:text /></input>
        <button type="submit" class="btn btn-sm btn-primary">
            <span class="glyphicon glyphicon-play" aria-hidden="true"><jsp:text /></span>
            START
        </button>
    </form>
   ```

3. The `@PUT` method in the `TaskController`.

   ```java
    @PUT
    @Path("{id}/status")
    public Response updateStatus(@PathParam(value = "id") Long id, @NotNull @FormParam(value = "status") String status) {
        log.log(Level.INFO, "updating status of the existed task@id:{0}, status:{1}", new Object[]{id, status});

        Task task = taskRepository.findById(id);

        task.setStatus(Task.Status.valueOf(status));

        taskRepository.update(task);

        flashMessage.notify(Type.info, "Task status was updated successfully!");

        return Response.ok("redirect:tasks").build();
    }
   ```

## Source codes

1. Clone the codes from my GitHub account.

   [https://github.com/hantsy/ee8-sandbox/](https://github.com/hantsy/ee8-sandbox/)

2. Open the **mvc** project in NetBeans IDE.
3. Run it on Glassfish.
4. After it is deployed and running on Glassfish application server, navigate [http://localhost:8080/ee8-mvc/mvc/tasks](http://localhost:8080/ee8-mvc/mvc/tasks) in browser.

