# Runtime Discovery of Servlet Mappings

When a servlet is activated, the mapping info can be discoverable at runtime.

Described in the Servlet spcefication.

>The method getHttpServletMapping() on HttpServletRequest returns an
HttpServletMapping implementation that provides information for the mapping
that caused the current Servlet to be invoked. Please see the javadocs for the
normative specification. Please see sections Section 9.3.1, “Included Request
Parameters” on page 9-101Section 9.4.2, “Forwarded Request Parameters” on
page 9-102 and Section 9.7.2, “Dispatched Request Parameters” on page 9-104 for
relevant request attributes. 

But please notice:

>As with the included and forwarded request parameters,
the HttpServletMapping is not available for servlets that have been obtained with
a call to ServletContext.getNamedDispatcher().

An sample to print the mapping information of a servlet.

```java
@WebServlet(name = "MyServlet",
        urlPatterns = {
            "/MyServlet",
            "/",
            "/path/*",
            "*.extension"
        }
)
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException, ServletException {
        
          
        HttpServletMapping mapping = request.getHttpServletMapping();
         
        response.getWriter()
                .append("Mapping match:")
                .append(mapping.getMappingMatch().name())
                .append("\n")
                .append("Match value:")
                .append(mapping.getMatchValue())
                .append("\n")
                .append("Pattern:")
                .append(mapping.getPattern());
    }
}
```

Here is an example for demonstrating forward, async dispatch, and include.

```java
@WebServlet(name = "ServletA",
        urlPatterns = {
            "/ServletA",
            "/AForwardToB",
            "/AIncludesB",
            "/AAsyncDispatchToB",
            "*.a"
        },
        asyncSupported = true
)
public class ServletA extends HttpServlet {

    public static void printCurrentMappingDetails(HttpServletRequest request,
            PrintWriter out) throws IOException {
            HttpServletMapping forwardMapping = (HttpServletMapping) request.getAttribute(RequestDispatcher.FORWARD_MAPPING);
            HttpServletMapping includeMapping = (HttpServletMapping) request.getAttribute(RequestDispatcher.INCLUDE_MAPPING);
            HttpServletMapping asyncMapping = (HttpServletMapping) request.getAttribute(AsyncContext.ASYNC_MAPPING);

            out.print("<p> " + request.getHttpServletMapping() + "</p>");
            out.print("<p> FORWARD_MAPPING: " + forwardMapping + "</p>");
            out.print("<p> INCLUDE_MAPPING: " + includeMapping + "</p>");
            out.print("<p> ASYNC_MAPPING: " + asyncMapping + "</p>");
            out.print("<hr />");


    }

    /**
     * Processes requests for both HTTP <code>GET</code> and <code>POST</code>
     * methods.
     *
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    protected void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        HttpServletMapping mapping = request.getHttpServletMapping();

        if (mapping.getPattern().equals("/AForwardToB")) {
            RequestDispatcher rd = request.getRequestDispatcher("/ServletB");
            rd.forward(request, response);
            return;
        }

        if (mapping.getPattern().equals("/AAsyncDispatchToB")) {
            AsyncContext asyncContext = request.startAsync();
            asyncContext.dispatch("/ServletB");
            return;
        }

        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        try {
            out.println("<!DOCTYPE html>");
            out.println("<html>");
            out.println("<head>");
            out.println("<title>Servlet ServletA</title>");
            out.println("</head>");
            out.println("<body>");
            out.println("<h1>Servlet ServletA at " + request.getContextPath() + "</h1>");

            printCurrentMappingDetails(request, out);

            if (mapping.getPattern().equals("/AIncludesB")) {
                RequestDispatcher rd = request.getRequestDispatcher("/ServletB");
                rd.include(request, response);
            }

            out.println("</body>");
            out.println("</html>");
        } finally {
            out.close();
        }
    }

    // <editor-fold defaultstate="collapsed" desc="HttpServlet methods. Click on the + sign on the left to edit the code.">
    /**
     * Handles the HTTP <code>GET</code> method.
     *
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        processRequest(request, response);
    }

    /**
     * Handles the HTTP <code>POST</code> method.
     *
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        processRequest(request, response);
    }

    /**
     * Returns a short description of the servlet.
     *
     * @return a String containing servlet description
     */
    @Override
    public String getServletInfo() {
        return "Short description";
    }// </editor-fold>

}

//Servlet B is a generic servlet.

@WebServlet(name = "ServletB", urlPatterns = {
    "/ServletB",
    "*.b"
})
public class ServletB extends HttpServlet {

    /**
     * Processes requests for both HTTP <code>GET</code> and <code>POST</code>
     * methods.
     *
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    protected void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        try {
            /* TODO output your page here. You may use following sample code. */
            out.println("<!DOCTYPE html>");
            out.println("<html>");
            out.println("<head>");
            out.println("<title>Servlet ServletB</title>");
            out.println("</head>");
            out.println("<body>");
            out.println("<h1>Servlet ServletB at " + request.getContextPath() + "</h1>");

            ServletA.printCurrentMappingDetails(request, out);

            out.println("</body>");
            out.println("</html>");
        } finally {
            out.close();
        }
    }

    // <editor-fold defaultstate="collapsed" desc="HttpServlet methods. Click on the + sign on the left to edit the code.">
    /**
     * Handles the HTTP <code>GET</code> method.
     *
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        processRequest(request, response);
    }

    /**
     * Handles the HTTP <code>POST</code> method.
     *
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        processRequest(request, response);
    }

    /**
     * Returns a short description of the servlet.
     *
     * @return a String containing servlet description
     */
    @Override
    public String getServletInfo() {
        return "Short description";
    }// </editor-fold>

}
```

Try to access ServletA with different urls(urlPatterns of `WebServlet` annotation on `ServletA`), which results in different mappings.


Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.
