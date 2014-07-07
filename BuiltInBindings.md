_More bindings that you can use_
### Built-in Bindings
Alongside explicit and [[just-in-time bindings|JustInTimeBindings]] additional bindings are automatically included in the injector. Only the injector can create these bindings and attempting to bind them yourself is an error. 

### Loggers
Guice has a built-in binding for `java.util.logging.Logger`, intended to save some boilerplate. The binding automatically sets the logger's name to the name of the class into which the Logger is being injected..
```java
@Singleton
public class ConsoleTransactionLog implements TransactionLog {

  private final Logger logger;

  @Inject
  public ConsoleTransactionLog(Logger logger) {
    this.logger = logger;
  }

  public void logConnectException(UnreachableException e) {
    /* the message is logged to the "ConsoleTransacitonLog" logger */
    logger.warning("Connect exception failed, " + e.getMessage());
  }
```


### The Injector
In framework code, sometimes you don't know the type you need until runtime. In this rare case you should inject the injector. Code that injects the injector does not self-document its dependencies, so this approach should be done sparingly.

### Providers
For every type Guice knows about, it can also inject a Provider of that type. [[Injecting Providers|InjectingProviders]] describes this in detail.

### TypeLiterals
Guice has complete type information for everything it injects. If you're injecting parameterized types, you can inject a `TypeLiteral<T>` to reflectively tell you the element type.

### The Stage
Guice supports a stage enum to differentiate between development and production runs.

### MembersInjectors
When binding to providers or writing extensions, you may want Guice to inject dependencies into an object that you construct yourself.  To do this, add a dependency on a `MembersInjector<T>` (where T is your object's type), and then call `membersInjector.injectMembers(myNewObject)`.