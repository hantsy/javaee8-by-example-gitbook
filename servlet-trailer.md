# Http Trailer

Servlet 4.0 added Http Trailer([RFC 7230](https://tools.ietf.org/html/rfc7230)) supports, which is a specific collection of http headers comes after response body.

It is useful in some case, such as chunked transfer encoding or implements some specific protocols.

* The reading side, `HttpServletRequest` has a method `isTrailerFieldsReady()` to check if the trailer fields are available, if it returns true, the trailer fields can be read via `getTrailerFields()` method.
* The writing side, `HttpServletResponse` has a method `setTrailerFields`, which accpets a `Supplier` as it's parameter.

An example of Http Trailer to handle chunked tranfer encoding.

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

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.
 