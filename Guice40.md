### Guice 4.0

Released April 28, 2015

#### Maven

Guice:

```xml
<dependency>
  <groupId>com.google.inject</groupId>
  <artifactId>guice</artifactId>
  <version>4.0</version>
</dependency>
```

Guice (No AOP):

```xml
<dependency>
  <groupId>com.google.inject</groupId>
  <artifactId>guice</artifactId>
  <version>4.0</version>
  <classifier>no_aop</classifier>
</dependency>
```

Extensions:

```xml
<dependency>
  <groupId>com.google.inject.extensions</groupId>
  <artifactId>guice-${extension}</artifactId>
  <version>4.0</version>
</dependency>
```

#### Downloads

 * Guice: [guice-4.0.jar](http://search.maven.org/remotecontent?filepath=com/google/inject/guice/4.0/guice-4.0.jar)
 * Guice (No AOP): [guice-4.0-no_aop.jar](http://search.maven.org/remotecontent?filepath=com/google/inject/guice/4.0/guice-4.0-no_aop.jar)
 * Guice extensions are all directly downloadable [from this search page](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.google.inject.extensions%22%20AND%20v%3A%224.0%22).  Just click on the "jar" link for the appropriate extension.

#### Docs

  * [API documentation](https://google.github.io/guice/api-docs/4.0/javadoc/index.html)
  * [API Changes from 3.0 to 4.0, by JDiff](http://google.github.io/guice/api-docs/4.0/api-diffs/changes.html)

#### Changelog (since Guice 3.0)
##### Some highlights:
   * Java8 runtime compatibility for Guice core & all extensions.
   * Added [testlib extension](http://search.maven.org/#artifactdetails%7Ccom.google.inject.extensions%7Cguice-testlib%7C4.0%7Cjar), initially with support for easily [binding fields in tests] (https://github.com/google/guice/wiki/BoundFields) using [BoundFieldModule](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/testing/fieldbinder/BoundFieldModule.html).
   * Added [dagger-adapter extension](http://search.maven.org/#artifactdetails%7Ccom.google.inject.extensions%7Cguice-dagger-adapter%7C4.0%7Cjar) to allow using dagger2 modules with Guice (using [DaggerAdapter](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/daggeradapter/DaggerAdapter.html)).
   * Added [OptionalBinder](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/OptionalBinder.html) to the Multibinder extension, to allow frameworks to depend on something that users may or may not bind.  (Also allows a default value for a binding that users can change.)
   * Provisioning can be intercepted with a [ProvisionListener](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/ProvisionListener.html).
   * Singletons can be initialized concurrently.
   * Injecting objects without @Inject constructors can be disabled with [Binder.requireAtInjectOnConstructors](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/Binder.html#requireAtInjectOnConstructors--).
   * Extensions can add bindings using @Provides-like methods, by binding a [ModuleAnnotatedMethodScanner](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/ModuleAnnotatedMethodScanner.html).
   * Multibinder or MapBinder items can be bound using [@ProvidesIntoSet](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/ProvidesIntoSet.html) or [@ProvidesIntoMap](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/ProvidesIntoMap.html) by installing a [MultibindingsScanner](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/MultibindingsScanner.html).
   * The current `@RequestScope` can be detached & reattached later using [ServletScopes.transferRequest](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/servlet/ServletScopes.html#transferRequest-java.util.concurrent.Callable-).
   * Lots of new validation for @Provides methods (see details below).

##### Details, per core + extension...
###### Guice Core:
  * Changed validation on @Provides methods -- it is now forbidden to override them.
  * Changed validation on null being passed into @Provides method parameters.  Parameters must now be @Nullable to allow null (the same as injecting methods or constructors).
  * Changed the way Singletons are initialized -- there is no system-wide lock anymore.  Singletons can now be instantiated concurrently.
  * Changed requireExplicitBindings on child injectors or PrivateModules so that implicit bindings aren't promoted to the parent injector/module, even if the parent doesn't have requireExplicitBindings set.
  * Added [ProvisionListener](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/ProvisionListener.html) (installed with [Binder.bindListener](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/Binder.html#bindListener-com.google.inject.matcher.Matcher-com.google.inject.spi.ProvisionListener...-)) to intercept provisioning.
  * Added [Binder.requireAtInjectOnConstructors](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/Binder.html#requireAtInjectOnConstructors--) to prevent Guice from creating instances of classes that have no-args public constructors w/o @Inject.
  * Added [Binder.requireExactBindingAnnotations](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/Binder.html#requireExactBindingAnnotations--) to prevent Guice from allowing a binding for @Named Foo to fulfill a request for @Named("foo") Foo, if the latter was missing but the former existed.  (This applies to all parameterized annotations, not just @Named.)
  * Added [ElementSource](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/ElementSource.html), returned from [Binding.getSource](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/Element.html#getSource--) when possible, to get more detailed information about the binding.  This includes keeping track of all the modules in the path that led to the binding.
  * Added an SPI for @Provides methods. Call Binding.acceptTargetVisitor with the new [ProvidesMethodTargetVisitor](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/ProvidesMethodTargetVisitor.html).
  * Added [ModuleAnnotatedMethodScanner](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/ModuleAnnotatedMethodScanner.html) (installed by [Binder.scanModulesForAnnotatedMethods](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/Binder.html#scanModulesForAnnotatedMethods-com.google.inject.spi.ModuleAnnotatedMethodScanner-) to allow extensions to hook into Modules, scanning for methods annotated with @Provides-like methods and creating bindings.
  * Added the ability to to stop recording line numbers of bindings, which can increase startup time drastically (at the expense of less useful error messages).  (Use the system property `guice_include_stack_traces` set to `OFF`, `ONLY_FOR_DECLARING_SOURCE` (the default), or `COMPLETE`).
  * Added [LinkedBindingBuilder.toProvider(javax.inject.Provider)](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/binder/LinkedBindingBuilder.html#toProvider-javax.inject.Provider-), for better JSR330 compatibility.
  * Added [InjectionPoint.forMethod](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/spi/InjectionPoint.html#forMethod-java.lang.reflect.Method-com.google.inject.TypeLiteral-) & [Binder.getProvider(Dependency)](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/Binder.html#getProvider-com.google.inject.spi.Dependency-) to allow building injection points off arbitrary methods.
  * Allowed duplicate listener, interceptor & scope bindings to be ignored (instead of failing or installing them twice).
  * Allowed method interceptors to capture the intercepted method and invoke it later.
  * Allowed `@ProvidedBy` to work with enums ([issue 295](https://github.com/google/guice/issues/295)).
  * Improved error messages to list out all modules that led to a failed binding.
  * Improved error messages when requestStaticInjection is called on an interface.
  * Fixed the way Key works with annotations, so that it considers `@Annotation` is equal to `Annotation.class` if the annotation has all optional methods.
  * Fixed interceptors to allow intercepting bridge methods that javac automatically creates sometimes.
  * Fixed circular dependencies between providers ([issue 626](https://github.com/google/guice/issues/626)).
  * Fixed classloader leak relating to AOP ([issue 643](https://github.com/google/guice/issues/643)).
  * Fixed anonymous TypeLiteral/Key instances so they don't retain references to the enclosing Module ([issue 910](https://github.com/google/guice/issues/910)).
  * Fixed exceptions when encountering circular just-in-time bindings, when scopes cached proxies for circular dependencies, and when requestInjection (and others) try to inject the same object more than once in a different order than they were bound.
  * Fixed `currentStage()` when used in `Modules.override`.
  * Fixed internal calls to `System.getProperty` so exceptions are ignored.
  * Optimized memory required when dealing with child injectors ([issue 756](https://github.com/google/guice/issues/756)).
  * Optimized @Provides methods (by invoking them through cglib, saving ~200-300ns per method invocation).
  * Optimized memory required for keeping binding source information.
  * Optimized TypeConverters.

###### Servlet extension
  * Added [ServletScopes.transferRequest](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/servlet/ServletScopes.html#transferRequest-java.util.concurrent.Callable-) to allow the current scope to be transferred to a different callable.
  * Added [ServletScopes.isRequestScoped](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/servlet/ServletScopes.html#isRequestScoped-com.google.inject.Binding-) to see if a binding is request scoped.
  * Added the Key being requested into OutOfScopeException's message.
  * Added a [`@ScopingOnly GuiceFilter`](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/servlet/ScopingOnly.html) that can be injected to provide scoping functionality without the dispatching.
  * Fixed ordering of filter processing -- the first binding is processed first.  (It could have been in an arbitrary order if filters were set by different modules.)
  * Fixed the request/response passed to filters & servlets so they get the latest request/response (even if it's wrapped by an earlier filter).
  * Fixed manipulation of the servlet path info ([issue 372](https://github.com/google/guice/issues/372)).
  * Fixed manipulation of the context path.
  * Fixed pattern matching to ignore the query portion of the URI ([issue 379](https://github.com/google/guice/issues/379)).
  * Fixed to decode the PathInfo ([issue 745](https://github.com/google/guice/issues/745)).
  * Fixed exception when getCookies returned null.
  * Fixed to work with requireExplicitBindings.
  * Optimized the stack size when calling filters and trimmed excessive calls to doFilter from exception stack traces.
  * Optimized performance per served request.
  * Optimized `scopeRequest`.

###### Multibinder
  * Changed Multibinder to also bind a `Collection<Provider<T>> and Collection<javax.inject.Provider<T>>`.
  * Changed MapBinder to also bind a `Map<K, javax.inject.Provider<V>>`.
  * Added [OptionalBinder](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/OptionalBinder.html), to allow frameworks to depend on something that users may or may not bind.  (Also allows a default value for a binding that users can change.)
  * Added [MultibindingsScanner](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/MultibindingsScanner.html) to allow using [@ProvidesIntoSet](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/ProvidesIntoSet.html), [@ProvidesIntoMap](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/ProvidesIntoMap.html) and [@ProvidesIntoOptional](https://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/multibindings/ProvidesIntoOptional.html) on methods in modules.
  * Fixed Multibinder/MapBinder to work better with Modules.override, ignoring exact same bindings instead of binding them twice.  (Most usages of permitDuplicates should be unnecessary now.)
  * Fixed Multibinder & MapBinder so they can't conflict with each other ([issue 670](https://github.com/google/guice/issues/670)).
  * Improved error messages for duplicate bindings so it tells you what the duplicates were and where they were bound.

##### AssistedInject
   * Changed the validation on assisted injected classes to fail if the class has a scoping annotation.

###### ThrowingProvider
   * Added [ThrowingProviderBinder.SecondaryBinder.providing(Class|TypeLiteral)](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/throwingproviders/ThrowingProviderBinder.SecondaryBinder.html#providing-java.lang.Class-), which creates a proxy CheckedProvider that will construct the given class.
   * Added aCheckedProviders([scopeExceptions](http://google.github.io/guice/api-docs/4.0/javadoc/com/google/inject/throwingproviders/CheckedProvides.html#scopeExceptions--)) method, to allow turning off exception scoping.
   * Improved exception messages when more than one underlying dependency fails.

###### Grapher
   * Allowed injecting non-scoped graphers.
   * Fixed the way PrivateModules are handled to show exposed nodes ([issue 489](https://github.com/google/guice/issues/489)).
   * Fix corrupt/bad looking graphs ([issue 663](https://github.com/google/guice/issues/663)).
   * Fix incorrect grapher config ([issue 755](https://github.com/google/guice/issues/755)).

###### Persist
   * Fixed so that log4j.properties file is not in the jar.
   * Fixed to remove the entityManager even if `close()` throw an exception ([issue 597](https://github.com/google/guice/issues/597)).
   * Allowed JPA properties to be arbitrary Objects (in addition to Strings).

#### Migrating from Guice 3.0
See the [JDiff change report](http://google.github.io/guice/api-docs/4.0/api-diffs/changes.html) for complete details.
