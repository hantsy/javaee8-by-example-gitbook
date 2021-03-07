# Http Trailer

Servlet 4.0 added Http Trailer\([RFC 7230](https://tools.ietf.org/html/rfc7230)\) supports, which is a specific collection of HTTP headers comes after response body.

It is useful in some case, such as chunked transfer encoding or implements some specific protocols.

* The reading side, `HttpServletRequest` has a method `isTrailerFieldsReady()` to check if the trailer fields are available, if it returns true, the trailer fields can be read via `getTrailerFields()` method.
* The writing side, `HttpServletResponse` has a method `setTrailerFields`, which accepts a `Supplier` as it's parameter.

An example of Http Trailer to handle chunked transfer encoding.

```java
@WebServlet("/test")
public class TestServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {

        res.setContentType("text/plain");
        res.addHeader("Transfer-encoding", "chunked");
        res.addHeader("TE", "trailers");
        res.addHeader("Trailer", "bar");

        StringBuilder sb = new StringBuilder();

        final InputStream in = req.getInputStream();
        int b;
        while ((b = in.read()) != -1) {
            sb.append((char) b);
        }

        String foo = null;
        int size = -1;

        if (req.isTrailerFieldsReady()) {
            Map<String, String> reqTrailerFields = req.getTrailerFields();
            size = reqTrailerFields.size();
            foo = reqTrailerFields.get("foo");
        }

        final String finalFoo = foo;
        final int finalSize = size;
        res.setTrailerFields(() -> {
            Map<String, String> map = new HashMap<>();
            map.put("bar", finalFoo + finalSize);
            return map;
        });
        res.getWriter().write(sb.toString());
    }
}
```

A client to send chunked data.

```java
@WebServlet("")
public class ClientServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {

        res.setContentType("text/plain");
        StringBuilder sb = new StringBuilder();

        String hostStr = req.getServerName();
        int port = req.getServerPort();

        try (
            Socket socket = new Socket(hostStr, port);
            OutputStream output = socket.getOutputStream();
            InputStream input = socket.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(input))
        ) {
            String reqStr = (new StringBuffer("POST /ee8-servlet-trailer/test HTTP/1.1\r\n")).
                append("Host: " + hostStr + "\r\n").
                append("Transfer-encoding: chunked\r\n").
                append("Connection: close\r\n").
                append("trailer: foo\r\n").
                append("\r\n").
                append("5\r\n").
                append("hello\r\n").
                append("0\r\n").
                append("foo: A\r\n").
                append("\r\n").
                toString();

            output.write(reqStr.getBytes(Charset.forName("US-ASCII")));

            String line = null;
            while ((line = reader.readLine()) != null) {
                if (line.startsWith("bar")) {
                    sb.append(line).append("\r\n");
                }
            }
        }

        res.getWriter().write(sb.toString());
    }
}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.

