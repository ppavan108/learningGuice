_Using Guice Servlet and Binding Language_

### Installing a Servlet Module

Once you have [[GuiceFilter|Servlets]] up and running, Guice Servlet is set up. However, you will need to install an instance of `ServletModule` in order to get real use out of Guice Servlet:
```java
   Guice.createInjector(new ServletModule());
```

This module sets up the request and session scopes, and provides a place to configure your filters and servlets from. While you are free to create the injector from any place of your choice, a logical place to do it in is a `ServletContextListener`. 

A `ServletContextListener` is a Java servlet component that is triggered as soon as a web application is deployed, and before any requests begin to arrive. Guice Servlet provides a convenience utility that you can subclass in order to register your own `ServletContextListener`s:

```java
public class MyGuiceServletConfig extends GuiceServletContextListener {

  @Override
  protected Injector getInjector() {
    return Guice.createInjector(new ServletModule());
  }
}
```

Next, add the following to `web.xml` so the servlet container triggers this class when the app is deployed:

```xml
<listener>
  <listener-class>com.example.MyGuiceServletConfig</listener-class>
</listener>
```

You can now use Guice Servlet as per your needs. Note that it is not necessary to use a `ServletContextListener` to use Guice Servlet, as long as you remember to install `ServletModule` when creating your injector.

### The Binding Language

Think of the `ServletModule` as an in-code replacement for the `web.xml` deployment descriptor. Filters and servlets are configured here using normal Java method calls. Here is a typical example of registering a servlet when creating your Guice injector:

```java
   Guice.createInjector(..., new ServletModule() {

     @Override
     protected void configureServlets() {
       serve("*.html").with(MyServlet.class);
     }
   }
```
 
This registers a servlet (subclass of `HttpServlet`) called `MyServlet` to serve any web requests ending in `.html`. You can also use a path-style syntax to register servlets as you would in `web.xml`:
```java
       serve("/my/*").with(MyServlet.class);
```

== Filter Mapping ==

You may also map Servlet Filters using a very similar syntax:
```java
      filter("/*").through(MyFilter.class);
```

This will route every incoming request through `MyFilter`, and then continue to any other matching filters before finally being dispatched to a servlet for processing.

_Note: Every servlet (or filter) is required to be a `@Singleton`. If you cannot annotate the class directly, you must bind it using `bind(..).in(Singleton.class)`, separate to the `filter()` or `servlet()` rules. Mapping under any other scope is an error. This is to maintain consistency with the Servlet specification. 
Guice Servlet does not support the deprecated `SingleThreadModel`._


### Available Injections

Installing the servlet module automatically gives you access to several classes from the servlet framework. These are useful for working with the servlet programming model and are injectable in *any* Guice injected object by default, when you install the ServletModule:

```java
@RequestScoped
class SomeNonServletPojo {

  @Inject
  SomeNonServletPojo(HttpServletRequest request, HttpServletResponse response, HttpSession session) {
    ...
  }

}
```

The request and response are scoped to the current http request. Similarly the http session object is scoped to the current user session. In addition to this you may also inject the current `ServletContext` and a map of the request parameters using the binding annotation `@RequestParameters` as follows:

```java
@Inject @RequestParameters Map<String, String[]> params;
```

This must be a map of strings to arrays of strings because http allows multiple values to be bound to the same request parameter, though typically there is only one. Note that if you want access to any of the request or session scoped classes from singletons or other wider-scoped instances, you should inject a `Provider<T>` instead.

### Dispatch Order

You are free to register as many servlets and filters as you like this way. They will be compared and dispatched in the order in which the rules appear in your `ServletModule`:

```java
   Guice.createInjector(..., new ServletModule() {

     @Override
     protected void configureServlets() {
       filter("/*").through(MyFilter.class);
       filter("*.css").through(MyCssFilter.class);
       // etc..

       serve("*.html").with(MyServlet.class);
       serve("/my/*").with(MyServlet.class);
       // etc..
      }
    }
```
 
This will traverse down the list of rules in lexical order. For example, a url `/my/file.js` will first be compared against the servlet mapping:
```java
       serve("*.html").with(MyServlet.class);
```
 
And failing that, it will descend to the next servlet mapping:
```java
       serve("/my/*").with(MyServlet.class);
```
 
Since this rule matches, Guice Servlet will dispatch the request to `MyServlet` and stop the matching process. The same process is used when deciding the dispatch order of filters (that is, matched to their order of appearance in the binding list).

### Varargs Mapping

The two mapping rules above can also be written in more compact form using varargs syntax:
```java
       serve("*.html", "/my/*").with(MyServlet.class);
```
 
This way you can map several URI patterns to the same servlet. A similar syntax is also available for filter mappings.


### Using RequestScope

Each servlet configured with a `ServletModule` is executed within `RequestScope`.  

By default, there are several elements available to be injected, each of which is bound in RequestScope:
  * `HttpServletRequest` / `ServletRequest`
  * `HttpServletResponse` / `ServletResponse`
  * `RequestParameters Map<String, String[]>`

Remember to use a `Provider` when injecting either of these elements if the injection point is on a class that is created outside of `RequestScope`.  For example, a singleton servlet is created outside of a request, and so it needs to call `Provider.get()` on any `RequestScoped` dependency only after the request is received (typically it its `service()` method.

The most common way to seed a value within `RequestScope` is to add a <a href="http://download.oracle.com/docs/cd/E17802_01/products/products/servlet/2.3/javadoc/javax/servlet/Filter.html">Filter</a> in front of your servlet.  This filter should seed the scope by adding the value as a request attribute.

For example, assume we want to scope the context path of each request as a `String` so that objects involved in processing the request can have this value injected.

The `Filter` to do this might look like this:

```java
   protected Filter createUserIdScopingFilter() {
     return new Filter() {
       @Override public void doFilter(
          ServletRequest request,  ServletResponse response, FilterChain chain)
           throws IOException, ServletException {
         HttpServletRequest httpRequest = (HttpServletRequest) request;
         // ...you'd probably want more sanity checking here
         Integer userId = Integer.valueOf(httpRequest.getParameter("user-id"));
         httpRequest.setAttribute(
             Key.get(Integer.class, Names.named("user-id")).toString(),
             userId);  
         chain.doFilter(request, response);
       }

      @Override public void init(FilterConfig filterConfig) throws ServletException { }

      @Override public void destroy() { }
     };
  } 
```

And the binding might look like this:
```java
  public class YourServletModule extends ServletModule {
     @Override protected void configureServlets() {
         .....
        filter("/process-user*").through(createUserIdScopingFilter());
    }
  }
```

And the servlet to use this might look like this:
```java
@Singleton
class UserProcessingServlet extends HttpServlet {

      private final Provider<Integer> userIdProvider;
      ....
   
      @Inject UserProcessingServlet(
         Provider<Integer> userIdProvider,
         .....) {
            this.userIdProvider = userIdProvider;
       }

      ....

     @Override public void doGet(
          HttpServletRequest req,  HttpServletResponse resp)
        throws ServletException, IOException {
             ...
            Integer userId = userIdProvider.get();
       }
}
```


        
