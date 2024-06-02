# MapReduce: Simplified Data Processing on Large Clusters

Jeffrey Dean and Sanjay Ghemawat of Google Inc, 2004

## Introduction

For years, Google's systems had processed large amounts of data (crawled documents and web request logs) to compute derived data such as:
- inverted indices
- graph representations of web documents
- number of pages crawled per host
- most frequent queries in a given day

Input data was large and computation would have to be distributed across many machines.
Although the computations were simple, issues of parallel computation, distributed data, and machine failures necessitated complexity.

Dean and Ghemawat designed the MapReduce abstraction to hide the details of parallelization, fault-tolerance, data distribution, and load balancing.
The library was inspired by map and reduce primitives found in functional languages.

## Programming Model

The computation takes an input of key/value pairs and outputs key/value pairs.

The map function produces intermediate key/value pairs and groups them together by their intermediate key. 
The reduce function accepts an intermediate key and its values. It merges the values to form a smaller set of values.

MapReduce usecases:
- Distributed Grep: map emits a line if matches the pattern, reduce simply copies to the output
- Count of URL Access Frequency: map function outputs URLs and reduce function counts them
- Reverse Web-Link Graph: map outputs target and source urls, reduce concatenates all sources for a given target
- Term-Vector per Host: map emits host + term vector pairs, reduce adds up term vector pairs
- Inverted Index: map emits word + document ID pairs, reduce outputs word + list of document ID pairs
- Distributed Sort: map emits a key + record pair, is sorted thanks to partitioning

## Implementation

Google's implementation of MapReduce was designed for large clusters of commodity machines. Clusters were so large that failures were common.
Data was managed by a distributed file system that provided availability and reliability through replication.

### Execution Overview

Input data was partitioned across machines distributing map invocations. Reduce invocations were distributed by hash partitioning intermediate keys.

The master program would assign each worker a map task or a reduce task.
A map worker processes its split, outputs the intermediate key/value pairs into memory, and periodically writes to disk.
The master notifies the reduce workers of the location of intermediate pairs. The reduce worker reads the pairs via RPC.
A reduce worker sorts by intermediate keys so all data with the same key is grouped together.

### Master Data Structures

For each map and reduce task, the master stores the state (idle, in-progress, or completed) and identity of worker machine (non-idle tasks).
For each completed map task, the master stores locations and sizes of intermediate file regions.

### Fault Tolerance

The library tolerates machine failures gracefully.

#### Worker Failure

The master pings workers and marks non-responsive workers as failed. Tasks on failed workers are reset to idle.
Completed map tasks are re-executed on failure because they are stored on the local disks of the failed machine.
Completed reduce tasks do not need to be re-executed since they are stored in the global file system.
Reduce workers are notified of any map re-executions so they can grab the correct pairs.

#### Master Failure



## Source

http://nil.csail.mit.edu/6.824/2020/papers/mapreduce.pdf
