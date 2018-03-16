## Agenda

* whoami
* What is Distributed Tracing
** ? What is slueth brave zipkin vs tracer
** ?What is open cenesus
* What is Zipkin
** Show off some screen of ours
* Terminology Lesson
** span traceid, traces, trace % etc, tracer
* How Data flows between services
* How data gets to zipkin
** all the options
** what we do and why
* App Setup
** Ratpack install
** Grails install
* Debug Example
** Where request failed
** where request is slow
** doing extra db work
* Mobile Clients 
* External Client
** Dangers of bad traceids
* How often to sample





-note
Abstract
As the complexity of our systems grow so must our tooling, Zipkin is a great distributed tracing tool that helps deal with tracing in microservices. But once we have to tool in place how do we actually fix problems in our system? We have been using zipkin in production at SmartThings so I’m going to share our insights.

In this talk we will review how to get quickly get zipkin tracing added to Ratpack and Grails then actually go through debugging some issues. We will also cover strategies for interactions with mobile / external clients and how to figure out an amount to sample at.

No zipkin experience required.



Random links
Trace with kafka partion etc https://tracing.smartthingstlm.com/zipkin/traces/a4d0696ac0233fc7
Auth with cassandra details etc https://tracing.smartthingstlm.com/zipkin/traces/2740418edded39d0
Trace of an event push https://tracing.smartthingstlm.com/zipkin/traces/8dafe05624989391

lots of oauth dance https://tracing.smartthingstlm.com/zipkin/traces/74a1370c94b6fd5a

Things to search on:
http.path=/v2/license/security/authorizeToken
kafka.partition=70


https://github.com/hyleung/ratpack-zipkin/tree/master


To add stuff to spans old way
span.tag("oauth.param.principal", principal.name)
new way SpanCustomizer
https://github.com/openzipkin/brave/blob/master/brave/src/main/java/brave/SpanCustomizer.java

good reading 
https://research.google.com/pubs/pub36356.html
https://zipkin.io/pages/architecture.html

https://cloud.spring.io/spring-cloud-sleuth/
http://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.0.M8/



docker run -d -p 9411:9411 openzipkin/zipkin
http://192.168.10.2:9411/zipkin/

```
spanCustomizer.tag("user.username", user.getUserName());

```

span customerizer provided by ServerTracingModule.java in ratpack via guice

----
## $ whoami

Jeff Beck

Software Architect at SmartThings

----
## What is Distributed Tracing



** ? What is slueth brave zipkin vs tracer
