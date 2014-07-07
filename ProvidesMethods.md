### @Provides Methods
When you need code to create an object, use an `@Provides` method. The method must be defined within a module, and it must have an `@Provides` annotation. The method's return type is the bound type. Whenever the injector needs an instance of that type, it will invoke the method. 
```java
public class BillingModule extends AbstractModule {
  @Override
  protected void configure() {
    ...
  }

  @Provides
  TransactionLog provideTransactionLog() {
    DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
    transactionLog.setJdbcUrl("jdbc:mysql://localhost/pizza");
    transactionLog.setThreadPoolSize(30);
    return transactionLog;
  }
}
```
If the `@Provides` method has a binding annotation like `@PayPal` or `@Named("Checkout")`, Guice binds the annotated type. Dependencies can be passed in as parameters to the method. The injector will exercise the bindings for each of these before invoking the method.
```java
  @Provides @PayPal
  CreditCardProcessor providePayPalCreditCardProcessor(
      @Named("PayPal API key") String apiKey) {
    PayPalCreditCardProcessor processor = new PayPalCreditCardProcessor();
    processor.setApiKey(apiKey);
    return processor;
  }
```

### Throwing Exceptions
Guice does not allow exceptions to be thrown from Providers.  Exceptions thrown by `@Provides` methods will be wrapped in a `ProvisionException`.  It is bad practice to allow any kind of exception to be thrown -- runtime or checked -- from an `@Provides` method.  If you need to throw an exception for some reason, you may want to use the [[ThrowingProviders extension|ThrowingProviders]] `@CheckedProvides` methods.