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

### Leader election

Leaders send periodic heartbeats to all followers

If a follower receives no heartbeat before the election timeout, it starts an election

When starting an election, the follower increments its term, becomes a candidate, votes for itself, and issues vote requests to the other servers

Three things can end the candidate state:
- the candidate wins the election
- another server establishes itself as leader
- there is no winner

A candidate wins the election if it receives votes from the majority of servers

Each server votes for at most one candidate in a given term

A new leader sends heartbeat messages to establish authority and prevent elections

If a candidate receives AppendEntries RPC and leader's term is greater than or equal to its term, it becomes a follower because that is a legitimate leader

If there is no election winner (no candidate gets a majority of votes) caniddates will start new elections after timeouts

Randomized timeouts (150-300 ms) ensure split votes (where there is no winner) are rare

### Log replication

only leaders can service client requests

leader appends a command from a client request to its log

then issues AppendEntries RPCs in parallel to the other servers

once entry is safely replicated, leader applies entry to its state machine and returns result to client

if there are failures or network partitions, leader retries AppendEntries RPC until all followers store the log entry

each log entry has a term number used to detect inconsistencies; also has an integer index in the log

a committed log entry has been replicated on a majority of servers and is safe to apply to state machines

committing an entry also commits all previous entries (including those from previous leaders)

leader includes highest committed idx in AppendEntries RPCs (how other servers find out)

Raft Log Matching Property:
- if 2 entries in different logs have the same index and term they store the same command
- if 2 entries in different logs have the same index and term, the logs are identical in all preceding entries

log inconistencies only occur when the leader crashes (why Append Entries has a conistency check)

if logs are found inconsistent, the leader deletes entries in followers log after latest agreement and sends the follower all the leaders entries after that point

Leader keeps a nextIndex for each follower; inits nextIndex to just after last one in its log

If logs are inconistent, leader keeps decrementing nextIndex and retrying RPCs until conistent, then does the reconciliation process

### Safety

Raft restricts who can be leader for safety

#### Election restriction

Raft guarantees all command entries from previous leaders are present on the new leader

to win an election a candidate's log must be atleast as up to date as any other log in the majority

voter denies its vote if its own log is more up to date than candidates log

#### Committing entries from previous terms

#### Safety Argument

### Follower and candidate crashes

If a RequestVote or AppendEntries RPC fails, Raft retries indefinitely

If a raft server crashes before completing an RPC, it will receive the same RPC again ... this is not a problem because the RPCs are idempotent

### Timing and availability

For Raft to elect and maintain a leader: broadcastTime < electionTimeout < MTBF

Randomized election timeouts make split votes unlikely

## Cluster membership changes

Raft needed to allow for automatic cluster config changes without system downtime

For safety, it must be impossible for two leaders to be elected during the same term during a config transition

Switching all the servers at once is unsafe

Raft config changes use a two phase approach:

Raft first switches to a transitional phase called *Joint Consensus*

The system completes the transition joint consensus is committed

The cluster is split into two independenct majorities: old and new

Any server from either configuration may serve as leader

Agreement for elections and entry committment requires separate majorities from both the old and new configurations

Joint consensus allows individual servers to safely transition while the system still serves requests

Cluster configurations are communicated in special entries in the replicated log

only candidates with the old and new config log can be elected

The old neither the new config can make unilateral decisions at any time

To mitigate time-consuming catchups for new servers, new servers join as non-voting members that still replicate log entries

Servers disregard RequestVote RPCs when they believe a current leader exists to mitigate disruptions some servers being removed from the cluster who call perpetual votes

## Log compaction

Snapshotting is used to free up space for more log entries

The entire system state is persisted as a snapshot which replaces the log up to a certain point

Snapshot also has metadata
- *last included index*
- *last included term*
- the latest cluster configuration

Leader sends an InstallSnapshot RPC when a follower lags so far behind the entries it neededs have been deleted

Upon reception, the follower will usually discard its entire log and then apply the snapshot

Followers typically create their own snapshots

The leader not often sending snapshots departs from the idea of a strong leader. Reasons for this:
- would slow network bandwidth
- cheaper for a follower to snapshot its own state
- the leader's implementation is simplified

## Client interaction

Client first connects to a random server in the cluster

If the server is not the leader, the server tells the client about the leader

If a leader crashses after committing a log entry but before responding to a client, the client will retry and there could be a duplicate request

Solution is for clients to assign serial numbers to each request, if the leader receives a command whose serial number has already been executed, it responds immediately without re-executing the request






















