## Agenda

* Intro
* Lab01 - Logging
* Lab02 - Tracing Terminology
* Lab02 - Tracing
* Lab03 - Metrics 
* Lab04 - Observability Bugs

-note
Abstract
This workshop will be a hands on exploration of observability. There will be a microservice architecture system that we work to debug and add needed insights to. We will have a mix of services written in Ratpack, Micronaut, and Grails all interacting over a mix of methodologies; the setup will model an evolving system maintained by a number of teams. Work with logging techniques such as dynamic log levels, correlation IDs and formatting tricks. We will add distributed tracing to services, then actually debug issues. Metrics will also be covered. I’ll even introduce a few scenarios of observability gone wrong. Most of this workshop will be hands on the keyboard.

----
## $ whoami

Jeff Beck

Software at SmartThings

----
## How this Workshop is Setup

Labs are in a state where they will compile but not all are 100% correct. The answers are in the corresponding modules.

---
## Infrastructure

All the infrastructure needed is setup as a docker-compose you can run in the `infra` folder.

---
## Task List

There is a high level task list in each lab, that has a rough order on the things to explore.

---
## Wrap Up

We will do a wrap up discussion after each lab talking about more complex real world applications of the topics.

----
## What is Observability

The property of systems that allows operators to clearly understand the state of the system.

----
## Lab 01 - Logging

---
## Commands

From infra dir:
```bash
docker-compose up 
```

---
## Apps

Grails from the dir, `user-grails`

```
./gradlew boRu
```

Others from their project directory
```
./gradlew run
```

----
## Lab 01 - Wrap Up

* Correlation IDs are lightweight, good for small retrofit
* Dynamic Logging is for cost savings
* Formatting Matters

----
## Lab 02 - Tracing


----
## What is Distributed Tracing

Distributed tracing systems collect end-to-end latency graphs
(traces) in near real-time.

* [Zipkin](https://zipkin.io/)
* [Jaeger](https://github.com/jaegertracing/jaeger)
* [Dapper](https://research.google.com/pubs/pub36356.html)

----
## Terminology Lesson

* **Span** - An operation that took place.
* **Event** - Something that occurs in a span.
* **Tag** - Key value pair on a span.

---

* **Trace** - End-to-end latency graph, made up of spans.
* **Tracer** - Library that records spans and passes context
* **Instrumentation** - Use of a tracer to record tasks.
* **Sample %** - How often to record a trace.

----
## Lab 02 -- Tracing

Same apps just add tracing.

----
## Lab 02 -- Wrap Up

* Customization is Key
* Service Mesh 
* When to use annotations?
* When to use tags?

----
## Lab 03 -- Metrics

----
## Lab 03 -- Discussion

* What metrics do you collect today?
* How do metrics lie to you?
* How do your metrics tie to users?

----
## Lab 04 -- Observability Bugs

----
## Lab 04 -- Error Stories

* Traces with > 10k spans
* Error rates thrown off by service reporting the wrong name.
* Lost traces
* Broken Traces
* Fixed correlation IDs

----
## Observability Discussion

* Data is step 1
* Actionable data is step 2
* Pair all the tools for maximum effect.

----
## Questions

----
## We are Hiring

http://bit.ly/SmartThingsJobs
