# Data Processing

## Implicit Sequences

Lazy computation involves the delay in the computation of a value until that value is needed. 

An example is computing values on demand rather than retrieving them from an existing representation.

### Iterators

An iterator is an object that provides a way to provide sequential values one by one.

An iterator has two components:
- a mechanism for retrieving the next element in the sequence
- a mechanism for signaling the end of the sequence has been reached

Iterators are useful because the underlying sequence may not be represented explicitly in memory.

Elements may be computed on demand.

Sequential access is not as flexible as random access but is often sufficient.

### Iterables

Any value that can produce iterators is called an iterable.

Iterables include sequences and containers.



## Source

composingprograms.com
