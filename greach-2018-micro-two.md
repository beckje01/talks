## Agenda

* whoami
* Introductions
* Our Service
* Basic Project Setup
* RDBMS
* Parsing
* Rendering
* Testing
* Caching
* Current Clear Easier Paths
* Questions


-note
Abstract

We will build out the same microservice in both Grails and Ratpack. See how they each framework compares. There will not be a winner but a show of strengths and differences of both frameworks. We will cover RDBMS interactions, NoSQL interactions, caching, json marshaling, and testing.

At the end of this talk you will have a better understanding of how these frameworks relate to each other and be able to use these differences to pick the right tool for the job.

----
## $ whoami

Jeff Beck

Software Architect at SmartThings

----
## Introductions - Grails

  * Convention over configuration
  * Spring Boot Based
  * Servlet Oriented
  * GORM

---
## What This Means

  * Standard Project Layout
  * Powerful configuration tooling
  * Easy to approach programing paradigm
  * Spring Boot interoperability
  * The power of GORM

-note

The Layout is easy to deal with giving good separation between control and services.  

Tied with boot you can easily build out new features as needed.

GORM is the easiest way to deal with ORMs, great way of modeling and getting these items for free.

----
## Introductions - Ratpack

  * Minimal Framework
  * Netty Based
  * Ratpacks Execution Model

---
## What This Means

  * Ratpack Stays Unopinionated
  * Use Guice or Spring for DI
  * Forces Async Non Blocking
  * Ratpack wants to manage execution
  * Netty under the covers

-note

While ratpack is Unopinionated you have to layout your code someway this can lead to different projects being setup very differently.

Ex: One project using packages for all a feature while another groups services etc.

Picking a DI while great most just use Guice out of the box or pull in spring.

You will need to adapt many libraries to ratpack's execution model this is can be easy but is more then just using the libs.

Netty is a well known very performant library allowing for very high performance

----
## Our Service

Hub Service

  * Responsible for tracking a hub
    * Is is claimed?
    * Where is it connected?
  * Knows about manufacturing in some cases

-note

Why this is important it will get data from multiple source sent in.

----
## Basic Setup

  * Gradle to Manage Both
  * Setup as a Multi Project Build
  * Show the Hello World

-note
Using the grails default profile

---
## Next Step Add Hub Stats Stub

Lets add a simple endpoint stubbed out with some JSON data.

Serving the traffic on `/hubs/stats`

-note

Git tag `hubstats`

---
## URL Mapping vs Handlers

  * Grails has URL Mappings
    * `url-mappings-report` Very Powerful
    * Paths defined at startup even with dynamic mappings
  * Ratpack uses Handlers and a Chain
    * Routing is dependent on the handlers
    * No report of paths available
    * Allows mixing some routing logic into the handler on the return logic

----
## RDBMS

We need to store data in our service. So starting with a RDBMS is a great idea. This is where you will see some major differences between Grails and Ratpack

---
## RDBMS - Grails

GORM support of domains allows quick easy setup with a full set of functionality. With hibernate backing GORM in this use case we have a solid system. With some pretty easy choices to make.

---
## Grails Domains

Domains are built into Grails meaning you get lots of great functionality right from the framework add on to everything from GORM. One of my favorite examples is `@Resource` when applied to a domain class.

---
## RDBMS - Ratpack

Ratpack has modules for Hikari, H2, and JDBC Transactions. But these are just raw components no ORM quick bootstrapping. So you have raw materials you can build with but the choice is mostly up to the developer.

---
## Ratpack Quick RDBMS

Many examples and projects use just straight SQL interactions with the DBs, this is lightweight for many users and gives developers full control over exactly what is happening in the SQL.

---
## Ratpack More Options

  * GORM - Yes you can use GORM with Ratpack
  * [jOOQ](https://danhyun.github.io/2016-jeeconf-rapid-ratpack-java/#database) DB Code Generation
  * [Liquibase](https://github.com/SmartThingsOSS/ratpack-liquibase) Provided as an external module

----
## Note about Read Only DBs

If you are using RDS or something similar. Make sure you understand how you will make use of Read Replicas, many times the easiest way is to create ReadOnly transactions and allow the DB driver to route the traffic for you.

----
## Parsing / DataBinding

Much of what we need to do is parse the incoming data into objects.

This is another area where there are some interesting differences between Ratpack and Grails.

---
## DataBinding - Grails

Grails will take all the data flowing into a request and put it into `params` this then can go through a DataBinding process to get into Command Objects or Controller arguments.

-note
So the concern of getting the data into the correct format for an object such as doing a lookup etc is done in the binding process and you can do things with annotations or value converters

---
## DataBinding - Grails

If you want to parse an unsupported mime type you can do that by implementing DataBindingSourceCreator. See the [guide](http://docs.grails.org/latest/guide/theWebLayer.html#dataBinding) for more details.

---
## Parsing - Ratpack

In Ratpack you must call for the body to be parsed in which case you will be a Promise to the type you ask for. This works similar to Grails but operates only on the body out of the box.

---
## Parsing Custom

If you want to create your own parser that is as simple as implementing the `Parser` interface. The major difference from Grails is a parser is tried and expected to return null if it can't support the media type. Parsers are not registered to a mimetype that is left up to the implementation.

----
## Adding Hub CRUD

  Bring together the RDBMS interactions and Parsing to create a basic CRUD interface for hubs.

-note

git tag `crud`


----
## Rendering

Actually outputting the data to the consumer is the job of rendering. Grails has Views such as the JSON View that is part of the REST Profile. While Ratpack has renders which are expected to handle any format for a given type.

---
## Rendering Just IDs

Lets just render IDs for our Hubs now.

-note

Show ratpack not support xml

git tag `idonly`

----
## Testing

While you can use Spock and Geb with both frameworks how you think about testing varies greatly.

---
## Grails Test Phases

Grails has a number of test phases that come with it out of the box, unit, integration, functional. While Ratpack doesn't have any special phases just what comes from Gradle.

---
## Grails Testing Focus

Unit tests depend on mixins for testing services and controllers. These will only have access to the mocks and things provided.

---
## Grails Testing Focus

Integration tests load up the application context so everything is wired and ready to go.

---
## Grails Testing Focus

Functional tests the whole app is running and listening to http requests.

---
## Grails Testing

Tests tend to be mostly unit based on the time to run the tests. Running the whole application tends to take a noticeable amount of time.

---
## Ratpack Testing

Ratpack testing tends to focus on lots of running apps. Its very easy to put just the part you are testing in an embedded app and test it directly.

---
## Ratpack Testing

Ratpack offers an ApplicationUnderTest idea that sets up the app and allows overriding registry elements via impositions. Ratpack apps start so fast it makes the most sense to just start up and teardown these apps with ever test.

---
## Ratpack Testing

When testing code interacting with executions you need to use helpers like `ExecHarness`. Or while working with handlers you can test with the `RequestFixture`. These are nice to test in a very isolated way but compared to setting up an Embedded app it tends to be harder most of the time.

----
## Caching

Caching while very important for production systems both frameworks have limited built in support.

---
## Caching Grails

Grails does offer a cache plugin that can cache your method calls via an annotation. So as long as your code is structured to take advantage of this it can be very powerful.

[Cache Plugin](http://grails-plugins.github.io/grails-cache/)

---
## Caching Ratpack

Generally caching tends to be done via direct control but you don't actually have to cache the values in Ratpack you can cache the promise itself saving overhead. This works very well with local in memory caches like Caffeine.

---
## Caching Ratpack Caveat

Make sure to call [`.cache()`](https://ratpack.io/manual/current/api/ratpack/exec/Promise.html#cache--) on the promise to insure you only execute the promise once. Otherwise you will redo work that isn't needed.

----
## Current Clear Easier Paths

  * Ratpack and Queues - nothing out of the box and harder to adapt to the execution model
  * Grails and 100% NonBlocking - If you want to do everything non blocking async you can with Grails but it won't force it like Ratpack will
  * ORM with Mongo - Use Grails and GORM lots of nice features there

----
## Questions
