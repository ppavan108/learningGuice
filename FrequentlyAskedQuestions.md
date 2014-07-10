### Frequently Asked Questions

##### How do I inject configuration parameters?
You need a binding annotation to identify your parameter. Create an annotation class that defines the parameter:
```java
/**
 * Annotates the URL of the foo server.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@BindingAnnotation
public @interface FooServerAddress {}
```
Bind the annotation to its value in your module:
```java
public class FooModule extends AbstractModule {
  private final String fooServerAddress;

  /**
   * @param fooServerAddress the URL of the foo server.
   */
  public FooModule(String fooServerAddress) {
    this.fooServerAddress = fooServerAddress;
  }

  @Override public void configure() {
    bindConstant().annotatedWith(FooServerAddress.class).to(fooServerAddress);
    ...
  }
}
```
Finally, inject it into your class:
```java
public class FooClient {

  @Inject
  FooClient(@FooServerAddress String fooServerAddress) {
    ...
  }
```
You may save some keystrokes by using Guice's built-in `@Named` binding annotation rather than creating your own.

##### How do I load configuration properties?
Use [Names.bindProperties()](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/name/Names.html#bindProperties(com.google.inject.Binder,%20java.util.Properties)) to create bindings for each of the properties in a configuration file.

##### How do I pass a parameter when creating an object via Guice?
You can't directly pass a parameter into an injected value. But you can use Guice to create a `Factory`, and use that factory to create your object.
```java
public class Thing {
  // note: no @Inject annotation here
  private Thing(A a, B b) {
    ...
  }

  public static class Factory {
    @Inject
    public Factory(A a) { ... }
    public Thing make(B b) { ... }
  }
}
```

```java
public class Example {
  @Inject
  public Example(Thing.Factory factory) { ... }
}
```
See [[AssistedInject]], which can be used to remove the factory boilerplate.

<a name="RobotLegs"/>
##### How do I build two similar but slightly different trees of objects?
This is commonly called the "robot legs" problem: How to create a robot with a two `Leg` objects, the left one injected with a `LeftFoot`, and the right one with a `RightFoot`. But only one `Leg` class that's reused in both contexts.

There's a [PrivateModules solution](http://docs.google.com/Doc?id=dhfm3hw2_51d2tmv6pc). It uses two separate private modules, a `@Left` one and an `@Right` one. Each has a binding for the unannotated `Foot.class` and `Leg.class`, and exposes a binding for the annotated `Leg.class`:
```java
class LegModule extends PrivateModule {
  private final Class<? extends Annotation> annotation;

  LegModule(Class<? extends Annotation> annotation) {
    this.annotation = annotation;
  }

  @Override protected void configure() {
    bind(Leg.class).annotatedWith(annotation).to(Leg.class);
    expose(Leg.class).annotatedWith(annotation);

    bindFoot();
  }

  abstract void bindFoot();
}
```
```java
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(
        new LegModule(Left.class) {
          @Override void bindFoot() {
            bind(Foot.class).toInstance(new Foot("leftie"));
          }
        },
        new LegModule(Right.class) {
          @Override void bindFoot() {
            bind(Foot.class).toInstance(new Foot("righty"));
          }
        });
  }
```
See also [Alen Vrecko's more complete example](http://pastie.org/368348).

##### How can I inject an inner class?
Guice doesn't support this.  However, you can inject a _nested_ class (sometimes called a "static inner class"):
```java
class Outer {
  static class Nested {
    ...
  }
}
```

##### How to inject class with generic type?
You may need to inject a class with a parameterized type, like `List<String>`:
```java
class Example {
  @Inject
  void setList(List<String> list) {
    ...
  }
}
```
You can use a [TypeLiteral](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/TypeLiteral.html) to create the binding. `TypeLiteral` is a special class that allows you to specify a full parameterized type.
```java
  @Override public void configure() {
    bind(new TypeLiteral<List<String>>() {}).toInstance(new ArrayList<String>());
  }
```

Alternately, you can use an @Provides method.
```java
 @Provides List<String> providesListOfString() {
   return new ArrayList<String>();
 }
```

##### How can I inject optional parameters into a constructor?
Neither constructors nor `@Provides` methods support optional injection. To work-around this, you can create an inner class that holds the optional value:
```java
class Car {
  private final Engine engine;
  private final AirConditioner airConditioner;

  @Inject
  public Car(Engine engine, AirConditionerHolder airConditionerHolder) {
    this.engine = engine;
    this.airConditioner = airConditionerHolder.value;
  }

  static class AirConditionerHolder {
    @Inject(optional=true) AirConditioner value = new NoOpAirconditioner();
  }
}
```
This also allows for a default value for the optional parameter.

##### How do I inject a method interceptor?
In order to inject dependencies in an AOP `MethodInterceptor`, use `requestInjection()` alongside the standard `bindInterceptor()` call.
```java
public class NotOnWeekendsModule extends AbstractModule {
  protected void configure() {
    MethodInterceptor interceptor = new WeekendBlocker();
    requestInjection(interceptor);
    bindInterceptor(any(), annotatedWith(NotOnWeekends.class), interceptor);
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

##### How can I get other questions answered?
Please post to the [google-guice](http://groups.google.com/group/google-guice) discussion group.