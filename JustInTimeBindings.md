_Bindings that are created automatically by Guice_
### Just-in-time Bindings
When the injector needs an instance of a type, it needs a binding. The bindings in a modules are called *explicit bindings*, and the injector uses them whenever they're available. If a type is needed but there isn't an explicit binding, the injector will attempt to create a *Just-In-Time binding*. These are also known as JIT bindings and implicit bindings.

### Eligible Constructors
Guice can create bindings for concrete types by using the type's *injectable constructor*. This is either a non-private, no-arguments constructor, or a constructor with the `@Inject` annotation:
```java
public class PayPalCreditCardProcessor implements CreditCardProcessor {
  private final String apiKey;

  @Inject
  public PayPalCreditCardProcessor(@Named("PayPal API key") String apiKey) {
    this.apiKey = apiKey;
  }
```
Guice will not construct nested classes unless they have the `static` modifier. Inner classes have an implicit reference to their enclosing class that cannot be injected.

### @ImplementedBy
Annotate types tell the injector what their default implementation type is. The `@ImplementedBy` annotation acts like a *linked binding*, specifying the subtype to use when building a type.
```java
@ImplementedBy(PayPalCreditCardProcessor.class)
public interface CreditCardProcessor {
  ChargeResult charge(String amount, CreditCard creditCard)
      throws UnreachableException;
}
```
The above annotation is equivalent to the following `bind()` statement:
```java
    bind(CreditCardProcessor.class).to(PayPalCreditCardProcessor.class);
```
If a type is in both a `bind()` statement (as the first argument) and has the `@ImplementedBy` annotation, the `bind()` statement is used. The annotation suggests a _default implementation_ that can be overridden with a binding. Use `@ImplementedBy` carefully; it adds a compile-time dependency from the interface to its implementation.

### @ProvidedBy
`@ProvidedBy` tells the injector about a `Provider` class that produces instances:
```java
@ProvidedBy(DatabaseTransactionLogProvider.class)
public interface TransactionLog {
  void logConnectException(UnreachableException e);
  void logChargeResult(ChargeResult result);
}
```
The annotation is equivalent to a `toProvider()` binding:
```java
    bind(TransactionLog.class)
        .toProvider(DatabaseTransactionLogProvider.class);
```
Like `@ImplementedBy`, if the type is annotated and used in a `bind()` statement, the `bind()` statement will be used.