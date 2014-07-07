_Overview of the Injector creation process_
### The Injector Creation Process
Guice builds an injector using configuration modules. If there are errors at the end of any phase, injector creation is halted and a `CreationException` is thrown. 

Phase 1: Static Building
In this phase, Guice interprets elements, creates bindings, and validates the configuration. The only user code executed in this phase is `Module.configure()`.

This is the only phase executed for `Stage.TOOL`.

#### Phase 2: Injection
During this phase, objects will be injected on-demand if necessary. For example, if satisfying a static injection requires a provider instance, the provider will be injected before it is used. If initialized objects are circularly dependent, the order of injection is undefined.

First, **statics** registered via `requestStaticInjection()` are injected. Next, **instances** that are the arguments to `requestInjection()`, `toInstance()` and `toProvider()` are injected.

#### Phase 3: Singleton Preloading
In `Stage.PRODUCTION`, all singletons are created. In `Stage.DEVELOPMENT`, only bindings scoped using `asEagerSingleton()` are created.