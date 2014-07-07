_How Guice initializes your objects_
### Injections
The dependency injection pattern separates behaviour from dependency resolution. Rather than looking up dependencies directly or from factories, the pattern recommends that dependencies are passed in. The process of setting dependencies into an object is called *injection*.

#### Constructor Injection
Constructor injection combines instantiation with injection. To use it, annotate the constructor with the `@Inject` annotation. This constructor should accept class dependencies as parameters. Most constructors will then assign the parameters to final fields.
```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processorProvider;
  private final TransactionLog transactionLogProvider;

  @Inject
  public RealBillingService(CreditCardProcessor processorProvider,
      TransactionLog transactionLogProvider) {
    this.processorProvider = processorProvider;
    this.transactionLogProvider = transactionLogProvider;
  }
```
If your class has no `@Inject`-annotated constructor, Guice will use a public, no-arguments constructor if it exists. Prefer the annotation, which documents that the type participates in dependency injection.

Constructor injection works nicely with unit testing. If your class accepts all of its dependencies in a single constructor, you won't accidentally forget to set a dependency. When a new dependency is introduced, all of the calling code conveniently breaks! Fix the compile errors and you can be confident that everything is properly wired up.


#### Method Injection
Guice can inject methods that have the `@Inject` annotation. Dependencies take the form of parameters, which the injector resolves before invoking the method. Injected methods may have any number of parameters, and the method name does not impact injection.
```java
public class PayPalCreditCardProcessor implements CreditCardProcessor {
  
  private static final String DEFAULT_API_KEY = "development-use-only";
  
  private String apiKey = DEFAULT_API_KEY;

  @Inject
  public void setApiKey(@Named("PayPal API key") String apiKey) {
    this.apiKey = apiKey;
  }
```


#### Field Injection
Guice injects fields with the `@Inject` annotation. This is the most concise injection, but the least testable.
```java
public class DatabaseTransactionLogProvider implements Provider<TransactionLog> {
  @Inject Connection connection;

  public TransactionLog get() {
    return new DatabaseTransactionLog(connection);
  }
}
```
Avoid using field injection with `final` fields, which has [weak semantics](http://java.sun.com/javase/6/docs/api/java/lang/reflect/Field.html#set(java.lang.Object,%20java.lang.Object)).


#### Optional Injections
Occasionally it's convenient to use a dependency when it exists and to fall back to a default when it doesn't. Method and field injections may be optional, which causes Guice to silently ignore them when the dependencies aren't available. To use optional injection, apply the `@Inject(optional=true)` annotation:
```java
public class PayPalCreditCardProcessor implements CreditCardProcessor {
  private static final String SANDBOX_API_KEY = "development-use-only";

  private String apiKey = SANDBOX_API_KEY;

  @Inject(optional=true)
  public void setApiKey(@Named("PayPal API key") String apiKey) {
    this.apiKey = apiKey;
  }
```
Mixing optional injection and just-in-time bindings may yield surprising results. For example, the following field is always injected even when `Date` is not explicitly bound. This is because `Date` has a public no-arguments constructor that is eligible for just-in-time bindings.
```java
  @Inject(optional=true) Date launchDate;
```


### On-demand Injection
Method and field injection can be used to initialize an existing instance. You can use the `Injector.injectMembers` API:
```java
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(...);
    
    CreditCardProcessor creditCardProcessor = new PayPalCreditCardProcessor();
    injector.injectMembers(creditCardProcessor);
```


### Static Injections
When [migrating an application](http://publicobject.com/2007/07/guice-patterns-1-horrible-static-code.html) from static factories to Guice, it is possible to change incrementally. Static injection is a helpful crutch here. It makes it possible for objects to _partially_ participate in dependency injection, by gaining access to injected types without being injected themselves. Use `requestStaticInjection()` in a module to specify classes to be injected at injector-creation time:
```java
  @Override public void configure() {
    requestStaticInjection(ProcessorFactory.class);
    ...
  }
```
Guice will inject class's static members that have the `@Injected` annotation:
```java
class ProcessorFactory {
  @Inject static Provider<Processor> processorProvider;

  /**
   * @deprecated prefer to inject your processor instead.
   */
  @Deprecated
  public static Processor getInstance() {
    return processorProvider.get();
  }
}
```
Static members will not be injected at instance-injection time. This API is not recommended for general use because it suffers many of the same problems as static factories: it's clumsy to test, it makes dependencies opaque, and it relies on global state.

### Automatic Injection
Guice automatically injects all of the following:
  * instances passed to `toInstance()` in a bind statement
  * provider instances passed to `toProvider()` in a bind statement
The objects will be injected while the injector itself is being created. If they're needed to satisfy other startup injections, Guice will inject them before they're used.