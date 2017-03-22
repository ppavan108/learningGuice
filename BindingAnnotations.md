### Binding Annotations
Occasionally you'll want multiple bindings for a same type. For example, you might want both a PayPal credit card processor and a Google Checkout processor. To enable this, bindings support an optional *binding annotation*. The annotation and type together uniquely identify a binding. This pair is called a *key*.

Defining a binding annotation requires two lines of code plus several imports. Put this in its own `.java` file or inside the type that it annotates. 
```java
package example.pizza;

import com.google.inject.BindingAnnotation;
import java.lang.annotation.Target;
import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;

@BindingAnnotation @Target({ FIELD, PARAMETER, METHOD }) @Retention(RUNTIME)
public @interface PayPal {}
```
You don't need to understand all of these meta-annotations, but if you're curious:
  * `@BindingAnnotation` tells Guice that this is a binding annotation.  Guice will produce an error if ever multiple binding annotations apply to the same member.
  * `@Target({FIELD, PARAMETER, METHOD})` is a courtesy to your users. It prevents `@PayPal` from being accidentally being applied where it serves no purpose.
  * `@Retention(RUNTIME)` makes the annotation available at runtime.

To depend on the annotated binding, apply the annotation to the injected parameter:
```java
public class RealBillingService implements BillingService {

  @Inject
  public RealBillingService(@PayPal CreditCardProcessor processor,
      TransactionLog transactionLog) {
    ...
  }
```
Lastly we create a binding that uses the annotation. This uses the optional `annotatedWith` clause in the `bind()` statement:
```java
    bind(CreditCardProcessor.class)
        .annotatedWith(PayPal.class)
        .to(PayPalCreditCardProcessor.class);
```


### @Named
Guice comes with a built-in binding annotation `@Named` that takes a string:
```java
public class RealBillingService implements BillingService {

  @Inject
  public RealBillingService(@Named("Checkout") CreditCardProcessor processor,
      TransactionLog transactionLog) {
    ...
  }
```
To bind a specific name, use `Names.named()` to create an instance to pass to `annotatedWith`:
```java
    bind(CreditCardProcessor.class)
        .annotatedWith(Names.named("Checkout"))
        .to(CheckoutCreditCardProcessor.class);
```
Since the compiler can't check the string, we recommend using `@Named` sparingly. Defining your own purpose-built annotations provides better type-safety.

### Binding Annotations with Attributes

Guice supports binding annotations that have attribute values (like `@Named`). In the rare case that you need such an annotation we encourage you to use [`@AutoAnnotation`](https://github.com/google/auto/blob/master/value/userguide/howto.md#annotation) from the Auto/Value project, as properly implementing an annotation is error-prone. If you do decide to manually create a custom implementation be sure to properly implement the `equals()` and `hashCode()` specifications detailed in the [`Annotation` Javadoc](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Annotation.html). Pass an instance of this class to the `annotatedWith()` binding clause.