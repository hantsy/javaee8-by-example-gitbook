# Inject support in Converter, Validator and Behavor

In JSF 2.3, Converter, Validator and Behavior are refactored to generic interfaces.

Have a look at the `Converter` interface.

```java
public interface Converter<T> {

    public T getAsObject(FacesContext context, UIComponent component,
                              String value);

    public String getAsString(FacesContext context, UIComponent component,
                              T value);  
}
```

And these interfaces also support `@Inject` and also can be injected into other beans, which are originally planned in JSF 2.2, but delayed to be implemented in JSF 2.3.

Let's create a simple sample to taste it. `TagsConverter` is used to convert tag string to a tag list in backing bean.

```java
@FacesConverter(value = "tagsConverter", managed = true)
public class TagsConverter implements Converter<List<String>> {

    @Inject
    ConverterUtils utils;

    @Override
    public List<String> getAsObject(FacesContext context, UIComponent component, String value) {
       return utils.stringToList(value);
    }

    @Override
    public String getAsString(FacesContext context, UIComponent component, List<String> value) {
       return utils.listToString(value);
    }

}
```

`managed = true` indicates the converter is a CDI managed converter.

To demonstrate `@Inject` supports in `Converter`, I extracted the conversion handling process into a CDI bean `ConverterUtils`, and inject it in `TagsConverter`.

```java
@ApplicationScoped
public class ConverterUtils {

    @Inject
    Logger LOG;

    public String listToString(List<String> value) {
        LOG.log(Level.INFO, "listToString:{0}", value);
        return value != null ? join(",", value) : "";
    }

    public List<String> stringToList(String value) {
        LOG.log(Level.INFO, "stringToList:{0}", value);
        return value != null ? Arrays.asList(value.split(",")) : Collections.<String>emptyList();
    }

}
```

Now you can apply this converter on an input field in a facelets template.

```markup
 <div>
    <h:outputLabel for="tags" value="Tags" />
    <h:inputText 
        id="tags" 
        value="#{backingBean.tags}">
        <f:converter converterId="tagsConverter" />
    </h:inputText>    
</div>
```

The `backingBean` code is:

```java
@RequestScoped
@Named
public class BackingBean {


    List<String> tags = new ArrayList<>();


    public List<String> getTags() {
        return tags;
    }

    public void setTags(List<String> tags) {
        this.tags = tags;
    }



}
```

And the `TagsConverter` can be injected into other beans like other generic CDI beans.

```java
@RequestScoped
@Named
public class BackingBean {

    @Inject
    @FacesConverter(value="tagsConverter", managed = true)
    TagsConverter mytagsConverter;

    List<String> mytags = new ArrayList<>();

    public List<String> getMytags() {
        return mytags;
    }

    public void setMytags(List<String> mytags) {
        this.mytags = mytags;
    }

//...

}
```

In the facelets template, we can bind it to the converter directly.

```markup
<div>
    <h:outputLabel for="mytags" value="My Tags" />
    <h:inputText 
        id="mytags" 
        value="#{backingBean.mytags}">
        <f:converter binding="#{backingBean.mytagsConverter}"/>
    </h:inputText>     
</div>
```

Let's run this sample on Glassfish v5, open browser and navigate to [http://localhost:8080/jsf-converter/converter.faces](http://localhost:8080/jsf-converter/converter.faces).

![jsf converter](../.gitbook/assets/jsf-converter.png)

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.

