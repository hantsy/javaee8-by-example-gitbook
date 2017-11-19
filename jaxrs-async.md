# Async improvements

In JAXRS 2.0, you can build an asynchronous RESTful APIs as the following.

```java
@GET
public void getAsync(final @Suspended AsyncResponse res) {
	res.setTimeoutHandler(
			(ar) -> {
				ar.resume(Response.status(Response.Status.SERVICE_UNAVAILABLE)
						.entity("Operation timed out --- please try again.").build());
			}
	);
	res.setTimeout(1000, TimeUnit.MILLISECONDS);
	executor.submit(() -> {
		//do long run operations.
		try {
			LOG.log(Level.INFO, " execute long run task in AsyncResource");
			//Thread.sleep(new Random().nextInt(1005));
			Thread.sleep(500);
		} catch (InterruptedException ex) {
			LOG.log(Level.SEVERE, "error :" + ex.getMessage());
		}
		res.resume(Response.ok("asynchronous resource").build());
	});
}
```

Inject `@Suspended AsyncResponse res` as the method parameter, call `res.resume()` to return response result asynchronously.

Or combined with EJB `@Asynchronous`.

```java
@GET
@Asynchronous
public void getAsync(final @Suspended AsyncResponse res) {

	//perform long run operations.
	try {
		LOG.log(Level.INFO, " execute long run task in EjbAsyncResource");
		Thread.sleep(500);
	} catch (InterruptedException ex) {
		LOG.log(Level.SEVERE, "error :" +ex.getMessage());
	}

	res.resume(Response.ok("Asynchronus EJB resource").build());
}
```


JAXRS 2.1 allow you return a `CompletionStage` directly.

```java
@GET
public CompletionStage<String> getAsync() {
	CompletionStage<String> cs = CompletableFuture
			.<String>supplyAsync(
					() -> {
						try {
							LOG.log(Level.INFO, " execute long run task in CompletionStageResource");
							Thread.sleep(500);
						} catch (InterruptedException ex) {
							LOG.log(Level.SEVERE, "error :" +ex.getMessage());
						}
						return "return CompletableFuture instead of @Suspended";
					}
				   // , executor
			);

	return cs;
}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.