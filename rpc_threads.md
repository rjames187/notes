# RPC and Threads

Go is a great language for building distributed systems:
- Good support for threads, locking, and synchronization
- Convenient RPC library
- Type-safe, memory-safe, and garbage-collected

## Threads

Concurrency is important in distributed systems because a program may be talking to many other programs.

Reasons to use threads:
- I/O Concurrency: overlapping of activities
- Parallelism: use of multiple cores
- Convenience (such as running something in the background)

When two threads mutate the same data at the same time it can lead to unintended behavior (race).

Races can be prevented by inserting locks into the code. 

Coordination is when threads interact or wait for eachother.

Techniques for coordination in Go:
- channels
- condition variables (```sync.Cond```)
- Wait group (good for launching an arbitrary number of goroutines and waiting for all to finish)

A deadlock occurs when two threads are unable to release their locks because they are waiting for eachother.

