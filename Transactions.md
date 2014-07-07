_Transactions with Guice Persist_

### Method-level @Transactional

By default, any method marked with `@Transactional` will have a transaction started before, and committed after it is called. 

```java
  @Transactional
  public void myMethod() { ... }
```

The only restriction is that these methods must be on objects that were created by Guice and they must not be `private`.

### Responding to exceptions

If a transactional method encounters an unchecked exception (any kind of `RuntimeException`), the transaction will be rolled back. Checked exceptions are ignored and the transaction will be committed anyway.

To change this behavior, you can specify your own exceptions (checked or unchecked) on a per-method basis:

```java
  @Transactional(rollbackOn = IOException.class)
  public void myMethod() throws IOException { ... }
```
 
Once you specify a `rollbackOn` clause, only the given exceptions and their subclasses will be considered for rollback. Everything else will be committed. Note that you can specify any combination of exceptions using array literal syntax:

```java
  @Transactional(rollbackOn = { IOException.class, RuntimeException.class, ... })
```
 
It is sometimes necessary to have some general exception types you want to rollback but particular subtypes that are still allowed. This is also possible using the `ignore` clause:

```java
  @Transactional(rollbackOn = IOException.class, ignore = FileNotFoundException.class)
```
 
In the above case, any `IOException` (and any of its subtypes) except FileNotFoundException will trigger a rollback. In the case of `FileNotFoundException`, a commit will be performed and the exception propagated to the caller anyway. 

_Note that you can specify any combination of checked or unchecked exceptions._

### Units of Work 

A unit of work is roughly the lifespan of an `EntityManager` (in JPA). It is the _session_ referred to in the `session-per-*` strategies. We have so far seen how to create a unit of work that spans a single transaction:

http://warp-persist.googlecode.com/svn/wiki/txn_unitofwork.png

...and one that spans an entire HTTP request:

http://warp-persist.googlecode.com/svn/wiki/request_unitofwork.png

Sometimes you need to define a custom unit-of-work that doesn't fit into either requests or transactions. For example, you may want to do some background work in a timer thread, or some initialization work during startup. Or perhaps you are making a desktop app and have some other idea of a unit of work.

http://warp-persist.googlecode.com/svn/wiki/custom_unitofwork.png

To start and end a unit of work arbitrarily, inject the `UnitOfWork` interface:

```java
public class MyBackgroundWorker {
  @Inject private UnitOfWork unitOfWork;
 
  public void doSomeWork() {
    unitOfWork.begin();
    try { 
      // Do transactions, queries, etc...
      //...
    } finally {
      unitOfWork.end(); 
    }
  } 
}
```
 
You are free to call any `@Transactional` methods while a unit of work is in progress this way. When `end()` is called, any existing session is closed and discarded. It is safe to call `begin()` multiple times--if a unit of work is in progress, nothing happens. Similarly, if one is ended calling `end()` returns silently. `UnitOfWork` is threadsafe and can be cached for multiple uses or injected directly into singletons.