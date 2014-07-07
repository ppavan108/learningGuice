_Getting started with Guice Persist_
_New in Guice 3.0_

### Introduction

Guice Persist provides abstractions for working with datastores and persistence providers in your Guice applications. It works in a normal Java desktop or server application, inside a plain Servlet environment, or even a Java EE container.

Guice Persist supports a simple approach to transactions and various unit-of-work semantics; both with the same declarative, annotation-based API. Switching between unit-of-work strategies is as simple as changing a configuration option.

It also follows the Guice idioms of using simple, expressive configuration and tries to remain type-safe as far as possible. 

##### Support

We currently support the following persistence providers:
  * JPA (Java Persistence API) - with any compliant vendor implementation.

And a simple mechanism to extend to other persistence providers.

##### Getting Started

To begin, you must obtain a copy of `guice-persist.jar` from the Guice 3.0 download.