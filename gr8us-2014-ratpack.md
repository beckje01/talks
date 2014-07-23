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

  * handler chain
  * context
  * request / response
  * blocking/async/execution
  * Specs

~~
## Handler Chain

~~
## context
~~
## request / response
~~
## blocking/async/execution
~~
## Specs

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
