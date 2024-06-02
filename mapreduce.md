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

Master failure is unlikely. If the master fails, the job is aborted. 

#### Semantics in the Presence of Failures

As long as the map and reduce functions are deterministic, the distributed implementation produces the same output as a sequential execution would thanks to atomic commits of map and reduce tasks.

### Locality

The MapReduce master tries to schedule map jobs on machines where the corresponding input data is stored.
If that doesn't work, it schedules a map task on a nearby machine.
Throughout the whole cluster, a significant fraction of input data is read locally without consuming network bandwidth.

### Task Granularity

Each worker performs many different tasks to improve dynamic load balancing and speed up recovery when a worker fails. 

### Backup Tasks

A straggler is a machine that takes an abnormally long time to complete a task.
Causes of stragglers:
- A bad disk slowing read performance
- Having been scheduled many tasks leading to resource scarcity
- Processor caches disabled by a bug

The solution to stragglers: when close to completion, schedule backup executions of remaining tasks.

## Refinements

This section details extensions to MapReduce.

### Partitioning Function

MapReduce supports custom partitioning functions.
For example, the keys may be URLs but a user may want to partition on hostnames extracted from the URLs.

### Ordering Guarantees

Within a given partition, intermediate key/value pairs are processed in increasing order.

### Combiner Function

A combiner function is essentially a reduce function that executes before intermediate data is sent to the reduce machines. Partial combining speeds up certain MapReduce operations.

### Input and Output Types

The MapReduce library provides support for reading and producing data in different formats.
For example, text mode treats each line as a key/value pair.
Users can add support for a new type by implementing an interface.

### Skipping Bad Records

Sometimes user-provided code crashes deterministically on certain records.
Fixing the bug may not be feasible when it occurs in a third-party library.
So there is an option to skip records causing deterministic crashes.
When the master sees more than one failure on a particular record, it tells new tasks to skip it.

### Local Execution

There is an alternative sequential implementation of MapReduce useful for debugging.

### Status Information

The master runs an HTTP server with status pages showing the progress of the computation.
Indicators of progress:
- how many tasks have been completed
- how many tasks are in progress
- bytes of input
- bytes of intermediate data
- bytes of output
- processing rates

For debugging, the status page also shows which workers have failed and which tasks they were processing.

### Counters

The MapReduce library can count occurrences of events. For example, the total number of words processed.

## Experience

MapReduce has been used for a wide range of problems including:
- large-scale machine learning
- clustering for Google News
- extraction of data to produce reports of popular queries
- extraction of properties of web pages for new experiments of products
- large-scale graph computations

### Large-Scale Indexing

MapReduce has been used in the production indexing system for Google web search.

The indexing code is simpler and more understandable because fault-tolerance, distribution, and paraelleization are abstracted away.

It has become easier to change the indexing process.

The indexing process is more operable because MapReduce automatically deals with machine failures, slow machines, and networking issues. New machines can easily be added to improve performance.

## Source

http://nil.csail.mit.edu/6.824/2020/papers/mapreduce.pdf
