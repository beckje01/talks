## Agenda
  * whoami
  * Force SSL
  * Simple Handler Example
  * Ratpack Sessions
  * Pac4j
  ** Intro to Pac4J
  ** How it works with ratpak
  * Twitter Auth pac4j Example
  * Basic Auth pac4j Example
  * Case Study CellarHQ
  ** What is CellarHQ
  ** What is special about their auth
  ** Multi identity providers
  ** User decoration
  ** How decorator works
  ** Using the user inside Ratpack

-note
Full Agenda

* whoami
* Simple Handler Auth Example _first handler in chain_
** How it works
** example of it working for token auth
** Why this isn't good enough
** When it can be used _forcing SSL_
* Pac4j
** Intro to Pac4J
** How it works with ratpak
* Twitter Auth pac4j Example
* Basic Auth pac4j Example
* Case Study CellarHQ
** What is CellarHQ
** What is special about their auth
** Multi identity providers
** User decoration
** How decorator works
** Using the user inside ratpack
----
## $ whoami

Jeff Beck

Engineer at SmartThings

-note
Abstract

So you are all excited about this hot framework Ratpack now you are about ready to launch into production. But you need security, I will go over using pac4j with Ratpack to secure your application. Showing integrations with Twitter, Basic Auth, and others. I will also go over a case study of CellarHQ and their security in Ratpack.

----
## Force SSL

//TODO

-note
AWS Headers: http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/TerminologyandKeyConcepts.html#x-forwarded-headers

--
## AWS Example

----
## Simple Handler

Specify a handler at the start of the chain that all request will go through.

```groovy
handlers{
  handler {
    //All traffic hits this handler frist
  }
}
```

-note
We can provide a simple handler at the start of the chain that intercepts all traffic and checks for security concerns.

--
## Simple Handler

```groovy
handler {
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

-note
Here we are actually checking a token now its a simple hard coded value but you could at this point easily do a check against a DB or other datastore.

--
## When This Falls Short

 * Identities
 * Roles / Authorization
 * Mixed Secure / Insecure Content
 * Multipul Authentication Options

--
## When This Works

 * Stateless MicroService
 * Prototyping

----
## Ratpack Sessions

The `Session` module provides the basics of sessions as well as a session scoped data store.

```
compile ratpack.dependency('session')
```

--
## Session Module

The SessionModule will make sure every request is set up with a session. The [MapSessionModule](http://ratpack.io/manual/current/api/index.html?ratpack/session/store/MapSessionsModule.html) provides an in memory map of sessions and data.

```groovy
bindings {
  add SessionModule
  add new MapSessionsModule(1000, 360)
}
```
-note
MapSessionsModule(int maxEntries, int idleTimeoutMinutes)

--
## Interact with the Session
```
def session = context.get(Session)

//Get Session ID
String sessionId = session.existingId

//Terminate Session
session.terminate()
```
--
## Interact with Session Data

```groovy
//Put Data
def sessionStorage = context.request.get(SessionStorage)
sessionStorage.put("example", "Galaxy")

//Get Data
sessionStorage.get("example")
```

--
## Future of Ratpack Sessions

The `Session` module will most likely be changing before the 1.0 release. Moving to promises, something like:

```java
public interface SessionStorage {

  Promise<Optional<String>> get(String key);

  Promise<Boolean> set(String key, String value);

  Promise<Integer> remove(String key);

  Promise<Integer> clear();

}
```
-note
We are doing this so session stores can be persistent/expensive stores EX DB, Cassandra, Memcached
See: https://github.com/ratpack/ratpack/issues/447

----
## Pac4j

-note
* Pac4j
** Intro to Pac4J
** How it works with ratpak
* Twitter Auth pac4j Example
* Basic Auth pac4j Example
