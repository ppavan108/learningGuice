_Automatically bind field's values to their types._

### Bound Fields (New in Guice 4.0)

Test code which uses Guice frequently follows the pattern whereby you create several fields in your test class and bind those fields to their types using a custom Module. `BoundFieldModule` can help simplify this pattern by automatically binding fields annotated with `@Bind`.

#### Binding Fields Manually
Frequently in order to test classes which are injected, you need to inject mocks or dummy values for the tested class's dependencies. In order to do this, a common pattern is the following:
```java
public class TestFoo {
  private Bar barMock;

  // Foo depends on Bar.
  @Inject private Foo foo;

  @Before public void setUp() {
    barMock = ...;
    Guice.createInjector(getTestModule()).injectMembers(this);
  }

  private Module getTestModule() {
    return new AbstractModule() {
      @Override protected void configure() {
        bind(Bar.class).toInstance(barMock);
      }
    };
  }

  @Test public void testBehavior() {
    ...
  }
}
```
This class creates a field for the tested class's dependencies, places in that field a mock and binds the field into Guice with a custom Module.

#### Binding Fields Using `BoundFieldModule`
`BoundFieldModule` scans a given object for fields annotated with `@Bind` and binds those fields automatically. Using this, we can simplify our `TestFoo` class to:
```java
public class TestFoo {
  // bind(Bar.class).toInstance(barMock);
  @Bind private Bar barMock;

  // Foo depends on Bar.
  @Inject private Foo foo;

  @Before public void setUp() {
    barMock = ...;
    Guice.createInjector(BoundFieldModule.of(this)).injectMembers(this);
  }

  @Test public void testBehavior() {
    ...
  }
}
```

This binding occurrs under the following rules:

 * For each `@Bind` annotated field of an object and its superclasses, this module will bind that field's type to that field's value at injector creation time. This includes both instance and static fields.
 * If `@Bind(to = ...)` is specified, the field's value will be bound to the class specified by `to` instead of the field's actual type.
 * If a [BindingAnnotation](http://google-guice.googlecode.com/git/javadoc/com/google/inject/BindingAnnotation.html) or [Qualifier](http://atinject.googlecode.com/svn/trunk/javadoc/javax/inject/Qualifier.html) is present on the field, that field will be bound using that annotation via [annotatedWith](http://google-guice.googlecode.com/git/javadoc/com/google/inject/binder/AnnotatedBindingBuilder.html). For example, `bind(Foo.class).annotatedWith(BarAnnotation.class).toInstance(theValue)`. It is an error to supply more than one [BindingAnnotation](http://google-guice.googlecode.com/git/javadoc/com/google/inject/BindingAnnotation.html) or [Qualifier](http://atinject.googlecode.com/svn/trunk/javadoc/javax/inject/Qualifier.html).
 * If the field is of type [Provider](http://google-guice.googlecode.com/git/javadoc/com/google/inject/Provider.html), the field's value will be bound as a provider using [toProvider](http://google-guice.googlecode.com/git/javadoc/com/google/inject/binder/LinkedBindingBuilder.html) to the provider's parameterized type. For example, `Provider<Integer>` binds to `Integer`. Attempting to bind a non-parameterized `Provider` without a `@Bind(to = ...)` clause is an error.

Additional information can be found in the [source](https://code.google.com/p/google-guice/source/browse/extensions/testlib/src/com/google/inject/testing/fieldbinder/BoundFieldModule.java). Javadoc link coming soon!