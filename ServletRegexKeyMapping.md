_Advanced topics: regex mapping, init params, etc._

### Regular Expressions

You can also map servlets (or filters) to URIs using regular expressions:
```java
    serveRegex("(.)*ajax(.)*").with(MyAjaxServlet.class)
```
 
This will map any URI containing the text `"ajax"` in it to `MyAjaxServlet`. Such as:
  * `http://www.google.com/`**`ajax`**`.html`
  * `http://www.google.com/content/`**`ajax`**`/index`
  * `http://www.google.com/it/is-totally-`**`ajax`**`y`

### Initialization Parameters

Servlets (and filters) allow you to pass in string values using the `<init-param>` tag in web.xml. Guice Servlet supports this using a `Map<String, String>` of name/value pairs. For example, to initialize `MyServlet` with two parameters `coffee="Espresso"` and `site="google.com"` you could write:
```java
  Map<String, String> params = new HashMap<String, String>();
  params.put("coffee", "Espresso");
  params.put("site", "google.com");

  ...
      serve("/*").with(MyServlet.class, params)
```

And the `ServletConfig` object passed to `MyServlet` will contain the appropriate input parameters when using `getInitParams()`. The same syntax is available with filters.

### Binding Keys

You can also bind keys rather than classes. This lets you hide implementations with package-local visbility and expose them using only a Guice module and an annotation:
```java
      filter("/*").through(Key.get(Filter.class, Fave.class));
```
 
Where `Filter.class` refers to the Servlet API interface `javax.servlet.Filter` and `Fave.class` is a custom binding annotation. Elsewhere (in one of your own modules) you can bind this filter's implementation:
```java
   bind(Filter.class).annotatedWith(Fave.class).to(MyFilterImpl.class);
```
 
See the [UserGuide User's Guide] for more information on binding and annotations.

### Injecting the injector

Once you have a servlet or filter injected for you using `ServletModule`, you can access the injector at any time by simply injecting it directly into any of your classes:

```java
@Singleton
public class MyServlet extends HttpServlet {
  @Inject private Injector injector;
  ...
}

// elsewhere in ServletModule
serve("/myurl").with(MyServlet.class);
```

This is often useful for integrating third party frameworks that need to use the injector themselves.