## Agenda

 * What Happened to Us
 * How we got out of it
 * Things we Learned
 * Ops Acrobatics
 * Where to get Help
 * Things Developers Need to Know
 * Odds and Ends

----
## What Happened to Us

 * Cluster was running hot.
 * Adding nodes failed due due to running out of disk
 * During the process 2.0 nodes stopped being able to be rebooted due to 2.1
 * Old version of Java Driver 2.1.4 had issues.

-note

Our cluster had been running hot almost all yellow nodes for months with a few nodes fluttering red in OpsCenter. We started to run low on diskspace on the hotter nodes due to compactions not keeping up. We use leveled compaction for some TTLed data so we depend on the compactions to clean up our dead data. We tried unsuccessfully to add nodes to the cluster in hopes of spreading out the load in general allowing compactions to catch up. We also unthrottled compactions during the night on some of the nodes backing up with little effect. The new nodes could never join always filling their disk before joining fully it seemed to be related to the streams coming in from the nodes very backed up on compactions.

Next we decieded to move to bigger nodes to give more CPU and disks in an attempt to catch up on compactions. Unfortunatlly this new bigger node ended up deleted by the ASG health check. At this point we lost a second node due to running out of disk. We were able to bring that node back online by removing snapshots and clearing up some root owned spaced that wasn't needed. Then we noticed a node had it's commit log going to almost 100gigs. Reviewing the logs we found it was not able to flush correctly due to a bug in an earlier version of 2.0.x we restarted that node but were not able to get it to boot with that much in the commit logs. We then turned off all nonessital features to shed load, then we moved our event traffic to another cluster. We then were able to remove the data that was the biggest problem to compact, allow the cluster to go back to green. But these changes exposed us to a known issue in our java driver, once we upgrade that library our error rate dropped. We then started the process of removing the down nodes. After that process completed we kicked off repairs for the cluster and are now in a better state.

----
## How we got out of it

 * Deleted Commit Logs and restarted a node
 * Shed load of non critical features
 * Split off isolated feature
 * Removed dead nodes

----
## Things Learned

 * Don't run mixed clusters for a long period
 * Single version of Cassandra everywhere
 * Monitor compactions
 * Read the logs
 * Use Java 8
 * Be proactive


----
## Ops Acrobatics

 * AutoJoin false
 * rsync directories to a new server
 * Remove node is faster than doing repairs

--
## AutoJoin False

 * Still serves traffic as a coordinator node
 * Can be repaired if it's been offline for a bit

--
## rsync to new server

 If you have the sstables and commit logs you can simply rsync to a new bigger server.

--
## BUT

If you are using the AWS Snitch, you have to be in the same availability zone and region otherwise its a topology change and the node won't be able to take over from where the data left off.

--
## Remove node is faster

While recovering once we shed enough load it was easier and faster for us to just remove the down nodes dropping the cluster size.

----
## Where to get Help

 * IRC Freenode
 * Consultants

--
## IRC

 * Get a ZNC Bouncer setup
 * Textual on Macs is great
 * Have a thick skin

--
## Consulatants

 * Datastax
 * The Last Pickle
 * Pythian

----
## Things Developers Need to Know

 * Ability to turn off features to shed load is great
 * nodetool access is critical for debugging
 * Tune your compations
 * Be able to turn off features depending on particular C* releases
 * How the driver is changing over time

--
## nodetool

Understanding how the system is being used is key to make adjustments to datamodels or other application issues.

--
## Tune your compations

There are lots of settings for the compactions if you are falling behind you may need to tune. For us we adjusted the unchecked compactions, `tombstone_compaction_interval`, and `sstable_size_in_mb`.

--
## Turn off features for C* releases

With 2.1 counters got an overhaul but they was we were using them caused a very different load profile being able to shut down this feature while we addressed the new load profile was very helpful.

--
## How the driver is changing over time

 * What bug fixes went in.
 * New features to help ie: SpeculativeExecution

----
## Odds and Ends

 * C* loves patches
 * Can't repair a node while it's sibling is down
 * dstat
 * az rebalancing needs to be turned off
 * ssd tuning
 * nodetool is mostly per node

--
## PATCHES

The C* community is all about patches, don't be surprised to be sent a git patch file. Be willing / able to run custom C* versions.

--
## dstat
Great tool for monitoring the server.

```bash
dstat -lvrn 10
```

--
## nodetool is per node

Example

```
nodetool setcompactionthroughput
```
Sets the throughput capacity for compaction in the *system*, or disables throttling.
