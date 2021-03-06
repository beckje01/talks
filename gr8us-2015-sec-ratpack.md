## Agenda
  * whoami
  * Simple Handler Example
  * Force SSL
  * Ratpack Sessions
  * Pac4j and Ratpack
----
## $ whoami

Jeff Beck

Engineer at SmartThings

-note
Abstract

So you are all excited about this hot framework Ratpack now you are about ready to launch into production. But you need security, I will go over using pac4j with Ratpack to secure your application. Showing integrations with Twitter, Basic Auth, and others. I will also go over a case study of CellarHQ and their security in Ratpack.

----
## Ratpack Version Note

This talk and slides are targeted at Ratpack v0.9.19-SNAPSHOT

----
## Simple Handler

Specify a handler at the start of the chain that all request will go through.

```java
handlers{
  all {
    //All traffic hits this handler first
  }
}
```

-note
We can provide a simple handler at the start of the chain that intercepts all traffic and checks for security concerns.

--
## Simple Handler

```java
all {
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
 * Multiple Authentication Options

--
## When This Works

 * Stateless MicroService
 * Prototyping

----
 ## Force SSL

 ```java
 all {
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
--

![](images/aws-elb.png)

----
## Ratpack Sessions

The `Session` module provides the basics of sessions as well as a in memory data store.

```
compile ratpack.dependency('session')
```

--
## Session Module

The SessionModule will make sure every request is set up with a session. Also by default provide an in memory session data store.

```java
bindings {
  module SessionModule
}
```
--
## Interact with the Session
```
def session = context.get(Session)

//Get Session ID
String sessionId = session.id

//Terminate Session
session.terminate().then()
```
[Code Branch](https://github.com/beckje01/ratpack-gr8us-sec/tree/session)
-note

Terminate the session is an Operation so you need to call then() or then {}

----
## ClientSideSession

An encrypted cookie that stores session data.

Use the [ClientSideSessionModule](http://ratpack.io/manual/0.9.19/api/index.html?ratpack/session/clientside/ClientSideSessionModule.html)

```java
bindings {
  module(ClientSideSessionModule, { config ->
    config.setSessionCookieName("s1")
    config.setSecretToken("fakeToken")
  })
}
```

--
## Advantages

 * No extra infrastructure to support shared sessions.
 * Encrypted so the session data can't be messed with.

--
## Disadvantages

 * Max cookie size
 * More data transfer every request
 * Requires managing a key

----
## Providing a SessionStore

You can easily change out the session store by providing an implementation of the [SessionStore](http://ratpack.io/manual/0.9.19/api/index.html?ratpack/session/SessionStore.html) interface.

```java
protected void configure() {
  bind(SessionStore).to(YourSessionStore).in(Singleton);
}
```
-note
Example of a redis SessionStore or other in host needs.
----
## Pac4j

There is a Pac4j [class](http://ratpack.io/manual/0.9.19/api/index.html?ratpack/pac4j/RatpackPac4j.html) that ties Pac4j into Ratpack well providing some basics you can extend.

--
## What has changed

 * No more Authorizer
 * No more guice module

----
## BasicAuth Example

build.gradle
```java
compile ratpack.dependency('pac4j')
compile "org.pac4j:pac4j-http:1.7.0"
```
--

```java
handlers{
  all(RatpackPac4j.authenticator(
    new BasicAuthClient(
      new SimpleTestUsernamePasswordAuthenticator(),
      new UsernameProfileCreator())))

  prefix("auth"){
    //Require all requests past this point to have auth.
    all(RatpackPac4j.requireAuth(BasicAuthClient))
    get{
      render "An authenticated page."
    }
  }
}
```

----
## Twitter Auth Example

build.gradle
```java
compile ratpack.dependency('pac4j')
compile "org.pac4j:pac4j-oauth:1.7.0"
```

--

```java
handlers {
  all(RatpackPac4j.authenticator(new TwitterClient("key", "secret")))

  prefix("auth") {
    //Require all requests past this point to have auth.
    all(RatpackPac4j.requireAuth(TwitterClient))
    get {
      render "An authenticated page."
    }
  }
}
```
--
## Create a Twitter App

 * [Apps Twitter](https://apps.twitter.com)
 * [Twitter Example](https://github.com/beckje01/ratpack-gr8us-sec/tree/twitterAuth)

----
## With multiple clients

Setup all the clients you want.

```java
all(RatpackPac4j.authenticator(
  new BasicAuthClient(
    new SimpleTestUsernamePasswordAuthenticator(),
    new UsernameProfileCreator()),
  new TwitterClient("key", "secret")))
```
--

```java
prefix("auth") {
  //Require all requests past this point to have auth.
  all({ ctx ->
    RatpackPac4j.userProfile(ctx).then { opUp ->
      if (opUp.isPresent()) {
        ctx.next(single(opUp.get()))
      } else {
        ctx.redirect(302, "/login")
      }
    }
  })

  get { UserProfile userProfile ->
    render "An authenticated page. ${userProfile.getId()}"
  }
}
```
--
## Try Github

 * [GitHub Apps](https://github.com/settings/applications)
 * [GitHub Scopes](https://developer.github.com/v3/oauth/#scopes)

----
## Real World Use

CellarHQ [Open Source](https://github.com/CellarHQ/cellarhq.com)

_Note_ A few versions behind but will get updated.
