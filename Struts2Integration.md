### Struts 2 Integration

To install the Guice Struts 2 plugin with Struts 2.2 or later, simply include `guice-struts2-plugin-3.0.jar` in your web application's classpath, 
create a subclass of GuiceServletContextListener that implements getInjector and adds any servlets and filters (using ServletModule) and the Struts2GuicePluginModule, and a web.xml file that references your GuiceServletContextListener subclass as a listener.  For examples, look at the example [web.xml](https://github.com/google/guice/blob/master/extensions/struts2/example/root/WEB-INF/web.xml) and [GuiceServletContextListener subclass](https://github.com/google/guice/blob/master/extensions/struts2/example/src/com/google/inject/struts2/example/ExampleListener.java).

Guice will inject all of your Struts 2 objects including actions and interceptors. You can even scope your actions.
#### A Counting Example
Say for example that we want to count the number of requests in a session. Define a `Counter` object which will live on the session:
```java
@SessionScoped
public class Counter {

  int count = 0;

  /** Increments the count and returns the new value. */
  public synchronized int increment() {
    return count++;
  }
}
```
Next, we can inject our counter into an action:
```java
public class Count {

  final Counter counter;

  @Inject
  public Count(Counter counter) {
    this.counter = counter;
  }

  public String execute() {
    return SUCCESS;
  }

  public int getCount() {
    return counter.increment();
  }
}
```
Then create a mapping for our action in our struts.xml file:
```xml
<action name="Count"
    class="mypackage.Count">
  <result>/WEB-INF/Counter.jsp</result>
</action>     
```
And a JSP to render the result:
```jsp
<%@ taglib prefix="s" uri="/struts-tags" %>

<html>   
  <body>
    <h1>Counter Example</h1>
    <h3><b>Hits in this session:</b>
      <s:property value="count"/></h3>
  </body>
</html>
```
We actually made this example more complicated than necessary in an attempt to illustrate more concepts. In reality, we could have done away with the separate `Counter` object and applied `@SessionScoped` to our action directly.

#### Struts2 support in earlier versions of Guice
In Guice 1.0, the struts2 extension supports struts 2.0.6 or later, and is included using the `guice-struts2-plugin-1.0.jar`.  The struts2 extension included in Guice 2.0 did not work properly.

You must select Guice as your ObjectFactory implementation in your `struts.xml` file:
```xml
<constant name="struts.objectFactory" value="guice" />
```
You can optionally specify a `Module` for Guice to install in your `struts.xml` file:
```xml
<constant name="guice.module" value="mypackage.MyModule"/>
```
If all of your bindings are implicit, you can get away without defining a module at all.