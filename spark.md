# Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing

## Introduction

Large-scale data processing frameworks like MapReduce lack abstractions for leveraging memory.

MapReduce is inefficient for applications that reuse intermediate results.

Data reuse is common in **iterative** algorithms such as:
- PageRank
- K-means clustering
- logistic regression

**Interactive** data mining (ad hoc queries) is a desired feature

With MapReduce, reusability of data requires writing to a distributed file system

Writing to distributed file systems increases execution times because of:
- data replication
- disk I/O
- serialization

Frameworks such as Pregel support iterative processing but only for specific usecases

Resilient Distributed Datasets (RDDs) enable efficient data reuse and provide:
- fault-tolerance
- parallel data structures

RDDs solve the challenge of implementing efficient fault-tolerance for **coarse-grained** workloads

RDDs log the transformations to build a dataset (lineage) rather than actual data

If data is lost, the RDD can recompute it without the need for replication

Spark is an implementation of RDDs that provides a Scala programming interface

## Resilient Distributed Datasets

### RDD Abstraction

An RDD is a read-only partiioned collection of records

RDDs are created through deterministc operations (transformations) on either:
- data in persistent storage
- other RDDs

Examples of transformations:
- map
- filter
- join

Users can configure persistence and partitioning (such as choosing a partition key)

### Spark Programming Interface

In the programming interface, datasets are objects and transformations are methods

Users can define RDDs through transformations on data in stable storage

RDDs can be used in actions, operations that return or export data

By default, RDDs are kept in memory but spill to disk if RAM is exhausted

### Advantages of the RDD Model

With distributed shared memory (DSM), applications read and write to a global address space

RDDs can only be created with coarse-grained transformations (no fine-grained writes)

So RDDs are restricted to builk write workloads but allow for efficient fault tolerance

RDDs do not need checkpointing because they have lineage-based recovery

RDDs can mitigate grey failures by running backup copies of slow tasks

RDD operations can be scheduled based on data locality to improve performance

Partitions that don't fit in RAM can be stored on disk (back to the performance of current frameworks)

### Applications Not Suitable for RDDs

RDDs are suited for batch processing

RDDs are not suited for asynchronous fine-grained updates to shared state such as with:
- a storage system for a web application
- incremental web crawler

## Representing RDDs

Spark uses a graph-based representation of RDDs

Each RDD exposes five pieces of information:
- a set of partitions
- a set of dependencies on parent RDDs
- a function for computing the dataset based on dependencies
- metadata about partitioning scheme and data placement

Two types of dependencies:
- narrow: each partition is used by at most one partition of the child
- wide: multiple child partitions may depend on the same parent partition

