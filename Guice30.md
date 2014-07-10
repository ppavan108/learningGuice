_Guice 3.0 Release_
### Guice 3.0
Released March 24, 2011

#### Downloads
 * **[guice-3.0.zip](https://github.com/google/guice/releases/download/3.0/guice-3.0.zip)** core & extension jars.  866KB
 * **[guice-3.0-src.zip](https://github.com/google/guice/archive/3.0.zip)** sources, tests, dependencies and Javadoc. 23.8MB
 * **[guice-3.0-no_aop.jar](https://github.com/google/guice/releases/download/3.0/guice-3.0-no_aop.jar)** core without AOP, suitable for Android.  470KB

#### Docs
  * [Javadoc API](http://google.github.io/guice/api-docs/3.0/javadoc/index.html)
  * [API Changes from 2.0 to 3.0, by JDiff](http://google.github.io/guice/api-docs/3.0/api-diffs/changes.html)

#### New Features
  * [[JSR 330 support|JSR330]]
  * [[New Persist extension|GuicePersist]]
  * [[ToConstructor Bindings|ToConstructorBindings]]
  * Better OSGi support (and generally improved support for multiple classloaders)
  * New methods in Binder: `requireExplicitBindings` & `disableCircularProxies`
  * Much simpler stack traces when AOP is involved
  * Exact duplicate bindings are ignored (instead of an exception being thrown)
  * Repackaged cglib, asm & guava classes are now hidden from IDE auto-imports
  * Source can be built with Maven
  * General extension & SPI improvements:
     * [[Extensions SPI|ExtensionSPI]], with support from the [[Multibindings]], [[ServletModule]] and [[AssistedInject]] extensions.
     * `@Toolable`, for use by extensions in Stage.TOOL
     * All binding implementations implement `HasDependencies`
  * Scope improvements:
     * `Scopes.isSingleton`
     * toInstance bindings are considered eager singletons in the Scope SPI
     * Singleton providers that return null are now scoped properly.
     * Fail when circularly dependent singletons cannot be proxied (previously two instances may have been created)     
  * Provider Method (@Provides) improvements:
     * Logger dependencies now use named instead of anonymous loggers
     * Exceptions produce a simpler stack 
  * Private Modules improvements:
     * If a single `PrivateModule` is passed to Modules.override, private elements can be overridden.
     * If many modules are passed to Modules.override, exposed keys can be overridden.
  * `Multibinder` & `MapBinder` extension improvements:
     * `permitDuplicates` -- allows `MapBinder` to inject a `Map<K, Set<V>>` and a `Map<K, Set<Provider<V>>` and `Multibinder` to silently ignore duplicate entries
     * Expose dependencies in Stage.TOOL, and return a dependency on Injector (instead of null) when working with Elements.
     * Co-exist nicely with Modules.override
     * Work properly with marker (parameterless) annotations
     * Expose details through the new extensions SPI
  * Servlet extension improvements:
     * Added `with(instance)` for servlet bindings.
     * Support multiple injectors using ServletModule in the same JVM.
     * Support thread-continuations (allow background threads to process an `HttpRequest`)
     * Support manually seeding a `@RequestScope`
     * Expose details through new extensions SPI
     * Fix request forwarding
     * Performance improvements for the filter & servlet pipelines
     * Expose the servlet context (`ServletModule.getServletContext`)
     * Fixed so a single ServletModule instance can be used twice (in, say Element.getElements & creating an Injector)
     * Fix to work with context paths
  * [[AssistedInject]] extension improvements
     * New `FactoryModuleBuilder` for building assisted factories
     * Injection is done through child injectors and can participate in AOP
     * Performance improvements
     * Multiple different implementations can be created from a single factory
     * Incorrect implementations or factories are detected in Stage.TOOL, and more error-checking with better error messages
     * Work better with parameterized types
     * Expose details through new extensions SPI
  * [[ThrowingProviders]] extension improvements
     * Added a new CheckedProviders interface that allows more than one exception to be thrown.
     * Added @CheckedProvides allowing @Provides-like syntax for CheckedProviders.
     * Dependencies are checked at Injector-creation time and during Stage.TOOL, so they can fail early
     * Bindings created by ThrowingProviderBinder now return proper dependencies instead of Injector.
  * [[Struts2|Struts2Integration]]
     * The Struts2 extension now works again! (It worked in Guice 1.0 and was somewhat broken in Guice 2.0)
  * Added to Injector: `getAllBindings`, `getExistingBinding`, `getScopeBindings`, `getTypeConverterBindings`.
  * Added to Key: `hasAttributes`, `ofType`, `withoutAttributes`
  * Various bug fixes / improvements:
     * Prevent child injectors from rebinding a parent just-in-time binding.
     * Non-intercepted methods in a class with other intercepted methods no longer go through cglib (reducing total stack frames).
     * Allow Guice to work on generated classes.
     * Fix calling Provider.get() of an unrelated Provider in a scoping method.
     * `MoreTypes.getRawTypes` returns the proper types for arrays (instead of Object[].class)
     * JIT bindings left in a parent injector when child injectors fail creating a JIT binding are now removed
     * Overriding an @Inject method (if the subclass also annotates the method with @Inject) no longer injects the method twice
     * You can now bind byte constants (using `ConstantBindingBuilder.to(byte)`)
     * Better exceptions when trying to intercept final classes, using keys already bound in child or private module, and many other cases.
     * ConvertedConstantBinding now exposes the TypeConverterBinding used to convert the constant.

#### Migrating from Guice 2.0
See the [JDiff change report](http://google.github.io/guice/api-docs/3.0/api-diffs/changes.html) for complete details.

#### JSR 330
Guice 3.0 now requires JSR 330 on your classpath.  This is the javax.inject.jar included in the guice download.

#### com.google.inject.internal
Many things inside com.google.inject.internal changed and/or moved.  This is especially true for repackaged Guava (formerly Google Collections), cglib, and asm classes.  All these classes are now hidden from IDE auto-import suggestions and are in new locations.  You will have to update your code if you relied on any of these classes.
