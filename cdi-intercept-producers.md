# Configurators APIs and Intercept Producers

In CDI 2.0, it is possible to add `Interceptor` to a producer programmaticially.

For example, there is a POJO.

```
public class Greeter {

    public Greeter() {
    }

    public void say(String name) {
        System.out.println("Hi, " + name);
    }
}
``` 

We want to count the number of calling the method.

Create a `Counted` quilifier and `CountedInterceptor` interceptor class.

```java
@Inherited
@InterceptorBinding
@Retention(RUNTIME)
@Target({METHOD, TYPE})
public @interface Counted {

    public static final class Literal extends AnnotationLiteral<Counted> implements Counted {

        public static final Literal INSTANCE = new Literal();

        private static final long serialVersionUID = 1L;

    }
}
```

```java

@Interceptor
@Counted
public class CountedInterceptor {

    @Inject
    Logger LOG;

    @Inject
    Counter counter;

    @AroundConstruct
    public void aroundConstructed(InvocationContext ctx) throws Exception {
        LOG.log(Level.INFO, "invoke constructor:" + ctx.getConstructor() + ", arguments:" + ctx.getContextData());
        Object o = ctx.proceed();
        counter.increase();
    }

    @AroundInvoke
    public Object aroundInvoked(InvocationContext ctx) throws Exception {
        LOG.log(Level.INFO, "invoke method:" + ctx.getMethod() + ", paraemters:" + ctx.getParameters());
        Object o = ctx.proceed();
        counter.increase();

        return o;
    }

}
```

The counting service is done by `Counter`.

```java

@ApplicationScoped
public class Counter {
    
    private int count = 0;
    
    @Inject
    Logger LOG;
    
    public void increase() {        
        count++;
        LOG.log(Level.INFO, "current counter:" + count);
    }
    
    public int getCount() {
        return count;
    }
    
}
```

Now put all together.

```java
@ApplicationScoped
public class GreeterProducer {

    @Produces
    public Greeter producesGreeter(InterceptionFactory<Greeter> factory) {
        factory.configure().add(Counted.Literal.INSTANCE);
        return factory.createInterceptedInstance(new Greeter());
    }

}
```

Verify it works as expected.

```java
@Test()
public void testGreeter() {
	assertNotNull(greeter);

	LOG.log(Level.INFO, "counter.getCount()::" + counter.getCount());
	//assertTrue(1 == counter.getCount());
	greeter.say("CDI 2.0");
	LOG.log(Level.INFO, "counter.getCount() after called Greeter.say()::" + counter.getCount());
	assertTrue(1 == counter.getCount());
}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.