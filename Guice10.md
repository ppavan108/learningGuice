_Guice 1.0 Release_

### Guice 1.0
Released March 8, 2007.

#### Downloads
  * [guice-1.0.zip](https://github.com/google/guice/releases/download/1.0/guice-1.0.zip)
  * [guice-1.0-src.zip](https://github.com/google/guice/archive/1.0.zip)
  * [guice-struts2-plugin-1.0.1.jar](https://github.com/google/guice/releases/download/1.0/guice-struts2-plugin-1.0.1.jar)


#### Docs
  * [Javadoc API](http://google.github.io/guice/api-docs/1.0/javadoc/index.html)
  * [Guice 1.0 User's Guide.pdf](http://google.github.io/guice/user-docs/Guice-1.0-Users-Guide.pdf)

#### Changes

##### 1.0
  * Changed constant binding API. Replaced `bindConstant(annotation)` with `bindConstant().annotatedWith(annotation)`. 
  * Fixed bug in ordering of instance injection.

##### 1.0rc3
  * Renamed `Container` to `Injector`
  * Renamed "container scope" to "singleton"
  * Guice now blows up if a custom provider returns `null`
  * Added `@ImplementedBy` and `@ProvidedBy`
  * Removed `Binder.link()`. `Binder.bind().to(...)` now links.
  * Further fleshed out binder expression language.
  * Added Spring integration.
  * Added JNDI integration.
  * Made `CreationException` a runtime exception.

##### 1.0rc2 (changes since 1.0rc1)
  * Added `@ScopeAnnotation`. If you forget to bind a scope, you'll get an error.
  * Added `Binder.addError()` so you can record custom error messages from modules.
  * Removed `Scopes.DEFAULT`. We now refer to this as "no scope."
  * Added up front checks to ensure annotations have runtime retention and the proper meta annotations.
  * Added support for injecting private and protected members (cglib fast reflection doesn't support this).
  * Renamed `Locator` to `Provider`
  * Replaced `Factory` with `Provider` also
  * Removed `Context` from public API. We may support this functionality via injection in a future release but we're leaving it out for now.
  * Extracted an interface for `Binding`
  * We now inject members of instances bound using `toInstance()` and `toProvider()` upon container creation.
  * When a binding annotation has attributes, if we can't find a binding to the same set of attribute values, we now look for a binding to just the annotation type.
  * Created a separate jar for the `servlet` package
  * Created a Struts 2 plugin jar
