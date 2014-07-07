### Scopes
By default, Guice returns a new instance each time it supplies a value. This behaviour is configurable via *scopes*. Scopes allow you to reuse instances: for the lifetime of an application (`@Singleton`), a session (`@SessionScoped`), or a request (`@RequestScoped`). Guice includes a servlet extension that defines scopes for web apps. [[Custom scopes|CustomScopes]] can be written for other types of applications.


### Applying Scopes
Guice uses annotations to identify scopes. Specify the scope for a type by applying the scope annotation to the implementation class. As well as being functional, this annotation also serves as documentation. For example, `@Singleton` indicates that the class is intended to be threadsafe.
```java
@Singleton
public class InMemoryTransactionLog implements TransactionLog {
  /* everything here should be threadsafe! */
}
```
Scopes can also be configured in `bind` statements:
```java
  bind(TransactionLog.class).to(InMemoryTransactionLog.class).in(Singleton.class);
```
And by annotating `@Provides` methods:
```java
  @Provides @Singleton
  TransactionLog provideTransactionLog() {
    ...
  }
```
If there's conflicting scopes on a type and in a `bind()` statement, the `bind()` statement's scope will be used. If a type is annotated with a scope that you don't want, bind it to `Scopes.NO_SCOPE`.

In linked bindings, scopes apply to the binding source, not the binding target. Suppose we have a class `Applebees` that implements both `Bar` and `Grill` interfaces. These bindings allow for *two* instances of that type, one for `Bar`s and another for `Grill`s:
```java
  bind(Bar.class).to(Applebees.class).in(Singleton.class);
  bind(Grill.class).to(Applebees.class).in(Singleton.class);
```
This is because the scopes apply to the bound type (`Bar`, `Grill`), not the type that satisfies that binding (`Applebees`). To allow only a single instance to be created, use a `@Singleton` annotation on the declaration for that class. Or add another binding:
```java
  bind(Applebees.class).in(Singleton.class);
```
This binding makes the other two `.in(Singleton.class)` clauses above unnecessary.

The `in()` clause accepts either a scoping annotation like `RequestScoped.class` and also `Scope` instances like `ServletScopes.REQUEST`:
```java
  bind(UserPreferences.class)
      .toProvider(UserPreferencesProvider.class)
      .in(ServletScopes.REQUEST);
```
The annotation is preferred because it allows the module to be reused in different types of applications. For example, an `@RequestScoped` object could be scoped to the HTTP request in a web app and to the RPC when it's in an API server.


### Eager Singletons
Guice has special syntax to define singletons that can be constructed eagerly:
```java
  bind(TransactionLog.class).to(InMemoryTransactionLog.class).asEagerSingleton();
```
Eager singletons reveal initialization problems sooner, and ensure end-users get a consistent, snappy experience. Lazy singletons enable a faster edit-compile-run development cycle. Use the `Stage` enum to specify which strategy should be used.

                      | *PRODUCTION* | *DEVELOPMENT* |
----------------------|--------------|---------------|
.asEagerSingleton()   | eager        | eager         |
.in(Singleton.class)  | eager        | lazy          |
.in(Scopes.SINGLETON) | eager        | lazy          |
@Singleton            | eager`*`     | lazy          |

`*` Guice will only eagerly build singletons for the types it knows about. These are the types mentioned in your modules, plus the transitive dependencies of those types.


### Choosing a scope
If the object is *stateful*, the scoping should be obvious. Per-application is `@Singleton`, per-request is `@RequestScoped`, etc. If the object is *stateless* and *inexpensive to create*, scoping is unnecessary. Leave the binding unscoped and Guice will create new instances as they're required.

Singletons are popular in Java applications but they don't provide much value, especially when dependency injection is involved. Although singletons save object creation (and later garbage collection), initialization of the singleton requires synchronization; getting a handle to the single initialized instance only requires reading a volatile. Singletons are most useful for:
  * stateful objects, such as configuration or counters
  * objects that are expensive to construct or lookup
  * objects that tie up resources, such as a database connection pool.


### Scopes and Concurrency
Classes annotated `@Singleton` and `@SessionScoped` *must be threadsafe*. Everything that's injected into these classes must also be threadsafe. [[Minimize mutability|MinimizeMutability]] to limit the amount of state that requires concurrency protection.

`@RequestScoped` objects do not need to be threadsafe. It is usually an error for a `@Singleton` or `@SessionScoped` object to depend on an `@RequestScoped` one. Should you require an object in a narrower scope, inject a `Provider` of that object.