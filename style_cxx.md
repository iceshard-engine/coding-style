This Coding Style based on https://github.com/niklasfrykholm/blog/blob/master/reference/coding-style.md. Changes where made over time resulting in the current version.

# C++ Coding Style

This document describes the style you should use when working on code within IceShard repositories.

## Table of Contents

* [Introduction](#introduction)
* [Includes and dependencies](#includes-and-dependencies)
* [Naming](#naming)
* [Braces and Scopes](#braces-and-scopes)
* [Indentation and Spacing](#indentation-and-spacing)
* [Comments](#comments)
* [Design and Implementation](#design-and-implementation)
* [Miscellaneous Tidbits](#miscellaneous-tidbits)

This section is dedicated for C++ code, however some things can be seens as generic and good to follow in different languages.

## Introduction

### **A common style is good**

A common style is useful. It makes it easier to communicate, and it makes the code easier to read.

### **Some choices are arbitrary**

Some of the style choices described in this manual are (at least according to me) very well motivated, other are just arbitrary. Sometimes there is no reason to pick one particular style over another, but it is still useful to mandate a specific choice to ensure a consistent style.

### **Some things don't matter**

The purpose of this style guide is not to cover every possible situation. If something is not covered by this guide it is probably something that doesn't matter that much and you are free to use whatever style you are most accustomed to. If you think something should be in the guide, just bring it up to discussion.

### **Avoid revision wars**

If you see something that obviously does not follow the standard in a source file you should feel free to change it. If you see something that perhaps does not follow the standard, but you are not sure, it is better to leave it.

There is not much point in going over the entire codebase looking for style violations. Such nitpicking is not very productive. Instead, just fix the style violations in code you are working on anyway.

Avoid changing the style back and forth in the same piece of code. If you cannot agree on the style, talk about it instead of having a "cold war" in the codebase.

If you disagree strongly with one of the rules in this style guide, you should propose a change to the rule rather than silently rebel.

## Includes and dependencies

Proper including is a way for successful dependency management. To make it as easy as possible there are a few rules in place in all projects which should help you find around the codebase.

### **The `public` and `private` directories**

Each project has at least one directory:
- `public` is used to hold __only__ interfaces and public functions.
  - You are not allowed to include a `private` file from a `public` file.
- `private` contains implementations and private helper _(utility)_ source files. These files are only accessible from the owning project.

It's not permitted to add compilation unit files to the `public` directory. Only header _(.hxx)_ and inline _(.inl)_ files are allowed.

### **Include rules**

If a header is placed in a `public` directory, no matter if it's the same project or a dependent project, you include it with arrow braces.

Same goes for SDK headers and external 3rd party libraries.

```cpp
// Same project public header
#include <project/public_header.hxx>
// Dependent project public header
#include <other/public_header.hxx>

// OS SDK header
#include <Windows.h>

// 3rdParty library header
#include <zlib.h>
```

Accessing headers from the `private` directory can only be done using the `"header/path.hxx"` syntax. It should be avoided to use __`../`__ backtracing paths if possible.

```cpp
// BAD
#include <private_header.hxx>
#include <../private_header.hxx>

// OK
#include "../private_header.hxx"

// GOOD
#include "private_implementation/feature_header.hxx"
```

## Naming

Naming is a fundamental part of programming. The ease-of-use and "feeling" of an API depends a lot on having good names.

Furthermore, names are harder to change than implementations. This is especially true for names that are exported outside the executable itself, such as names for script functions and parameters in JSON files. For this reason, some care should be taken when selecting a name.

### **Names should provide all necessary information, nothing more**

A name should provide all the necessary information to understand what a function or variable is doing but no more than that. I.e., it should not contain redundant information or information that is easily understood from the context.

```cpp
// BAD: Name provides too little information
char *pstoc(char *);
float x;
int s;
void Image::draw(float, float);

// BAD: Name provides too much information
char *convert_string_in_pascal_string_format_to_c_string_format(char *);
float the_speed;
int time_elapsed_in_seconds;
void Image::draw_image_at(float, float);

// BETTER: Just the right amount
char *pascal_string_to_c(char *s);
float speed;
int elapsed_seconds;
void Image::draw_at(float x, float y);
```

If you cannot come up with a good name for something -- consult your colleagues or look up synonyms. Never use a name that you know is bad.

```cpp
// BAD:
// What is link2? How it is different from link?
void link();
void link2();

// These names don't mean anything.
Stuff the_thing;
```

### **A bigger scope warrants a more descriptive name**

The more visible a variable is (the larger scope it has) the more descriptive its name needs to be, because less information is provided by the context.

Consider the number of players in a network game. In a small local loop, it is fine to call the variable `num` because there is not much to confuse it with, and the context immediately shows where the variable is coming from.

```cpp
int num = player_count();
for (int idx = 0; idx < num; ++idx)
    ...
```

If it is a static variable in the network class we need to be more verbose, because `n` could mean any number of things in that context.

```cpp
class Network
{
    ...
    int _connected_player_count;
    ...
}
```

If it is a global variable, we must be even more verbose, because we no longer have a Network class context that tells us the variable has something to do with the network:

```cpp
int NetworkConnectedPlayerCount;
```

_NOTE: Global variables should be avoided. If you really need a global variable, you should hide it behind a function interface (e.g., `console_server::get()`). This reduces the temptation of misusing the variable._

### **Avoid use of abbreviations in names.**

_NOTE: This rule is applied to public sybols, it does not apply to local function scope variables or private members._

There are two problems with abbreviations in names:

* It gets harder to understand what the name means. This is especially the case with extreme and nonsensical abbreviations like `wbs2mc()`.
* Mixing abbreviated and non-abbreviated names introduces inconsistency in the API. Sometimes you gonna see a function called `set_pos` and in pother places you will see `world_position`, which can get ugly. Keeping everything consistent should be the priority. `set_position(world_position())`

### **Use sensible names**

* Spell your names correctly.
* Do not "simplify" the words `to` and `for` as numbers `2` and `4`.
* All names and comments should be in American English.

### **Name functions, arguments and variables `like_this()` _(snake case)_**

Use lower case characters and underscores where you would put spaces in a normal sentence.

This style is preferred for functions and variables, because it is the most readable one _(most similar to ordinary language)_. Because functions and variables are the things we have most of it helps reading the code.

Do not use any kind of Hungarian notation when naming variables and functions. Hungarian serves little purpose other than making the code less readable.

#### **Getters and setter functions**

Getter and setter functions should look like this.

```cpp
auto circle() const -> Circle const& { return _circle; }
void set_circle(Circle& circle) { _circle = circle; }
```

The getter is called `circle` rather than `get_circle`, since the `get_` prefix is superfluous. However, we use a `set_` prefix to emphasize that we are changing the state of the object.

The `get_` prefix can be used when a method needs to do additional work to return the value, as **get** is also an action verb in english. However it's still recommented to use `query_`, `search_` and similar instead.

### **Name classes `LikeThis` _(camel case)_**

It is good to use a different standard for classes than variables, because it means that we can give temporary variables of a class good names:

```cpp
// GOOD
class Circle { ... };
Circle circle;

// BAD
class circle { ... };
circle a_circle;
```

If the class was called `circle`, the variable would have to be called something horrible like `a_circle` or `the_circle` or `tmp`.

### **Name member variables `_like_this` _(snake case)_**

Being able to quickly distinguish member variables from local variables is good for readability... and it also allows us to use the most natural syntax for getter and setter methods:

```cpp
auto circle() const -> Circle const& { return _circle; }
void set_circle(Circle const& circle) { _circle = circle; }
```

A single underscore is used as a prefix, because a prefix with letters in it (like `m_`) makes the code harder to read.

    This _sentence can _be _easily read _even though _it _has _extra underscores.

    But m_throw in m_some letters m_and it m_is m_not so m_easy m_anymore, m_kay.

Also, using underscores makes the member variables stand out more, since there could be other variables starting with m.

**!! ATTENTION !!**
* Take care while naming member variables with a underscore
* The standard reserves all names using two or more underscores in a row and names starting with a underscore followed by a Big letter.
* It also reserves all names starting with a underscore in the global namespace / file scope.

```cpp
// Reserved:
int __age;
int __age__;
int _Age;

// Allowed (in any user class or namespace context, excluding `std::`)
int _age;
```

### **Name macros `LIKE_THIS` _(capital snake case)_**

It is good to have `#define` macro names really standing out, since macros can be devious traps when it comes to understanding the code. _(Like when Microsoft redefines `GetText` to `GetTextA`)_

### **Name namespaces `like_this` _(snake case)_**

This is the most readable syntax, so we prefer this when we don't have any reason to do otherwise.

Use a single namespace identifier if parent namespaces do not define or declare anything new.

```cpp
// BAD
namespace foo
{
    namespace bar
    {
        namespace foo_bar
        {
            ...
        }
    }
}

// GOOD
namespace foo::bar::foo_bar
{
    ...
}
```

### **Name enums and enum values `LikeThis` _(camel case)_**

Enums are types, just as classes and structs and should follow the same naming convention.

Enums should use the `enum class` feature of C++11 to avoid exporting the enum names to the enclosing scope:

```cpp
enum class Align {Left, Right, Center};
```

Enums not using the `enum class` feature are required to prefix each value with the enum name followed by a underscore.
```cpp
enum CommonValues { CommonValues_Foo, CommonValues_Bar, CommonValues_FooBar };
```

### **Name source files `like_this.cxx`, and header files `like_this.hxx`**

Again, this is the most readable format, so we choose that when we don't have a reason to do something else.

### **Using common types in code**

When developing in the `iceshard-engine` project use the introduced project types, like: `ice::u32`, `ice::f32`, etc...

Otherwise prefer standarized types like `uint32_t` before `int` or `unsigned`.

Use `east const` instead of `const west`, this allows for better code readability as the only rule to remember is: `const` applies to the type on the left.

When appliciable use `const` types for local variables or class members.

Use `constexpr` for compile time available values.

Avoid static mutable values, because this introduces a lot of problems in concurrent code.

## Braces and Scopes

Use braces to increase readability in nested scopes

Instead of:

```cpp
// BAD
while (a)
    if (b)
        c;
```

Write:

```cpp
while (a)
{
    if (b)
    {
        c;
    }
}
```

Only the innermost scope is allowed to omit its braces, but should still be avoided.

### **Fit matching braces on a single screen**

The opening and closing of a brace should preferably fit on the same screen of code to increase readability.

Class and namespace definitions can of course cover more than one screen.

Function definitions can sometimes cover more than one screen -- if they are clearly structured -- but preferably they should fit on a single screen.

`while`, `for` and `if` statements should always fit on a single screen, since otherwise you have to scroll back and forth to understand the logic.

Use `continue`, `break` to avoid deep nesting. `goto` is **disallowed**.

Code that is indented four or five times can be very hard to read. Often such indentation comes from a combination of loops and `if`-statements:

```cpp
// BAD
for (int i = 0; i < parent->num_children(); ++i)
{
    Child child = parent->child(i);
    if (child->is_cat_owner())
    {
        for (int j = 0; j < child->num_cats(); ++j)
        {
            Cat cat = child->cat(j);
            if (cat->is_grey())
            {
                ...
```

Using continue to rewrite gives a clearer structure:

```cpp
for (int i = 0; i < parent->num_children(); ++i)
{
    Child child = parent->child(i);
    if (!child->is_cat_owner())
        continue;

    for (int j = 0; j < child->num_cats(); ++j)
    {
        Cat cat = child->cat(j);
        if (!cat->is_grey())
            continue;

        ...
```

Excessive indentation can also come from error checking, however this case requires probably more thought to make it clearer.

```cpp
File f = open_file();
if (f.valid())
{
    std::string name;
    if (f.read(&name))
    {
        int age;
        if (f.read(&age))
        {
            ...
        }
    }
}
```

Using local lambda functions is another good way of avoiding deep nesting.

### **The two bracing styles and when to use them**

There are two bracing styles used:

```cpp
// Single line
int f() const { return 3; }

// New line
int X::f() const
{
    return 3;
}
```

The first style is typically used for getter and setter functions in the header file to make the header more compact.

This second is used for any other case like: while loops, for-loops, class declarations and function declarations.

Consistent bracing style is not super important, but in general the rule should be that the more that is enclosed by the brace, the more space there should be in the brace.

## Indentation and Spacing

### **Use spaces for indentation**

Spaces are used as it's constant accross most editors. Flexibility in controlling the indentation can be achieved using git pre and post hooks for local repository setup.

### **Avoid aligning any entities in code**

Aligning should be avoided. It does not really serve any purpose and work required to keep it up-to-date is not justified.

```cpp
void f()
{
    // DONT DO THIS
    int some_var    = 1;
    int another_var = 2;
    int x           = 3;

    // OK
    int some_var = 1;
    int another_var = 2;
    int x = 3;
}
```

### **No extra spaces at end-of-line**

There should be no whitespace at the end of a line. Such invisible whitespace can lead to merge issues.

Empty lines are __not__ an exception. Empty lines may not contain any tabs or spaces.

_NOTE: There are plugins, settings and other means to see these extra spaces and even remove them on saving in various text editors._

### **Think about evaluation order when placing spaces**

For statements, put a space between keywords and parenthesis, put a space before braces on the same line. Do not put any space before a semicolon.

```cpp
while (x == true)
{
    do_stuff();
}
```

Placement of spaces in expressions is not that important. We generally tend to put a space around every binary operator, but not around unary operators (such as array access, function calls, etc).

```cpp
z = x * y(7) * (3 + p[3]) - 8;
```

You can use a more terse or a more loose style if you want to, but make sure that the placement of spaces reflects the evaluation order of the expression. I.e. begin by removing spaces around operators that have a higher order of precedence.

### **Make lines reasonably long**

A lot of style guides say that lines should never be more than 80 characters long. This is overly restrictive. We all have displays that can show more than 80 characters per line and nobody prints their code anymore.

Never write code like this:

```cpp
// BAD
int x = the + code +
    is + indented +
    and + I + dont +
    want + to + create
    + long + lines;
```

Either use less indentation or write longer lines.

Don't go crazy with line lengths, scrolling to see the end of the line is annoying. Also, make sure not to put very important stuff far to the right where it might be clipped from view.

### **General guidelines for spaces**

* Put a space between `if`, `for`, `while` and the parenthesis that follows.
* Do not put a space between the function name and the parenthesis in a function call.
* Do not put spaces inside parenthesis.
* Put spaces after commas, do not put spaces before commas.
* In a variable declaration, do not put a space before `*` or `&` and put a space after.

```cpp
// GOOD
if (x)
for (int i = 0; i < 3; ++i)
memset(&a, 0, sizeof(a));
void f(T const& t)

// BAD
if(x)
for(int i = 0; i < 3; ++i)
memset ( &a,0,sizeof(a) );
void f(const T & t)
```

### **Indent `#if` statements**

By default, the visual studio editor left flushes all preprocessing macros. This is idiotic and makes the code really hard to read, especially when the macros are nested:

```cpp
// BAD
void f()
{
#ifdef _WIN32
#define RUNNING_WINDOWS
#ifdef PRODUCTION
    bool print_error_messages = true
#else
    bool print_error_messages = false
#endif
#else
    bool win32 = false
#endif
```

Instead, indent your macros just as you would normal C code:

```cpp
void f()
{
#   ifdef _WIN32
#       define RUNNING_WINDOWS
#       ifdef PRODUCTION
            bool print_error_messages = true
#       else
            bool print_error_messages = false
#       endif
#   else
        bool win32 = false
#   endif
}
```

## Comments

### **Use `//` for descriptive comments `/*` for disabling code**

`//!` should be used for documentation purposes as it's recognized by tools like `doxygen`.

`//` comments are better for comments that you want to leave in the code, because they don't have any nesting problems, it is easy to see what is commented, etc.

`/*` is useful when you want to quickly disable a piece of code.

### **Do not leave disabled code in the source**

Commenting out old bad code with `/* ... */` is useful and necessary.

It can be useful to leave the old commented out code in the source file *for a while*, while you check that the new code does not have any bugs, performance problems, etc. But once you are sure of that you should remove the commented out code from the file.

Having a lot of old, unused, commented out code in the source files makes them harder to read, because you constantly ask yourself why was this commented out, maybe the solution to my problem lies in this commented out code, etc. Source control already keeps a version history, we don't need to keep old code in comments.

### **Use comments as hints to the reader**

The main source of information about what the code does should be the code itself. The code is always up-to-date, it doesn't lie and no extra effort is required to maintain it. You should not need to add comments that explain what the code does:

```cpp
// BAD

// Returns the speed of the vehicle
float sp() {return _sp;}

// Computes speed from distance and time
s = d / t;

// Check for end of file
if (c == -1)
```

Instead, write code that is self-explanatory.

```cpp
float speed() const { return _speed; }

speed = distance / time;

if (c == END_OF_FILE_MARKER)
```

Source code comments should be used as hints to the reader who tries to understand the code. They should point out when the code does something which is a little bit clever or tricky, something that may not be immediately obvious from reading just the code. In complicated algorithms that consist of several steps, they are also useful for identifying the separate steps and giving the user a sense of context.

```cpp
// Use Duff's device for loop unrolling
// See for example: http://en.wikipedia.org/wiki/Duff's_device
switch (count % 8)
{
    case 0: do { *to = *from++;
    case 7:      *to = *from++;
    case 6:      *to = *from++;
    case 5:      *to = *from++;
    case 4:      *to = *from++;
    case 3:      *to = *from++;
    case 2:      *to = *from++;
    case 1:      *to = *from++;
               } while ((count -= 8) > 0);
}
```

### **Avoid boilerplate comments**

The purpose of comments is to convey information. Avoid big cut-and-paste boilerplate comments in front of classes and functions. Make the comments succint and to the point. There is no point in repeating information in the comment that is already in the function header, like this:

```cpp
// BAD
//! Returns the distance between p1 and p2
//! p1 a point
//! p2 another point
float distance(const Vector3 &p1, const Vector3 &p2);
```

You don't have to comment every single function in the interface. If the function's meaning is clear from its name, then adding a comment conveys no extra information. I.e... this is pointless:

```cpp
// BAD
//! Returns the speed.
float speed();
```

Do not add a super heavy boilerplate comments to functions with parameters, return values, etc. Such comments tend to contain mostly fluff anyway. They convey no more information than a simple comment and they make it much harder to get an overview of the code.

I.e. avoid fluff pieces like this:

```cpp
// BAD
/************************************************************
* Name: cost
*
* Description: Returns the cost of going from point p1 to p2.
* Note: Cost of going in z direction is 2 times as expensive.
*
* Parameters:
* p1 - The one point
* p2 - The other point
* Return value:
* The cost of going from p1 to p2.
*************************************************************/
static inline float cost(const Vector3 &p1, const Vector3 &p2) const;
```

Currently `IceShard` does not make use of Doxygen.
However you are allowed to use Doxygen markup because many editors or plugins make use of these to provide better tooltips.

It is still preferet to just use the plain `Markdown` syntax if you need to hilight specific words or phrases in your comment.

### **Don't put high level documentation in source code comments**

Source code comments are not and should not be the only kind of documentation. Source code comments are good for documenting details that are directly related to the code, such as reference documentation for an API.

Aside from detail documentation, systems also need high level documentation. The high level documentation should provide an overview of the system and an entry point for programmers who are new to the system. It should explain the different concepts that are used in the system and how they relate to each other, the goals of system and the different design choices that have been made.

High level documentation should not be put in source code comments. That makes it fragmented and hard to read. Instead, it should be created as an HTML document, where the user can read it as a single continuous text with nice fonts, illustrations, examples, etc.

### **Put interface documentation in the .hxx file**

Put interface _(function and class documentation)_ in the .hxx file. This makes it easier to find all the relevant interface documentation for someone browsing the .hxx files.

A drawback of this is that the .hxx files will become bigger and harder to grasp, but that is a price we are willing to pay.

## Design and Implementation

### **Optimize wisely**

All the code in the engine does not have to be super optimized. Code that only runs once-per-frame has very little impact on a game's performance. Do not spend effort on optimizing that code. Consider what are the heavy-duty number-crunching parts of the code and focus your efforts on them. Use the profiler as a guide to finding the parts of the code that matter.

Be very wary of sacrificing simplicity for code efficiency. Your code will most likely live for a long time and go through several rounds of optimization and debugging. Every time you add complexity you make future optimizations more difficult. Thus, an optimization that makes the code faster today may actually make it slower in the long run by preventing future optimizations. Always strive for the simplest possible code. Only add complexity when it is absolutely necessary.

Be aware that the rules of optimization have changed. Cycle counts matter less. Memory access patterns and parallelization matter more. Write your optimizations so that they touch as little memory as possible and as linearly as possible. Write the code so that it can be parallelized and moved to SPUs. Focus on data layouts and data transforms. Read up on data oriented design.

### **Make use of pure interfaces**

On good practice is to use pure interfaces. This generally forces you to think about what needs to be visible and what is an implementation detail.

Having good interfaces allows to make the API cleaner and more understandable for the end user. It also allows for more flexibility, as you are not forced to use members from yor parent type.


### **RAII and Dependency injection**

Always design your objects in the **RAII** principle. If an object is created, it needs to be fully constructed and ready for use. You should avoid any `init` and `shutdown` like behavior, as this leads to overly complicated initialization scenarios.

Additionally when you are depending on a different interface / API, always take a reference to it and use the stored pointer / reference. This enforces proper use of APIs and solves problems with hidden dependencies as you need to provide the interface the object depends on during construction.

If you also follow **RAII** properly, you won't end up with dependencies getting destroyed before the objects that they use.

_NOTE: If we would use a Singleton inside a method implementation, there is no information about this relation in the public interface._

#### Factory functions

If the creation of an object is complicated or consists of multiple stages, create a factory function for it. Such a function is allowed to return a nullptr when construction fails, or a fully constructor object.

You can initialize your dependencies and object in the factory function however you like. The only requirement is to return an object that works perfectly with the **RAII** principle. That means, that if the objects gets destroyed it will also release all dependent resources that have a shorter lifetime that the object itself.

## C++11 Features

The IceShard engine is compiled using C\+\+20, and aims to use only features up to this standard, nothing above. The features listed below are the C\+\+17 features that are known to be working and that we recommend using.

### **Keyword: `auto`**

Outside of function signatures the use of the `auto` keyword should be avoided.

It can be used when declaring variables of complex types such as `Array<Item>::const_iterator`.

### **Keyword: `decltype()`**

Use this if you need it.

### **Range based for-each loops**

Use this instead of regular for loops where it makes the code simpler to read.

### **Lambda functions**

Use lambda functions whenever you need a small local helper function in local scope.

```cpp
void format_markdown(const char *s)
{
    auto has_macro = [](const char *line) {
        return line[0] == '#';
    };

    ...
}
```

### **Keyword: `noexcept`**

As this is a game engine, we do not want expetions to be used at all, thus it's generally a rule of thumb to mark every engine function as `noexcept` and ensure no exceptions are thrown.

### **`stdint.h` types `int8_t`, `uint8_t`, `int32_t`, `uint32_t`, ...**

We assume that any compiler compiling the IceShard project has sensible type sizes, i.e.:

* `char` = 8 bits
* `short` = 16 bits
* `int` = 32 bits

Still, for integer sizes other than 32 bit, the `stdint.h` types should be used as they are less ambiguous:

* Use: `int8_t`, `int16_t`, `uint64_t`, ...
* Rather than: `char`, `short`, `size_t`, `long long`, ...

For 32-bit integers, the `stdint.h` types are prefered:

* `int32_t` instead of `int`
* `uint32_t` instead of `unsigned`

When you are referring to a string or a buffer of raw data you should still use `char *` rather than `int8_t *`. Use `int8_t` when you want a small integer.

### **Keywords: `override`, `final`**

These keywords should be used to document the intent of virtual methods.

### **Keyword: `static_assert`**

Use this everywhere possible to provide compile time errors instead of runtime bugs.

## Miscellaneous Tidbits

### **Use `#pragma once` to avoid multiple header inclusion**

All current compilers understand the `#pragma once` directive. And it is a lot easier to read than the standard `#ifndef` syntax:

```cpp
// BAD
#ifndef _MY_UNIQUE_HEADER_NAME_H_
#define _MY_UNIQUE_HEADER_NAME_H_
    ...
#endif

// GOOD
#pragma once
```

### **Output arguments**

Returning value via arguments should be avoided, but it is not always possible to do so.

When defining a function that returns values via an argument, that argument should be prefixed with `out_`.

```cpp
// OKAY
void do_something(ArrayType<int>& out_indices);
```

### **Arguments: `(pointer, size)`**

When writing a function that takes a pointer and a size/count as parameters, following the standard of the standard C library (`memcpy`, etc) the pointer argument should be passed first and the size argument last. I.e.:

```cpp
void do_something(char *data, unsigned len);
```

An even better approach is to use the `ice::Data` type as an argument, which is used to define arbitrary constant data.

If you need to pass a mutable data block, use the `ice::Memory` type instead.

### **Arguments: `(type, name)`**

When writing functions that take a resource type and resource name as arguments, the type argument should precede the name argument, i.e. as in:

```cpp
bool can_get(ResourceID const& type, ResourceID const& name) const;
```

### **Function signatures**

Prefer using trailing return types instead of the old syntax. This allows to focus on the function name and it's arguments instead of reading a multiline return type first, which might not even be used.

The exeptions are the `bool` and `void` return types, which also have 4 letters and are so common it is of no value to put them at the end.

```cpp
// NOT OK
FooResult foo_task();
std::unique_ptr<WithAResultType, AndACustomDeleterType> get_future();

// OK
void bar_task();
bool is_foobar_done();

auto foo_task() -> FooResult;
auto get_future() -> std::unique_ptr<WithAResultType, AndACustomDeleterType>;
```

### **Const the world**

Because C++ does implicitly allow to modify everything _(not like Rust)_ try to `const` eveything that seems to be a good candidate.

If you need to access something, __DO NOT EVER__ `const_cast` it away, search for another way to access the mutable version of that value or if you created the interface, change it to a mutable one.

You can also always provide two method definitions in a class.

```cpp
class Object
{
    auto some_value() -> int&; // Can be only accessed on non-const Object's.
    auto some_value() const -> int const&; // Can be only accessed on const Object's.

    // BAD: Do not ever return mutable values from const objects!
    auto some_value() const -> int&; // Can be only accessed on non-const Object's.
};
```

### **C++ Attributes**

Make use of C++ attributes like `[[nodiscard]]` and `[[maybe_unused]]`.

```cpp
[[nodiscard]]
auto create_expensive_object() -> std::unique_ptr<ExpensiveObject>;

void do_something_on_windows([[maybe_unused]] int some_param)
{
#if IS_WINDOWS_BUILD
    // `some_param` only used on windows builds
#endif
}
```

IceShard embraces modern C++ and defines are replaced or also available as `constexpr` values.

When working platform specific code, you might want to use compile-time if expressions instead of `#if/#endif`. This will also take care of variables beeing `unused` in unhandled scenarios.

```cpp
void do_something_on_windows(int some_param)
{
    // if the expression turns to be false, the compiler still parse it's body and marks `some_param` as used.
    // This will not trigger an `unused parameter` warning or error.
    if constexpr(is_windows_build)
    {
        // `some_param` only used on windows builds
    }
}
```
