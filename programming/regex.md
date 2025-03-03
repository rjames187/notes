# Regular Expressions

Regular expressions are a declarative way to match patterns

## Syntax

Special characters in regex: \ ( ) [ ] { } + * ? | $ ^ .

A preceding \ can escape special characters.

The . character matches any single character that is not a newline.

[] matches characters in a set. [^] matches characters outside of the set. 

\d matches any digit character. \D matches any non-digit character. 

\w is equivalent to [A-Za-z0-9_]. \W matches the inverse.

\s matches any whitespace character. \S matches the inverse.

\* matches zero or more of the previous pattern.

\+ matches one or more of the previous pattern.

? matches zero or one of the previous pattern.

{min, max} matches a quantity between min and max (inclusive) of the previous pattern.

p1|p2 matches p1 or p2.

() is used for grouping.

^ matches the beginning of a string.

$ matches the end of a string.

\b matches the beginning or end of a word.



