# Dynamo: Amazon's Highly Available Key-value Store

## Introduction

Reliability is one of the most important requirements for Amazon's platform. A slight outage has significant financial consequences and impacts customer trust.

High scalability is another important requirement.

Amazon uses a decentralized, service-oriented architecture necessitating the need for storage technologies that are always available.

Failure is a norm in a system with millions of components. 

Amazon has designed Dynamo to meet high availability and scalability requirements. It is also configurable.

Dynamo provides a simple primary-key only interface (no secondary indexes).

Techniques for achieving scalability and availability:
- Data is partitioned and replicated using consistent hashing
- Consistency is facilitated by object versioning
- Consistency among replicas is maintained by a quorum-like technique and decentralized synchronization protocol
- A gossip based distributed failure detection and membership protocol
- Minimal need for manual administration
- Storage nodes can be added and rmeoved without requiring manual partitioning or redistribution

Dynamo has been able to scale to peak loads without downtime.

Dynamo is eventually consistent.

## Background

Traditionally, production systems store state in a relational database. But many services only need to retrieve data by primary key and do not provide complex querying.

For RDBMS, replication technologies are limited and prioritize consistency over availability. 

Scaling relational databases can be hard.

### System Assumptions and Requirements

Query Model: simple read and write operations, primary key identifies a small blob

ACID Properties: databases that provide ACID transactions have poor availability, Dynamo sacrifices consistency and isolation

Efficiency: Dynamo needs to function on commodity hardware while meeting stringent SLAs

Dynamo is only used by internal services, no security related requirements

### Service Level Agreements (SLA)

SLA includes:
- client's expected request rate distribution
- expected service latency

Example: response within 300ms for 99.9% of requests for peak client load of 500 request per second

SLAs ensure that dependencies in a huge call graph all obey their contracts

Amazon measures SLAs at high percentiles to ensure all customers have a good experience

If business logic is lightweight, storage management becomes the most important part of an SLA

### Design Considerations

Synchronous replication provides high consistency and low availability

Availability can be increased by using optimistic replication techniques where replica updates propagate in the background.

The challenge with asynchronous replication is that conflicting changes must be resolved.

To ensure the data store is "always writeable" and provide a good customer experience, Dynamo does not perform conflict resolution on write. 

For example, rejecting updates to a shopping cart would result in poor CX. 

So Dynamo pushes the complexity of conflict resolution to reads.

An application developer can either choose to let the data store resolve the conflict (simple policies such as "last write wins") or the application can handle conflict resolution.

Other design principles:
- Incremental scalability: Dynamo should scale out one storage host at a time
- Symmetry: every node has the same set of responsibilities as peers (leaderless replication)
- Decentralization: uses peer-to-peer techniques
- Hereteogeneity: work distribution must be proportional to the capabilites of the individual servers

## System Architecture

A production storage system needs scalable and robust solutions for:
- persistence
- load balancing
- membership
- failure detection
- failure recovery
- replica synchronization
- overload handling
- state transfer
- concurrency and job scheduling
- request marshalling
- request routing
- system monitoring and alarming
- configuration management

### System Interface

Dynamo exposes get() and put() operations.

get(key) returns an object or list of objects with conflicting versions and a context

put(key, context, object)

context encodes system metadata such as version

Dynamo applies a MD5 hash on the key to generate a 128-bit identifier - used to determine storage nodes that serve the key

### Partitioning Algorithm

In order to scale incrementally, Dynamo must dynamically partition data over the nodes

Dynamo uses consistent hashing

With consistent hashing, the departure or arrival of a node only affects its neighbors on the hash ring minimizing the amount of rebalancing

Problems with consistent hashing:
- random position assignment can lead to non-uniform distribution
- oblivious to heterogeneity in nodes

Dynamo uses a variant of consistent hashing with virtual nodes. The advantages are:
- if a node becomes unavailable, the load is evenly dispersed across remaining nodes
- when a node becomes available again or a new node is added, the new node accepts an equivalent amount of load from the other nodes
- the number of virtual nodes a node is responsible for can be decided based on capacity

### Data Versioning

Dynamo provides eventual consistency (asynchronous replication)

If there are no failures, there is a bound to update propagation times. But propagation may be severely delayed in certain failure scenarios.

Some Amazon applications (like the shopping cart) can operate under a network partition

Dynamo allows multiple versions of an object to be present in the system

Most of the time, new versions subsume the previous version ... but there can be conflicting versions

Dynamo uses vector clocks to capture causality between different versions

A vector clock is a list of (node, counter) pairs. Each version of an object has a vector clock.

If one object's clock is <= to all nodes in the second object's clock ... the first object is an ancestor and can be forgotten ... otherwise there is a conflict requiring reconciliation

When a client updates an object, it must pass in the version it wants (context as a vector clock)

### Handling Failures: Hinted Handoff

Dynamo does not enforce strict quorum membership (sloppy quorum).

