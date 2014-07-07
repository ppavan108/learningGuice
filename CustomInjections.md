_Performing custom injections with type and injection listeners_
### Custom Injections
In addition to the standard `@Inject`-driven injections, Guice includes hooks for custom injections. This enables Guice to host other frameworks that have their own injection semantics or annotations. Most developers won't use custom injections directly; but they may see their use in extensions and third-party libraries. Each custom injection requires a type listener, an injection listener, and registration of each.

[TypeListeners](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/spi/TypeListener.html) get notified of the types that Guice injects. Since type listeners are only notified once-per-type, they are the best place to run potentially slow operations, such as enumerating a type's members. With their inspection complete, type listeners may register instance listeners for values as they're injected.

[MembersInjectors](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/MembersInjector.html) and [InjectionListeners](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/spi/InjectionListener.html) can be used to receive a callback after Guice has injected an instance. The instance is first injected by Guice, then by the custom members injectors, and finally the injection listeners are notified. Since they're notified once-per-instance, these should execute as quickly as possible.

#### Example: Injecting a Log4J Logger
Guice includes built-in support for injecting `java.util.Logger` instances that are named using the type of the injected instance. With the type listener API, you can inject a `org.apache.log4j.Logger` with the same high-fidelity naming. We'll be injecting fields of this format:
```java
import org.apache.log4j.Logger;
import com.publicobject.log4j.InjectLogger;

public class PaymentService {
  @InjectLogger Logger logger;

  ...
}
```
We start by registering our custom type listener `Log4JTypeListener` in a module. We use a matcher to select which types we listen for:
```java
  @Override protected void configure() {
    bindListener(Matchers.any(), new Log4JTypeListener());
  }
```
You can implement the [TypeListener](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/spi/TypeListener.html) to scan through  a type's fields, looking for Log4J loggers. For each logger field that's encountered, we register a `Log4JMembersInjector` on the passed-in [TypeEncounter](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/spi/TypeEncounter.html):
```java
  class Log4JTypeListener implements TypeListener {
    public <T> void hear(TypeLiteral<T> typeLiteral, TypeEncounter<T> typeEncounter) {
      for (Field field : typeLiteral.getRawType().getDeclaredFields()) {
        if (field.getType() == Logger.class
            && field.isAnnotationPresent(InjectLogger.class)) {
          typeEncounter.register(new Log4JMembersInjector<T>(field));
        }
      }
    }
  }
```
Finally, we implement the `Log4JMembersInjector` to set the logger. In this example, we always set the field to the same instance. In your application you may need to compute a value or request one from a provider.
```java
  class Log4JMembersInjector<T> implements MembersInjector<T> {
    private final Field field;
    private final Logger logger;

    Log4JMembersInjector(Field field) {
      this.field = field;
      this.logger = Logger.getLogger(field.getDeclaringClass());
      field.setAccessible(true);
    }

    public void injectMembers(T t) {
      try {
        field.set(t, logger);
      } catch (IllegalAccessException e) {
        throw new RuntimeException(e);
      }
    }
  }
```