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
