_Guice 2.0 Release_
### Guice 2.0
Released May 19, 2009

##### Downloads
  * **[guice-2.0.zip](http://google-guice.googlecode.com/files/guice-2.0.zip)** jars and Javadoc. 1.1MB
  * **[guice-2.0-src.zip](http://google-guice.googlecode.com/files/guice-2.0-src.zip)** sources, tests, dependencies and Javadoc. 16.5MB
  * **[guice-2.0-no_aop.jar](http://google-guice.googlecode.com/files/guice-2.0-no_aop.jar)** replaces guice-2.0.jar on platforms that don't support AOP (ie. Android). 430KB

##### Docs
  * [Javadoc API](http://google-guice.googlecode.com/svn/tags/2.0/javadoc/index.html)
  * [User's Guide](http://code.google.com/docreader/#p=google-guice&s=google-guice&t=Motivation)

##### New Features

###### Small Features
  * `Binder.getProvider` and `AbstractModule.requireBinding` allow modules to declare and use dependencies.
  * `Binder.requestInjection` allows modules to register instances for injection at Injector-creation time.
  * `Providers.of()` always provides the same instance, useful for testing.
  * `Scopes.NO_SCOPE` allows you make no-scoping explicit.
  * `Matcher.inSubpackage` matches all classes in any child package of the given one.
  * `Types` utility class for creating implementations of generic types.
  * `AbstractModule.requireBinding` allows modules to document dependencies programatically

###### Provider Methods
Creating custom providers without all of the boilerplate. In any module, annotate a method with `@Provides` to define logic that will be used to provide that type:
```java
class PaymentsModule extends AbstractModule {
  public void configure() {
    ...
  }

  @Provides @Singleton
  PaymentService providePaymentService(CustomerDb database) {
    DatabasePaymentService result = new DatabasePaymentService();
    result.setDatabase(database);
    result.setReplicationLevel(5);
    return result;
  }
```
Parameters to the @Provides method will be injected. It can optionally be annotated with a scoping annotation (like `@Singleton`). The method's returned type is the bound type. Annotate the method with a binding annotation (like `@Named("replicated")`) to create a binding for the annotated type.

###### Binding Overrides
[Override](http://publicobject.com/2008/05/overriding-bindings-in-guice.html) the bindings from one module with another:
```java
Module functionalTestModule 
        = Modules.override(new ProductionModule()).with(new OverridesModule());
```

###### Multibindings, MapBindings
Bind elements of [Sets](http://publicobject.com/2008/05/guice-multibindings-extension-checked.html) and [Maps](http://publicobject.com/2008/05/mapbinder-checked-in.html), then inject the full collection. Bindings are aggregated from all modules.
```java
public class SnacksModule extends AbstractModule {
  protected void configure() {
    MapBinder<String, Snack> mapBinder
        = MapBinder.newMapBinder(binder(), String.class, Snack.class);
    mapBinder.addBinding("twix").toInstance(new Twix());
    mapBinder.addBinding("snickers").toProvider(SnickersProvider.class);
    mapBinder.addBinding("skittles").to(Skittles.class);
  }
}
```

###### Private Modules
[PrivateModules](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/PrivateModule.html) can be used to create bindings that are not externally visible. This makes it easier to encapsulate dependencies and to avoid bind conflicts.

###### Servlets
`ServletModule` now supports programmatic configuration of servlets and filters (get rid of web.xml). 

These servlets may be:
  * constructor injected directly
  * support AOP
  * circular references, and all Guice POJO idioms.

Guice Servlet also supports regex URL mapping and the ability to package and bundle your own servlet libraries (for example, an SOAP web service).

`GuiceServletContextListener` can be used to help bootstrap a Guice application in a servlet container.

###### Child Injectors
[Injector.createChildInjector](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/Injector.html#createChildInjector(java.lang.Iterable)) allows you to create child injectors that inherit the bindings, scopes, interceptors and converters of their parent. This API is primarily intended for extensions and tools.

###### Even Better Error Reporting
Exceptions in Guice 1.0 tend to include long chains of 'caused by' exceptions. We've tidied this up! Now a single exception describes concisely what Guice was doing when the problem occurred.

###### Introspection API
Like `java.lang.reflect`, but for Guice. It lets you rewrite a Module, tweaking bindings programatically. It also lets you inspect a created injector, and examine its bindings. This is intended to enable simpler, more powerful extensions and tools for Guice.

###### Custom Injections
[[Type and instance listeners|CustomInjections]] enable Guice to host other frameworks that have their own injection semantics or annotations.

###### Pluggable Type Converters
Constant String bindings can be converted to arbitrary types (such as dates, URLs, or colours) using pluggable type converters.

###### Available without AOP
Guice 2.0 is available [[without AOP|OptionalAOP]] for platforms like [Android](http://code.google.com/android/) that don't support bytecode manipulation.

###### OSGi-friendly AOP
Guice does bytecode generation internally to implement AOP. In version 2.0, generated classes are loaded by a bridge classloader that works in managed environments such as OSGi.

###### Type Resolution
[Parameterized injection points](http://groups.google.com/group/google-guice/browse_thread/thread/1355313a074d8094/88270edbbeae2df8) allow you to inject types like `Reducer<T>` or `Converter<A, B>`. Guice will figure out what `T` is and find the binding for that type. [TypeLiteral injection](http://publicobject.com/2008/11/guice-punches-erasure-in-face.html) means you can inject a `TypeLiteral<T>` into your classes. Use this to reify [Java 5 type erasure](http://java.sun.com/docs/books/tutorial/java/generics/erasure.html). The `TypeLiteral` class now includes library methods for manual type resolution.

###### Migrating from Guice 1.0
Guice 2.0 breaks compatibility with Guice 1.0 as described below. See the [JDiff change report](http://google-guice.googlecode.com/svn/trunk/latest-api-diffs/2.0/changes.html) for complete details. 

###### Exception Types
In Guice 1.0, when a custom provider throws an unchecked exception, sometimes Guice wraps the exception and sometimes it doesn't. This depends on whether the provider is being called directly (via Provider.get()) or indirectly (such as for injecting into another type).

In Guice 2.0, any time a provider or injection throws, Guice wraps it in a ProvisionException. [This rule is simpler](http://publicobject.com/2008/04/future-guice-providers-that-throw.html), and it makes it easier to write fault-tolerant code with Guice.

Injector creation problems are always reported as CreationException. Runtime configuration problems (ie. programmer errors) are always reported as ConfigurationException.

ProvisionException, ConfigurationException and OutOfScopeException are now public.

###### Abstract Types
Guice doesn't support injecting into abstract types. The messaging around this has been improved since 1.0, and some code that was silently failing now throws exceptions.

###### Inner Classes
Guice used to support constructor injection of nonstatic inner classes. So this used to work, but it won't anymore:
```java
public class FooTest extends TestCase {
  public void testFoo() {
    Foo foo = Guice.createInjector().getInstance(FakeFoo.class)
  }

  class FakeFoo implements Foo {
    @Inject TestFoo() {...}
  }
}
```

###### Keys
Guice now [canonicalizes](http://publicobject.com/2008/06/integerclass-and-intclass-as-guice-keys.html) primitive types (like `int.class`) and array types (like `Integer[].class`) when they're used in Keys. It now supports wildcards like `List<?>` in keys - use `@Provides` to bind these.

###### Annotation Implementations
Guice 2.0 fixes the treatment of equals() and hashCode() for fieldless annotations. Annotation implementations that don't implement equals() and hashCode() may have worked in 1.0 but will be broken in 2.0.

###### Injector.getBinding
Guice 2.0 throws an exception if the binding cannot be resolved. The old version used to return null. To get the old behaviour, use `injector.getBindings().get(key)`.

###### SPI Changes
SourceProviders have been replaced with `Binder.withSource` and `Binder.skipSources`. These new methods are easier to call and test. They don't require static initialization or static dependencies.