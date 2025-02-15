# Building Abstractions with Data

Native data types evaluate to values of native data types called literals.

There are built-in functions and operators to manipulate the values of native types.

## Data Abstraction

An abstract data type allows the manipulation of compound objects as units. 

Data abstractions: functions enforce an abstraction barrier between the representation/implementation and use of data.

## Sequences

A sequence is an ordered collection of values.

Common behavior of sequences:
- finite length
- every element corresponds to a non-negative integer index less than the sequence's length (starting at zero)

Examples of sequences:
- lists
- ranges
- tuples
- arrays
- strings
- linked lists (non-native)

## Mutable Data

Instances of primitive built-in objects are typically immutable.

Mutable objects represent values that change over time.

## Object-Oriented Programming

Like functions, classes create abstraction barriers between the use and representation of data.

Objects respond to behavioral requests and have local state.

Objects abstract the complexity of both state and behavior.

A class is a template. An object is an instance of a class.

## Multiple Representations

Abstraction barriers enable the separation of the use and representation of data.

It can be useful for a system to deal with multiple representations of data.

Multiple representations are necessary when working on large software systems with a lot of people for a long time because it is not possible for everyone to agree in advance on data representation.

Abstraction barriers can also isolate design choices from eachother allowing them to coexist.

An interface is a set of shared attribute names with behaviorial specifications.

## Efficiency

Different ways of representing and processing data have different efficiencies (measures of time and memory required for computation)

Determining how much space and time a program uses is difficult because it depends on many factors

It is easier to categorize processes based on how the resource requirements grow as a function of input




