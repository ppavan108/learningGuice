_How the Injector resolves injection requests_
### Binding Resolution
The injector's process of resolving an injection request depends on the bindings and the annotations of the types involved.  Here's how an injection request is resolved:
  1. **Use explicit bindings.**
    * If the binding links to another, follow this resolution algorithm for that.
    * If the binding specifies an instance, return that.
    * If the binding specifies a provider, use that.
  2. **Ask a parent injector.** If this injector has a parent injector, ask that to resolve the binding. If it succeeds, use that. Otherwise proceed.
  3. **Ask child injectors.** If any child injector already has this binding, give up. A blacklist of bindings from child injectors is kept so injectors don't need to maintain references to their child injectors.
  4. **Handle Provider injections.** If the type is `Provider<T>`, resolve `T` instead, using the same binding annotation, if it exists. 
  5. **Convert constants.** If there is a constant string bound with the same annotation, and a `TypeConverter` that supports this type, use the converted String.
  6. **If the dependency has a binding annotation, give up.** Guice will not create default bindings for annotated dependencies.
  7. **If the dependency is an array or enum, give up.**
  8. **Handle TypeLiteral injections**. If the type is `TypeLiteral<T>`, inject that value, using context for the value of the type parameter.
  9. **Use resolution annotations.** If the dependency's type has `@ImplementedBy` or `@ProvidedBy`, lookup the binding for the referenced type and use that.
  10. **If the dependency is abstract or a non-static inner class, give up.**
  11. **Use a single `@Inject` or public no-arguments constructor.**
    1. Validate bindings for all dependencies — the constructor's parameters, plus `@Inject` methods and fields of the type and all supertypes.
    2. Invoke the constructor.
    3. Inject all fields. Supertype fields are injected before subtype fields.
    4. Inject all methods. Supertype methods are injected before subtype methods.