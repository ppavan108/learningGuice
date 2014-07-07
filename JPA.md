_Using JPA with Guice Persist._

### Java Persistence API (JPA)

JPA is a standard released as part of JSR-220 (or EJB3). It is roughly an analog to Hibernate or Oracle TopLink; and they are also the two most prominent vendors of JPA. Guice Persist supports JPA in a vendor-agnostic fashion, so it should be easy to swap between implementations (such as AppEngine's datastore) should you need to.

##### Enabling Persistence Support

To enable persistence support, simply install the JPA module:

```java
Injector injector = Guice.createInjector(..., new JpaPersistModule("myFirstJpaUnit"));
```
 
In JPA, you specify your configuration in a *persistence.xml* file in the `META-INF` folder of that is available on the classpath (if inside a jar, then at the root for example). In this file you declare several _Persistence Units_ that are roughly different profiles (for example, backed by different databases). Here is an example of a simple JPA configuration:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
	http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd" version="1.0">

	<!-- A JPA Persistence Unit -->
	<persistence-unit name="myFirstJpaUnit" transaction-type="RESOURCE_LOCAL">
		<provider>org.hibernate.ejb.HibernatePersistence</provider>
 
		<!-- JPA entities must be registered here -->
		<class>com.wideplay.warp.jpa.JpaTestEntity</class>

		<properties>
			<!-- vendor-specific properties go here -->
		</properties>
	</persistence-unit>

</persistence>
```

To tell Guice Persist which persistence unit you wish you use, you specify its name when creating your module:

```java
Injector injector = Guice.createInjector(..., new JpaPersistModule("myFirstJpaUnit"));
```

Finally, you must decide when the persistence service is to be started by invoking `start()` on `PersistService`. I typically use a simple initializer class that I can trigger at a time of my choosing:

```java
public class MyInitializer { 
	@Inject MyInitializer(PersistService service) {
		service.start(); 
 
 		// At this point JPA is started and ready.
	} 
}
```

It makes good sense to use Guice's [Service API] to start all services in your application at once. I recommend you think about how it fits into your particular deployment environment when doing so. 

However, in the case of web applications, you do not need to do anything if you simply install the `PersistFilter` (see below).

##### Session-per-transaction strategy

This is a popular strategy, where a JPA `EntityManager` is created and destroyed around each database transaction. Set the transaction-type attribute on your persistence unit's configuration:

```xml
 <persistence-unit name="myFirstJpaUnit" transaction-type="RESOURCE_LOCAL">
```

##### Using the EntityManager inside transactions
Once you have the injector created, you can freely inject and use an `EntityManager` in your transactional services:

```java
import com.google.inject.persist.Transactional;
import javax.persistence.EntityManager; 
 
public class MyService {
	@Inject EntityManager em; 
 
	@Transactional 
	public void createNewPerson() {
		em.persist(new Person(...)); 
	} 
}
```
 
This is known as the _session-per-transaction_ strategy. For more information on transactions and units of work (sessions), see [[this page|Transactions]].

Note that if you make `MyService` a `@Singleton`, then you should inject `Provider<EntityManager>` instead.

##### Web Environments (session-per-http-request)

So far, we've seen the session-per-transaction strategy. In web environments this is atypical, and generally a _session-per-http-request_ is preferred (sometimes also called _open-session-in-view_). To enable this strategy, you first need to add a filter to your `ServletModule`:

```java
public class MyModule extends ServletModule {
  protected void configureServlets() {
    install(new JpaPersistModule("myJpaUnit"));  // like we saw earlier.

    filter("/*").through(PersistFilter.class);
  }
}
```
 
You should typically install this filter before any others that require the `EntityManager` to do their work.
 
Note that with this configuration you can run multiple transactions within the same `EntityManager` (i.e. the same HTTP request).