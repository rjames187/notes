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



## Source

http://nil.csail.mit.edu/6.824/2020/papers/gfs.pdf
