_throwing providers extension tutorial_

### Throwing Providers

Guice isn't very good at handling exceptions that occur during provision.
  * Implementers of Provider can only throw RuntimeExceptions.
  * Callers of Provider can't catch the exception they threw, because it may be wrapped in a ProvisionException.
  * Injecting an instance directly rather than a Provider can cause creation of the injected object to fail.
  * Exceptions cannot be advertised in the API.

The ThrowingProviders extension offers an alternative to Providers that allow a checked exception to be thrown.
  * API consistent with Provider.
  * Scopable
  * Standard binding DSL

#### ThrowingProvider _(new in Guice 2.0)_
Guice 2.0 offers the ThrowingProvider interface.  It is similar to Provider, but with a generic Exception type:
```java
public interface ThrowingProvider<T,E extends Exception> {
  T get() throws E;
}
```
For each application exception, create an interface that extends the ThrowingProvider. For our news widget application, we created the FeedProvider interface that throws a FeedUnavailableException:
```java
public interface FeedProvider<T> extends ThrowingProvider<T, FeedUnavailableException> { }
```

#### CheckedProvider _(new in Guice 3.0)_
Guice 3.0 offers the CheckedProvider interface.  It improves upon ThrowingProvider by allowing more than one exception type to be thrown:
```java
public interface CheckedProvider<T> {
  T get() throws Exception;
}
```
For each object where providing may throw an exception, create an interface that extends the CheckedProvider. For our news widget application, we created the FeedProvider interface that throws a FeedUnavailableException and SecurityException:
```java
public interface FeedProvider<T> extends CheckedProvider<T> {
  T get() throws FeedUnavailableException, SecurityException;
}

```

#### Binding the provider
After implementing our WorldNewsFeedProvider and SportsFeedProvider, we bind them in our module using ThrowingProviderBinder:
```java
public static class FeedModule extends AbstractModule {
  protected void configure() {
    ThrowingProviderBinder.create(binder())
        .bind(FeedProvider.class, BbcFeed.class)
        .annotatedWith(WorldNews.class)
        .to(WorldNewsFeedProvider.class)
        .in(HourlyScoped.class);

    ThrowingProviderBinder.create(binder())
        .bind(FeedProvider.class, BbcFeed.class)
        .annotatedWith(Sports.class)
        .to(SportsFeedProvider.class)
        .in(QuarterHourlyScoped.class);
  }
}
```

#### Using @CheckedProvides _(new in Guice 3.0)_
You can also bind checked providers use an @Provides-like syntax: @CheckedProvides.  This has all the benefits of [ProvidesMethods @Provides methods], and also allows you to specify exceptions.
```java
public static class FeedModule extends AbstractModule {
  protected void configure() {
    // create & install a module that uses the @CheckedProvides methods
    install(ThrowingProviderBinder.forModule(this)); 
  }

  @CheckedProvides(FeedProvider.class) // define what interface will provide it
  @HourlyScoped // scoping annotation
  @WorldNews // binding annotation
  BbcFeed provideWorld(FeedFactory factory)
      throws FeedUnavailableException, SecurityException {
    return factory.tryToFeed("bbc"); // may throw an exception
  }

  @CheckedProvides(FeedProvider.class) // define what interface will provide it
  @QuarterlyHourlyScoped // scoping annotation
  @Sports // binding annotation
  BbcFeed provideSports(FeedFactory factory)
      throws FeedUnavailableException, SecurityException {
    return factory.tryToFeed("bbc"); // may throw an exception
  }
```
  
#### Injecting the provider
Finally, we can inject the FeedProviders throughout our application code. Whenever we call get(), the compiler reminds us to handle the FeedUnavailableException and any other exceptions declared in the interface:
```java
public class BbcNewsWidget {
  private final FeedProvider<BbcFeed> worldNewsFeedProvider;
  private final FeedProvider<BbcFeed> sportsFeedProvider;

  @Inject
  public BbcNewsWidget(
      @WorldNews FeedProvider<BbcFeed> worldNewsFeedProvider,
      @Sports FeedProvider<BbcFeed> sportsFeedProvider) {
    this.worldNewsFeedProvider = worldNewsFeedProvider;
    this.sportsFeedProvider = sportsFeedProvider;
  }

  public GxpClosure render() {
    try {
      BbcFeed bbcWorldNews = worldNewsFeedProvider.get();
      BbcFeed bbcSports = sportsFeedProvider.get();
      return NewsWidgetBody.getGxpClosure(bbcWorldNews, bbcSports);
    } catch (FeedUnavailableException e) {
      return UnavailableWidgetBody.getGxpClosure();
    }
  }
}
```
#### Notes on Scoping
Scopes work the same way they do with Providers. Each time get() is called, the returned object will be scoped appropriately. Exceptions are also scoped. For example, when worldNewsFeedProvider.get() throws an exception, the same exception instance will be thrown for all callers within the scope.