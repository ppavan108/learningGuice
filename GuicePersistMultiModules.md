_Multiple persistence providers with Guice Persist_

### Multiple Modules

So far we've seen how to use Guice Persist and JPA with a single persistence unit. This typically maps to one data store. Guice Persist supports using multiple persistence modules using the `private modules` API.

This means that you must be careful to install all JPA modules only in individual private modules:

```java
Module one = new PrivateModule() {
  protected void configure() {
    install(new JpaModule("persistenceUnitOne"));
    // other services...
  }
};

Module two = new PrivateModule() {
  protected void configure() {
    install(new JpaModule("persistenceUnitTwo"));
    // other services...
  }
};

Guice.createInjector(one, two);
```

The injector now contains two parallel sets of bindings which are isolated from each other. When you bind services in module `one` those services will get the `EntityManager` from `persistenceUnitOne` and transactions around its database.

Similarly services in module `two` will have their own transactions, `EntityManager`s etc., and these should typically not cross.