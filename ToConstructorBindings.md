### Constructor Bindings
_New in Guice 3.0_

Occasionally it's necessary to bind a type to an arbitrary constructor. This comes up when the `@Inject` annotation cannot be applied to the target constructor: either because it is a third party class, or because _multiple_ constructors that participate in dependency injection. [[@Provides methods|ProvidesMethods]] provide the best solution to this problem! By calling your target constructor explicitly, you don't need reflection and its associated pitfalls. But there are limitations of that approach: manually constructed instances do not participate in [[AOP|AOP]].

To address this, Guice has `toConstructor()` bindings. They require you to reflectively select your target constructor and handle the exception if that constructor cannot be found:
```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
    try {
      bind(TransactionLog.class).toConstructor(
          DatabaseTransactionLog.class.getConstructor(DatabaseConnection.class));
    } catch (NoSuchMethodException e) {
      addError(e);
    }
  }
}
```
In this example, the `DatabaseTransactionLog` must have a constructor that takes a single `DatabaseConnection` parameter. That constructor does not need an `@Inject` annotation. Guice will invoke that constructor to satisfy the binding.

Each `toConstructor()` binding is scoped independently. If you create multiple singleton bindings that target the same constructor, each binding yields its own instance.