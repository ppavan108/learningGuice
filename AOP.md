_Intercepting methods with Guice_
### Aspect Oriented Programming
To compliment dependency injection, Guice supports *method interception*. This feature enables you to write code that is executed each time a _matching_ method is invoked. It's suited for cross cutting concerns ("aspects"), such as transactions, security and logging. Because interceptors divide a problem into aspects rather than objects, their use is called Aspect Oriented Programming (AOP).

Most developers won't write method interceptors directly; but they may see their use in integration libraries like [Warp Persist](http://www.wideplay.com/guicewebextensions2). Those that do will need to select the matching methods, create an interceptor, and configure it all in a module.

[Matcher](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/matcher/Matcher.html) is a simple interface that either accepts or rejects a value. For Guice AOP, you need two matchers: one that defines which classes participate, and another for the methods of those classes. To make this easy, there's [factory class](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/matcher/Matchers.html) to satisfy the common scenarios. 

[MethodInterceptors](http://aopalliance.sourceforge.net/doc/org/aopalliance/intercept/MethodInterceptor.html) are executed whenever a matching method is invoked. They have the opportunity to inspect the call: the method, its arguments, and the receiving instance. They can perform their cross-cutting logic and then delegate to the underlying method. Finally, they may inspect the return value or exception and return. Since interceptors may be applied to many methods and will receive many calls, their implementation should be efficient and unintrusive.

### Example: Forbidding method calls on weekends
To illustrate how method interceptors work with Guice, we'll forbid calls to our pizza billing system on weekends. The delivery guys only work Monday thru Friday so we'll prevent pizza from being ordered when it can't be delivered! This example is structurally similar to use of AOP for authorization.

To mark select methods as weekdays-only, we define an annotation:
```java
@Retention(RetentionPolicy.RUNTIME) @Target(ElementType.METHOD)
@interface NotOnWeekends {}
```
...and apply it to the methods that need to be intercepted:
```java
public class RealBillingService implements BillingService {

  @NotOnWeekends
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    ...
  }
}
```
Next, we define the interceptor by implementing the `org.aopalliance.intercept.MethodInterceptor` interface. When we need to call through to the underlying method, we do so by calling `invocation.proceed()`:
```java
public class WeekendBlocker implements MethodInterceptor {
  public Object invoke(MethodInvocation invocation) throws Throwable {
    Calendar today = new GregorianCalendar();
    if (today.getDisplayName(DAY_OF_WEEK, LONG, ENGLISH).startsWith("S")) {
      throw new IllegalStateException(
          invocation.getMethod().getName() + " not allowed on weekends!");
    }
    return invocation.proceed();
  }
}
```
Finally, we configure everything. This is where we create matchers for the classes and methods to be intercepted. In this case we match any class, but only the methods with our `@NotOnWeekends` annotation:
```java
public class NotOnWeekendsModule extends AbstractModule {
  protected void configure() {
    bindInterceptor(Matchers.any(), Matchers.annotatedWith(NotOnWeekends.class), 
        new WeekendBlocker());
  }
}
```
Putting it all together, (and waiting until Saturday), we see the method is intercepted and our order is rejected:
```java
Exception in thread "main" java.lang.IllegalStateException: chargeOrder not allowed on weekends!
	at com.publicobject.pizza.WeekendBlocker.invoke(WeekendBlocker.java:65)
	at com.google.inject.internal.InterceptorStackCallback.intercept(...)
	at com.publicobject.pizza.RealBillingService$$EnhancerByGuice$$49ed77ce.chargeOrder(<generated>)
	at com.publicobject.pizza.WeekendExample.main(WeekendExample.java:47)
```

### Limitations
Behind the scenes, method interception is implemented by generating bytecode at runtime. Guice dynamically creates a subclass that applies interceptors by overriding methods. If you are on a platform that doesn't support bytecode generation (such as Android), you should use [[Guice without AOP support|OptionalAOP]].

This approach imposes limits on what classes and methods can be intercepted:
  * Classes must be public or package-private.
  * Classes must be non-final
  * Methods must be public, package-private or protected
  * Methods must be non-final
  * Instances must be created by Guice by an `@Inject`-annotated or no-argument constructor
It is not possible to use method interception on instances that aren't constructed by Guice.

### Injecting Interceptors
If you need to inject dependencies into an interceptor, use the `requestInjection` API.
```java
public class NotOnWeekendsModule extends AbstractModule {
  protected void configure() {
    WeekendBlocker weekendBlocker = new WeekendBlocker();
    requestInjection(weekendBlocker);
    bindInterceptor(Matchers.any(), Matchers.annotatedWith(NotOnWeekends.class), 
       weekendBlocker);
  }
}
```
Another option is to use Binder.getProvider and pass the dependency in the constructor of the interceptor.
```java
public class NotOnWeekendsModule extends AbstractModule {
  protected void configure() {
    bindInterceptor(any(),
                    annotatedWith(NotOnWeekends.class),
                    new WeekendBlocker(getProvider(Calendar.class)));
  }
}
```
Use caution when injecting interceptors. If your interceptor calls a method that it itself is intercepting, you may receive a `StackOverflowException` due to unending recursion.

### AOP Alliance
The method interceptor API implemented by Guice is a part of a public specification called [AOP Alliance](http://aopalliance.sourceforge.net/). This makes it possible to use the same interceptors across a variety of frameworks.