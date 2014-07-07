### Using Guice with Google App Engine

You can use Guice with to write modular applications for [Google App Engine](http://code.google.com/appengine/).

### Supported Builds
Google App Engine support requires Guice 2 (with or without AOP), plus the guice-servlet extension.

### Setup

##### Servlet and Filter Registration
[[Configure servlets and filters|ServletModule]] by subclassing `ServletModule`:
```java
package com.mycompany.myproject;

import com.google.inject.servlet.ServletModule;

class MyServletModule extends ServletModule {
  @Override protected void configureServlets() {
    serve("/*").with(MyServlet.class);
  }
}
```

##### Injector Creation
Construct your Guice injector in the `getInjector()` method of a class that extends GuiceServletContextListener. Be sure to include your application's servlet module in the list of modules.
```java
package com.mycompany.myproject;

import com.google.inject.servlet.ServletModule;
import com.google.inject.servlet.GuiceServletContextListener;
import com.google.inject.Guice;
import com.google.inject.Injector;

public class MyGuiceServletContextListener extends GuiceServletContextListener {

  @Override protected Injector getInjector() {
    return Guice.createInjector(
        new MyServletModule(),
        new BusinessLogicModule());
  }
}
```

##### Web.xml Configuration
You must register both the [[GuiceFilter|Servlets]] and your subclass of [GuiceServletContextListener](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/servlet/GuiceServletContextListener.html) in your application's `web.xml` file. All other servlets and filters may be configured in your servlet module.
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app 
   xmlns="http://java.sun.com/xml/ns/javaee" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
   version="2.5"> 
  <display-name>My Project</display-name>
  
 <filter>
    <filter-name>guiceFilter</filter-name>
    <filter-class>com.google.inject.servlet.GuiceFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>guiceFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <listener>
    <listener-class>com.mycompany.myproject.MyGuiceServletContextListener</listener-class>
  </listener>
</web-app>
```

##### WAR Layout
Ensure the AOP alliance, Guice, and Guice servlet jars are in the `WEB-INF/lib` directory of your `.war` file (or `www` directory):
```xml
  www/
      WEB-INF/
          lib/
              aopalliance.jar
              guice-servlet-snapshot.jar
              guice-snapshot.jar
              ...
          classes/
              ...
          appengine-web.xml
          web.xml
```