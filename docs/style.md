© 2025 V-Nova International. All rights reserved.  
Licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0).

If you use or adapt this document, please include the following attribution:
"Adapted from the V-Nova Coding Standard, © [Year] V-Nova International. Licensed by V-Nova Limited under CC BY 4.0."


# Introduction

This style guide standard intends to establish a common set of style and formatting rules so that
our code remains consistent between all teams and projects. This helps avoid context switching for
engineers while also allowing them to focus less on how formatting and more on functionality.

# Tooling

If you are the maintainer of a project, and that project does *not* follow the standard style, then
you are expected to provide your own `.clang-format` and `.clang-tidy` files for your project.
Every popular IDE has either built in support for both tools or has a plugin that can provide this
support.

If your project *is* following the standard style, then you can use the pre-established ones [here](configs).

If you are introducing them for the first time then the project should be formatted in the same
merge they are introduced.

# Spelling

All code and comments should use American English spelling. Most third party libraries use American
English and by using American English a line of code calling these third party libraries has
consistent spelling, avoiding a context switch.

Additionally, as V-Nova is a multicultural company, many people do not speak English as their
first language. These people have typically learned American English. By sticking to American
English we make it easier on these people, who may not be as familiar with the nuances of
British vs American English.

## Language

- Code and comments should only use language suitable for a professional environment. Do not use
  expletives or provocative language.

- Do not use naming or phrasing that could be offensive or upsetting to some. An example of this
  would be Master/Slave. It's free to use different language and helps create a more inclusive
  environment.

## Non-ASCII Characters

Non-ASCII characters should be rare and if present should use UTF-8 formatting.

# Formatting
## Line Length

No line should exceed roughly 100 characters.

The limit of 100 can be exceeded when the excess characters are mostly syntactical, such as closing
parenthesis or braces. This is to avoid forcing a second line just because the closing parenthesis
is the 101st character.

The main reasons for the line limit is:
- Allow viewing one or many files simultaneously without needing to scroll horizontally.
- When the limit is met it serves as an indication that a line is potentially overly complex and
  should be broken down.

## Naming

Specific rules on naming will be outlined for every type of identifier in their own sections. To
avoid repetition these are the rules that are applied globally:

1. Names should be descriptive, this means no single letter names beyond the notable exceptions of
   `i`, `j` and `k` for loops.
2. Do not abbreviate words by selectively removing letters. An exception is when the abbreviation is
   common such as "config", "tmp" or "img" .
3. Do not use uncommon acronyms. Generally a good test is whether the acronym appears on the
   subject's Wikipedia article.

### File Names

File names should be all lower case with words separated by underscores. Header files should use
the `.h` extension and source files should use the `.c` for C and `.cpp` extension for C++ files.
Where C++ files define classes, the name of the file should match the name of the class being
defined.

e.g. my_class.h should define `MyClass` and declare its functions, and my_class.cpp should define
those functions.

### Exceptions

Naming rules may be deviated from, but only when necessary for a program to function correctly, such
as specializing a Standard Library or third-party template.

## Whitespace
### Horizontal whitespace

Only use spaces, indenting 4 spaces at a time. There should be no trailing whitespace on any line.

The specifics of when to use whitespace in a line are quite verbose. A full list can be found in the
clang-format file. In summary:

- Insert a space between separate symbols:
    - bad: `int a+=5*2;`,
    - good: `int a += 5 * 2;`
- Insert a space after control statements:
    - bad: `if(someCondition)`
    - good: `if (someCondition)`
- Insert a space after a comment specifier:
    - bad: `//example`
    - good: `// example`
- Insert a space after a template declaration:
    - bad: `template<typename T>`
    - good: `template <typename T>`
- Align trailing comments vertically
- Don't insert a space between paired symbols, i.e.:
  ```c++
    // BAD                      // GOOD
    template < typename T >     template <typename T>
    void function( int a );     void function(int a);
    int a[ 2 ] = { 1, 2 };      int a[5] = {1, 2};
  ```

### Vertical Whitespace

Any line break should be a LF (`\n`). Do not use CRLF (`\r\n`).

There should be at most one consecutive blank line. Try to break your code into paragraphs,
separating each prose with a single blank line.

Some further rules to keep in mind:
- Do not start or end a new block with a blank line.
- Always insert a blank line before a comment.
- Insert a blank line between a namespace declaration or a block of namespace declarations and then
  enclosed symbols.

## General Syntax

Your code should use the minimal amount of syntax needed to make an expression functionally correct.
Redundant syntax should only be used if it improves the overall readability.

```c++
const auto y = (((a * x * x) + (b * x)) + c); // BAD: Superfluous syntax inhibits readability
const auto y = a * x * x + b * x + c;         // OK: Minimal syntax but hard to read.
const auto y = (a * x * x) + (b * x) + c;     // GOOD: Redundant syntax but more readable
```

## Variables
### Naming
#### Local Variables

Local variables should be in [camel case].

```c++
const char* exampleString = "example";
```

#### Global Constants

Global constants should be prefixed with a `k` and be in [pascal case].

```c++
static const int kExampleConstant = 42;
```

#### Member Variables

Member variables should follow the same naming rules as their non-member counterparts. Private
member variables, however, should be prefixed with `m_`.

```c++
class ExampleClass
{
    static const int kMeaningOfLife = 42;
public:

private:
    int m_examplePrivateMember;
};

struct ExampleStruct
{
    int intValue;
    float floatValue;
};
```

### Formatting

#### Pointers and references

Pointer and reference modifiers should be left aligned, immediately following type. There should be
no space before the asterisks or ampersand.

```c++
int x = 0;
int* examplePointer = &x;
int& exampleReference = x;
```

#### Const and Volatile

`const` and `volatile` should always be placed in their left most position

```c++
const int example = 0;
const int* exampleConstPointer = &example;  // int is immutable, variable is mutable
const int& exampleConstReference = example; // int is immutable, variable is mutable
int* const constExamplePointer = &example;  // int is mutable, variable is immutable
int& const constExampleReference = example; // int is mutable, variable is immutable
const int* const constExampleConstPointer = &example;  // int is immutable, variable is immutable
const int& const constExampleConstReference = example; // int is immutable, variable is immutable
```

#### Declarations

Each variable should be declared on its own line. Do not declare multiple variables in a single statement.

```c++
// BAD
int a = 1, *b = nullptr;

// GOOD
int a = 1;
int* b = nullptr;
```

## Functions

These rules apply to any function and function calls, this includes:
- Functions
- Member functions 
- Lambdas

### Naming

All functions should be in [camel case]. Generally function names should contain a verb i.e. `get`
or `find`.

```c++
class Person
{
public:
    const char* getName() const;
};

Person& findPerson(const std::vector<Person>& people, std::string_view name);
```

#### Parameter names

Parameter names should follow the naming rules for local variables. If the function definition and
declaration are separate then ensure that parameter names are consistent between the two.

### Formatting
#### Declarations and Definitions

- The return type should be on the same line as the function name
- Braces should be on their own lines
- If the arguments do not fit on the same line as the function name they should wrap to a new line
  and be vertically aligned to the first argument.
- There should be no blank lines at the top or bottom of the function
- Empty functions should have their braces on the same line

```c++
void example(int parameter)
{
    // contents
}

void AnExcessivelyLongClassName::anExtremelyLongFunctionName(int first, int second, int third,
                                                             int fourth)
{
    // contents
}

void exampleEmptyFunction() {}
```
#### Template Functions

- Template definitions should be on their own line immediately before the function definition
- There should be a single space between the `template` keyword and the opening angled bracket.
- If the template arguments do not all fit on the same line as the `template` keyword then they
  should wrap to a new line and be vertically aligned to the first argument.

```c++
template <typename T>
void example(T t)
{
    // contents
}

template <typename T, typename... Args,
          typename = std::enable_if_t<std::is_same_v<T, MySuperSpecialType>>>
void exampleLongTemplateArguments(T t, Args&&... args)
{
    // contents
}
```


#### Function Calls

A function call should have all the arguments on the same line where possible.

If the arguments do not fit on the same line then they should be put on following line below,
aligned to the opening parenthesis.

If the first argument does not fit on the same line then the arguments should start on the line
below, indented with 4 spaces.

```c++
auto result = example(1);
auto result = anExteremelyLongFunctionNameWithLotsOfArguments(aVeryLongVariableName, argument2,
                                                              argument3);

if (...) {
    if (...) {
        auto result = aLongObjectName->anExteremelyLongMemberFunctionWithLotsOfArguments(
            aVeryLongVariableName, argument2, argument3, argument4);
    }
}
```

## Inline Functions

A function should only be declared inline if it is small (no more than ten lines). This includes
stand alone and member functions. Generally a function should not be forcibly inlined through
compiler specific extensions.

In scenarios where a function is implicitly inline, such as it being defined in a class body, the
`inline` specifier should be omitted.

## Lambdas
### Naming

Lambdas should follow the same naming rules as functions.

### Layout

- There should be no space between the capture list and the argument list
- There should be a single space between each element of the capture list
- Short lambdas (a single expression) may be on a single line
- There should be a single space before the opening brace
- For by-reference captures there should be no space between the ampersand and the variable:
- For explicit return types there should be a space single space before and after the `->`
- Multi-line lambdas should be formatted like a multi-line function

```c++
int x = 0;
int n = 5;
auto example = [&x, n]() -> int { return x + n; }
auto longerExample = [&x, n](SomeLongClassName& a, AnotherLongerClassName& b) {
    a.DoSomething();
    b.DoSomething();
};
```

## Types

The following set of rules apply to any custom type declarations, this includes:
- Classes
- Structs
- Enumerations (`enum` and `enum class`)
- Template Parameters
- Type aliases (`typedef` and `using`)
- Unions

### Naming

Type names should be in [pascal case]. Ideally type names should be nouns or noun phrases.

```c++
class Encoder {};
struct EncoderConfig {};
enum /* class */ Format {};
template <typename T>
using PropertyMap = std::map<std::string, Property<T>> {};
```

### Formatting

#### Whitespace

- The opening and closing braces should be on their own lines
- Access modifiers (`public`, `private` and `protected`) should be on their own line at the same
  indent level as the braces
- There should be an empty line before the access modifier if it is not appearing at the top of the
  class body
- There should be no empty line after the access modifiers
- The contents of the type should be indented one level above the braces
- There should be at most one definition per line
- Templates should be on their own line immediately preceding the type

```c++
template <typename T>
class Example
{
public:
    Example();

private:
    int m_member;
};
```

#### Class Layout

A class should follow the below ordering as best as possible:
- Static constants
- Nested types
- The public constructors and assignment operators
- The public destructor
- Public member functions
- Public member variables
- Protected member functions
- Protected member variables
- Private member functions
- Private member variables

#### Initializer Lists

- Indented one level above the constructor
- New line before the colon
- First member on the same line as the colon
- Every subsequent member on its own line with a leading comma

```
Constructor()
    : m_firstMember{1}
    , m_secondMember{"example"}
{}
```

#### Inheritance Lists

If there is a only one base class inherited it may be on the same line as the class name, otherwise:

- There should be a new line before the colon
- The inheritance list should be indented one level above the class
- The first base class should be on the same line as the colon
- Every subsequent base class should be on its own line with a leading comma

```c++
class Example
    : public Base1
    , public Base2
{}
```

## Extern blocks
### Formatting
#### Whitespace

Extern blocks, like top-level [namespaces](#namespaces), do not need to be indented. This is
because we very frequently wrap entire files, namely c interfaces, in `extern "C"` blocks.

```c++
#ifdef __cplusplus
extern "C"
{
#endif // __cplusplus

struct ExampleStruct
{
    int variable;
};

void exampleFunction(ExampleStruct* ptrToStruct)
{
    if(ptrToStruct != nullptr) {
        printf("%d\n", ptrToStruct->variable);
    }
}

#ifdef __cplusplus
}
#endif // __cplusplus

```

## Namespaces
### Naming

A namespace should be in [snake case], preferably only a single word. If you need to use multiple
words that generally, but not always, indicates that there should be a nested namespace.

### Trailing Comment

The closing brace of a namespace should indicate which namespace it closes with a comment of the
format `// namespace <namespace name>`.

```c++
namespace vnova {
} // namespace vnova
```

### Formatting
#### Whitespace

- There should be no new line after the start of the namespace
- There should be a single new line after the closing brace of a nested namespace
- The opening brace should be on the same line as the namespace.
- Namespaces after the top level namespace should have their contents indented

```c++
namespace vnova {
namespace {
    void nestedExample();
} // namespace

void topLevelExample();
} // namespace vnova
```

### Compact Nesting

Namespaces that have the same scope should be compacted to a single line. If using C++17 then use
the nested namespace syntax instead.

```c++
// pre C++17
namespace vnova { namespace util {
}} // namespace vnova::util

// post C++17
namespace vnova::util {
} // namespace vnova::util
```

## Control statements

The following set of rules apply to any control statement, this includes:
- `if` and `else`
- `for`
- `while`
- `do while`
- `switch`
- `try` and `catch`

### Formatting

- The braces should never be omitted
- The body of the control statement should never be on the same line as the control statement

```
while (true) {
    if (const int next = f(); next > 20) {
        break;
    }
}
```

#### Whitespace

- There should be a space after the control statement
- There should be no new line before the opening brace with a single space immediately preceding it
- The closing brace should be on its own line or the same line as the paired `else` if it is present
- There should be a space between any init statement and the condition

```c++
if (statement) {
    if (const int x = f(); x > 3) {
    }
} else {

}
```

#### Switch cases

- The case label should be one level of indent above the switch
- There should be at most one case label per line
- Short case bodies may be on the same line as the case
- Cases with more than one line, other than `break`, should be enclosed in braces
- If a case is given a scope the opening brace should be on the same line as the the label, the
  closing brace on its own line.

```c++
switch(example) {
    case 1: return 5;
    case 2:
    case 3: return 10;
    case 4: break;
    case 5: doSomething(); break;
    case 6: {
        int x = f();

        if (x > 10) {
            return 15;
        }

        break;
    }
}
```

##### Fallthrough

Deliberate fallthrough should be marked with the C++17 attribute `[[fallthrough]];` - if your
project is using an earlier version of C++ then it should be marked with a `// fallthrough` comment.

```c++
// C++17 onwards
switch(example) {
    case 1: [[fallthrough]];
    case 2: return 0;
}

// Prior to C++17
switch(example) {
    case 1: // fallthrough
    case 2: return 0;
}
```

#### Condition statements

If your condition statement exceeds the maximum line length, consider breaking out individual
parts into their own variables. If this cannot be done then the line breaks should occur after the
logical operators and the following line should be indented one level above the control statement.

```c++
if (firstVariable >= secondVariable &&
    thirdVariable == fourthVariable &&
    someBooleanVariable) {
    // contents
}
```

## Macros and Preprocessor Directives
### Naming

Macros used as part of the code should be in [CamelCase] with  a `VN` prefix.
Macros that are for private use within other macros should have a  `_VN` prefix:

```C++
#define _VNConcatDo(a, b) a##b
#define VNConcat(a, b) _VNConcatDo(a, b)
````

### Include Guards

Include guards should be in [UPPER_CASE] or the form `VN_<Project>_<Module>_<Filename>, e.g.:

```c++
#ifndef VN_LCEVC_UTILITY_BASE_DECODER_H
#define VN_LCEVC_UTILITY_BASE_DECODER_H

...

#endif` 
```

### Formatting

- There should be no space after the `#`
- They should not be indented
- When using `#ifdef` then the closing `#endif` should have a trailing comment with the name of
  macro the `#ifdef` was checking.
- Multiline macros should:
    - Start on the line after the `#define`
    - Indented one level above the `#define`
    - Have the trailing `\` vertically aligned one space after the longest line

```c++
#define VNExample

#ifdef VNExample
#define VNConditional 1
#endif // VNExample

#define VNExampleMultilineMacro(op)                       \
    if ((op) < 0) {                                       \
        VNLog(LogLevel::Error, "Something went wrong\n"); \
        return false;                                     \
    }
```

## Includes
### Formatting

Includes should be split into distinct blocks that are sorted alphabetically and case-
insensitively. For the avoidance of doubt: short words precede long ones, and underscores precede
alphanumeric characters.

Each block should be separated by a single new line.

1. If the file is the source for a header file, the first include should be that header.
2. Local/private headers, i.e. anything in the format `#include ".*"`
3. Third-party library headers
4. System includes

## Comments

Comments should only be present if they add value to the code. For example:

- Don't add horizontal rules (i.e. `=` or `-` spanning the full line), these are just noise
- Don't add author comments, these quickly rot and version control software provides a better
  alternative
- Do not state the obvious, i.e.
```c++
// Finds the element in the vector -- BAD: States the obvious
if (std::find(std::begin(values), std::end(values), value) != std::end(values)) {
    process(value);
}

// Processes the value unless it was already processed -- OK
if (std::find(std::begin(values), std::end(values), value) != std::end(values)) {
    process(value);
}

// Better
if (!isAlreadyProcessed(value)) {
    process(value);
}
```

### Formatting

- Prefer `//` as it is much more common, however, `/* */` is okay so long as you are consistent.
- Do not exceed the line length
- Add a single space after the comment specifier
- Prefer to put the comment before the line it refers to, as opposed to trailing it

```c++
// This is an example variable
int example;

/* This is an example function */
void doSomething();
```

### Todo Comments

Todo comments should start with the word `TODO` in all caps followed by some identifying
information, such as a name or ticket number, that gives context to the TODO.

```c++
// TODO(James Maddison): An example of a todo comment
// TODO(XYZ-1234): An example of a todo comment with a ticket
```

[configs]: configs/
[camel case]: glossary.md#camel-case
[pascal case]: glossary.md#pascal-case
[snake case]: glossary.md#snake-case
[UPPER_CASE]: glossary.md#upper_case
