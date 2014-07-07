_Guice's SPI for authors of tools, extensions and plugins_

The [service provider interface](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/package-summary.html) exposes Guice's internal models to aid in the development in tools, extensions, and plugins. 

### Core Abstractions

|    |    | 
|----|----|
|[InjectionPoint](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/InjectionPoint.html) | A constructor, field or method that can receive injections. Typically this is a member with the `@Inject` annotation. For non-private, no argument constructors, the member may omit the annotation. Each injection point has a collection of dependencies. |
|[Key](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/Key.html) | A type, plus an optional binding annotation.|
|[Dependency](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/Dependency.html) | A key, optionally associated to an injection point. These exist for injectable fields, and for the parameters of injectable methods and constructors. Dependencies know whether they accept null values via an `@Nullable` annotation.|
|[Element](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/Element.html) | A configuration unit, such as a `bind` or `requestInjection` statement. Elements are [visitable](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/ElementVisitor.html). |
|[Module](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/Module.html) | A collection of configuration elements. [Extracting the elements](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/Elements.html#getElements(java.lang.Iterable)) of a module enables **static analysis** and **code-rewriting**. You can inspect, rewrite, and validate these elements, and use them to build new modules. |
|[Injector](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/Injector.html) | Manages the application's object graph, as specified by modules. SPI access to the injector works like **reflection**. It can be used to retrieve the application's bindings and dependency graph. |

The [[Elements SPI|InspectingModules]] page has more information about using these classes.
<p>

### Abstractions for Extension Authors

_(New in Guice 3.0)_

|    |    | 
|----|----|
[@Toolable](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/Toolable.html) | An annotation used on methods also annotated with `@Inject`.  This instructs Guice to inject the method even in `Stage.TOOL`.  Typically used for extensions that need to gather information to implement `HasDependencies` or validate requirements and fail early for easier testing.
[ProviderWithExtensionVisitor](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/ProviderWithExtensionVisitor.html) | An interface that provider instances implement to allow extensions to visit custom subinterfaces of [BindingTargetVisitor](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/spi/BindingTargetVisitor.html).  See [[Extensions SPI|ExtensionSPI]] for more information.

The [[Extensions SPI|ExtensionSPI]] page has more information about writing extensions that expose an SPI

### Examples

Log a warning for each static injection in your Modules:
```java
  public void warnOfStaticInjections(Module... modules) { 
    for (Element element : Elements.getElements(modules)) { 
      element.acceptVisitor(new DefaultElementVisitor<Void>() { 
        @Override 
        public Void visit(StaticInjectionRequest element) { 
          logger.warning("Static injection is fragile! Please fix " 
              + element.getType().getName() + " at " + element.getSource()); 
          return null; 
        } 
      }); 
    } 
  } 
```