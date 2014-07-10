_an easier way to get Guice to build auto-wired factories._
### AssistedInject

Factories are a well established pattern for creating value objects, model/domain objects (entities), or objects that combine parameterization and dependencies.  Factories can be brittle and contain a lot of boilerplate.  Guice can eliminate a lot of that boilerplate by auto-generating Factory implementations from simple interfaces.  This process is (possibly misleadingly) known as assisted injection.

##### Factories by Hand
Sometimes a class gets some of its constructor parameters from the Guice Injector and others from the caller:
```java
public class RealPayment implements Payment {
  public RealPayment(
        CreditService creditService,  // from the Injector
        AuthService authService,  // from the Injector
        Date startDate, // from the instance's creator
        Money amount); // from the instance's creator
  }
  ...
}
```
The standard solution to this problem is to write a factory that helps Guice build the objects:
```java
public interface PaymentFactory {
  public Payment create(Date startDate, Money amount);
}
```
```java
public class RealPaymentFactory implements PaymentFactory {
  private final Provider<CreditService> creditServiceProvider;
  private final Provider<AuthService> authServiceProvider;

  @Inject
  public RealPaymentFactory(Provider<CreditService> creditServiceProvider,
      Provider<AuthService> authServiceProvider) {
    this.creditServiceProvider = creditServiceProvider;
    this.authServiceProvider = authServiceProvider;
  }

  public Payment create(Date startDate, Money amount) {
    return new RealPayment(creditServiceProvider.get(),
      authServiceProvider.get(), startDate, amount);
  }
}
```
...and a corresponding binding in the module:
```java
   bind(PaymentFactory.class).to(RealPaymentFactory.class);
```
It's annoying to write the boilerplate factory class each time this situation arises. It's also annoying to update the factories when the implementation class' dependencies change. 


##### Factories by AssistedInject
AssistedInject generates an implementation of the factory class automatically. To use it, annotate the implementation class' constructor and the fields that aren't known by the injector:
```java
public class RealPayment implements Payment {
  @Inject
  public RealPayment(
        CreditService creditService,
        AuthService authService,
        @Assisted Date startDate,
        @Assisted Money amount);
  }
  ...
}
```
Then bind a `Provider<Factory>` in the Guice module:
##### AssistedInject in Guice 2.0
```java
bind(PaymentFactory.class).toProvider(
    FactoryProvider.newFactory(PaymentFactory.class, RealPayment.class));
```

##### AssistedInject in Guice 3.0
Guice 3.0 improves AssistedInject by offering the ability to allow different factory methods to return different types or different constructors from one type.  See the [FactoryModuleBuilder javadoc](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/assistedinject/FactoryModuleBuilder.html) for complete details.
```java
install(new FactoryModuleBuilder()
     .implement(Payment.class, RealPayment.class)
     .build(PaymentFactory.class));
```

##### How & Why

AssistedInject maps the `create()` method's parameters to the corresponding `@Assisted` parameters in the implementation class' constructor. For the other constructor arguments, it asks the regular Injector to provide values. 

With AssistedInject, it's easier to create classes that need extra arguments at construction time:
  1. Annotate the constructor and assisted parameters on the implementation class (such as `RealPayment`) 
  2. Create a factory interface with a `create()` method that takes only the assisted parameters. Make sure they're in the same order as in the constructor
  3. Bind that factory to a provider created by AssistedInject.

##### Inspecting AssistedInject Bindings _(new in Guice 3.0)_

Visiting an assisted inject binding is useful for tests or debugging.  AssistedInject uses the [[extensions SPI|ExtensionSPI]] to let you learn more about the factory binding.  You can visit bindings with an [AssistedInjectTargetVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/assistedinject/AssistedInjectTargetVisitor.html) to see details about the binding.

```java  
  Binding<PaymentFactory> binding = injector.getBinding(PaymentFactory.class);
  binding.acceptTargetVisitor(new Visitor());

  class Visitor
      extends DefaultBindingTargetVisitor<Object, Void>
      implements AssistedInjectTargetVisitor<Object, Void> {

    @Override void visit(AssistedInjectBinding<?> binding) {
      // Loop over each method in the factory...
      for(AssistedMethod method : binding.getAssistedMethods()) {
        System.out.println("Non-assisted Dependencies: " + method.getDependencies()
                       + ", Factory Method: " + method.getFactoryMethod()
                       + ", Implementation Constructor: " + method.getImplementationConstructor()
                       + ", Implementation Type: " + method.getImplementationType());
      }
    }
  }
```