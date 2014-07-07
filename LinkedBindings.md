### Linked Bindings
Linked bindings map a type to its implementation. This example maps the interface `TransactionLog` to the implementation `DatabaseTransactionLog`:
```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);
  }
}
```
Now, when you call `injector.getInstance(TransactionLog.class)`, or when the injector encounters a dependency on `TransactionLog`, it will use a `DatabaseTransactionLog`. Link from a type to any of its subtypes, such as an implementing class or an extending class. You can even link the concrete `DatabaseTransactionLog` class to a subclass:
```java
    bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
```
Linked bindings can also be chained:
```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);
    bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
  }
}
```
In this case, when a `TransactionLog` is requested, the injector will return a `MySqlDatabaseTransactionLog`.