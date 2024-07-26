# The Google File System

## Introduction

Goals of GFS:
- Performance
- Scalability
- Reliability
- Availability

GFS is built from inexpensive commodity machines and failures are the norm.

Causes of failures
- application bugs
- operating system bugs
- human errors
- failures of disks, memory, connectors, networking, and power supplies

Due to failures, monitoring, error detection, fault tolerance, and automatic recovery are essential to the system

GFS deals with multi-GB files which are larger than in traditional file systems

Most of the files are mutated via appends rather than overwrites

Files are written once and then only read

The GFS API has been codesigned with the applications that use it for flexibility

## Design Overview

### Assumptions

Assumptions for the design
- GFS will be built from cheap commodity components that often fail (needs monitoring, fault tolerance, failure recovery, etc.)
- GFS will store a lot of large files that need to be managed efficiently
- Two main workloads: large streaming reads and small random reads
- Workloads also have writes that append data to files
- GFS needs support for high append concurrency
- High sustained bandwidth is more important than low latency

### Interface

Files are organized into directories identified by path-names

Supported operations on files:
- create
- delete
- open
- close
- read
- write
- snapshot append
- record append

A snapshot creates a copy of a file or directory at a low cost.

Record append supports concurrent appends with atomicity

### Architecture

A GFS cluster has one master and many chunk servers which are accessed by clients

Files divided into fixed-size chunks

A chunk is identified by a globally unique 64 bit handle assigned by the master at chunk creation

Chunk servers store chunk data on local disk and read/write data specified by handle and byte range

Each files is replicated on different chunk servers for reliability (3 times by default)

The master has filesystem metadata:
- namespace
- access control info
- mapping from files to chunks
- locations of chunks

Other activities controlled by the master:
- chunk lease management
- garbage collection of orphaned chunks
- chunk migration between chunk servers
- sending heartbeat messages to chunk servers

Clients implement the file system API

Clients talk to the master for metadata operations but go directly to chunk servers for reads/writes

Clients do not cache because the data is too large to be cached

Chunk servers do not cache because the data is already local (so automatically cached by the OS)

### Single Master

A single master leads to a simple design

Because there is only one, reads and writes to the master should be minimized

Interactions for read:
1. Client sends master a file name and chunk index
2. Master replies with a chunk handle and replica locations
3. Client caches this info under the chunk index
4. Client reads from the nearest replica
5. Client can do additional reads from the same chunk before cached info expires

### Chunk Size

Chunk size is large at 64 MB

Advantages of large chunk size:
- Reduces clients' need to interact with master (r/w within same chunk only requires one request to master)
- Clients may r/w to the same chunk many times reducing network overhead via persistent TCP conncetion
- Reduces metadata stored in master (fewer chunks means the metadata can be stored in memory)

### Metadata

#### In-Memory Data Structures

Because metadata is stored in memory, operations are fast and periodic scanning is efficient

Periodic scanning is used for:
- chunk garbage collection
- re-replication when chunk servers fail
- chunk migration to balance load and disk space

Memory-only technically limits the capacity of the system but metadata is so small its not a concern

In-memory storage is worth it because it brings simplicity, reliability, performance, flexibility

#### Chunk Locations

The master does not persist chunk server replica locations (polls chunk servers for the info at startup)

Not persisting such info simplifies the design as master and chunk servers do not have to kept in sync in the face of failures, restarts, etc.

#### Operation Log

Operation log is a historical record of metadata changes (only persistent record of metadata)

Its a logical timeline that defines the order of concurrent operations

The operation log must be flushed to disk and replicated before responding to clients

Log operations are flushed in batches to reduce impact on overall system throughput

The master recovers file system state by replaying the operation log

To keep the operation log small (to reduce startup time) state is checkpointed in a B tree that can be directly mapped to memory

Checkpoints are created in a separate thread to not delay incoming operations

Incomplete (failed) checkpoints are skipped

### Consistency Model

GFS has relaxed consistency

#### Guarantees by GFS

File namespace mutations (like file creation) are handled by the master

Such mutations have atomicity provided by namespace locking

A file region is *consistent* if all clients see the same data regardless of replica

A file region is *defined* if mutations happen in their entirety (no partial writes)

Concurrent successful mutations leave a region consistent but undefined (incorrect)

A failed mutation makes the region inconsistent and undefined

How GFS guarantees regions are defined:
- Applies mutations in the same order in all replicas
- Uses chunk version numbers to detect stale replicas because of chunkserver downtime

GFS identifies failed chunkservers by handshakes

GFS detects data corruption via checksums

If GFS detects a problem with a replica, it is restored from other replicas

#### Implications for Applications

GFS applications must accomodate the relaxed consistency model

## System Interactions

### Leases and Mutation Order

Leases maintain conistent mutation order across all replicas of a chunk

Leases minimize management overhead at the master

During a mutation, the replica with the lease (the primary) essentially controls the replication of writes

### Data Flow

Data flow is decoupled from control flow

Control flows from client to primary to secondaries

Data flows linearly along a chain of chunk servers to fully utilize network bandwidth

Data is always forwarded to the closest machine that hasn't received it

### Atomic Record Appends

Traditional writes break with concurrency (not serializable)

Record appends to the end of a file atomically

Record appends eliminate the need for complicated synchronization among clients

### Snapshot

Operation that copies a file or directory tree

Before snapshotting, mastser revokes outstanding leases (so subsequent writes will have to find the new leaseholder)

Master logs operation to disk and duplicates metadata

## Master Operation

### Namespace Management and Locking

GFS represents its namespace as a lookup table mapping pathnames to metadata

Prefix compression enables the table to be efficiently represented in memory

Each node in the namespace tree (either a file or directory) has a read-write lock

Each master operation acquires locks on the nodes involved

Example involving snapshot of `/home/user` to `/save/user`:
1. Read locks acquired on `/home` and `/save`
2. Write locks acquired on   `/home/user` and `/save/user`

A read lock on a directory prevents deletion, renaming, and snapshotting

A write lock on a file serializes file creation with the same name

### Replica Placement

Chunk replicas are spread across racks for availability (no single point of failure)

Read traffic can exploit the aggregate bandwidth of multiple racks

### Creation, Re-replication, Rebalancing

Factors master considers when placing chunk replicas:
- Seeks chunk servers with below-average disk space utilizaion
- Limit the number of recent creations on each chunkserver (because of imminent heavy write traffic)
- Spread chunk replicas across racks

Master re-replicates a chunk when number of available replicas gets too low for reasons such as:
- a chunkserver becomes unavailable
- a replica is corrupted
- disk is disabled because of errors
- replication goal is increased

Factors influencing the prioritization of chunk re-replication
- Prioritizes chunks that are farther from the replication goal
- Prioritize re-replication of live files over deleted files
- Prioritize chunks blocking client progress

New replicas are placed with the same goals as with creation

Master limits active cloning operations to not overwhelm client traffic

Each chunkservers limits bandwidth for clone operations by throttling reads to source chunk server

Sometimes the master rebalances (moves) replicas for better disk space and load balancing

### Garbage Collection

Rather than immediately, GFS lazily reclaims storage after a file is deleted

#### Mechanism

When a file is deleted, it's renamed to a hidden name including the deletion timestamp

A file can be undeleted by renaming it

As the master scans the file system, it deletes hidden files (including memory metadata) that are more than 3 days old

Orphaned chunks are also deleted 

In regular heartbeats a chunkserver reports the chunks it has and the master replies with chunks that are safe to be deleted

#### Discussion

Advantages of garbage collection over immediate deletion:
- simple and reliable; when chunk creation doesnt succeed, there may be replicas the master doesnt know about; replica deletion messages could be lost and need to be resent
- running it in batches in the background amortizes cost and frees the master to respond to more important things
- delay in reclamation safeguards against accidental irreversible deletion

A disadvantage is that not being able to immediately reclaim space can lead to storage being tight

Addressed by:
- permanently deleting a file if explicitly deleted again
- allowing the user to apply different replication and reclamation policies to parts of the namespace

### Stale Replica Detection

Chunk replicas become stale if a chunkserver goes down and misses mutations

For each chunk, the master keeps a chunk version number which is incremented when a new lease is granted

Online chunk servers will increment their chunk version numbers when master 

Chunk servers report their chunk version numbers so master can see which have stale chunks

## Fault Tolerance and Diagnosis

### High Availability

Two strategies for high availability: fast recovery and replication

#### Fast Recovery

Master and chunkserver restore their state and restart in seconds

No distinction between normal and abnormal  termination

#### Chunk Replication

Chunks are replicated on multiple chunk servers across racks

Users can specify replication goal for different namespaces

Num replicas may fall under replication goal due to chunkservers going offline or detection of corrupted replicas via checksums

#### Master Replication

Master's operation log and checkpoints are replicated on multiple machines

A state mutation is only committed after a log record is flushed to disk locally and on all replicas

One master process remains in charge of mutations and background activities

If the master fails, a new master process is started elsewhere with a replica's operation log

The name the clients use to reach the master is a DNS alias that can be changed if the master's machine changes

If a master goes down, shadow masters still provide read-only access (they lag a bit)

Shadow master applies replicated operation log to its state machine

### Data Integrity

Chunkservers use checksumming to detect data corruption

Disk failures can cause data corruption and regularly occur

Checksumming is needed because comparing across replicas is impractical

Each 64 KB block of a chunk has a 32 bit checksum

Checksums are kept in memory and stored persistently with logging

During reads, chunkserver verifies the checksum of the requested data blocks

If a block doesn't match the checksum, chunkserver returns an error, another replica is read, and chunk is cloned from a non-corrupt replica to a new replica

Checksum computation is optimized for append writes because that is the dominant workload

### Diagnostic Tools

Logging has helped in problem isolation, debugging, and performance analysis

Diagnostic logs record events such as:
- chunkservers going up or down
- RPC requests and replies

Performance impact of logging is small because logs are written sequentially and asynchronously

Most recent events are kept in memory

## Experiences

GFS was originally designed for production systems but grew to include R&D tasks

Grew to support permissions and quotas

Mismatches between Linux driver and disk protocols led to data corruption and motivated the use of checksums

## Source

http://nil.csail.mit.edu/6.824/2020/papers/gfs.pdf
