## Agenda
  * whoami
  * Simple Handler Example
  * Force SSL
  * Ratpack Sessions
  * Pac4j
  * BasicAuth Example
  * Twitter Auth Example
  * Case Study CellarHQ
----
## $ whoami

Jeff Beck

Engineer at SmartThings

-note
Abstract

So you are all excited about this hot framework Ratpack now you are about ready to launch into production. But you need security, I will go over using pac4j with Ratpack to secure your application. Showing integrations with Twitter, Basic Auth, and others. I will also go over a case study of CellarHQ and their security in Ratpack.

----
## Ratpack Version Note

This talk and slides are targeted at Ratpack v0.9.19

----
## Simple Handler

Specify a handler at the start of the chain that all request will go through.

```groovy
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

```groovy
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

 ```groovy
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
