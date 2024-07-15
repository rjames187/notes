# In Search of an Understandable Consensus Algorithhm

## Introduction

Consensus algorithms allow a distributed system to survive failures

Paxos has been the dominant consensus algorithm for a while

But Paxos is hard to understand and requires complex changes to be put into practice

Ongaro and Ousterhout designed a new understandable consensus algorithm called Raft by:
- decomposing the problem
- reducing the state space

Novel features of Raft:
- Strong leader
- Leader election using randomized timers
- Membership changes using joint consensus enabling cluster to operate normally during config changes

## Replicated State Machines

Large scale systems use replicated state machines like Chubby or Zookeeper to manage leader election and store config information that survives leader crashes

Replicated state machines include logs of commands which are in the same order on all machines

A consensus algorithm keeps the replicated log consistent

Job of a consensus module:
- Adds commands from clients to its log
- Communicates with other servers to ensure every log contains the same requests in the same order
- The state machine processes the log's commands

Properties of consensus algorithms
- ensure safety despite network delays, partitions, and packet loss, duplication, and reordering
- fully available as long as the majority of servers are operational and can communicate
- do not depend on timing to ensure log consistency (not vulnerable to faulty clocks)
- a command can complete if the majority of servers respond (not slowed down by a few slow servers)

## What's wrong with Paxos?

Single-decree Paxos is a protocol for reaching agreement on a single decision (log entry)

Multi-Paxos combines combines multiple instances of single-decree Paxos for many decisions (such as a log)

Good things about Paxos:
- ensures safety and liveness
- supports changes in cluster membership
- proven correct
- typically efficient

Drawbacks of Paxos:
- very difficult to understand
- single-decree Paxos is dense and decomposed unintuitively
- poor architecture for implementing practical systems

## Designing for understandability

Goals of Raft:
- complete and practical foundation for building practical systems
- reduces amount of design work required
- safe under all ocnditions
- available under typical conditions
- efficient for common operations
- intuitive and understandable by a large audience

Raft is decomposed into leader election, log replication, safety, and membership changes

Raft has a reduced state space

## The Raft consensus algorithm

Raft elects a leader to manage the replicated log

Responsibilities of the leader:
- accept log entries from clients
- replicate them on other servers
- tells servers when they can apply log entries to their state machines

If the leader fails or becomes disconnected, a new leader is elected

### Raft basics

A server can be on one of these states:
- leader
- follower
- candidate

Followers simply respond to requests from leaders and candidates

The leader handles all client requests

A candidate elects a new leader

Time is divided into *terms* and each term starts with an election

If a candidate wins, it serves as leader for the rest of the term (until it fails)

In the case of a split vote, a new term and election start

There is always one leader in a given term

Terms act as a logical clock and allow servers to detect stale leaders

Each server stores a current term number which is monotonic

If a candidate or leader discovers its term is outdated, it becomes a follower

If a follower receives a request with a stale term number, it rejects it

The basic Raft algorithm only has 2 RPCs:
- RequestVote RPC
- AppendEntries RPC (also used as a heartbeat)
- There can also be a third RPC for transferring snapshots
