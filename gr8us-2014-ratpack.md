## Agenda

   * whoami
   * What is Ratpack
   * HelloWorld
   * key concepts
   * Bigger example (subscription service)
   * Testing
      * unit
      * functional
   * fat jar
   * docker deployment (aws?)
~~~~
## $ whoami

Jeff Beck

TechLead at ReachLocal

~~~~
## What Is Ratpack

A set of libraries focused on making fast and efficient HTTP applications.

~~
## Core

 * Java 7 with an eye towards Java 8
 * Netty
 * Guava

~~
## Groovy Support

 * Static Compilation Support
 * Typing Features via @DelegatesTo
 * Good IDE Support

~~
## Gradle Plugin

 * Provides Base setup
 * Exposes Ratpack extension libraries as easy dependencies
  * `compile ratpack.dependency("jackson")`

~~~~
## HelloWorld

```groovy
@Grab('io.ratpack:ratpack-groovy:0.9.6')
import static ratpack.groovy.Groovy.*

ratpack {
  handlers {
    get {
      render "Hello world!"
    }
  }
}

```

~~~~
## Key Concepts

  * Handlers
  * Context
  * Request / Response  
  * Specs
  * blocking/async/execution

~~~~
## Handler

A handler acts on a Context and will either send a response or delegate to another handler.

```groovy
public class HelloWorld implements Handler {
  public void handle(Context context) {
      context.response().send("Hello world!")
  }
}
```

~~
## Handler Delegating

The handler can delegate to the next handler or it can add some new handlers to do work based on logic in the handler.

Simple delegation:
```groovy
context.next()
```
Or insert handlers:
```groovy
def handler = new FooHandler()
context.insert(handler)
```
~~
## Composition

Allowing many handlers to act on a context we are able to use composition to great effect. We can do complex routing based on this composition by inserting other handlers.

~~
## Handler Chain

//TODO Review

The Chain provides a DSL for common routing to handlers. Allowing a very clean and concise expression of some routing with handler support.

If you want to provide a few handlers with some routing to insert to do further work you can use the chain to build out the handlers for you easily.

~~~~
## Context

Container for all contextual information. It is based around the idea of a registry, allowing handlers to look up needed objects.

The context is where you will put objects for downstream handlers to access.

~~
## Working with the Context

```groovy
import static ratpack.registry.Registries.just

def p = new Person(name: "Test")
context.next(just(Person.class, p))
```

This examples uses `just` to create a Registry with a single new entry that will be appended onto current the context.
~note
Example of exposing an object for further handlers.
~~
## Adding Multipul Values

```
//todo
```

~~
## Retrieving Values

```groovy
def person = context.get(Person.class)

def foo = context.maybeGet(Foo.class)
```

Doing a `get` will return an object of the type requested or throw a NotInRegistryException, while the `maybGet` will simply return a null if no object was found.

~~~~
## Request / Response

The request and response objects are also available on the context but they have their own convenience methods.

```groovy
context.getRequest()
context.getResponse()
```
~~
## Request

The HTTP request sent from the client to the server. It holds some of the expected items the handlers will need to process:

 * Headers
 * Body
 * Query Parameters
 * Cookies

~~~~
## Specs
~~~~
## blocking/async/execution
~~~~
## Bigger Example Sub Service

~~
## Lazybones

~~
## Build Out

~~
## Gradle Run

~~~~
## Testing

~~~~
## Fat Jar

~~~~
## Docker Deployment
