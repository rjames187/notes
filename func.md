# Functional Programming Notes
In functional programming there is no mutation of variables.

Everything is immutable.

Functional programming is a programming paradigm intended to make code cleaner, like to object-oriented programming.

## First Class Functions
A language has first class functions if functions can be stored in variables, passed as arguments, or returned. \
A higher order function is a function that takes a function as an argument or returns a function.

## Pure Functions
A pure function is deterministic and has no side effects.

A pure function cannot ...
- mutate variables outside of the function scope
- reference variables outside of the function scope
- have an I/O operation

It often makes sense to use pure functions because they make code easier to reason about.

## Functional Transformations
A functional transformation is a higher order function that takes one or more functions as input and returns a function.

Functional transformations can reduce code duplication.

## Closures
A closure is function that references variables from outside its function scope.

Closures are stateful.

## Currying
Currying is a functional transformation that translates a function with multiple parameters into a sequence of single-parameter functions.

Currying can be useful for satisfying a particular function signature.

## Sum Types
In functional programming, it is good to limit the amount of possible values a variable can have.

Product types like arrays and maps can have many possible values.

Sum types like booleans have a limited number of specific values.

An enum is basically a sum type.

