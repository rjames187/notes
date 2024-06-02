# Intro to Distributed Systems

A distributed system is a set of cooperating computers communicating over a network.

Lots of critical infrastructure is built out of distributed systems.

A single computer is simpler. Only build a distributed system if necessary.

Reasons people build distributed systems:
- Parallelism (more CPU, more memory, etc)
- Fault Tolerance
- Inherent physical distribution
- Security / Isolation

Distributed systems are tricky. Concurrency is difficult to deal with. There can be unexpected failure patterns. There can be partial failures.

Getting good performance out of a distributed system requires careful design.

## Infrastructure for Applications

Kinds of infrastructure:
- Storage
- Communication
- Computation

It's hard to build abstractions for distributed systems that behave just like single-node systems.

## Implementation

RPC is an abstraction over an unreliable network.

Threads are a way to structure concurrent applications.

Concurrency control is a key concept in building distributed systems.

## Scalability

More computers can provide greater throughput.

Just adding more computers is a cheap way to improve performance.

## Fault Tolerance

In large-scale systems with thousands of computers, failure is common.

So masking of failures has to be built into the design of a distributed system.

Types of fault tolerance:
- Availability: when a system continues to work even when part of the system fails
- Recoverability: system can be recovered after the failed component is repaired

Non-volatile storage is a tool for recoverability.
NV storage is expensive to access and is linked to performance.

Replication is a tool for fault tolerance. 

## Consistency

In distributed systems there are often multiple copies of data (replication, caching).

Consistency is when the copies are the same.

Weakly consistent systems may allow for stale reads.

Communication required for stong consistency may be very expensive.

## Source

https://www.youtube.com/watch?v=cQP8WApzIQQ&t=1s
