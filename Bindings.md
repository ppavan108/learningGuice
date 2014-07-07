_Overview of bindings in Guice_
### Bindings
The injector's job is to assemble graphs of objects. You request an instance of a given type, and it figures out what to build, resolves dependencies, and wires everything together. To specify how dependencies are resolved, configure your injector with bindings.

#### Creating Bindings
To create bindings, extend `AbstractModule` and override its `configure` method. In the method body, call `bind()` to specify each binding. These methods are type checked so the compiler can report errors if you use the wrong types. Once you've created your modules, pass them as arguments to `Guice.createInjector()` to build an injector.

Use modules to create [[linked bindings|LinkedBindings]], [[instance bindings|InstanceBindings]], [[@Provides methods|ProvidesMethods]], [[provider bindings|ProviderBindings]], [[constructor bindings|ToConstructorBindings]] and [[untargetted bindings|UntargettedBindings]].

#### More Bindings
In addition to the bindings you specify the injector includes [[built-in bindings|BuiltInBindings]]. When a dependency is requested but not found it attempts to create a [JustInTimeBindings just-in-time binding]. The injector also includes bindings for the [[providers|InjectingProviders]] of its other bindings.