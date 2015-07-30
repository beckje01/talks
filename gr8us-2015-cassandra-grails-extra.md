## Tools

 * [CCM](https://github.com/pcmanus/ccm)
 * [DevCenter](http://www.datastax.com/what-we-offer/products-services/devcenter)
 * [Pillar](https://github.com/comeara/pillar)
 
----
## Glossary

  * [CAP Theorem](http://en.wikipedia.org/wiki/CAP_theorem) - Consistency, Availability, and Partition tolerance
  * Cluster, DataCenter, Rack
  * Partitioner
  * Thrift
  * Quorum, Local Quorum

--
## Glossary

  * DSE
  * Node Discovery
  * Load Balancing
  * [Replication Strategy](http://www.datastax.com/documentation/cassandra/2.0/cassandra/architecture/architectureDataDistributeReplication_c.html)
  * [Snitches](http://www.datastax.com/documentation/cassandra/2.0/cassandra/architecture/architectureSnitchesAbout_c.html)
----
## Interesting Cassandra Modeling

```groovy
session.prepare("""
   BEGIN BATCH
     INSERT INTO ${keyspace}.oauth_access_token
        (oauth_token, access_token, authentication, last_modified)
        VALUES (?, ?, ?, ?);
     INSERT INTO ${keyspace}.oauth_access_token_by_refresh
        (refresh_token, access_token, authentication, last_modified)
        VALUES (?, ?, ?, ?);
     INSERT INTO ${keyspace}.oauth_access_token_by_authentication
        (authentication_id, access_token, authentication, last_modified)
        VALUES (?, ?, ?, ?);
     INSERT INTO ${keyspace}.oauth_access_token_by_client
        (client_id, client_mod, user_id, access_token, authentication, last_modified)
        VALUES (?, ?, ?, ?, ?, ?);
   APPLY BATCH;
   """).setConsistencyLevel(ConsistencyLevel.ONE)
```

--
## Client Mod

Create a Mod for every client that goes to a known set of buckets

```groovy
Hashing.consistentHash(Hashing.murmur3_32()
    .hashString(userName, Charsets.UTF_8), 10)
```
