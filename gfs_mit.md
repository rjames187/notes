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

## Reads

1. The application has a filename and offset in mind to read from (sends to master)
2. Master looks up filename and offset to find a specific chunk
3. Master looks up list of chunk servers that have needed replicas
4. Client caches the chunk handle and chunk servers sent by the Master (for minimzing load on master)
5. Chunk server reads data at the offset and returns the data to the client

## Writes

1. If there is no primary, find up to date replicas (having the highest version number)
2. Pick primary and secondaries from up-to-date replicas
3. Master increments version number to tells primary and secoondaries the version number
4. Master gives lease to primary and writes new version num to disk
5. Client sends the data to the primary and secondaries
6. After all replicas report they have the data, the client tells the primary to append the data
7. Primary picks an offset, all replicas told to write at the offset
8. If primary gets yes from all secondaries, primary sends success to client
9. If primary gets no answer or problem from secondary, primary send no to client
10. If client gets a 'no' from primary, client must restart record append operation
11. 


