_Blogs, articles, books on Guice._
### External Documentation

#### Books

##### [Dependency Injection by Dhanji Prasanna](http://www.manning.com/prasanna/)
  In a traditional object-oriented application, a primary program controls secondary pieces of code, such as classes in a module, library, or framework. Dependency Injection (DI) is a technique that inverts this control, using an external mechanism to insert—or inject—a reference to an implementation of a service into an object. This allows you to build complex OO applications in a more testable, maintainable, and business-focused manner.

##### [Google Guice by Robbie Vanbrabant](http://www.apress.com/9781590599976)
  _Google Guice: Agile Lightweight Dependency Injection Framework_ will not only tell you “how,” it will also tell you “why” and “why not,” so that all the knowledge you gain will be as widely applicable as possible. Filled with examples and background information, this book is an invaluable addition to your knowledge of modern agile Java.

#### Videos
##### [Big Modular Java with Guice](http://code.google.com/events/io/sessions/BigModularJavaGuice.html)
An introduction to Guice by Dhanji R. Prasanna and Jesse Wilson, at Google I/O 2009.
  Learn how Google uses the fast, lightweight Guice framework to power some of the largest and most complex applications in the world. Supporting scores of developers, and steep testing and scaling requirements for the web, Guice proves that there is still ample room for a simple, type-safe and dynamic programming model in Java. This session will serve as a simple introduction to Guice, its ecosystem and how we use it at Google.

##### [Introduction to Google Guice: Programming is fun again!](http://developers.sun.com/learning/javaoneonline/sessions/2009/pdf/TS-5434.pdf)
PDF slides from the sold-out technical session at !JavaOne 2009.

#### Articles
##### [Dependency injection with Guice by Nicholas Lesiecki](http://www.ibm.com/developerworks/java/library/j-guice/index.html)
  Guice is a dependency injection (DI) framework. I've suggested for years that developers use DI, because it improves maintainability, testability, and flexibility. By watching engineers react to Guice, I've learned that the best way to convince a programmer to adopt a new technology is to make it really easy. Guice makes DI really easy, and as a result, the practice has taken off at Google. I hope to continue in the same vein in this article by making it really easy for you to learn Guice.


##### [Guicing Up Your Testing by Dick Wall](http://www.developer.com/design/article.php/3684656/Guicing-Up-Your-Testing.htm)
  This article examines the simplest and most obvious use case for the Guice container, for mocking or faking objects in unit tests.

  _Also available in [Portuguese](http://fabiolnm.blogspot.com/2009/11/teste-unitario-com-junit-e-google-guice.html)_

##### [Squeezing More Guice from Your Tests with EasyMock by Dick Wall](http://www.developer.com/design/article.php/3688436/Squeezing-More-Guice-from-Your-Tests-with-EasyMock.htm)
   It should be apparent that making Invoice something that Guice creates directly is an incorrect design decision. Instead, you can mix it with another pattern instead: Factory.


#### Blogs

##### [Guice with GWT](http://stuffthathappens.com/blog/2009/09/14/guice-with-gwt/)
Configure your serverside app to serve up a backend for GWT apps.
  Obtain the Guice JAR files, extend GWT’s `RemoteServiceServlet`, extend Guice’s `GuiceServletContextListener`, extend Guice’s `ServletModule`, set all `RemoteService` relative paths to _GWT.rpc_, and configure `GuiceFilter` and your context listener in _web.xml_.

##### [Refactoring to Guice: Part 1 of N](http://publicobject.com/2007/07/guice-patterns-1-horrible-static-code.html)
Guide for migrating from factories to dependency injection.
  In this N-part series, I'm attempting to document some patterns for improving your code with Guice. In each example, I'll start with sample code, explain what I don't like about it, and then show how I clean it up.

##### [What's a Hierarchical Injector?](http://publicobject.com/2008/06/whats-hierarchical-injector.html)
  The premise is simple. @Inject anything, even stuff you don't know at injector-creation time. So our DeLorean class would look exactly as it would if EnergySource was constant:

##### [TypeResolver tells you what List.get() returns](http://publicobject.com/2008/07/typeresolver-tells-you-what-listget.html)
What the new methods on `TypeLiteral` do.

##### [Simpler service interfaces with Guice](http://publicobject.com/2007/11/simpler-service-interfaces-with-guice.html)
A brief discussion on contextual APIs.

##### [Guice AOP Example](http://sarah-a-happy.livejournal.com/145875.html)
```java
  @Trace("note goes here")
  public void bye() {
    System.out.println("see you later");
  }
```

##### [Alternative way to integrate Guice with Wicket](http://headtoscreencollision.blogspot.com/2010/05/wicket-and-guice-alternate-route.html)
Explains a better way to integrate Guice and Wicket, using Guice 2.0 and ServletModule which allows you to get rid of even more XML in your projects!