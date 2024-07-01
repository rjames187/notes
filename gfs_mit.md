# Google File System Case Study

GFS touches on parallel performance, fault tolerance, replication, and consistency

## Why is distributed storage hard?

It starts with the desire to harness the aggregate performance of many machines

This requires a massive volume of data distributed across different machines (sharding)

In large distributed systems, fault tolerance is needed

Replication is the best way to get fault tolerance

Replication leads to inconsistencies in data

Consistency degrades performance

## Strong Consistency

Requests see data that reflects all previous operations in order

## Bad Replication Design Example

What if two replicas process concurrent requests in a different order?

## GFS

The paper published around the first time academic ideas about distributed systems were used in industry

Main ideas of GFS:
- Big
- Fast
- Global
- Sharding
- Automatic Recovery
- Single data center
- For internal use
- Big sequential access
- Throughput over latency
- Single master

## General Structure of GFS

Master separated from chunk servers (where data is stored)

Master keeps track of a list of chunk IDs for each file

For each chunk ID, master keeps a list of chunk servers with version # and primary and lease expiration

The above data structure is in memory

Master has a log and checkpoint on disk
