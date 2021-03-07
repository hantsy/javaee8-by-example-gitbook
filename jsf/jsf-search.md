# Search expression framework

In before JSF versions, it uses an absolute hierarchical ID, or a relative local ID to locate a component, it is useful to rerender a component in an ajax request. For more flexible to query the component, JSF added `@all`, `@form`, `@this` expression to search the component to be re-rendered.

JSF 2.3 extends these expression by introducing new **Component search expression framework**, which adds some useful and powerful keywords, and also provides APIs to define your own keywords in the expression.

The following table lists all new keywords added in JSF 2.3, it is just a copy from [Arjan Tijms' blog entry: What's new in JSF 2.3](http://arjan-tijms.omnifaces.org/p/jsf-23.html).

| Keyword | Description |
| :--- | :--- |
| @child\(n\) | The nth child of the base component |
| @composite | The closest composite component ancestor of the base component |
| @id\(id\) | All component descendants of the base component with the specified component id |
| @namingcontainer | The closest NamingContainer ancestor of the base component |
| @next | The next component in the view after the base component |
| @parent | The parent of the base component |
| @previous | The previous component to the base component |
| @root | The UIViewRoot |

Let's try to create a custom keyword to find the _grant parent_ node of the certain component in the component tree.

Create a new `SearchKeywordResolver`.

```java
public class GrandParentKeywordResolver extends SearchKeywordResolver {

    @Override
    public boolean isResolverForKeyword(SearchExpressionContext searchExpressionContext, String keyword) {
        return "grandParent".equals(keyword);
    }

    @Override
    public void resolve(SearchKeywordContext searchKeywordContext, UIComponent current, String keyword) {
        UIComponent parent = current.getParent();
        if (parent != null) {
            searchKeywordContext.invokeContextCallback(parent.getParent());
        } else {
            searchKeywordContext.setKeywordResolved(true);
        }
    }
}
```

Register it via a `@WebListener` class.

```java
@WebListener
public class WebInit implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        FacesContext.getCurrentInstance()
                .getApplication()
                .addSearchKeywordResolver(new GrandParentKeywordResolver());

    }
}
```

A simple facelets template.

```markup
<h:panelGroup id="panelgroup">
    <h:form id="form">
        <h:button id="button" outcome="foo2">Button foo</h:button>
        <h:commandButton id="commandButton" action="#{backingBean.foo()}" value="invoke foo"/>
        <h:outputText id="body" value="body"/>
    </h:form>
</h:panelGroup>
```

Get the _grandParent_ id of the certain component at runtime.

```java
public void foo() {
    SearchExpressionContext searchContext = createSearchExpressionContext(context, context.getViewRoot());

    context.getApplication()
            .getSearchExpressionHandler()
            .resolveComponent(
                    searchContext,
                    ":form:@parent",
                    (c, target) -> out.print(":form:@parent -> "+target.getId()));

    context.getApplication()
            .getSearchExpressionHandler()
            .resolveComponent(
                    searchContext,
                    ":form:@grandParent",
                    (c, target) -> out.print(":form:@grandParent -> "+target.getId()));
}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.

