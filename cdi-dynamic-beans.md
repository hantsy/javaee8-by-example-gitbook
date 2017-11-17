# Register Beans dynamicially

Before CDI 2.0 register a bean dynamicially is a little complex. CDI 2.0 simple the work.

```java
public void addABean(@Observes AfterBeanDiscovery event) {
	// get an instance of BeanConfigurator
	event.addBean()
			// set the desired data
			.types(Greeter.class)
			.scope(ApplicationScoped.class)
			.addQualifier(Default.Literal.INSTANCE)
			//.addQualifier(Custom.CustomLiteral.INSTANCE);
			//finally, add a callback to tell CDI how to instantiate this bean
			.produceWith(obj -> new Greeter());
}

```

`AfterBeanDiscovery` event add `addBean` method to register a bean manually.

You can check if the bean is existed in CDI container.

```java
Set<Bean<?>> greeters = CDI.current().getBeanManager().getBeans(Greeter.class);
assertTrue(greeters.size() == 1);
assertNotNull(greeter);
```		

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my github account, and have a try.