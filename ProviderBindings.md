### Provider Bindings
When your `@Provides` methods start to grow complex, you may consider moving them to a class of their own. The provider class implements Guice's `Provider` interface, which is a simple, general interface for supplying values:
```java
public interface Provider<T> {
  T get();
}
```
Our provider implementation class has dependencies of its own, which it receives via its `@Inject`-annotated constructor. It implements the `Provider` interface to define what's returned with complete type safety:
```java
public class DatabaseTransactionLogProvider implements Provider<TransactionLog> {
  private final Connection connection;

  @Inject
  public DatabaseTransactionLogProvider(Connection connection) {
    this.connection = connection;
  }

  public TransactionLog get() {
    DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
    transactionLog.setConnection(connection);
    return transactionLog;
  }
}
```
Finally we bind to the provider using the `.toProvider` clause:
```java
public class BillingModule extends AbstractModule {
  @Override
  protected void configure() {
    bind(TransactionLog.class)
        .toProvider(DatabaseTransactionLogProvider.class);
  }
```
If your providers are complex, be sure to test them!

### Throwing Exceptions
Guice does not allow exceptions to be thrown from Providers.  The Provider interface does not allow for checked exception to be thrown.  RuntimeExceptions may be wrapped in a `ProvisionException` or `CreationException` and may prevent your `Injector` from being created.  If you need to throw an exception for some reason, you may want to use the [[ThrowingProviders extension|ThrowingProviders]].