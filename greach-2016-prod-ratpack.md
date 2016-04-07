## Agenda

 * whoami
 * Our Usage
 * Organizing Your App
 * When reusable handlers
 * Sharing a set of functionality
 * Monitoring Ratpack
 * Adapting Libs
 * Etc


-note
Abstract

You’ve seen all the getting started talks, even little bits about securing your Ratpack application. Now what? We will go over some best practices for organizing your applications. How to build libraries to easily share across different projects. We will also get into monitoring your Ratpack application. Lastly we will work through some things to think about when adapting both blocking libraries and non-blocking libraries with Ratpack.

----
## $ whoami

Jeff Beck

Software Architect at SmartThings

----
## Our Usage

 * 5 in production
 * 2+ being built now
 * 4 different teams working on services

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
--
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

_Ratpack 1.3 Feature_

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
--
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

--
## Differences from Chain

  * Allows Bindings
  * Merges the handlers block
    * Chain method can bind to a path include doesn't
    * Order with include vs handlers{} matters  
<br>
See [Ratpack Include Example](https://github.com/beckje01/greach-ratpack-include-example) and [API Doc](https://ratpack.io/manual/1.3.0/api/ratpack/groovy/Groovy.Ratpack.html#include-java.lang.String-)

----
## What Goes in a Handler

 * Parsing
 * Async calls to do the business logic
 * Call to render in error cases and complete cases

----
## Ratpack Service

 * Implements the [Service](https://ratpack.io/manual/1.2.0/api/ratpack/server/Service.html) interface.
 * Notified of Starting and Stopping
 * Added to the registry, commonly in the bindings block

--
## Service Ordering

 * Services ordered based on how they come out of the registry
 * Coming in Ratpack 1.3 [DependsOn](https://ratpack.io/manual/1.3.0/api/ratpack/service/DependsOn.html) and [ServiceDependencies](https://ratpack.io/manual/1.3.0/api/ratpack/service/ServiceDependencies.html)
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
## Sharing a set of functionality

 * Just JARs share via Bintray / Artifactory
 * Use lowest version of Ratpack possible
 * Separate service interfaces into different artifacts
 * [HandlerDecorator](https://ratpack.io/manual/1.2.0/api/ratpack/handling/HandlerDecorator.html) for common routes

-note
Service interfaces example Rx vs Ratpack Promises ala Cassandra

--
## HttpModule Example

 * Provides known endpoints for all our Ratpack apps
   * Healthcheck route
   * Build info route
 * Provides custom rendering of the metrics modules healthcheck

--
## HandlerDecorator

```groovy
public class HealthHandlerDecorator implements HandlerDecorator {
  @Override
  public Handler decorate(Registry serverRegistry, Handler rest) throws Exception {
    return Handlers.chain(Handlers.chain(serverRegistry, { c ->
      c.get("buildinformation", BuildMetadataHandler)
      c.get("healthcheck", HealthCheckHandler)
    }), rest);
  }
}
```

-note

Adds the two routes at the start of the handlers this is done in our case to make sure they are done before anything else can respond to requests. We always want these 2 available.


----
# Monitoring Ratpack

 * HealthCheck
 * DropwizardMetricsModule

----
## HealthCheck

Inspired by Metrics HealthCheck but async focused.

```groovy
@Override
public String getName() {
  return "SimpleCheck";
}

@Override
public Promise<Result> check(Registry registry) throws Exception {
  //Do the check
}
```

-note

All healthchecks just need to be in the registry then HealthCheckHandler will use them. It is an active check when the handler is called.

Great for checking datastores or reporting out the state of known dependencies such as open cricuitbreakers etc.

----
## DropwizardMetrics Module

 * Will time all requests once turned on
 * New timer for every different request by default
 * Lots of available reporters

--
## Example config

```yaml
metrics:
  jvmMetrics: true
  jmx:
    enabled: true
  requestMetricGroups:
    hubShard: "hubs/.*/shard"
    hubClaim: "hubs/.*/claim"
    hub: "hubs/.*"
    destination: "locations/destination"
    locationsForUser: "locations"
    locationUser: "locations/.*/user/.*"
    location: "locations/.*"
    other: .*
```
-note

The other is key to make sure you don't leak random timers

----
## Adapting Libs

 * Async / Non Blocking Libs
 * Synchronous Libraries

----
## Async

 * ListenableFuture
 * CompletableFuture
 * CompletionStage

-note
Easy to adapt any of these to Ratpack's execution model

--
## Promise.of

```groovy
return Promise.of(upstream -> {
  //Cassandra Java Driver runs ListenableFuture
  ResultSetFuture resultSetFuture = session.executeAsync(statement);
  upstream.accept(resultSetFuture);
});
```

--
## More Complicated Promise.of

Redis Lettuce Example:

```groovy
return Promise.<Boolean>of(d ->
  connection.set(sessionId, sessionData).handleAsync({ value, failure ->
    if (failure == null) {
      if (value != null && value.equalsIgnoreCase("OK")) {
        d.success(true);
      } else {
        d.error(new RuntimeException("Failed to set session data"))
      }
    } else {
      d.error(new RuntimeException("Failed to set session data.", failure))
    }
    return null
  }, Execution.current().getEventLoop())
)
```
-note
Mark what happens as failure or success lettuce will return a failure not as an exception so we want to send it as an error in the
--
## Sharing EventLoopGroup

If your library uses Netty you can actually share the EventLoopGroup with Ratpack.

In Cassandra Java Driver it looks like:
```groovy
@Override
public EventLoopGroup eventLoopGroup(ThreadFactory threadFactory) {
  return Execution.current().getController().getEventLoopGroup()
}

@Override
public void onClusterClose(EventLoopGroup eventLoopGroup) {
  //Noop let Ratpack deal with the loop shutdown.
}
```
-note
Be careful here don't let the library manage stopping the event loop group

----
## Synchronous Libs

We need to use the [Blocking](https://ratpack.io/manual/1.2.0/api/ratpack/exec/Blocking.html#get-ratpack.func.Factory-) to make sure the work is not done in the main threads.

```groovy
def aPromise = Blocking.get({ ->
  datastore.doSomeLongQuery()
})
```

----
## Etc

 * Run with both `run` and `runShadow`
 * `gradle -t check` & `gradle -t run`
 * Caching promises not just values

--
## Cache Promises
```groovy
promiseOAuthToken = upstreamValidator.validate(token);

//Make sure we only call the validate on the upstream once
promiseOAuthToken = promiseOAuthToken.cache();

promiseOAuthToken.then(optionalOAuth -> {
  //Promise.value of the returned value actually
  //Storing the promise in the cache means we only build that promise once
  cache.put(token, Promise.value(optionalOAuth));
});
```

--
## RxRatpack
```groovy
ratpack {
  RxRatpack.initialize()
  serverConfig {
    port System.getProperty("ratpack.port", "8181") as int
  }
  bindings {
    //...
  }

  //...
}
```
