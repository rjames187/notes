# Conflict-free Replicated Data Types

## Definition

A CRDT is an abstract data type designed to be replicated at multiple processes

Exhibits these properties:
- Any replica can be modified without coordinating with other replicas
- When two replicas have received the same set of updates, they deterministically converge to the same state

## Overview

Distributed systems often replicate data across geographic locations to provide:
- Low latency
- High Availability

Inter-datacenter replication usually happens **asynchronously**

Replicas may temporarily diverge which can lead to update conflicts

CRDTs enable uncoordinated conflict-free updates

**Concurrency semantics** determine how a CRDT behaves

The **synchronization model** defines the requirements a system must meet and helps in choosing a CRDT

## Concurrency semantics

Data types that intrinsically commute naturally converge to the same state

For example, a Counter's operations will yield the same result no matter the order of execution

Examples of concurrency semantics based on the happens-before relation:
- Add-wins set: during concurrent add and remove, add wins
- Remove-wins set: during concurrent add and remove, remove wins

The total order relation allows for last-writer-wins semantics

Add-wins, Remove-wins, and last-writer-wins are all applicable to sets

A register CRDT maintains an opaque value and provides a single update operation

Register CRDTs:
- multi-value: all concurrently written values are kept
- last-writer-wins: only the last write is kept

Counter CRDTs with only increment and decrement operations naturally commute

Introducing a write operation to the counter complicates things. Possible semantics:
- last-writer-wins for concurrent writes
- write-wins for concurrent write and inc/dec operations

CRDTs for lists, graphs, and JSON documents have all been proposed with specific concurrency semantics

## Synchronization Model

There are two approaches to propagating updates (synchronizing replicas):
- state-based (bi-directional merging)
- operation-based (may require operations be delivered in a specific order)

Operation-based CRDTs define a generator and effector function for each operation

A generator executes in the replica where the operation is submitted and has no side-effects

The generator generates an effectro that encodes the side-effects (and replicates to all replicas)

Most operation-based CRDTs require causal delivery

An alternative model is the Delta-state CRDT:
- does not have to propagate the entire state, just delta-mutators

Propagation of effectors or delta-mutators can be deferred and the outbound log can be compressed

## Examples of applications

CRDTs have been used in systems with weak consistency models

Riak, Redis, and Akka are storage systems that use CRDTs to be highly available

CRDT libraries have also been created to be embedded into applications

## Future directions of research

### Scalability

The metadata cost from causality trakcing can limit scalability of CRDTs

Large metadata footprint can impact:
- computation time of local operations
- required storage
- communication

Possible solutions include more compact causality representations


