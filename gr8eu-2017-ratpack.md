## Agenda

* whoami
* What is Ratpack
* HelloWorld
* Key Concepts
* Organizing Your App
* Testing
* Fat Jar

-note
Abstract

Ratpack is a simple but very powerful framework for web applications.

We will go through the steps of getting started with a Ratpack project. We will use lazybones to bootstrap ourselves. Then use gradle to run our application locally.

But don’t worry this talk will go beyond hello world you will learn about the power of using Ratpack’s promises as well as best practices of app structure to make sure you Promise code is easy to compose and test.

At the end you should be ready to start building Ratpack apps for yourself.
----
## $ whoami

Jeff Beck

Software Architect at SmartThings

----
## What Is Ratpack

  * Minimal HTTP Focused Framework
  * Netty Based

----
## Hello World

Kick start a basic Ratpack app.

```
sdk use lazybones
lazybones create ratpack rat-app
cd rat-app
gradle run
```
---
## Debugging

```
gradle run --debug-jvm
```
Or

Setting up IDEA as a Groovy Script Run

[Debugging Application Defined Using Groovy DSL In IntelliJ IDEA](http://mrhaki.blogspot.dk/2016/01/ratpacked-debugging-application-defined.html) - Mr Haki

---
## Continuous Build

Using Continuous build from gradle you get reloading and a great development flow.

```shell
gradle -t run
```

---
## Continuous Build

This also works for tests.

```shell
gradle -t test
```

----
## Key Concepts

  * Handlers
  * Context
  * Request / Response  
  * Promises

----
## Handler

A handler acts on a Context and will either send a response or delegate to another handler.

```groovy
public class HelloWorld implements Handler {
  public void handle(Context context) {
      context.response().send("Hello world!")
  }
}
```

---
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
---
## Composition

Allowing many handlers to act on a context we are able to use composition to great effect. We can do complex routing based on this composition by inserting other handlers.

---
## Handler Chain

The Chain provides a DSL for common routing to handlers. Allowing a very clean and concise expression of some routing with handler support.

If you want to provide a few handlers with some routing to insert to do further work you can use the chain to build out the handlers for you easily.

----
## Context

Container for all contextual information. It is based around the idea of a registry, allowing handlers to look up needed objects.

The context is where you will put objects for downstream handlers to access.

---
## Working with the Context

This examples uses `just` to create a Registry with a single new entry that will be appended onto current the context.

```groovy
import static ratpack.registry.Registry.single

def p = new Person(name: "Test")
context.next(single(p))
```
-note
Example of exposing an object for further handlers.
---
## Adding Multiple Values

We can use the RegistrySpec to build a registry with many values:

```
next(Registry.of({ RegistrySpec registrySpec ->
                registrySpec.add(Foo.class, new Foo(value: "fooValue"))
                def people = []
                people.add(new Person(name: "PersonList1"))
                people.add(new Person(name: "PersonList2"))
                registrySpec.add(PersonList, people)
            }))
```

---
## Retrieving Values

Doing a `get` will return an object of the type requested or throw a NotInRegistryException, while the `maybGet` will simply return a null if no object was found.

```groovy
def person = context.get(Person.class)

def foo = context.maybeGet(Foo.class)
```
---
## Retrieving Values Groovy Sugar

In a handler you can also retrieve values by declaring them as params to the handler closures, this works like a `context.get()`

```groovy

get { Person p ->
    render "Hello ${p.name}"
}

```

----
## Request / Response

The request and response objects are also available on the context but they have their own convenience methods.

```groovy
context.getRequest()
context.getResponse()
```
---
## Request

The HTTP request sent from the client to the server. It holds some of the expected items the handlers will need to process:

 * Headers
 * Body
 * Query Parameters
 * Cookies

---
## Request Body

Working with the request body is a Promise. We will see more shortly

---
## Response

The response is the low level HTTP response the server is sending. Here you can work with the expected items:

 * Cookies
 * Status Codes
 * Headers
 * Body

---
## Sending a Response

Handlers need to eventually send the response that is done at a low level by calling `send` on a response.

In general this is a low level interface for sending data and you may not want to work directly with it but instead depending on the Rendering framework.
---
## Rendering

The [render][1] method available on the context will send an object off to be rendered by the Rendering Framework.

[1]: https://ratpack.io/manual/1.4.5/api/ratpack/handling/Context.html#render-java.lang.Object-

---
## Json Rendering


```groovy
import static ratpack.jackson.Jackson.json

ratpack {
  handlers {
    get("some-json") {
      render json([user: 1])  // will render '{user: 1}'
    }
  }
}
```
----
## Promises

In general Ratpack exposes [Promise](https://ratpack.io/manual/1.4.5/api/ratpack/exec/Promise.html).

---
## Working with Promises

  * Promises are don't execute unless consumed
  * Methods with `map` in the name transforms data
  * `then` is the method that activates the Promise
  * `onError` allows you to deal with error cases

---
## Example Body

```
Promise<TypedData> bodyPromise = request.body
Promise<String> bodyStringPromise = bodyPromise.map({ TypedData typedDate ->
  return typedDate.text
})

bodyStringPromise.then { String bodyText ->
  render bodyText
}
```

---
## Simplified

```
request.body.map({TypedData typedDate ->
  return typedDate.text
}).then { String bodyText ->
  render bodyText
}
```

---
## Even

```
render request.body.map { TypedData typedDate ->
  return typedDate.text
}
```

-note
The then in this case is actually part of the renderer for Promises

----
## Organizing Your App

 * All routing *shouldn't* live in `Ratpack.groovy`
 * All application logic *shouldn't* live in Handlers
 * Related things *should* be grouped

----
## Ratpack.groovy

 Separate out routing to related routes. Such as everything under `/locations` to its own chain.


 ```groovy
handlers {
 all(new BearerTokenAuthHandler(registry.get(TokenValidator)))

 prefix('hubs') {
   all chain(registry.get(HubEndpoints))
 }

 prefix('locations') {
   all chain(registry.get(LocationEndpoints))
 }

 get('countries', CountryHandler)
}
```
---
## Chain for Organizing
```groovy
class HubEndpoints implements Action<Chain> {
  @Override
	void execute(Chain chain) throws Exception {
    Groovy.chain(chain) {
      //Same DSL here as handlers { } in Ratpack.groovy
      path(':id') {
        byMethod {
          String hubId = pathTokens.id
          delete {
            render "DELETE at /hubs/${hubId}"
          }
          get {
            render "GET at /hubs/${hubId}"
          }
        }    
      }
    }
  }
}
```
----
## Ratpack.groovy Include

Include other Groovy script files into Ratpack.groovy

```groovy
ratpack {
  bindings {
    //...
  }

  handlers {
    //...
  }

  include("users.groovy")
}
```
---
## users.groovy

```groovy
import static ratpack.groovy.Groovy.ratpack

ratpack {
  bindings {

  }

  handlers {
    path("users") {
      byMethod {
        get {
          render "GET on users"
        }
        post {
          render "POST on users"
        }
      }
    }
  }
}
```

---
## Differences from Chain

  * Allows Bindings
  * Merges the handlers block
    * Chain method can bind to a path include doesn't
    * Order with include vs handlers{} matters  
<br>
See [Ratpack Include Example](https://github.com/beckje01/greach-ratpack-include-example) and [API Doc](https://ratpack.io/manual/1.4.5/api/ratpack/groovy/Groovy.Ratpack.html#include-java.lang.String-)

----
## What Goes in a Handler

 * Parsing
 * Async calls to do the business logic
 * Call to render in error cases and complete cases

----
## Ratpack Service

 * Implements the [ratpack.service.Service](https://ratpack.io/manual/1.4.5/api/ratpack/service/Service.html) interface.
 * Notified of Starting and Stopping
 * Added to the registry, commonly in the bindings block

---
## Service Ordering

 * Services ordered based on dependencies.
 * [DependsOn](https://ratpack.io/manual/1.4.5/api/ratpack/service/DependsOn.html) and [ServiceDependencies](https://ratpack.io/manual/1.4.5/api/ratpack/service/ServiceDependencies.html)
   * Allows for annotations to declare simple dependencies via `DependsOn`
   * After the fact or more complex dependencies can be expressed with `ServiceDependencies`
-note

Great for doing dependencies between libraries that didn't know about each other such as making sure datastore migrations are done before preparing statements.
----
## Guice Modules for Grouping

Useful to group sets of business logic

```groovy
class HubModule extends AbstractModule {
  @Override
  protected void configure() {
    bind(ShardLib).in(Scopes.SINGLETON)
    bind(HubEndpoints).in(Scopes.SINGLETON)
    bind(HubService).in(Scopes.SINGLETON)
  }
}
```
-note

Here we add to the registry a Chain for routing, a library, and a Ratpack Service

ShardLib is just a pogo

----
## Unit Testing

Using Spock is the standard you will see across many of the example applications and it's included from lazybones.

---
## Unit - Handler

If you want to work with testing a handler itself Ratpack provides an easy method. See [RequestFixture](https://ratpack.io/manual/current/api/ratpack/test/handling/RequestFixture.html#handle-ratpack.handling.Handler-ratpack.func.Action-) and [HandlingResult](https://ratpack.io/manual/current/api/ratpack/test/handling/HandlingResult.html)

```
HandlingResult result = GroovyRequestFixture.handle(handler) { RequestFixture rf ->
	rf.registry.add(objectMapper)
	rf.header(HttpHeaderNames.ACCEPT, "application/octet-stream")
	rf.body(objectMapper.writeValueAsString(initialRequestBody), 'application/json')
}
```
---
## Note About HandlingResult

If you are calling `render` you get back the object passed into rendering not the rendered output.

----
## Functional Testing

Functional testing for Ratpack is built up around the [ApplicationUnderTest](https://ratpack.io/manual/1.4.5/api/ratpack/test/ApplicationUnderTest.html).

This interface provides the address of the running application. Implementations of this interface will take care of starting the server for you.

---
## Example Usage

```
class ExampleAUT extends GroovyRatpackMainApplicationUnderTest {
	@Override
	protected void addImpositions(ImpositionsSpec impositions) {
		impositions.add(UserRegistryImposition.of {
			Registry.of {
				it.add(HandlerDecorator, RemoteControl.handlerDecorator())
				it.add(TokenValidator, new NoOpTokenValidator())
			}
		})
	}

	@Override
	TestHttpClient getHttpClient() {
		return TestHttpClient.testHttpClient(this) { RequestSpec request ->
			request.headers.set(HttpHeaderNames.AUTHORIZATION, "Bearer FakeToken")
		}
	}
}
```

---
## Functional - TestHttpClient

Ratpack also provides a [TestHttpClient](https://ratpack.io/manual/1.4.5/api/ratpack/test/http/TestHttpClient.html) in the groovy test area.
```groovy
def aut = new GroovyRatpackMainApplicationUnderTest()
@Delegate TestHttpClient client = aut.httpClient()

def "Check Site Index"() {
	when:
	get("index.html")

	then:
	response.statusCode == 200
	response.body.text.contains('<title>Ratpack: A toolkit for JVM web applications</title>')

}
```  
Example: [Ratpack Site](https://github.com/ratpack/ratpack/blob/master/ratpack-site/src/test/groovy/ratpack/site/SiteSmokeSpec.groovy)

---
## Functional - Geb

Geb can also be used we just need to do a little work to set up the correct base URL.
```groovy
private final static ServerBackedApplicationUnderTest applicationUnderTest = new LocalScriptApplicationUnderTest()

def setup() {
  URI base = applicationUnderTest.address
  browser.baseUrl = base.toString()
}
```
[Example](https://github.com/ratpack/ratpack/blob/master/ratpack-site/src/browserTest/groovy/ratpack/site/SiteBrowserSmokeSpec.groovy)

----
## Fat Jar

Uses ShadowJar

```
gradle shadowJar
```

---
## Run as a Fat Jar

We can also just run the jar quickly.

```
gradle runShadow
```
