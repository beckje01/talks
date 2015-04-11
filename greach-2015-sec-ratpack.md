## Agenda
  * whoami
  * Simple Handler Example
  * Force SSL
  * Ratpack Sessions
  * Pac4j
  * BasicAuth Example
  * Twitter Auth Example
  * Case Study CellarHQ
  ** What is CellarHQ
  ** What is special about their auth
  ** Multi identity providers
  ** User decoration
  ** How decorator works
  ** Using the user inside Ratpack
----
## $ whoami

Jeff Beck

Engineer at SmartThings

-note
Abstract

So you are all excited about this hot framework Ratpack now you are about ready to launch into production. But you need security, I will go over using pac4j with Ratpack to secure your application. Showing integrations with Twitter, Basic Auth, and others. I will also go over a case study of CellarHQ and their security in Ratpack.

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
 ## Force SSL

 ```groovy
 handler {
   if (!checkForSSL) {
       redirect(301, request.rawUri)
   }
   next()
 }
 ```

 -note
 AWS Headers: http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/TerminologyandKeyConcepts.html#x-forwarded-headers

 --
 ## AWS Example

Using the ELB for SSL termination, we can easily detect if the request was made with https.

```
request.headers.get('X-Forwarded-Proto') != 'https'
```
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

[Session Example](https://github.com/beckje01/ratpack-greach-simple/blob/session/src/ratpack/Ratpack.groovy)

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

Provides an easy framework to work across many Java security libraries and authentication systems. Such as authenticating with Twitter.

[pac4j.org](http://www.pac4j.org/#1)

--
## Pac4j with Ratpack

There is a pac4j [module](http://ratpack.io/manual/current/api/index.html?ratpack/pac4j/package-summary.html) that ties pac4j into Ratpack well providing some basics you can extend.

--
## Ratpack Authorizer

[Authorizer](http://ratpack.io/manual/current/api/ratpack/pac4j/Authorizer.html) The interface that is used to determine if authentication required and deals with authorization.

You can extend [AbstractAuthorizer](http://ratpack.io/manual/current/api/ratpack/pac4j/AbstractAuthorizer.html) to just check if authentication is required and just not do authorization.

----
## BasicAuth Example

build.gradle
```groovy
compile ratpack.dependency('pac4j')
compile "org.pac4j:pac4j-http:1.6.0"
```
--

```groovy
bindings {
  add SessionModule
  add new MapSessionsModule(1000, 360)
  add new Pac4jModule(new BasicAuthClient(new DumbUsernamePasswordAuthenticator()), new SecureAllAuthorizer())
}

handlers {
  handler {
    render "Hello World"
  }
}
```
--

[This Example](https://github.com/beckje01/ratpack-greach-simple/tree/basicAuth)

 * [DumbUsernamePasswordAuthenticator](https://github.com/beckje01/ratpack-greach-simple/blob/basicAuth/src/main/groovy/DumbUsernamePasswordAuthenticator.groovy)
 * [SecureAllAuthorizer](https://github.com/beckje01/ratpack-greach-simple/blob/basicAuth/src/main/groovy/SecureAllAuthorizer.groovy)

----
## Twitter Auth Example

build.gradle
```groovy
compile ratpack.dependency('pac4j')
compile "org.pac4j:pac4j-oauth:1.6.0"
```

--

```groovy
ratpack {
	bindings {
		add SessionModule
		add new MapSessionsModule(1000, 360)
		add new Pac4jModule(new TwitterClient("key", "secret"), new SecureAllAuthorizer())
	}

	handlers {
		handler {
			render "Hello Twitter"
		}
	}
}
```
--
## Create a Twitter App

[Apps Twitter](https://apps.twitter.com)

[Twitter Example](https://github.com/beckje01/ratpack-greach-simple/tree/twitter)
----
## Case Study CellarHQ

A online list of your beer cellar.

--
## Why do we care

Written is Ratpack

--
## What do they do

 * Dual Twitter and Web Form Authentication
 * Complex Authorizer
 * Handler Decoration
 * Uses Roles

----
## Dual Twitter and Web Form

Supply their own AuthenticationModule instead of the Pac4j Module.

--
## Authorizer

Based around a white list for anonymous action

```groovy
final static List<String> ANONYMOUS_WHITELIST = [
        '',
        'about',
        /styles\/.*/,
        /images\/.*/,
        /scripts\/.*/,
        /pac4j.*/,
        'health-checks'
]

```
--

```
boolean isAuthenticationRequired(Context context) {
    return !matchesAnyPath(context.request.path, ANONYMOUS_WHITELIST)
}

private boolean matchesAnyPath(String subject, List<String> patterns) {
    return matchesAny(subject, patterns.collect { String pattern -> (String) "^${pattern}/?\$"})
}

private boolean matchesAny(String subject, List<String> patterns) {
    return patterns.any { String pattern -> subject.matches(pattern) }
}
```
--
## Authorization

```
void handleAuthorization(Context context, UserProfile userProfile) throws Exception {
    if (matchesAnyPath(context.request.path, ADMIN_ROLE_REQUIRED) && !userHasRole(userProfile, Role.ADMIN)) {
        context.redirect('/login?error=' + Messages.UNAUTHORIZED_ERROR)
        return
    }
    context.next()
}

private boolean userHasRole(UserProfile userProfile, Role role) {
    return userProfile.roles.contains(role.toString())
}
```

--
## Handler Decoration

CellarHQ uses [HandlerDecorator](http://ratpack.io/manual/current/api/index.html?ratpack/handling/HandlerDecorator.html) to apply their login handlers to the chain. The same method used by the Pac4j Module.

-note
This is why we don't have to put a login handler directly in the chain it is done for us.
