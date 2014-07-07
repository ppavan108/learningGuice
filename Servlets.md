_Guice Servlet Extensions_

### Introduction

Guice Servlet provides a complete story for use in web applications and servlet containers. Guice's servlet extensions allow you to completely eliminate `web.xml` from your servlet application and take advantage of type-safe, idiomatic Java configuration of your servlet and filter components.

This has advantages not only in being able to use a nicer API for configuring your web applications, but also in tying together dependency injection with web components. Meaning that your servlets and filters benefit from:

  * Constructor injection
  * Type-safe, idiomatic configuration
  * Modularization (package and distribute custom Guice Servlet libraries)
  * Guice AOP

While keeping the benefits of the standard servlet lifecycle.

### Getting Started

Before you begin, you will require the latest version of the *guice-servlet* jar file, which is available along with the full Guice distribution from the [project homepage](http://github.com/google/guice) (or can be built from source using `ant dist distjars`). Once you have this library in your classpath, along with the core guice jar, you ready to go.

Start by placing `GuiceFilter` at the top of your `.web.xml` file:

```xml
  <filter>
    <filter-name>guiceFilter</filter-name>
    <filter-class>com.google.inject.servlet.GuiceFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>guiceFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

This tells the Servlet Container to re-route all requests through `GuiceFilter`. The nice thing about this is that any servlets or JSPs you already have will continue to function as normal, and you can migrate them to Guice Servlet at your own pace.