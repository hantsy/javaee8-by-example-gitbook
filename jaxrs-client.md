# Reactive Client

In JAXRS 2.0, a client to handle async resources looks like.

```java
public class AsyncClient {

    public final static void main(String[] args) throws Exception {

        WebTarget target = ClientBuilder.newClient().target("http://localhost:8080/jaxrs-async/rest/ejb");

        Future<String> future = target.request()
                .async()
                .get(String.class);

        System.out.println("ejb resource future:" + future.get());

        target.request()
                .async()
                .get(AsyncClient.responseInvocationCallback());
    }

    private static InvocationCallback<Response> responseInvocationCallback() {
        return new InvocationCallback<Response>() {
            @Override
            public void completed(Response res) {
                System.out.println("Status:" + res.getStatusInfo());
                System.out.println("Entity:" + res.getEntity());
                System.out.println("Request success!");
            }

            @Override
            public void failed(Throwable e) {
                System.out.println("Request failed!");
            }

        };
    }

}
```

JAXRS 2.1 embraces the concept, added a `rx()` method switch to Reactive APIs and handle the response in stream.

By default, it supports Java 8 `CompletionStage`.

```java
public class CompletionStageClient {

    public final static void main(String[] args) throws Exception {

        WebTarget target = ClientBuilder.newClient().target("http://localhost:8080/jaxrs-async/rest/ejb");

        CompletionStage<Void> future = target.request()
                .rx()
                .get(String.class)
                .thenAccept(t -> System.out.println(t));
              

    }
}
```

Jersey added extra support fro rxjava1.

```java
public class ObservableClient {

    public final static void main(String[] args) throws Exception {
        Client client = ClientBuilder.newClient();
        
        client.register(RxObservableInvokerProvider.class);
        WebTarget target = client.target("http://localhost:8080/jaxrs-async/rest/ejb");

        target.request()
                .rx(RxObservableInvoker.class)
                .get(String.class)
                .subscribe(new Observer<String>() {
                    @Override
                    public void onCompleted() {
                        System.out.println("onCompleted");
                    }

                    @Override
                    public void onError(Throwable e) {
                        System.out.println("onError:" + e.getMessage());
                    }

                    @Override
                    public void onNext(String t) {
                        System.out.println("onNext:" + t);
                    }
                });

    }
}
```

And rxjava2.

```java
public class FlowableClient {

    public final static void main(String[] args) throws Exception {
        Client client = ClientBuilder.newClient();

        client.register(RxFlowableInvokerProvider.class);
        WebTarget target = client.target("http://localhost:8080/jaxrs-async/rest/ejb");

        target.request()
                .rx(RxFlowableInvoker.class)
                .get(String.class)
                .subscribe(new Subscriber<String>() {

                    @Override
                    public void onError(Throwable e) {
                        System.out.println("onError:" + e.getMessage());
                    }

                    @Override
                    public void onNext(String t) {
                        System.out.println("onNext:" + t);
                    }

                    @Override
                    public void onSubscribe(Subscription s) {
                        System.out.println("onSubscribe:" + s);
                        s.request(1);                    
                    }

                    @Override
                    public void onComplete() {
                        System.out.println("onComplete");
                    }
                });

    }
}
```

And Guava's `ListenableFuture`.

```java
public class ListenableFutureClient {

    public final static void main(String[] args) throws Exception {
        Client client = ClientBuilder.newClient();

        client.register(RxListenableFutureInvokerProvider.class);
        WebTarget target = client.target("http://localhost:8080/jaxrs-async/rest/ejb");

        ListenableFuture<String> future = target.request()
                .rx(RxListenableFutureInvoker.class)
                .get(String.class);

        FutureCallback<String> callback = new FutureCallback<String>() {
            @Override
            public void onSuccess(String result) {
                System.out.println("result :" + result);
            }

            @Override
            public void onFailure(Throwable t) {
                System.out.println("error :" + t.getMessage());
            }
        };

        Futures.addCallback(future, callback, Executors.newFixedThreadPool(10));

        System.out.println("ListenableFuture:" + future.get());

    }
}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.