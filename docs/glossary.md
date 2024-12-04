© 2025 V-Nova International. All rights reserved.  
Licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0).

If you use or adapt this document, please include the following attribution:
"Adapted from the V-Nova Coding Standard, © [Year] V-Nova International. Licensed by V-Nova Limited under CC BY 4.0."

##### Camel Case
The first letter of the first word is not capitalized, every subsequent word starts with a capital
letter and has no symbols between them: `anExampleOfCamelCase`

##### Pascal Case
Every word starts with a capital letter with no symbols between them: `AnExampleOfPascalCase`

##### Snake Case
Every word is in lower case with underscores between them: `an_example_of_snake_case`

##### UPPER_CASE
Every word is in upper case with underscores between them: `AN_EXAMPLE_OF_SNAKE_CASE`

##### Translation Unit
A basic unit of compilation. It consists of the contents of a source file and the contents of any
header files directly or indirectly included by it.

##### RAII
**R**esource **A**quisition **I**s **I**nitialization is the paradigm that an object's resources
(compositional members) are bound to the object's lifetime. They should be acquired on construction
and released on destruction.

##### Virtual Table
A lookup table of function pointers used to resolve function calls at runtime.

##### Trivially Copyable

The only trivially copyable types are scalar types such a `int`, `float` and pointers. Arrays and
classes are trivially copyable only if they are composed of other trivially copyable types.

A class is only trivially copyable if it does not have virtual member functions or a non-default
copy constructor.

##### Copy elision
Also referred to as return value optimization (RVO).

Under certain conditions compilers are required to omit the copy and move construction of a class,
even if they have observable side-effects. This is achieved by the object being constructed directly
where they would have been copied to. For example, a function's return value will be constructed on
the stack above.

##### Internal Linkage
Anything whose scope does not go beyond its own [translation unit]

#### Ownership
Ownership of an instance of a type refers to the responsibility for managing that instance's lifetime.
For example, if an instance of something is created on the stack, typically the enclosing scope owns
that instance, however, if something is created with `new`, the ownership of that instance has no
implicit owner and must be managed manually.

[translation unit]: glossary.md#translation-unit
