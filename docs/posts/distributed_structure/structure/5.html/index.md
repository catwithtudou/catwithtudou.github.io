# MIT6824LEC3笔记:GFS


# GFS

> [课程地址](https://pdos.csail.mit.edu/6.824/schedule.html)

## Before gfs

### why is distributed storage hard

High performance -> shard data over many servers

Many servers -> constant faults

Fault tolerance -> replication

replication -> potential inconsistencies

Better consistency -> low performance

it can sum up the all thinking points like:


- performance
- fault tolerance
- replication
- consistency

these points have connection and influence each other.

### what would we like for consistency?

There are two conditions we can think:

- single server

C1: wx1

C2: wx2

C3: 		 rx?

C4:					rx?

C3 or C4 can see either 1 or 2, but both have to see the same value.

It's a strong consistency model in the single server, but it's has poor fault-tolerance.

So we can think the replication for the fault-tolerance, but it will make strong consistency tricky.

- Replication servers:S1 S2

C1: write S1 or S2

C2: write S1 or S2

C3: read S1
C4: read S2

C3 or C4 will see the different results. It can't promise the same result or special result, because there is no standard protocol.

So that's not strong consistency. For consistency, we need to use the special protocol to fix these problems.

The gfs is one case about the distributed system's protocol.

## Gfs

### Context

Many Google services needed a big fast unified storage system, like Mapreduce, crawler/indexer, log storage/analysis, Youtube...

- Big: large data set

- Fast: automatic sharding
    - For parallel performance
    - To increase space available

- Global(over a single data center)
    - any client can read any file
    - Allows sharing of data among applications

- Fault-tolerance
    -  Automatic recovery from failures
    -  Just one data center per deployment
    -  Just Google applications/users

- Aimed at sequential access to huge files; read or append
    - not a low-latency DB for small items

> What was new about this in 2003? How did they get an SOSP paper accepted?
>
> - Not the basic ideas of distribution, sharding, fault-tolerance.
> - Huge scale.
> - Used in industry, real-world experience.
> - Successful use of weak consistency.
> - Successful use of single master.

### Design

- clients (library, RPC -- but not visible as a UNIX FS)
- each file split into independent 64 MB chunks
- chunk servers, each chunk replicated on 3
- every file's chunks are spread over the chunk servers
    - for parallel read/write (e.g. MapReduce), and to allow huge files
- single master, and master replicas
- division of work: master deals w/ naming, chunkservers w/ data

![image-20210515112556187](https://img.zhengyua.cn/20210515112556.png)

### Master state

- in RAM (for speed, must be smallish)

file name / index -> array of chunk handles

Chunk handle(stable)

-> version(stable)

-> list of chunk servers(volatile)

-> primary, secondaries(volatile)

-> lease time(volatile)

- on disk: stable storage

log ->

master will first write the log.

when master fails or crashes, the log can reconsturct or fix the master for the backup.

Checkpoint ->

when master fails or crashes, it can replay the last part or state basically then can quickly reconsturct master from the log.

> what state does need to end up in stable storage for the mass detection function coreectly?

### steps when read a file

1. C sends filename and offset to master M (if not cached)

2. M finds chunk handle for that offset

3. M replies with list of chunkservers
   only those with latest version

4. C caches handle + chunkserver list

   -> caches is to reduce the load on this single machine

5. C sends request to nearest chunkserver
   chunk handle, offset

   -> nearest chunkserver is to minimize network traffific or maximize the throughput about the joint setup form the clients to the servers

6. chunk server check the reversion then reads from chunk file on disk, returns

   -> check the version is to avoid reading the stale data

### steps when write a file: append

![image-20210515115327279](https://img.zhengyua.cn/20210515115327.png)

1. C asks M about file's last chunk
2. if M sees chunk has no primary (or lease expired):
   2a. if no chunkservers w/ latest version **# error**
   2b. pick primary P and secondaries from those w/ latest version #
   2c. increment version #, write to log on disk
   2d. tell P and secondaries who they are, and new version #
   2e. replicas write new version **# to disk**
    3. M tells C **the primary and secondaries + version**
    4. C sends data to all (just temporary...), waits
    5. C tells P to append
    6. P checks that **lease hasn't expired**, and chunk has space
    7. P picks an offset (at end of chunk)
    8. P writes chunk file (a Linux file)
    9. P tells each secondary the offset, tells to append to chunk file
    10. P waits for **all secondaries to reply**, or timeout
        secondary can reply "error" e.g. out of disk space
    11. P tells C "ok" or "error"
    12. C retries from start if error

> What consistency guarantees does GFS provide to clients?
>
> Needs to be in a form that tells applications how to use GFS.
>
> Here's a possibility:
>
> - If the primary tells a client that a record append succeeded, then any reader that subsequently opens the file and scans it will see the appended record somewhere.

### question

* What if an appending client fails at an awkward moment?
  Is there an awkward moment?

* What if the appending client has cached a stale (wrong) primary?

* What if the reading client has cached a stale secondary list?

* Could a master crash+reboot cause it to forget about the file?
  Or forget what chunkservers hold the relevant chunk?

* Two clients do record append at exactly the same time.
  Will they overwrite each others' records?

* Suppose one secondary never hears the append command from the primary.
  What if reading client reads from that secondary?

* What if the primary crashes before sending append to all secondaries?
  Could a secondary that *didn't* see the append be chosen as the new primary?

* Chunkserver S4 with an old stale copy of chunk is offline.
  Primary and all live secondaries crash.
  S4 comes back to life (before primary and secondaries).
  Will master choose S4 (with stale chunk) as primary?
  Better to have primary with stale data, or no replicas at all?

* What should a primary do if a secondary always fails writes?
  e.g. dead, or out of disk space, or disk has broken.
  Should the primary drop secondary from set of secondaries?
  And then return success to client appends?
  Or should the primary keep sending ops, and having them fail,
  and thus fail every client write request?

* What if primary S1 is alive and serving client requests,
  but network between master and S1 fails?
  "network partition"
  Will the master pick a new primary?
  Will there now be two primaries?
  So that the append goes to one primary, and the read to the other?
  Thus breaking the consistency guarantee?
  "split brain"

* If there's a partitioned primary serving client appends, and its
  lease expires, and the master picks a new primary, will the new
  primary have the latest data as updated by partitioned primary?

* What if the master fails altogether.
  Will the replacement know everything the dead master knew?
  E.g. each chunk's version number? primary? lease expiry time?

* Who/what decides the master is dead, and must be replaced?
  Could the master replicas ping the master, take over if no response?

* What happens if the entire building suffers a power failure?
  And then power is restored, and all servers reboot.

* Suppose the master wants to create a new chunk replica.
  Maybe because too few replicas.
  Suppose it's the last chunk in the file, and being appended to.
  How does the new replica ensure it doesn't miss any appends?
  After all it is not yet one of the secondaries.

* Is there *any* circumstance in which GFS will break the guarantee?
  i.e. append succeeds, but subsequent readers don't see the record.
  All master replicas permanently lose state (permanent disk failure).
  Could be worse: result will be "no answer", not "incorrect data".
  "fail-stop"
  All chunkservers holding the chunk permanently lose disk content.
  again, fail-stop; not the worse possible outcome
  CPU, RAM, network, or disk yields an incorrect value.
  checksum catches some cases, but not all
  Time is not properly synchronized, so leases don't work out.
  So multiple primaries, maybe write goes to one, read to the other.

What application-visible anomalous behavior does GFS allow?
Will all clients see the same file content?
Could one client see a record that another client doesn't see at all?
Will a client see the same content if it reads a file twice?
Will all clients see successfully appended records in the same order?

Will these anomalies cause trouble for applications?
How about MapReduce?

### strict consistency

I.e. all clients see the same file content.
Too hard to give a real answer, but here are some issues.

* Primary should detect duplicate client write requests.
  Or client should not issue them.
* All secondaries should complete each write, or none.
  Perhaps tentative writes until all promise to complete it?
  Don't expose writes until all have agreed to perform them!
* If primary crashes, some replicas may be missing the last few ops.
  New primary must talk to all replicas to find all recent ops,
  and sync with secondaries.
* To avoid client reading from stale ex-secondary, either all client
  reads must go to primary, or secondaries must also have leases.

> Performance (Figure 3)
>   large aggregate throughput for read (3 copies, striping)
>     94 MB/sec total for 16 chunkservers
>       or 6 MB/second per chunkserver
>       is that good?
>       one disk sequential throughput was about 30 MB/s
>       one NIC was about 10 MB/s
>     Close to saturating network (inter-switch link)
>     So: individual server performance is low
>         but scalability is good
>         which is more important?
>     Table 3 reports 500 MB/sec for production GFS, which is a lot
>   writes to different files lower than possible maximum
>     authors blame their network stack (but no detail)
>   concurrent appends to single file
>     limited by the server that stores last chunk
>   hard to interpret after 15 years, e.g. how fast were the disks?
>
>
>
> Random issues worth considering
>   What would it take to support small files well?
>   What would it take to support billions of files?
>   Could GFS be used as wide-area file system?
>     With replicas in different cities?
>     All replicas in one datacenter is not very fault tolerant!
>   How long does GFS take to recover from a failure?
>     Of a primary/secondary?
>     Of the master?
>   How well does GFS cope with slow chunkservers?
>
>
>
> Retrospective interview with GFS engineer:
>   http://queue.acm.org/detail.cfm?id=1594206
>   file count was the biggest problem
>     eventual numbers grew to 1000x those in Table 2 !
>     hard to fit in master RAM
>     master scanning of all files/chunks for GC is slow
>   1000s of clients too much CPU load on master
>   applications had to be designed to cope with GFS semantics
>     and limitations
>   master fail-over initially entirely manual, 10s of minutes
>   BigTable is one answer to many-small-files problem
>   and Colossus apparently shards master data over many masters

## Summary

- case study of performance, fault-tolerance, consistency

  specialized for MapReduce applications

- good ideas:
    - global cluster file system as universal infrastructure
    - separation of naming (master) from storage (chunkserver)
    - sharding for parallel throughput
    - huge files/chunks to reduce overheads
    - primary to sequence writes
    - leases to prevent split-brain chunkserver primaries
- not so great:
    - single master performance:ran out of RAM and CPU
    - chunkservers not very efficient for small files
    - lack of automatic fail-over to master replica
    - maybe consistency was too relaxed
