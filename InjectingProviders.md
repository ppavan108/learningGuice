### Injecting Providers
With normal dependency injection, each type gets exactly *one instance* of each of its dependent types. The `RealBillingService` gets one `CreditCardProcessor` and one `TransactionLog`. Sometimes you want more than one instance of your dependent types.  When this flexibility is necessary, Guice binds a provider. Providers produce a value when the `get()` method is invoked:
```java
public interface Provider<T> {
  T get();
}
```
The provider's type is _parameterized_ to differentiate a `Provider<TransactionLog>` from a `Provider<CreditCardProcessor>`. Wherever you inject a value you can inject a provider for that value.
```java
public class RealBillingService implements BillingService {
  private final Provider<CreditCardProcessor> processorProvider;
  private final Provider<TransactionLog> transactionLogProvider;

  @Inject
  public RealBillingService(Provider<CreditCardProcessor> processorProvider,
      Provider<TransactionLog> transactionLogProvider) {
    this.processorProvider = processorProvider;
    this.transactionLogProvider = transactionLogProvider;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = processorProvider.get();
    TransactionLog transactionLog = transactionLogProvider.get();

    /* use the processor and transaction log here */
  }
}
```
For every binding, annotated or not, the injector has a built-in binding for its provider.


### Providers for multiple instances
Use providers when you need multiple instances of the same type. Suppose your application saves a summary entry and a details when a pizza charge fails. With providers, you can get a new entry whenever you need one:
```java
public class LogFileTransactionLog implements TransactionLog {

  private final Provider<LogFileEntry> logFileProvider;

  @Inject
  public LogFileTransactionLog(Provider<LogFileEntry> logFileProvider) {
    this.logFileProvider = logFileProvider;
  }

  public void logChargeResult(ChargeResult result) {
    LogFileEntry summaryEntry = logFileProvider.get();
    summaryEntry.setText("Charge " + (result.wasSuccessful() ? "success" : "failure"));
    summaryEntry.save();

    if (!result.wasSuccessful()) {
      LogFileEntry detailEntry = logFileProvider.get();
      detailEntry.setText("Failure result: " + result);
      detailEntry.save();
    }
  }
```


### Providers for lazy loading
If you've got a dependency on a type that is particularly *expensive to produce*, you can use providers to defer that work. This is especially useful when you don't always need the dependency:
```java
public class DatabaseTransactionLog implements TransactionLog {
  
  private final Provider<Connection> connectionProvider;

  @Inject
  public DatabaseTransactionLog(Provider<Connection> connectionProvider) {
    this.connectionProvider = connectionProvider;
  }

  public void logChargeResult(ChargeResult result) {
    /* only write failed charges to the database */
    if (!result.wasSuccessful()) {
      Connection connection = connectionProvider.get();
    }
  }
```

### Providers for Mixing Scopes
It is an error to depend on an object in a _narrower_ scope. Suppose you have a singleton transaction log that needs on the request-scoped current user. Should you inject the user directly, things break because the user changes from request to request. Since providers can produce values on-demand, they enable you to mix scopes safely:
```java
@Singleton
public class ConsoleTransactionLog implements TransactionLog {
  
  private final AtomicInteger failureCount = new AtomicInteger();
  private final Provider<User> userProvider;

  @Inject
  public ConsoleTransactionLog(Provider<User> userProvider) {
    this.userProvider = userProvider;
  }

  public void logConnectException(UnreachableException e) {
    failureCount.incrementAndGet();
    User user = userProvider.get();
    System.out.println("Connection failed for " + user + ": " + e.getMessage());
    System.out.println("Failure count: " + failureCount.incrementAndGet());
  }
```