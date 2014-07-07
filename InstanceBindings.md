### Instance Bindings
You can bind a type to a specific instance of that type. This is usually only useful only for objects that don't have dependencies of their own, such as value objects:
```java
    bind(String.class)
        .annotatedWith(Names.named("JDBC URL"))
        .toInstance("jdbc:mysql://localhost/pizza");
    bind(Integer.class)
        .annotatedWith(Names.named("login timeout seconds"))
        .toInstance(10);
```
Avoid using `.toInstance` with objects that are complicated to create, since it can slow down application startup. You can use an `@Provides` method instead.