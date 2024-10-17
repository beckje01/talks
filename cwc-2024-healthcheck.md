### Agenda

 * whoami
 * Definitions
 * Mental Model
 * Properties of a Healthcheck
 * Demo - What do they look like?
 * What are they good for?
 * Healthcheck Patterns

----
### $ whoami

Jeff

@beckje01

----
### Warning

I would consider this a work in progress. Please give flow and materials feedback.

Is it Health Check or Healthcheck?

----
### Definitions 

 * *Healthcheck* - Action that confirms some bit of functionality
 * *Heartbeat* - Message sent out
 * *Healthcheck Endpoint* - An where you read the state of healthchecks.

--
### Heartbeat vs Healthcheck?

 * Heartbeat is pushed out from a μService
 * Healthcheck is something you read from a μService
 * Heartbeats are usecase specific

-note
A heartbeat is where the microservice calls out or pushes data / messages out of a known  channel many times it can also be used to keep a long lived connection working. Kept in a thread it doesn't typically expose how the service is doing just the simple fact that it is still doing stuff.

I've always found it better to treat a heartbeat as outbound and a healthcheck endpoint an inbound read.

 * Payload of a heartbeat may include protected information.

--
### Deep vs Shallow Healthcheck

----
### Mental Model

The healthcheck represent a single instance. Not the health of your whole application. 

<img src="images/healthcheck/hc-single-bad.png" height="500"/>

--
### Mental Model

A set of nodes with a shared dependency can easily cause a group issue.

<img src="images/healthcheck/hc-region-bad.png" height="500"/>

--
### Mental Model

A single shared dependency can lead to full failure.

<img src="images/healthcheck/hc-many-bad.png" height="500"/>

----
### Properties of a Healthcheck

* Checks a single condition
* Reflects the state of the health when possible
* No Side Effects
* Represents a value for a given service

----
### Properties of a Healthcheck Endpoint

* Unauthenticated 
* Must not generate load with repeated calls.
* 