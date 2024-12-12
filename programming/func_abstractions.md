# Building Abstractions with Functions

Fundamental parts of computing:
- Representing information
- Specifying logic to process it
- Designing abstractions to manage the complexity of the logic

An **expression** is typically a computation.

A **statement** is typically an action.

A **function** encapsulates logic that manipulates data.

An **object** bundles together data and the logic that manipulates that data, managing the complexity of both.

Expressions are evaluated while statements are executed and change the interpreter's state.

Principles of debugging:
- Test incrementally (individually test small, modular components)
- Isolate errors
- Check your assumptions

Programs are written for people to read and for machines to execute.

An assignment statement binds a name to a value. Assignment is a simple abstraction.

The environment keeps track of names, values, and bindings.

Bindings are often called variables.

A function definition can bind a name to a compound operation.

When a function is executed, the arguments are bound to the names of the function's formal parameters in its local frame.

When a name is accesible to a frame, it is said to be in-scope.

Aspects of a functional abstraction:
- Domain: set of arguments it can take
- Range: set of values it can return
- Intent: relationship it computes between inputs and outputs (and any side effects)

Principles for designing functions:
- Each function should have one job identifiable with a short name
- Create functional abstractions instead of coding redundant logic
- Functions should be defined generally

Doc strings may describe the job, arguments, and behavior of a function.



## Source:

https://www.composingprograms.com/pages/11-getting-started.html
