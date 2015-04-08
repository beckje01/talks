## Agenda

 * whoami
 * What is Cassandra
  * When Cassandra
  * When Not Cassandra
 * Data Modeling Overview
  * span out on write not read
  * focus on query first modeling
 * Thrift vs CQL
 * What is available for Grails
  * Cassandra ORM
  * Cassandra GORM
  * Astyanax
  * Java Native Driver
//TODO more

-note
Abstract

As our applications grow and need to deal with new big data challenges and global distribution we look to new data stores designed to deal with these new challenges. Cassandra is one great tool to deal with both of these problems, we will go over some of the different ways of dealing with Cassandra from Grails. I will cover the various Cassandra plugins for grails, including the Cassandra ORM, Astyanax, and Cassandra GORM. Also, I will talk about using the Cassandra Java driver directly.

By the end of the talk you should have a overview of the data modeling that will working well in Cassandra paired with Grails, when to use or not use each plugin, and some of the basic connection configuration details needed.

----
## $ whoami

Jeff Beck

Engineer at SmartThings

-note
I've been doing Cassandra for the last 3 years, production around 2 - 2 and a half years.

----
## What is Cassandra

  * Highly-Available, Distributed, Tuned Consistency
  * Master-less Replication
  * Redundancy Configurable
  * Cluster can span DCs

--
## When Cassandra

 //TODO

--
## When Not Cassandra

 * When you depend on transactions
 //TODO more

-note

You probably don't need transactions but its a different mindset check out Steve's talks about event sourcing.

----
## Data Modeling

It's hard you are going to make mistakes.

-note

No really you are going to do things wrong for a while, I still do.
