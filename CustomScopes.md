### Custom Scopes
It is generally recommended that users **do not** write their own custom scopes — the built-in scopes should be sufficient for most applications. If you're writing a web application, the `ServletModule` provides simple, well tested scope implementations for HTTP requests and HTTP sessions. 

Creating custom scopes is a multistep process:
  1. Define a scoping annotation
  2. Implementing the `Scope` interface
  3. Attaching the scope annotation to the implementation
  4. Triggering scope entry and exit

#### Defining a scoping annotation
The scoping annotation identifies your scope. You'll use it to annotate Guice-constructed types, `@Provides` methods, and in the `in()` clause of a bind statement. Copy-and-customize this code to define your scoping annotation:
```java
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.TYPE;
import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import java.lang.annotation.Target;

@Target({ TYPE, METHOD }) @Retention(RUNTIME) @ScopeAnnotation
public @interface BatchScoped {}
```
**Tip:** If your scope represents a request or session (such as for SOAP requests), consider using the `RequestScoped` and `SessionScoped` annotations from Guice's servlet extension. Otherwise, you may import the wrong annotation by mistake. That problem can be quite frustrating to debug.

#### Implementing Scope
The scope interface ensures there's at most one type instance for each scope instance. `SimpleScope` is a decent starting point for a per-thread implementation. Copy this class into your project and tweak it to suit your needs.
```java
import static com.google.common.base.Preconditions.checkState;
import com.google.common.collect.Maps;
import com.google.inject.Key;
import com.google.inject.OutOfScopeException;
import com.google.inject.Provider;
import com.google.inject.Scope;
import java.util.Map;

/**
 * Scopes a single execution of a block of code. Apply this scope with a
 * try/finally block: <pre><code>
 *
 *   scope.enter();
 *   try {
 *     // explicitly seed some seed objects...
 *     scope.seed(Key.get(SomeObject.class), someObject);
 *     // create and access scoped objects
 *   } finally {
 *     scope.exit();
 *   }
 * </code></pre>
 *
 * The scope can be initialized with one or more seed values by calling
 * <code>seed(key, value)</code> before the injector will be called upon to
 * provide for this key. A typical use is for a servlet filter to enter/exit the
 * scope, representing a Request Scope, and seed HttpServletRequest and
 * HttpServletResponse.  For each key inserted with seed(), you must include a
 * corresponding binding:
 *  <pre><code>
 *   bind(key)
 *       .toProvider(SimpleScope.&lt;KeyClass&gt;seededKeyProvider())
 *       .in(ScopeAnnotation.class);
 * </code></pre>
 *
 * @author Jesse Wilson
 * @author Fedor Karpelevitch
 */
public class SimpleScope implements Scope {

  private static final Provider<Object> SEEDED_KEY_PROVIDER =
      new Provider<Object>() {
        public Object get() {
          throw new IllegalStateException("If you got here then it means that" +
              " your code asked for scoped object which should have been" +
              " explicitly seeded in this scope by calling" +
              " SimpleScope.seed(), but was not.");
        }
      };
  private final ThreadLocal<Map<Key<?>, Object>> values
      = new ThreadLocal<Map<Key<?>, Object>>();

  public void enter() {
    checkState(values.get() == null, "A scoping block is already in progress");
    values.set(Maps.<Key<?>, Object>newHashMap());
  }

  public void exit() {
    checkState(values.get() != null, "No scoping block in progress");
    values.remove();
  }

  public <T> void seed(Key<T> key, T value) {
    Map<Key<?>, Object> scopedObjects = getScopedObjectMap(key);
    checkState(!scopedObjects.containsKey(key), "A value for the key %s was " +
        "already seeded in this scope. Old value: %s New value: %s", key,
        scopedObjects.get(key), value);
    scopedObjects.put(key, value);
  }

  public <T> void seed(Class<T> clazz, T value) {
    seed(Key.get(clazz), value);
  }

  public <T> Provider<T> scope(final Key<T> key, final Provider<T> unscoped) {
    return new Provider<T>() {
      public T get() {
        Map<Key<?>, Object> scopedObjects = getScopedObjectMap(key);

        @SuppressWarnings("unchecked")
        T current = (T) scopedObjects.get(key);
        if (current == null && !scopedObjects.containsKey(key)) {
          current = unscoped.get();

          // don't remember proxies; these exist only to serve circular dependencies
          if (current instanceof CircularDependencyProxy) {
            return current;
          }

          scopedObjects.put(key, current);
        }
        return current;
      }
    };
  }

  private <T> Map<Key<?>, Object> getScopedObjectMap(Key<T> key) {
    Map<Key<?>, Object> scopedObjects = values.get();
    if (scopedObjects == null) {
      throw new OutOfScopeException("Cannot access " + key
          + " outside of a scoping block");
    }
    return scopedObjects;
  }

  /**
   * Returns a provider that always throws exception complaining that the object
   * in question must be seeded before it can be injected.
   *
   * @return typed provider
   */
  @SuppressWarnings({"unchecked"})
  public static <T> Provider<T> seededKeyProvider() {
    return (Provider<T>) SEEDED_KEY_PROVIDER;
  }
}
```

#### Binding the annotation to the implementation
You must attach your scoping annotation to the corresponding scope implementation. Just like bindings, you can configure this in your module's `configure()` method. Usually you'll also bind the scope itself, so interceptors or filters can use it.
```java
public class BatchScopeModule {
  public void configure() {
    SimpleScope batchScope = new SimpleScope();

    // tell Guice about the scope
    bindScope(BatchScoped.class, batchScope);

    // make our scope instance injectable
    bind(SimpleScope.class)
        .annotatedWith(Names.named("batchScope"))
        .toInstance(batchScope);
  }
}
```

#### Triggering the Scope
`SimpleScope` requires that you manually enter and exit the scope. Usually this lives in some low-level infrastructure code, like a filter or interceptor. Be sure to call `exit()` in a `finally` clause, otherwise the scope will be left open when an exception is thrown.
```java
  @Inject @Named("batchScope") SimpleScope scope;

  /**
   * Runs {@code runnable} in batch scope.
   */
  public void scopeRunnable(Runnable runnable) {
    scope.enter();
    try {
      // explicitly seed some seed objects...
      scope.seed(Key.get(SomeObject.class), someObject);

      // create and access scoped objects
      runnable.run();

    } finally {
      scope.exit();
    }
  }
```