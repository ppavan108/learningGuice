_Creating bindings that don't have targets_
### Untargetted Bindings

You may create bindings without specifying a target. This is most useful for concrete classes and types annotated by either `@ImplementedBy` or `@ProvidedBy`. An untargetted binding informs the injector about a type, so it may prepare dependencies eagerly. Untargetted bindings have no _to_ clause, like so:
```java
    bind(MyConcreteClass.class);
    bind(AnotherConcreteClass.class).in(Singleton.class);
```

When specifying binding annotations, you must still add the target binding, even it is the same concrete class.  For example:
```java
    bind(MyConcreteClass.class)
        .annotatedWith(Names.named("foo"))
        .to(MyConcreteClass.class);
    bind(AnotherConcreteClass.class)
        .annotatedWith(Names.named("foo"))
        .to(AnotherConcreteClass.class)
        .in(Singleton.class);
```