### Guice 4.1

Released June 17, 2016

#### Maven

Guice:

```xml
<dependency>
  <groupId>com.google.inject</groupId>
  <artifactId>guice</artifactId>
  <version>4.1</version>
</dependency>
```

Guice (No AOP):

```xml
<dependency>
  <groupId>com.google.inject</groupId>
  <artifactId>guice</artifactId>
  <version>4.1</version>
  <classifier>no_aop</classifier>
</dependency>
```

Extensions:

```xml
<dependency>
  <groupId>com.google.inject.extensions</groupId>
  <artifactId>guice-${extension}</artifactId>
  <version>4.1</version>
</dependency>
```

#### Downloads

 * Guice: [guice-4.1.jar](http://search.maven.org/remotecontent?filepath=com/google/inject/guice/4.1/guice-4.1.jar)
 * Guice (No AOP): [guice-4.1-no_aop.jar](http://search.maven.org/remotecontent?filepath=com/google/inject/guice/4.1/guice-4.1-no_aop.jar)
 * Guice extensions are all directly downloadable [from this search page](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.google.inject.extensions%22%20AND%20v%3A%224.1%22).  Just click on the "jar" link for the appropriate extension.

#### Docs

  * [API documentation](https://google.github.io/guice/api-docs/4.1/javadoc/index.html)
  * [API Changes from 4.0 to 4.1, by JDiff](http://google.github.io/guice/api-docs/4.1/api-diffs/changes.html)

#### Changes since Guice 4.0
##### Guice Core:
  * `@ProvidedBy` supports javax.inject.Provider [issue #808](https://github.com/google/guice/issues/808)
  * Reduced the number of lines in error messages.
  * Fix potential rare deadlock when there are multi-threaded cycles in singletons.
  * Allow members injection to happen in parallel if it was requested from different threads.
  * Make disableCircularProxies more strict.
  * Updated the internal cglib to 3.2.0 to allow Guice to be used with signed jars.
  * Various performance optimizations.

##### AssistedInject:
  * Skip static methods in interfaces (for Java8 compatibility) [issue #937](https://github.com/google/guice/issues/937)

##### Servlet:
  * Add Iterable overloads for the varargs methods in ServletModule.
  * Add an AutoCloseable-style API for transferring the request scope.
  * Normalize the path before applying filters.

##### Multibindings:
  * Add more bound types when using MapBinder.  By default, `Set<Map.Entry<K, javax.inject.Provider<V>>` will be bound.  And additionally if `permitDuplicates()` is called, the following are also bound: `Map<K, Set<javax.inject.Provider<V>>>`, `Map<K, Collection<Provider<V>>>`, and `Map<K, Collection<javax.inject.Provider<V>>>`.

##### Testing Libraries:
  * Fix `@Bind` to support javax.inject.Provider.
  * Allow `@Bind` to bind to null if annotated with `@Nullable`.
  * Allow `@Bind(lazy=true)` with Providers.
  * 

##### Dagger Adapter:
  * Support the new multibinding declaration style (`@IntoSet`, etc..)
  * 


#### Migrating from Guice 4.0
See the [JDiff change report](http://google.github.io/guice/api-docs/4.1/api-diffs/changes.html) for complete details.
