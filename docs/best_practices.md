© 2025 V-Nova International. All rights reserved.  
Licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0).

If you use or adapt this document, please include the following attribution:
"Adapted from the V-Nova Coding Standard, © [Year] V-Nova International. Licensed by V-Nova Limited under CC BY 4.0."

# Introduction

This document outlines best practices to ensure code quality, readability, and maintainability across V-Nova projects. 
All engineers are expected to adhere to these guidelines as part of the V-Nova Coding Standard.

The main focus of this standard will be on C/C++, and *all* rules apply to C++. If a rule *can* 
also apply to C (e.g. [`const` by default](#bp1-const-by-default)), then assume it *does*, *unless*
the body of the rule makes a specific distinction (e.g. rules regarding
[macros](#bp7-avoid-using-macros) and [`goto`](#r5-never-use-goto-in-c-and-resist-it-in-c)). For
some rules (e.g. [never use C-style casts](#r13-never-use-c-style-casts)), you can use your common
sense to determine what language to apply it to.

Most languages other than C/C++ have predetermined industry standards and conventions, which you
are encouraged to follow for those languages.

## Aims and Intent

The main goals of the best practices are:

- Making code explicit
    - When code is explicit in its intent it makes it easier to identify exactly what the intended
      behavior is. This makes it easier for both the reader *and* the compiler to find unintended
      behavior.
- Promote runtime bugs to compile time bugs
    - The more code fails at compile time the fewer bugs you have at runtime.
- Standardized code
    - Simply put if all projects under the V-Nova umbrella follow the same rules then it makes it
      easier for any engineer to drop into any project and focus only on the problem they are there
      to solve.

The coding standard will specifically try and avoid removing any of an engineer's autonomy with
regards to architectural decisions. To this end it will try to avoid making rules around this
subject and where it does, the rules will be suggestions rather than requirements. That said there
are a small number of things this standard explicitly disallows as they are largely considered
harmful practices across the industry.

# Bug Prevention

## BP.1 `const` by default

Every variable you define should be declared `const` unless it needs to be mutable. The reasons for
this are three-fold:

1. It protects you from accidentally modifying something that you didn't intend to modify. Promoting
   runtime bugs to compile time errors.

   The below is fine if `x` and `y` are non-`const` but will fail at compile time if `x` is `const`.
    ```c++
    if (x = y)
    ```

2. It makes the code easier to follow, the reader can understand the meaning of a variable by its
   definition. They know it cannot change, and therefore they know what that variable represents for
   the rest of the scope.

3. It makes interfaces explicit. If you see the function declaration: `void func(const std::string& str);`
   The reader can instantly infer that `func` will not modify `str`.

## BP.2 Prefer braced initialization

C++11 introduced the syntax `std::string str{"hello"};` in addition to the usual
`std::string str("hello");` with one key difference: braced initialization prohibits implicit casts:

```c++
class Foo
{
public:
  Foo(float x) : m_x{x} {}

private:
  float m_x;
};

void example()
{
  int x = 1;

  Foo a{x}; // error: non-constant-expression cannot be narrowed from type int to float
  Foo b(x); // no error
}
```

This is preferred because it prevents accidental conversions between types. It forces the writer to
explicitly cast `x` to `float` which means nothing is hidden from the reader.

There is a small caveat to this syntax that you must be aware of. If the type is constructable
from a `std::initializer_list<T>` this syntax will prefer that constructor. You will typically encounter 
this issue in the Standard Library containers such as `std::vector`.

For example, trying to construct a vector, which contains 3 integers of value `0` using braced
initialization lists: `std::vector<int> vec{3, 0};`. This will instead construct a vector containing
the values `3` and `0`.

## BP.3 Avoid default in enum switches

When a switch is being used to perform some operation based on an enumeration you should avoid using
a default case. This is due to the fact that the compiler can detect whether all enumerations are
being handled, and if one is not being handled, it will produce a compile time error. If a switch
has a default case, then from the compiler's point of view, all enumerations are being handled and so
it cannot produce an error.

Consider the following:
```c++
enum class Color
{
    Red,
    Green,
    Blue,
};

const char* ToString(Color color)
{
    switch(color) {
        case Red: return "Red";
        case Green: return "Green";
        case Blue: return "Blue";
        default: break;
    }

    return "Unknown";
}
```

If we introduce a new enumeration `Color::Yellow` the code will compile fine. If we then call
`ToString(Color::Yellow)` we get the value `Unknown`, which is incorrect. We are able to compile the
code with a bug. If we remove the default case the code will not compile with sensible flags.

## BP.4 Never mix signed and unsigned arithmetic

It is extremely easy to introduce hard to find bugs caused by mixing signed and unsigned arithmetic.
If a signed value appears in an expression with an unsigned value it will be promoted to an unsigned
value and all arithmetic will be unsigned. Prefer to use signed values if there is any subtraction
involved as this is how we, as humans, expect numbers to behave.

For example:
```c++
#include <iostream>

int main()
{
    int a = -3;
    unsigned b = 7;

    std::cout << a + b << "\n"  // 4, Correct, a is converted to 4294967293, adding b then overflow to 4
              << a - b << "\n"  // possibly 4294967285 (wrong)
              << a * b << "\n"; // possibly 4294967275 (wrong)
}
```

If `a` and `b` were not so clearly defined, this would be much harder to identify.

## BP.5 Always initialize variables

Using an uninitialized variable is undefined behavior. To make matters worse a lot of compilers
will initialize a variable to 0 in debug mode if it is left uninitialized, meaning a bug is only
present in release mode. To avoid these issues, all variables should *always* be initialized.

## BP.6 Use include guards

The contents of a header should always be encapsulated in include guards, this prevents the contents
from being defined twice if it is included in multiple headers. The define should not start with
underscores as these names are reserved by the standard, instead it should be of the format
`VN_<MODULE>_<FILENAME>_H_`.

```c++
#ifndef VN_CODING_STANDARD_EXAMPLE_H_
#define VN_CODING_STANDARD_EXAMPLE_H_

// Contents

#endif // VN_CODING_STANDARD_EXAMPLE_H_
```

Some implementations offer vendor extensions like `#pragma once` as alternative to include guards.
It is not standard and it is not portable. It injects the hosting machine’s filesystem semantics
into your program, in addition to locking you down to a vendor and therefore should not be used.

## BP.7 Avoid using macros

Macros are hazardous for a number of reasons:
- They don't obey scopes
- They don't obey rules for passing arguments
- They don't obey type rules

If you are using a macro for something other than source control, there is almost always a better
alternative, i.e. inline functions, constant variables and enums. This is especially true for C++,
where you can use templates for type-agnostic functions, and [RAII] for boilerplate initialize/
release pairing. In C, there are usually alternatives too, but it may require judgement to decide
if those alternatives are preferable.

For example, macros as constants:
```c++
#define PI 3.14
// Should become (in C++)
inline constexpr double PI = 3.14;
// Or (in C)
static const double PI = 3.14;
```

Macros as type-agnostic functions:
```c++
#define Mul(a, b) ((a) * (b))
// MAY be acceptable in C, but should become (in C++)
template <typename T>
constexpr auto Mul(const T& a, const T& b)
{
    return a * b;
}
// Or PERHAPS (in C)
uint8_t MulU8 (uint8_t a, uint8_t b) { return a * b; }
uint16_t MulU16 (uint16_t a, uint16_t b) { return a * b; }
uint32_t MulU32 (uint32_t a, uint32_t b) { return a * b; }
```

Macros as a substitute for RAII:
```c++
#define VN_TIME_FUNCTION(fn)   \
    Stopwatch stopwatch;       \
    stopwatchStart(stopwatch); \
    fn;                        \
    stopwatchStop(stopwatch);
// MAY be acceptable in C, but should become (in C++)
class Stopwatch{
    Stopwatch() { start(); }
    ~Stopwatch() { stop(); }
};

void fn()
{
    Stopwatch stopwatch;
    ...
}
// Or PERHAPS (in C, using `exit` as a `goto` label)
void fn()
{
    Stopwatch stopwatch;
    stopwatchStart(stopwatch);
    ...
exit:
    stopwatchStop(stopwatch);
}
```

If you do find yourself in a situation where you absolutely must use a macro, strongly prefer
a function-like macro. Function-like macros will raise compile time errors if they are used but
undefined, meaning typos do not silently get ignored.

An example of where a non function-like macro would make sense is something that stands as a keyword:

```C++
#define VNThreadLocal __thread
````

If the function-like macro takes arguments, keep in mind that *they are not evaluated like a normal
function*. What's passed to the macro will literally be pasted wherever the argument is used. To
prevent common bugs:

- If you use the argument more than once, then it should be assigned to a local temporary
- Wherever a macro's argument is used, it should be in parentheses, to prevent bugs such as the
  following:
```c++
#define VNSquare(a) a * a

VNSquare(1 + 1); // 1 + 1 * 1 + 1 != 4
```

For compile time build flags, preferably they should also be function-like macros that return 1 or
0 depending on whether the feature is enabled, Ideally following such a pattern (which require
support from the build system to generate a build configuration file).

```c++
#define VNFeature(f) VNFeature##f()
#define VNFeatureProfiler() 1
#define VNFeatureTraceLogging() 1

#if VNFeature(Profiler)
    // profiler code
#endif

#if VNFeature(TraceLogging)
    // trace logging code
#endif

#if VNFeature(Porfiler) // Error: function-like macro is not defined
#endif
```

## BP.8 Integers

Of the built in integer types the only one that should be used is `int`. If a different sized
integer is needed then the precise width integer types from `<cstdint>` (`<stdint.h>` for C) such as
`uint16_t` and `int64_t` should be used instead.

This helps the reader understand that a variable's width is important in its given context.

This does not mean you should not use types like `size_t` and `ptrdiff_t`. These are well-defined
to have a certain length depending on the word size and are the portable way to handle size
differences between 32 and 64 bit builds.

# Readability

## R.1 Don't use non-const globals

Non const globals make it hard to follow the state of a program. They make functions unpredictable,
masking behavior and potentially having side-effects on one another. Furthermore, the order that
global objects are initialized is not fixed. If you have globals initializing from other globals
you can hit undefined behavior unless you know exactly what you are doing.

## R.2 Declare variables as close to their first use as possible

This improves readability as the definition is usually on the same line, limiting the need to jump
around to find its use. Keeping a variable in the scope that it is used also prevents bugs that may
occur from the variable having some unknown state from a previous scope:

Bad:
```c++
int i;
int result;

for (i = 0; i < 10; ++i) {
    result = doSomething(i);

    // ...
}
```

Good:
```c++
for (int i = 0; i < 10; ++i) {
    int result = doSomething(i);

    // ...
}
```

## R.3 Minimize nested scopes

Nested scopes can reduce readability and make the context harder to understand. Below are a few examples
of how to minimize nested scoping:

- Combine tests

Instead of:
```c++
if (a) {
    if (b) {
        // ...
    }
}
```

Prefer:
```c++
if (a && b) {
    // ...
}
```

- `return` or `continue` early:
Instead of:
```c++
for (int i = 0; i < 10; ++i) {
    if (a) {
        // ...
    }
}

if (b) {
    // ...
}
```

Prefer:
```c++
for (int i = 0; i < 10; ++i) {
    if (!a) {
        continue;
    }

    // ...
}

if (!b) {
    return;
}

// ...
```

- Omit `else` after return:

Instead of:
```c++
if (a) {
    return /* ... */;
} else {
    return /* ... */;
}
```

```c++
if (a) {
    return /* ... */;
}

return /* ... */;
```

## R.4 Declare one variable per line

Don't use inline variable declarations, these can be hard to read and can be ambiguous when pointers
or references are involved:
```c++
int* a, b, c;
```

Here only `a` is a pointer, `b` and `c` are just `int`. Instead declaring one variable per line:
```c++
int* a;
int b;
int c;
```

## R.5 Never use `goto` in C++ (and resist it in C)

`goto` is a relic from pre-structured programming that are now largely obsolete. They negatively
impact readability and make code error prone. C++ has better, more reliable tools to handle almost
any instance that might previously have been solved with a `goto`.

A classic use of `goto` is to have a single section for cleaning up any resources allocated by the
function that is shared between the success and error paths. One problem with this approach is that -
ignoring the lack of top-to-bottom follow - it is easy to accidentally omit a `goto` statement.
Someone who is less familiar with that section of code may add another branch that conditionally
returns, not realizing that instead of returning in place they are meant to use `goto`.

C++ can solve this with [RAII]. If the resource were allocated in a `std::unique_ptr` then the
programmer no longer has to be concerned about the lifetime of the resource. It will automatically
be freed as soon as it falls out of scope, thus making it impossible for an errant return to cause
a bug.

This rule is less strict for C as it does not have alternative methods. However, `goto` should still
be avoided and only used as a last resort.

The single use-case where `goto`s may be considered useful are to break out of nested for loops,
though it would be preferred if the deep nesting is avoided.
```c++
    for(int i = 0; i < 100; ++i) {
        for(int j = 0; j < 100; ++j) {
            if (someCondition) {
                goto breakout;
            }
        }
    }

breakout:
   // ...
```

## R.6 Use reference modifiers when using `auto`

`auto` uses the same rules for type deduction as template type deduction. That means the rules used
to determine `T` in the below example, are the same ones used to determine what type `auto` should
be.

```c++
template<typename T>
void example(T value);

example(1);
```

This is important to highlight, as this set of rules for type deduction will **never** resolve to
a reference type.

For example:
```c++
std::string& getString();

void example()
{
    auto str = getString();
}
```

`str` will resolve to `std::string` instead of `std::string&`. As a result the string returned by
`getString` will be *copied* into `str`. To have `str` be a reference it should instead be declared
as `auto&`. As such it is important to always include the `&` when you intend for `auto` to be
a reference.

## R.7 Use pointer modifiers when using `auto`

Unlike reference modifiers, `auto` can and will resolve to a pointer type where sensible. Unlike
reference modifiers, however, we need to be aware of whether a variable is a pointer as it affects
how we use and access it. As such when `auto` is going to resolve to a pointer keep the `*` as
a courtesy to anyone who comes along later.

This practice, when used in conjunction with the practice on reference modifiers, also means it is
explicitly clear to a reader when `auto` is intended to be a value type.

## R.8 Prefer type aliases over `typedef`

C++11 introduced type aliases as a better alternative to `typedef`. These are preferred as they read
in the typical left-to-right order and have a consistent style between function and type aliases:

```c++
typedef int MyInt;
typedef int (*IntOperation)(int, int);
```

```c++
using MyInt = int;
using IntOperation = int (*)(int, int);
```

A further benefit is that type aliases can be templated, allowing for convenient use of a templated
type:
```c++
template<typename T>
struct ExpandedInteger;

template<>
struct ExpandedInteger<int32_t>
{
    using Type = int64_t;
};

template<typename T>
using ExpandedIntegerType = typename ExpandedInteger<T>::Type;

ExpandedIntegerType<int> x; // vs typename ExpandedInteger<int>::Type x;
```

## R.9 Headers should include everything, and only those things, that they require

A header should always include everything it needs: users shouldn't need to include something
before a header to make it work. This can be with direct `#include`s or (preferably) with forward
declarations. This is important to ensure that the order of includes does not matter.

Additionally a header should not rely on an include provided by another header (transient include).
If the header uses something it should directly include the necessary header, so that it doesn't
break if the transient include is removed.

Lastly, a header should not include things it doesn't need. Avoid "bundled" headers.

## R.10 Source files should include everything, and only those things, that they require/define

Like headers, source files should include everything that they require.

This specifically means that source files:
1. *Should not* include headers which they don't use.
2. *Should not* define declarations made in *unrelated* headers.
3. *Should* define declarations made in *their own* header.

In other words, a source file should have *one* header which could be called "its own header".
The first include of a source file should always be its own header file, to facilitate this. Note,
however, that the converse is not true: you may have different source files for the same header
(e.g. platform-specific source files).

## R.11 Use `struct` for value types, `class` for classes

`struct` should only be used for grouping relevant data together - all members should be publicly
accessible:

```c++
struct Point
{
    int x = 0;
    int y = 0;
};
```

`class` should be used for anything more complex and all members should be private, exposing them
through relevant accessors:

```c++
class Animal
{
public:
    virtual void makeSound() const = 0;

    const Point& getPosition() const { return m_position; }

private:
    Point m_position;
};
```

## R.12 Write Comments

Code should strive to be self documenting through appropriate naming, however this can only provide
so much information. Anything which cannot be inferred by reading the code without any additional
knowledge should have an appropriate comment.

For example:
```c++
class Vector
{
public:
    int getX() const { return m_x; }
    int getY() const { return m_y; }
    int getZ() const { return m_z; }

    void rotate(const Vector& normal, int angle);

private:
    int m_x = 0;
    int m_y = 0;
    int m_z = 0;
};
```

The accessor methods do not need an accompanying comment as they are self explanatory. The `rotate`
function, however, definitely needs to have a comment explaining what `normal` and `angle` are and
the outcomes.

## R.13 Never use C-style casts

C-style casts will do whatever is necessary in order to cast something to the desired type, even if
it's not what you intended. Using C++ style casts comes with several advantages:

1. It makes the intent explicit, allowing the compiler and reader to understand exactly what is
   happening.
2. As the compiler now knows the exact intent it can promote runtime bugs to compile time errors.
3. Easily searchable

Consider the following:
```c++
int64_t total = 0;

for (int i = 0; i < 100; ++i) {
    int* value = getValue(i);
    total += (int64_t)value;
}
```

The use of a C-style cast has caused a runtime bug. The address of `value` is being added to the
total, rather than the value of `value`. The C-style cast is reinterpreting the `int*` into
an `int64_t`. Had we used the appropriate `static_cast` the compiler could have thrown a compile
time error preventing the bug.

## R.14 Avoid using `reinterpret_cast`

It is very hard to use `reinterpret_cast` without triggering undefined behavior. There is a very
short list of permitted conversions in the standard. The only *safe* ways to use `reinterpret_cast`
are:

- Casting something to `std::byte*`, `unsigned char*` or `char*`, this allows anything to be viewed
  as bytes.
- Adding/removing signedness from something, i.e. `reinterpret_cast<uint32_t>(int32_t{1});` - This
  is only valid if both types have the same `const` and `volatile` modifiers.
- Casting to the same type
- Any of the above but for the pointer, array or function pointer equivalents.

The correct way to do any other sort of type conversion would be through `memcpy`. This can be done
without a performance impact as modern compilers are smart enough optimize the copy away.

## R.15 Avoid using `const_cast`

Modifying a `const` object through a non-const access path by using `const_cast` to remove the
`const` qualifier is undefined behavior.

If `const_cast` is being used in such a manner it is almost always an indicator that there is
something wrong in the bigger picture.

To illustrate the point the following is undefined behavior, the problem can be much harder to
identify in a real world scenario.
```c++
static const int kExampleConstant = 1;

int main()
{
    int& i = const_cast<int&>(kExampleConstant);

    i = 3; // undefined behaviour

    return i;
}
```

## R.16 Avoid using `dynamic_cast`

There are a couple of reasons to avoid using `dynamic_cast`:

1. It generally indicates a flaw in design, either:
    - The method you need access to on the derived class should probably be a virtual function of
      the base.
    - The place that's doing the dynamic cast shouldn't accept the base type if it actually needs
      the derived type.
2. It is not free. When you perform a dynamic cast you are actually looking up the [virtual table]
   of the object, checking whether it contains the [virtual table] of the type you are casting too
   and then returning either null or the pointer to the correct [virtual table]. This is something
   that has to happen at runtime.

# Pointers
## P.1 Use smart pointers

Prefer to use smart pointers to handle memory allocation and lifetimes of objects. Raw pointers
should be used as a nullable reference rather than an owning pointer. There are many benefits to
using smart pointers but to list a few:

- The code directly explains the lifetime of an object.

- Object lifetimes are managed through RAII. It's all too easy to miss an exit path and cause
  something to leak.

- Don't need to implement any of the special member operations to manage memory owned by a class.
  The implicitly defined operations will handle copying and freeing of the memory, allowing you to
  follow the rule of zero.

A few small notes on shared pointers:

- Prefer `std::unique_ptr` over `std::shared_ptr`, only use `std::shared_ptr` when shared
  [ownership] is absolutely necessary.

- `std::shared_ptr` should only be passed to things that need [ownership] of the object, preferring
  a raw pointer if [ownership] is not needed. When you do pass a `shared_ptr` it should be passed as
  a const reference. Passing a `std::shared_ptr` by value causes a copy and when a `std::shared_ptr`
  is copied (and subsequently destroyed) it has to update the reference counter. Passing by const
  reference avoids this.

- Use `std::make_shared` where possible. When constructed through `std::make_shared` the smart
  pointer allocates a single block of memory big enough for both the reference counter and the
  object being created. This means only one allocation occurs and the object and reference counter
  are next to each other in memory.

## P.2 Use `nullptr` not `NULL`

The main reason to not use `NULL` is that it is a macro defined as `#define NULL 0`. It has no type
safety, therefore all of the below is valid:
```c++
int x = NULL;
int* y = NULL;
```

Comparatively `nullptr` is the special `std::nullptr_t` that is only implicitly convertible to any
pointer type. As a result the compiler would give an error on `int x = NULL;`, possibly preventing
a bug. Additionally, due to `nullptr` being its own type, functions can be overloaded for (compile
time) `nullptr`s.

For example, the two `func` calls below resolve to different functions.
```c++
int func(int* a);
int func(std::nullptr_t a);

void example()
{
    int a;

    func(&a);
    func(nullptr);
}
```

## P.3 Prefer passing references if null is an error case

If a function takes a pointer to something and the first thing the function does is checks that the
given pointer is not null the function should instead take a reference. This promotes runtime errors
to compile time and makes the usage clear from the function declaration alone.

```c++
bool example(uint8_t* value); // is example(nullptr) valid?
bool example(uint8_t& value); // example(nullptr) is invalid and won't compile
```

# Classes

## C.1 Avoid Singletons

Singletons are essentially non-const globals in disguise, but can be worse as they can contain more
state than a non-const global. A singleton may seem like a good idea when initially designing
something but it can cause a lot of work down the road as soon as there is a need for a second
instance of the singleton.

Additionally a singleton, or something depending on a singleton can be very hard or nearly
impossible to write unit tests for as you cannot inject a mock object in place of the singleton.

## C.2 Overridden virtual member functions should specify either `override` or `final`

There are a growing number of ways in which a function signature can be modified in C++ and
an overridden function must match them all. Marking virtual methods as overridden has many benefits:
- Allows the compiler to validate a matching virtual function signature exists in one of the base
  classes.
- Clear indication of intent to the reader, documents that a method is virtual override
- If a change is made to a base class's virtual member functions all broken derivations will be
  caught at compile time.

Consider the following:
```c++
struct Base
{
    void f1();
    virtual void f2(int) const;
};

struct Derived : Base
{
    void f1();
    void f2(int);
};
```

This will compile fine but neither method in `Derived` overrides the inherited methods.
- `Derived::f1()` shadows `Base::f1()`, does not override. Which `f1()` is called
  depends on whether it is accessed as `Base` or `Derived`
- `Derived::f2(int)` does not override `Base::f2(int)` as it is not marked `const`.  Instead a new
  non-const method is defined.

If instead these were marked with `override` then both errors would be elevated to compile time
errors.

```c++
struct Derived : Base
{
    void f1() override;    // error: only virtual member functions can be marked 'override'
    void f2(int) override; // error: non-virtual member function marked 'override' hides virtual member function
};
```

## C.3 Declare constructors and assignment operators as `noexcept` where possible

Parts of the standard library have exception guarantees that they must meet. To ensure these
guarantees are met they will have different overrides depending on whether the templated class is no
throw constructible and assignable.

For example, `std::vector::emplace_back` is guaranteed to have no effect on the container if an
exception is thrown. To meet this `emplace_back` will prefer to copy construct if the object being
stored is not `noexcept` constructible, essentially nullifying the benefit of `emplace_back`:

```c++
#include <vector>
#include <cstdio>

struct Noisey
{
    Noisey() { printf("Constructor\n"); }
    Noisey(const Noisey& other) { printf("Copy\n"); }
    Noisey(Noisey&& other) { printf("Move\n"); }
    ~Noisey() { printf("Destructor\n"); }
};

int main()
{
    std::vector<Noisey> v;

    v.emplace_back();
    v.emplace_back();
}
```

Gives the output:
> Constructor  
> Constructor  
> Copy  
> Destructor  
> Destructor  
> Destructor  

From lines 2-4 we can see that a temporary object is created in the second `emplace_back`, copied
into the vector then destroyed.  If we instead mark the constructors as `noexcept` the copy is
replaced with a move, as excepted:

```c++
struct Noisey
{
    Noisey() noexcept { printf("Constructor\n"); }
    Noisey(const Noisey& other) noexcept { printf("Copy\n"); }
    Noisey(Noisey&& other) noexcept { printf("Move\n"); }
    ~Noisey() noexcept { printf("Destructor\n"); }
};
```

> Constructor  
> Constructor  
> Move  
> Destructor  
> Destructor  
> Destructor  

## C.4 Follow the Rule of Zero

This rule means that if you can avoid defining default operations, do. This prevents inadvertent
behavioral changes caused by explicitly defining default operations.

The default operations are:
- A constructor: `X()`, this will default initialize all members.
- A copy constructor `X(const X&)`: this will copy construct all members with their counterparts.
- A copy assignment: `X& operator=(const X&)`: this will copy assign all members with their
  counterparts.
- A move constructor: `X(X&&)`: this will move construct all members with their counterparts.
- A move assignment: `X& operator=(X&&)`: this will move assign all members with their counterparts.
- A destructor: `~X()`

Consider this small program:
```c++
#include <cstdio>
#include <utility>

class Noisey
{
public:
  Noisey() noexcept { printf("Constructor\n"); }
  ~Noisey() noexcept { printf("Destructor\n"); }
  Noisey(const Noisey& other) noexcept { printf("Copy\n"); }
  Noisey(Noisey&& other) noexcept { printf("Move\n"); }
};

class Zero
{
public:
  ~Zero() = default;

private:
  Noisey m_noisey;
};

int main() {
  Zero a;
  Zero b{std::move(a)};
}
```

On initial inspection, you see there is an object `a` being moved into another object `b`.
Therefore, you may expect the following output:
> Constructor  
> Move  
> Destructor  
> Destructor  

You may be surprised to discover that, actually, it outputs:
> Constructor  
> Copy  
> Destructor  
> Destructor  

This is due to the fact that, when a class contains a user-declared copy constructor, copy or move
assignment operator or a destructor there cannot be an implicitly declared move constructor. By
giving `Zero` a destructor, even though we declared it as default, we have removed `Zero`'s move
constructor.

If we follow the rule of zero and define `Zero` without a destructor, we get the expected output.
```c++
class Zero
{
private:
    Noisey m_noisey;
};
```

### C.4.1 Otherwise, follow the Rule of Five

Due to the behavior described in the previous section if you absolutely must implement any of the
default operations, except the main constructor, then you should implement all 5 - preferably
declaring them default where possible.

For example, assuming we absolutely must implement a destructor in our above `Zero` class, we could
have achieved the same behavior with:
```c++
class Zero
{
public:
  Zero() = default;

  Zero(const Zero&) = default;
  Zero& operator=(const Zero&) = default;

  Zero(Zero&&) noexcept = default;
  Zero& operator=(Zero&&) noexcept = default;

  ~Zero() { /* ... */ }

private:
  Noisey m_noisey;
};
```

## C.5 Be careful of self assignment in assignment operators

If you must declare an assignment operator yourself then you should ensure that you protect against
self assignment.

For example the following code would have undefined behaviour:
```c++
class Example
{
public:
    Example(uint8_t value)
        : m_value{new uint8_t{value}}
    {}

    Example& operator=(const Example& other)
    {
        // when other == this, other's m_value is also being deleted
        delete m_value;

        // and now we dereferenced the deleted memory
        m_value = new uint8_t{*other.m_value};

        return *this;
    }

private:
    uint8_t* m_value = nullptr;
};

int main()
{
    Example e{5};

    e = e;
}
```

The protection against the above error if quite simple:
```c++
    Example& operator=(const Example& other)
    {
        if (this != &other) {
            delete m_value;
            m_value = new uint8_t{*other.m_value};
        }

        return *this;
    }
```

A better solution would be the copy-and-swap idiom. This prevents bugs related to self assignment
whilst also making a copy assignment resistant to problems such as allocation failures.

```c++
class Example
{
public:
    friend void swap(Example& first, Example& second)
    {
        using std::swap; // enable ADL

        swap(first.m_value, second.m_value);
    }

    Example(const Example& other) // = default;
        : m_value{new uint8_t{other.value}}
    {}

    // Take the argument by value so a new object is constructed via the copy constructor
    Example& operator=(Example other)
    {
        std::swap(*this, other);

        return *this;
    }

private:
    uint8_t* m_value = nullptr;
};
```

## C.6 Single-argument constructors and conversion operators should be marked explicit

If you inadvertently pass arguments of the wrong type to a function the compiler will attempt to
construct an argument of the correct type from the provided argument. This means when an instance of
the correct type can be constructed or implicitly casted from the given type, it will be which can
result in hidden, unexpected behavior.

Consider the following:
```c++
class Bar {};

class Foo
{
public:
    Foo(Bar& bar);
};

class Baz
{
public:
    operator Foo();
};

void func(const Foo& foo);

void example(Bar& bar, Baz& baz)
{
    func(bar); // implicit construction of foo from bar using Foo::Foo(Bar& bar)
    func(baz); // implicit construction of foo from baz using Baz::operator Foo()
}
```

Both the calls to `func` cause an implicit construction of a `Foo` object. This can be especially
detrimental if the construction of a `Foo` is not cheap. If `example` was read in isolation, on
first inspection it would not be clear on that this conversion is happening.

If `Foo::Foo(Bar& bar);` and `Baz::operator Foo();` were marked as explicit a compile time error
would be generated if the conversion was not intentional. If it was intentional then there would
have to be an explicit construction or cast, making it obvious on first inspection what is
happening.

## C.7 If you have virtual methods, the destructor must either be virtual and public or protected and non-virtual

When you have virtual methods the destructor should be public and virtual:
```c++
class Base
{
    virtual ~Base() = default;
    // ...
};
```

or protected and non-virtual:
```c++
class Base
{
protected:
    ~Base() = default;
};
```

This is because it is undefined behavior to attempt to destroy an instance of a derived class
through a pointer to the base class (`Base* b = new Derived{};`) unless the base class has a virtual
destructor. By making the base class's destructor virtual, you avoid undefined behavior.

By making the destructor protected an instance of the base can only be destroyed by itself or
a derivation of the base, meaning it cannot be destroyed indirectly.

### C.7.1 Object Slicing

When you need to define a virtual destructor, even as default, you should still follow the [Rule of
Five] and suppress the public copy/move operations. This is to prevent a problem known as object
slicing.

Object slicing is the copying of the base portion of a derived class. For example:
```c++
struct Base 
{
    virtual ~Base() = default;
    virtual const char* name() const { return "Base"; }
};

struct Derived : Base 
{
    const char* name() const override { return "Derived"; }
};

void example()
{
    Derived d;
    Base& b1 = d;
    Base b2 = d; // Lack of reference, d was sliced

    b1->name(); // returns "Derived" as expected
    b2->name(); // returns "Base", unexpected as we copied a Derived object
}
```

The solution is to follow the Rule of Five, suppressing the copy and move operations:

```c++
struct Base 
{
    Base() = default;

    Base(const Base&) = delete;
    Base& operator=(Base&) = delete;

    Base(Base&&) = delete;
    Base& operator=(Base&&) = delete;

    virtual ~Base() = default;
    virtual const char* name() const { return "Base"; }
};

Base b2 = d; // No longer compiles
```

## C.8 Avoid making a class's members const

If a class's member that is not [trivially copyable] is marked `const` the implicit move constructor
is deleted. This is because a move is a *destructive* operation. Therefore if the object being moved
from has a `const` member then that member cannot be moved from.

```c++
#include <cstdio>
#include <utility>
#include <array>

class Noisey
{
public:
  Noisey() noexcept { printf("Constructor\n"); }
  ~Noisey() noexcept { printf("Destructor\n"); }
  Noisey(const Noisey& other) noexcept { printf("Copy\n"); }
  Noisey(Noisey&& other) noexcept { printf("Move\n"); }
};

class Example
{
    const Noisey m_noisey;
};

int main()
{
    Example a;
    Example b{std::move(a)};
}
```

> Constructor  
> Copy  
> Destructor  
> Destructor  

If a class's internal state should not be modified the class itself should be `const` and have
appropriate `const` member functions.

In the unlikely event that a member should absolutely be const, you should follow the [Rule of
five]

## C.9 Non-static member initialization

Prefer to initialize a class's non-static members with default member initialization. If the member
cannot be default initialized it should always be initialized in the member initializer list.

Default member initialization defines the default value of the member in one location, without
having to set its value in a constructor. This is beneficial for a few reasons:
- The member's initialization cannot be accidentally omitted
- If there are multiple constructors, the single assignment applies to them all, no need to repeat
  it multiple times

Member initializer lists should always be used where necessary to initialize variables that have not
been default initialized or need initializing to a different value. The construction of a class's
members happens *before* the body of the constructor. If a member is omitted from the initialization
list then it will be default constructed, if the member is then assigned to in the constructor's
body then the member would have been constructed twice. The members will be initialized in the order
they appear in the class definition, so care should be taken to ensure that the initializer list has
the same order.

```c++
// Don't do this
class BadExample
{
    std::string m_member;

public:
    BadExample()
    {
        m_member = "example"; // Member gets initialized twice
    }
};

// Slightly better
class OkayExample
{
    std::string m_member;

public:
    // Member is initialized once, but we defined a constructor just for it
    OkayExample()
        : m_member{"example"}
    {
    }
};

// Best version, prefer to write code like this
class GoodExample
{
    // member gets initialized once, no need to explicitly define a constructor
    std::string m_member = "example";
};
```

## C.10 Forward declare where possible

Forward declaring reduces compilation times. It avoids the chain reaction of one header being
modified causing many [translation unit]s to be recompiled.

Consider the following:
- `Foo.h`:
```c++
class Foo {};
```

- `Example.h`:
```c++
#include "Foo.h"

void example(const Foo&);
```

If someone were to modify the `Foo.h` in a way that is inconsequential to our example, for instance:
```c++
class Foo {};
class Bar {};
```

The compiler has no choice but to recompile any [translation unit] that includes `Foo.h` or
`Example.h`. `#include` essentially does a copy and paste so from the compiler's point of view
`Example.h` has changed, therefore anything including it has changed and must be recompiled.

If we were to instead use forward declarations:
```c++
class Foo;

void example(const Foo&) {}
```

`Example.h` no longer contains `Foo.h`. Therefore no matter how much `Foo.h` changes `Example.h`
will not change and reduce the amount of code that needs to be recompiled.

## C.11 `mutable`

`mutable` should generally be avoided with one exception: synchronization primitives. `mutable`
should be preferred over removing the `const` modifier from member variables if the only thing being
modified are the synchronization primitives.

i.e.:
```c++
// Bad: loses const correctness
class Foo
{
public:
    size_t size()
    {
        std::unique_lock lock{m_mutex};

        return m_values.size();
    }

private:
    std::mutex m_mutex;
    std::queue<int> m_values;
};

// Good: maintains const correctness without losing thread safety
class Bar
{
public:
    size_t size() const
    {
        std::unique_lock lock{m_mutex};

        return m_values.size();
    }

private:
    mutable std::mutex m_mutex;
    std::queue<int> m_values;
};
```

## C.12 Use pure virtual destructor for abstract class

Clearly mark abstract base classes as such by using a pure virtual destructor.

```c++
class AbstractBase
{
public:
    AbstractBase() = default;

    virtual ~AbstractBase() = 0;

    AbstractBase(const AbstractBase&) = delete;
    AbstractBase& operator=(AbstractBase&) = delete;

    AbstractBase(AbstractBase&&) = delete;
    AbstractBase& operator=(AbstractBase&&) = delete;
};

```

# Functions

## F.1 Keep functions simple

Prefer to have functions do as few things as possible, ideally a single logical operation.
A function neatly packages code and gives it a label. This describes the bigger picture to a reader,
rather than forcing the reader to dissect a function line by line to understand what's happening.

Consider the following:
```c++
void example(const Rect& area, const std::vector<Rect>& rectangles)
{
    std::vector<Rect> selected;

    for (const auto& rect : rectangles) {
        if (area.left() <= rect.right() && rect.left() <= area.right() &&
            area.top() <= rect.bottom() && rect.top() <= area.bottom()) {
            selected.emplace_back(rect);
        }
    }

    for (auto head = selected.begin(); head != selected.end(); ++head) {
        for (auto it = head + 1; it != selected.end(); ++it) {
            if (head->left() > it->left()) {
                std::iter_swap(head, it);
            }
        }
    }

    std::cout << "Selected rectangles from left to right are:\n";

    for (const auto& rect : selected) {
        std::cout << rect << "\n";
    }
}
```

This function is doing two things that are not immediately clear without inspection. If we move each
logical operation into their own functions the code becomes much more self explanatory:

```c++
bool overlap(const Rect& a, const Rect& b)
{
    return a.left() <= b.right() && b.left() <= a.right() &&
           a.top() <= b.bottom() && b.top() <= a.bottom();
}

std::vector<Rect> select(const Rect& area, const std::vector<Rect>& rectangles)
{
    std::vector<Rect> selected;

    for (const auto& rect : rectangles) {
        if (overlap(area, rect)) {
            selected.emaplce_back(rect);
        }
    }

    return selected;
}

void example(const Rect& area, const std::vector<Rect>& rectangles)
{
    auto selection = select(area, rectangles);

    std::sort(selection.begin(), selection.end(), [](auto& a, auto& b) {
        return a.left() < b.left();
    });

    std::cout << "Selected rectangles from left to right are:\n";

    for (const auto& rect : selected) {
        std::cout << rect << "\n";
    }
}
```

We can now clearly see that the first loop is *select*ing the rectangles in an area, the selection
happens by checking whether a rectangle *overlap*s with an area. They are then sorted from left to
right.

## F.2 Avoid return variables

When you use a return variable you are asking the compiler to do more work, making it less likely
that the compiler can optimize things. Instead prefer returning string literals or rvalues where
possible.

Consider the following:
```c++
std::string example(const bool b)
{
    std::string value;

    if (b) {
        value = "hello";
    } else {
        value = "world":
    }

    return value;
}
```

The declaration of value is telling the compiler to default construct a `std::string`. Then you are
telling the compiler to do a copy assignment.

If, instead, we returned the string literals directly:

```c++
std::string example(const bool b)
{
    if (b) {
        return "hello";
    } else {
        return "world":
    }
}
```

All you are telling the compiler to do is construct a string *with* the value "hello" or "world".
Not only have you given the compiler less steps, you've given it more detail in those steps.

Both gcc and clang generate better assembly from the second example. Neither one can optimize away
the default construction in the first example, even with max optimization.

## F.3 Never return std::move(x)

Returning from a function with `std::move(x)` impedes [copy elision] from happening. Return the
value directly form the function instead:

```c++
#include <cstdio>

class Noisey
{
public:
  Noisey() noexcept { printf("Constructor\n"); }
  ~Noisey() noexcept { printf("Destructor\n"); }
  Noisey(const Noisey& other) noexcept { printf("Copy\n"); }
  Noisey(Noisey&& other) noexcept { printf("Move\n"); }
};

// Outputs:
// Constructor
// Move
// Destructor
Noisey Bad()
{
    Noisey noisey;

    return std::move(noisey);
}

// Outputs:
// Constructor
Noisey Good() {
    Noisey noisey;

    return noisey;
}
```

## F.4 Do not use `va_args`

`va_args` are dangerous as they erase the type of the arguments. If an argument in the variadic
arguments is not convertible to the type passed to `va_arg` then the behaviour is undefined. The
compiler has no way to check this at compile time.

Instead of `va_args` you should use C++11's parameter packs:
```
template<typename... Args>
void example(Args&&... args);

example(1.0, 2u, 3.0f, false);
```

The type of each argument is not erased, as such the compiler can correctly catch any compile time
errors.

## F.5 Internal Linkage

When a function or variable is not needed outside of the source file it is defined in it should be
given [internal linkage] by placing it inside of an anonymous namespace instead of giving it the
`static` modifier. Placing a function in an anonymous namespace in a header file does not provide
internal linkage and therefore should never be present in a header file.

```c++
namespace vnova {
namespace {

// this function now has internal linkage
void example();

} // namespace
} // namespace vnova
```

# Enums

## E.1 Prefer scoped enums

Scoped enums (`enum class EnumName`) have a few major benefits over non-scoped enums (`enum EnumName`):

1. They don't pollute their namespaces.
2. They aren't implicitly convertible to their underlying type.
3. Subjectively, they can be more readable.

The exception is bitfields. This is because, with bitfields, we DO want implicit conversion, and
we'd probably otherwise be using a huge list of constants instead.

For example, AVOID this:
```c++
enum PipelineGPUBitdepth
{
    PipelineGPUBitdepth8Bit,
    PipelineGPUBitdepth16BitFloat,
    PipelineGPUBitdepth16BitInt,

    Count
};

enum Platforms
{
    Windows,
    Linux,
    Android,
    MacOS,
    IOS,

    Count // ERROR (namespace pollution): Count is now defined as both 3 (above) and 5 here.
};

class MobileDecoderSettings
{
public:
    // BUG (implicit conversion): m_bitdepth is never 16, because it's an enum from 0 to 3.
    bool isPipeline16Bits() const { return m_bitdepth == 16; }

    // BUG (implicit conversion): 2 isn't the max value anymore, this function was probably
    // written before the introduction of PipelineGPUBitdepth16BitInt.
    bool isBitdepthValid() const { return m_bitdepth < 2; }

    // BUG (readability): Someone has written this function on the mistaken belief that the enum
    // is listing types of pipeline, rather than types of GPU pipeline bitdepth. They likely
    // wrongly expect that there is some valid enum entry which doesn't have "GPU" in it.
    bool isUsingGPUPipeline() const
    {
        return (m_bitdepth == PipelineGPUBitdepth8Bit) || (m_bitdepth == PipelineGPUBitdepth8Bit) ||
               (m_bitdepth == PipelineGPUBitdepth8Bit);
    }

private:
    PipelineGPUBitdepth m_bitdepth = Count;
};
```

Instead, do this:
```c++
enum class PipelineGPUBitdepth
{
    // Same as the names in the bad example, but you don't need to start with "PipelineGPUBitdepth"
    // anymore (also, had to change the names so they don't start with numbers).
    Int8bit,
    Float16bit,
    Int16bit,
    Count
};

int numBits(PipelineGPUBitdepth bitdepth)
{
    switch (bitdepth) {
        case PipelineGPUBitdepth::Int8bit: return 8;
        case PipelineGPUBitdepth::Int16bit:
        case PipelineGPUBitdepth::Float16bit: return 16;
        case PipelineGPUBitdepth::Count: break;
    }
    VNLogError("Unhandled bitdepth enum: %d\n", static_cast<int>(bitdepth));
    return -1;
}
// Fine, make your own == operator if you want to compare against an int.
bool operator==(PipelineGPUBitdepth lhs, int rhs) { return numBits(lhs) == rhs; }
bool operator==(int lhs, PipelineGPUBitdepth rhs) { return lhs == numBits(rhs); }

// Fine: we're going to use this for a bitfield, m_supportedPlatforms, below, so we
enum PlatformFlags : uint32_t
{
    SupportsWindows = 0x00000001,
    SupportsLinux = 0x00000002,
    SupportsAndroid = 0x00000004,
    SupportsMacOS = 0x00000008,
    SupportsiOS = 0x00000010,
};

class MobileDecoderSettings
{
public:
    // Fine: we've explicitly defined this so that it'll use our numBits function.
    bool isPipeline16Bits() const { return m_bitdepth == 16; }

    // Fine: no conversion need for this comparison.
    bool isBitdepthValid() const { return m_bitdepth < PipelineGPUBitdepth::Count; }

private:
    PipelineGPUBitdepth m_bitdepth = PipelineGPUBitdepth::Count;
    uint32_t m_supportedPlatforms = SupportsAndroid | SupportsiOS;
};
```

# Unit Testing

## U.1 Unit testing best practices

Unit testing is a key part of making sure a product is stable. Unit tests are a piece of code that runs a unit of code and checks it gives the expected output and should be written as part of doing the work that it tests. While there are several different ideas on the best way to do Unit testing (such as TDD) this section only aims to define what a unit test should be. Unit tests should be:

### Simple
Unit tests should test the smallest unit of code in the simplest way possible. If a simple unit test requires a lot of unnecessary setup it is a sign that the code may need refactoring. Unit tests code should also be as simple as possible to avoid bugs in unit test code

### Thorough
All code paths in the code should be triggered. Tests should also consider other scenarios such as numerical overflows, extreme input values and null/invalid inputs. If the code is written as part of a Jira ticket the functional acceptance criteria of that ticket should also be tested

### Isolated
A unit test should be able to run on its own without the need to run the entire test suite. This way if a single test is failing it is quick and easy to run that single test over and over to fix the issue. The use of setup and tear down functions can help avoid repetition here

### Immediate
The utility of a unit test is allowing Developers to have confidence that their changes are correct. Thus unit tests should be able to point out any problems with the code every time they are run

### Deterministic
Unit tests should always run under the same conditions and produce the same outcome. Inconsistent results can lead to confusion and can make it difficult to diagnose and fix problems.

### Cross-platform
Unit tests should behave the same way across all platforms. Any tests for platform specific code(e.g. SIMD) should not run at all on platforms where they are not supported

### Parameterized
Prefer use of parameterization over repeated test implementations or a single test that test multiple parameters. This makes test code clean and makes it easy to isolate individually failing tests

### Fast
Unit test suites should be quick to run. Avoid testing large amounts of data/parameters and optimize assert code where possible


Note that these apply specifically to Unit testing. Other tests such as Functional testing, perf testing and stress testing may need to violate some of these principles.


# Misc

## M.1 Do not merge commented out code

Commented out code rots quickly. If there are changes in the surrounding code that impact the
commented out code there are no guarantees that it will compile the next time someone uncomments it.

If it's commented out, it's dead code and no longer serves a purpose, thus it should be removed to
prevent it from rotting.

## M.2 Never have `using` in a header file

`using namespace XXX;` should never be used in a header. This pollutes the namespace and as a result
anywhere that includes that header also has all the names brought into its scope.

## M.3 Static Linkage through anonymous namespaces

In source files, anything contained inside of an anonymous namespace has static linkage. This is
equivalent to defining a function or variable as `static`.

For example, these two declarations are equivalent:
```c++
namespace {
    const char* const kMessage = "Hello, World\n";

    void example()
    {
        printf(kMessage);
    }
}
```

```c++
static const char* const kMessage = "Hello, World\n";

static void example()
{
    printf(kMessage);
}
```

Therefore, declaring a function or variable as static inside of an anonymous namespace is redundant.
The idiomatic C++ way is to use anonymous namespaces, which should be preferred.

An important note is that anonymous namespaces do not give static linkage inside of a header file.
Instead, in header files, objects declared inside of an anonymous namespace will be unique per
translation unit.

For example:
```c++
// example.h
namespace {
    const int kExample = 1;
}
```

Two separate translation units including `example.h` will have their own versions of `kExample` and
as a result they have different addresses. This can be desirable in certain scenarios, but is not
the same behavior as static linkage. In addition, in C++17 and beyond it is better to declare such
variables as `inline`.


[translation unit]: glossary.md#translation-unit
[virtual table]: glossary.md#virtual-table
[RAII]: glossary.md#RAII
[trivially copyable]: glossary.md#trivially-copyable
[copy elision]: glossary.md#copy-elision
[internal linkage]: glossary.md#internal-linkage
[Rule of Five]: best_practices.md#c41-otherwise-follow-the-rule-of-five
[ownership]: glossary.md#ownership
