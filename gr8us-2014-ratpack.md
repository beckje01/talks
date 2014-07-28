## Agenda

   * whoami
   * What is Ratpack
   * HelloWorld
   * Key Concepts
   * Example Subscription API
   * Testing
   * Fat Jar
   * Docker Deployment
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

The Chain provides a DSL for common routing to handlers. Allowing a very clean and concise expression of some routing with handler support.

If you want to provide a few handlers with some routing to insert to do further work you can use the chain to build out the handlers for you easily.

~~~~
## Context

Container for all contextual information. It is based around the idea of a registry, allowing handlers to look up needed objects.

The context is where you will put objects for downstream handlers to access.

~~
## Working with the Context

This examples uses `just` to create a Registry with a single new entry that will be appended onto current the context.

```groovy
import static ratpack.registry.Registries.just

def p = new Person(name: "Test")
context.next(just(Person.class, p))
```
~note
Example of exposing an object for further handlers.
~~
## Adding Multipul Values

We can use the RegistrySpec to build a registry with many values:

```
next(registry({ RegistrySpec registrySpec ->
                registrySpec.add(Foo.class, new Foo(value: "fooValue"))
                def people = []
                people.add(new Person(name: "PersonList1"))
                people.add(new Person(name: "PersonList2"))
                registrySpec.add(PersonList, people)
            }))
```

~~
## Retrieving Values

Doing a `get` will return an object of the type requested or throw a NotInRegistryException, while the `maybGet` will simply return a null if no object was found.

```groovy
def person = context.get(Person.class)

def foo = context.maybeGet(Foo.class)
```
~~
## Retrieving Values Groovy Sugar

In a handler you can also retrive values by declaring them as params to the handler closures, this works like a `context.get()`

```groovy

get { Person p ->
    render "Hello ${p.name}"
}

```
~~
## Example Script
```
handlers {
    handler {
        request.register(Person.class, new Person(name: "Test"))
        next()
    }
    handler {
        next(registry({ RegistrySpec registrySpec ->
            registrySpec.add(Foo.class, new Foo(value: "fooValue"))
            def people = []
            people.add(new Person(name: "PersonList1"))
            people.add(new Person(name: "PersonList2"))
            registrySpec.add(PersonList, people)
        }))
    }
    get { Person p ->
        def people = maybeGet(PersonList.class)
        render "Hello ${p.name}, ${people?.size()}"
    }
}
```
[Gist](https://gist.github.com/beckje01/136fce42cdba7036a988)

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

~~
## Request Scope Objects

One use more unique to Ratpack of the request object is storage of request scope objects.

~~
Instead of adding the person object to the context it is better to add it to the request.

```groovy
request.register(Person.class, new Person(name: "Test"))
```
~~
## Response

The response is the low level HTTP response the server is sending. Here you can work with the expected items:

 * Cookies
 * Status Codes
 * Headers
 * Body

~~
## Sending a Response

Handlers need to eventually send the response that is done at a low level by calling `send` on a response.

In general this is a low level interface for sending data and you may not want to work directly with it but instead depending on the Rendering framework.
~~
## Rendering

The [render][1] method available on the context will send an object off to be rendered by the Rendering Framework.

[1]: http://www.ratpack.io/manual/0.9.7/api/ratpack/handling/Context.html#render(java.lang.Object)

~~
## Json Rendering

The Jackson module offers a json renderer, note the need to bind the Jackson module without that you will not have the json renderer available.

```
ratpack {
  bindings {
    add new JacksonModule()
  }
  handlers {
    get("some-json") {
      render json(user: 1)  // will render '{user: 1}'
    }
  }
}
```
~~~~
## Specs

Spec classes in the Ratpack API are lazily executed and something you never instantiate yourself.

~~
## Specs

To work with Spec the Ratpack API will have methods that accept params like `Action<? super RequestSpec> action`

In Groovy we can just pass in a closure that acts on the RequestSpec in the above case.

~~
##Spec Example

requestSpec { RequestSpec request ->  
  request.body.type("text/plain").stream { it << "foo" }
}

~~
## Why Specs

 * Easy Composition
 * Interfaces Everywhere
~~
## Specs

Important Examples

 * BindingsSpec
 * HttpUrlSpec
 * RequestSpec
 * RegistrySpec

~~~~
## Blocking / Async

The core of Ratpack is really about building a non-blocking HTTP app. So when ever you are working with blocking IO you want to wrap that in something that will be non-blocking for the handler.

[Reading Homework](http://www.ratpack.io/manual/0.9.7/async.html#performing_async_operations)
~~
## Blocking DSL

Inside of the handlers block we can do the following assuming the datastore is blocking. [Blocking DSL][2]

```
handlers {
  get("deleteOlderThan/:days") { Datastore datastore ->
    blocking {
      datastore.deleteOlderThan(pathTokens.asInt("days"))
    } then {
      context.render("$it records deleted")
    }
  }
}
```
[2]: http://www.ratpack.io/manual/0.9.7/api/ratpack/handling/Context.html#blocking(java.util.concurrent.Callable)
~~
## Ratpack Async

In general Ratpack exposes a low level [Promise](http://www.ratpack.io/manual/0.9.7/api/ratpack/exec/Promise.html) that is the base way to interact with the Async / Non Blocking.

A user can adapt external Async libraries into ratpack by using [context.promise][3]

[3]: http://www.ratpack.io/manual/0.9.7/api/ratpack/handling/Context.html#promise(ratpack.func.Action)

~~
## Better Ratpack Asyc

Check out the ratpack-rx module, a clean integration of RX Java. Makes it much easier to deal with async.

~~~~
##Example Subscription Service

A simple REST resource representing a subscription.

~~
## Lazybones

Kick start a basic Ratpack app.

```
gvm use lazybones
lazybones create ratpack rat-app
cd rat-app
gradle run
```
~~
## Stub Out

In Ratpack.groovy stub out all the handlers I will be using for the app.

[Code](https://github.com/beckje01/ratpack-subscription-api/blob/Step-1/src/ratpack/Ratpack.groovy)

Under the covers the handlers closure delegates to GroovyChain which is a Handler Chain with groovy sugar.

~~
## Gradle Run
With the simple `gradle run` supports a few key items:

 * Reloading
 * Debug available via `gradle run --debug-jvm`

~~
## Adding Simple Token Auth

Here you see the handler delegating to other handlers.
```groovy
handler {
    //A very simple handler to check token auth on a header
    if (request.headers['Authorization'] != "Token faketoken") {
        response.status(401)
        //We must send some response or the request will hang.
        response.send()
    } else {
        //We can choose to do nothing but allow the next handler in the chain to deal with the request.
        next()
    }
}
```
~~~~
## Unit Testing

Using Spock is the standard you will see across many of the example applications.

Add the following to dependencies:
```
testCompile "org.spockframework:spock-core:0.7-groovy-2.0"
```
~~
## Unit - Handler

If you want to work with testing a handler itself Ratpack provides an easy method to create a invocation of a handler. See [UnitTest.invoke()](http://www.ratpack.io/manual/current/api/ratpack/test/UnitTest.html#invoke%28ratpack.handling.Handler,%20ratpack.func.Action%29) and [GroovyUnitTest.invoke()](http://www.ratpack.io/manual/current/api/ratpack/groovy/test/GroovyUnitTest.html#invoke%28ratpack.handling.Handler,%20groovy.lang.Closure%29)
```
Invocation invocation = invoke(new MyHandler(), new Action<InvocationBuilder>() {
  public void execute(InvocationBuilder builder) {
    builder.header("input-value", "foo");
    builder.uri("some/path");
  }
});
```

The manual is up to date on this topic so check [here](http://www.ratpack.io/manual/current/testing.html#unit_testing_ratpack_handlers).

~~~~
## Functional Testing

Functional testing for Ratpack is built up around the [ApplicationUnderTest](http://www.ratpack.io/manual/current/api/index.html?ratpack/groovy/test/TestHttpClient.html).

This interface provides the address of the running application. Implementations of this interface will take care of starting the server for you.

~~
## Functional - TestHttpClient

Ratpack also provides a [TestHttpClient](http://www.ratpack.io/manual/current/api/index.html?ratpack/groovy/test/TestHttpClient.html) in the groovy test area.
```groovy
def aut = new LocalScriptApplicationUnderTest()
@Delegate TestHttpClient client = aut.httpClient()

def "Check Site Index"() {
	when:
	get("index.html")

	then:
	response.statusCode == 200
	response.body.asString().contains('<title>Ratpack: A toolkit for JVM web applications</title>')

}
```  
Examples: [Ratpack Site](https://github.com/ratpack/ratpack/blob/master/ratpack-site/src/test/groovy/ratpack/site/SiteSmokeSpec.groovy) - [Midcentury Ipsum](https://github.com/robfletcher/midcentury-ipsum/blob/master/src/test/groovy/app/FunctionalSpec.groovy)
~note
The TestHttpClient interface will be changing soon. To be based around ratpacks internal http client.
~~
## Functional - Geb

Geb can also be used we just need to do a little work to set up the correct base URL and make sure the test app is shut down.
```groovy
private final static ServerBackedApplicationUnderTest applicationUnderTest = new LocalScriptApplicationUnderTest()

def setup() {
  URI base = applicationUnderTest.address
  browser.baseUrl = base.toString()
}

def cleanup() {
  applicationUnderTest.stop()
}
```
[Example](https://github.com/ratpack/ratpack/blob/master/ratpack-site/src/browserTest/groovy/ratpack/site/SiteBrowserSmokeSpec.groovy)


~~~~
## Fat Jar

As of Ratpack 0.9.7 the Gradle Shadow plugin has built in support inside Ratpack core.

```
buildscript {
  repositories {
    jcenter()
    maven { url "http://oss.jfrog.org/artifactory/repo" }
  }
  dependencies {
    classpath "io.ratpack:ratpack-gradle:0.9.7-SNAPSHOT"
    classpath 'com.github.jengelman.gradle.plugins:shadow:1.0.2'
  }
}

apply plugin: 'com.github.johnrengelman.shadow'
```
~~~~
## Docker Deployment

 * Use our Fat Jar
 * Create image with [Docker Gradle Plugin](https://github.com/Transmode/gradle-docker)
 * Push Image to Docker Hub
 * Deploy to AWS BeanStalk

~~
## Docker Gradle Plugin

Update build.gradle

```
buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'se.transmode.gradle:gradle-docker:1.2'
  }
}

apply plugin: 'docker'

group='beckje01'

distDocker {
    exposePort 5050
}
```

~~
## Docker Gradle Plugin

You will need docker set up locally, `boot2docker` is great for OSX.

Then run `gradle distDocker`.

~~
## Push Image

```shell
docker login
docker push beckje01/ratpack-subscription-api
```

~~
## AWS Beanstalk

```
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "beckje01/ratpack-subscription-api",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "5050"
    }
  ]
}
```
[Running Link](http://default-environment-3gyepqbfsi.elasticbeanstalk.com/subscriptions)

~~
## Docker Deploy Links

 * [Docker AWS Deployment](https://www.youtube.com/watch?v=lBu7Ov3Rt-M&feature=youtu.be)
 * [Gradle Docker Plugin](https://github.com/Transmode/gradle-docker)
 * [Boot2Docker](https://github.com/boot2docker/boot2docker)
 * [Docker Hub](https://hub.docker.com/)
