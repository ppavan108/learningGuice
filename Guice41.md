### Guice 4.1

Released June 17, 2016

#### Maven

Guice:

```xml
<dependency>
  <groupId>com.google.inject</groupId>
  <artifactId>guice</artifactId>
  <version>4.1</version>
</dependency>
```

Guice (No AOP):

```xml
<dependency>
  <groupId>com.google.inject</groupId>
  <artifactId>guice</artifactId>
  <version>4.1</version>
  <classifier>no_aop</classifier>
</dependency>
```

Extensions:

```xml
<dependency>
  <groupId>com.google.inject.extensions</groupId>
  <artifactId>guice-${extension}</artifactId>
  <version>4.1</version>
</dependency>
```

#### Downloads

 * Guice: [guice-4.1.jar](http://search.maven.org/remotecontent?filepath=com/google/inject/guice/4.1/guice-4.1.jar)
 * Guice (No AOP): [guice-4.1-no_aop.jar](http://search.maven.org/remotecontent?filepath=com/google/inject/guice/4.1/guice-4.1-no_aop.jar)
 * Guice extensions are all directly downloadable [from this search page](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.google.inject.extensions%22%20AND%20v%3A%224.1%22).  Just click on the "jar" link for the appropriate extension.

#### Docs

  * [API documentation](https://google.github.io/guice/api-docs/4.1/javadoc/index.html)
  * [API Changes from 4.0 to 4.1, by JDiff](http://google.github.io/guice/api-docs/4.1/api-diffs/changes.html)

#### Changes since Guice 4.0 (coming soon!)

#### Migrating from Guice 4.0
See the [JDiff change report](http://google.github.io/guice/api-docs/4.1/api-diffs/changes.html) for complete details.
