# Don't Settle for Eventual: Scalable Causal Consistency for Wide-Area Storage with COPS

## Introduction

Ideally, data stores would always be strongly consistent, totally available, and partition-tolerant.

According to CAP theorem, all three are not possible. 

Web services have largely chosen availability for low latency.

Properties of ALPS systems:
- Availability
- low Latency
- Partition-tolerance
- high Scalability

Causal consistency with convergent conflict handling may be the strongest form of consistency possible within the ALPS constraints.

Convergent conflict handling ensures replicas never permanently diverge.

Causal+ consistency ensure clients only see preogressively newer versions of keys.

This paper's COPS (Clusters of Order-Preserving Servers) provides causal+ consistency.

It provides linearizability within datacenters and causal consistency between data centers.

## Causal+ Consistency

Rules defining potential causality between operations:
- Execution Thread: if a and b are operations in the same execution thread
- Gets From: If a is a put operation and b is a get operation that returns the value written by a
- Transitivity: If a comes before b and b comes before c then a comes before c

### Definition

Causal consistency does not order concurrent operations which allows for increased efficiency.

However, if the same key is written concurrently, there is a conflict. 

Conflicts are undesirable because they can led to permanent replica divergence.

Convergent conflict handling requires an arbitrary handler function.

One way to handle conflicts is last-writer-wins.

COPS can be configured to use either the last-writer-wins rule or an application-defined resolution procedure.


