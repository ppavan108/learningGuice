_How Guice decides what gets injected_
### Injection Points

#### Constructor Selection
When Guice instantiates a type using its constructor, it decides which constructor to invoke by following these rules:
  1. Find and return an `@Inject`-annotated constructor
    * If any are `@Inject(optional=true)` constructors, record an error. Constructors may not be optional.
    * If there are multiple `@Inject`-annotated constructors, record an error. There may be at most one `@Inject`-annotated constructor.
    * If this constructor has binding annotations (like `@Named`), record an error. Binding annotations on the constructor's _parameters_ are okay, but not on the constructor itself.
  2. Find and return a no-arguments constructor
    * If this constructor is `private` but the class is not `private`, record an error. Private constructors on non-private classes are not injectable without the `@Inject` annotation.
    * If this constructor itself has binding annotations (like `@Named`), record an error. Binding annotations on the constructor's _parameters_ are okay, but not on the constructor itself.
  3. No suitable constructor was found. Record an error.

#### Injecting Fields and Methods
Injections are performed in a specific order. All fields are injected and then all methods. Within the fields, supertype fields are injected before subtype fields. Similarly, supertype methods are injected before subtype methods.

#### What Gets Injected
Guice injects **instance methods** and fields of:
  * All values that are bound using `toInstance()`. These are injected at injector-creation time.
  * All providers that are bound using `toProvider(Provider)`. These are injected at injector-creation time.
  * All instances that are registered for injection using `requestInjection`. These are injected at injector-creation time.
  * The arguments to `Injector.injectMembers` when that method is invoked.
  * All values instantiated by Guice via its injectable constructor, immediately after it is instantiated. Regardless of how the value is scoped, it is only injected once.

Guice injects **static methods** and fields of:
  * All classes that are registered for static injection using `requestStaticInjection`.

#### How Guice Injects
**To inject a field**, Guice will:
  * Lookup the binding, creating a just-in-time binding if necessary. If no just-in-time binding can be created, an error is reported. 
  * The binding is exercised to provide a value. How this works depends on the type of the binding.
  * The binding's value is set into the field. Injecting `final` fields is not recommended because the injected value may not be visible to other threads.
If the field injection is `@Inject(optional=true)` and the binding neither exists, nor can be [[created|JustInTimeBindings]], the field injection is skipped. 

**To inject a method**, Guice will:
  * Lookup bindings for all of the parameters, creating just-in-time bindings if necessary. If any just-in-time binding cannot be created, an error is reported. 
  * The bindings are each exercised to provide a value. How this works depends on the types of the bindings.
  * The method is called, using the bindings' values as arguments.
If the method injection is optional and a parameter's binding neither exists nor can be [[created|JustInTimeBindings]], the method injection will be skipped. 