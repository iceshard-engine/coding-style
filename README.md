This Coding Style based on https://github.com/niklasfrykholm/blog/blob/master/reference/coding-style.md. Changes where made over time resulting in the current version.

# Coding Style

This document describes the coding style you should use when working on code for the IceShard engine and tools.

## Table of Contents

* [C++ Code](#c++-code)
  * [Introduction](#introduction)
  * [Includes and dependencies](#includes-and-dependencies)
  * [Naming](#naming)
  * [Braces and Scopes](#braces-and-scopes)
  * [Indentation and Spacing](#indentation-and-spacing)
  * [Comments](#comments)
  * [Design and Implementation Issues](#design-and-implementation-issues)
  * [Miscellaneous Tidbits](#miscellaneous-tidbits)
* [Jinx scripts](#jinx-scripts)
* [Lua Scripts](#lua-scripts)
* [Asset metadata](#asset-metadata)
* 

## C++ Code

This section describes the coding style for C++ code, and also general coding guidelines that are useful regardless of what languge you are using.

### Introduction

#### A common style is good

A common coding style is useful. It makes it easier to communicate, and it makes the code easier to read.

#### Some choices are arbitrary

Some of the style choices described in this manual are (at least according to me) very well motivated, other are just arbitrary. Sometimes there is no reason to pick one particular style over another, but it is still useful to mandate a specific choice to ensure a consistent style.

#### Some things don't matter

The purpose of this style guide is not to cover every possible situation. If something is not covered by this guide it is probably something that doesn't matter that much and you are free to use whatever style you are most accustomed to. If you think something should be in the guide, just bring it up to discussion.

#### Avoid revision wars

If you see something that obviously does not follow the standard in a source file you should feel free to change it. If you see something that perhaps does not follow the standard, but you are not sure, it is better to leave it.

There is not much point in going over the entire codebase looking for style violations. Such nitpicking is not very productive. Instead, just fix the style violations in code you are working on anyway.

Avoid changing the style back and forth in the same piece of code. If you cannot agree on the style, talk about it instead of having a "cold war" in the codebase.

If you disagree strongly with one of the rules in this style guide, you should propose a change to the rule rather than silently rebel.

### Includes and dependencies

Proper including is a way for successful dependency management. To make it as easy as possible there are a few rules in place in all projects which should help you find around the codebase.

#### The `public` and `private` directories

Each project has at least one of them, where `public` is used to hold __only__ interfaces and public functions, and `private` holds all implementation details private, and utility code.

It's not permitted to add source files to the `public` directory, only header and inline files are allowed.

#### Include rules 

If a header is placed in a `public` directory, no matter if it's the same project or a dependent project, you include it with arrow braces.

Same goes for SDK headers and external 3rd party libraries.

```cpp
#include <project/public_header.hxx> // Same project 'public` header 
#include <other/public_header.hxx> // Other dependency project 'public` header 
#include <Windows.h> // OS SDK header
#include <zlib.h> // 3rdParty library geader
```

Accessing a header from the `private` directory can only use the `""` syntax, __but__ it is not alowed to use __..__ backwards paths.

```cpp
// BAD
#include "../private_root_header.hxx"

// GOOD 
#include "private_implementation/feature_header.hxx"
```

#### Providing common tools

Because of the above `include` rules it seems there is no way to include a common `root` header with helpfull utility functions in sub-directories. But thats not the case.

As far as it comes to include a giant utility header _(which tend to appear)_ it's better to create such utility header for each sub directory and fill with with function declaration used only for this sub-directory.

Functions can be declared this way, and implemented in a single place in the private root directory.

### Naming

#### Naming is important

Naming is a fundamental part of programming. The ease-of-use and "feeling" of an API depends a lot on having good names.

Furthermore, names are harder to change than implementations. This is especially true for names that are exported outside the executable itself, such as names for script functions and parameters in JSON files. For this reason, some care should be taken when selecting a name.

#### Names should provide all necessary information, nothing more

A name should provide all the necessary information to understand what a function or variable is doing but no more than that. I.e., it should not contain redundant information or information that is easily understood from the context (such as the class name).

```cpp
// BAD: Name provides too little information
char *pstoc(char *);
float x;
void Image::draw(float, float);

// BAD: Name provides too much information
char *convert_string_in_pascal_string_format_to_c_string_format(char *);
float the_speed;
void Image::draw_image_at(float, float);

// GOOD: Just the right amount
char *pascal_string_to_c(char *s);
float speed;
void Image::draw_at(float x, float y);
```

If you cannot come up with a good name for something -- think harder and consult your colleagues. Never use a name that you know is bad.

```cpp
// BAD:
// What is link2? How it is different from link?
void link();
void link2();

// These names don't mean anything.
Stuff the_thing;
```

#### A bigger scope warrants a more descriptive name

The more visible a variable is (the larger scope it has) the more descriptive its name needs to be, because less information is provided by the context.

Consider the number of players in a network game. In a small local loop, it is fine to call the variable `n` because there is not much to confuse it with, and the context immediately shows where the variable is coming from.

```cpp
int n = num_players();
for (int i = 0; i < n; ++i)
    ...
```

If it is a function in the network class we need to be more verbose, because `n` could mean any number of things in that context.

```cpp
int Network::num_players();
```

If it is a global variable, we must be even more verbose, because we no longer have a Network class context that tells us the variable has something to do with the network:

```cpp
int _num_players_in_network_game;
```

**NOTE:** Global variables should be avoided. If you really need a global variable, you should hide it behind a function interface (e.g., `console_server::get()`). This reduces the temptation of misusing the variable.

#### Do not use abbreviations in names

There are two problems with abbreviations in names:

* It gets harder to understand what the name means. This is especially the case with extreme and nonsensical abbreviations like `wbs2mc()`.
* Once you start to mix abbreviated and non-abbreviated names, it becomes hard to remember which names where abbreviated and how. It is not hard to understand that `world_pos` means `world_position`. But it can be hard to remember whether the function was called `world_pos` or `world_position` or something else. Never using abbreviation makes it much easier to guess what a function should be called.

The general rule is "do not use any abbreviations at all". The only allowed exception is:

* `num_`

which means number of, e.g. `num_players()` instead of `number_of_players()`.

Note that the rule against abbreviations only applies to exported symbols. A local variable can very well be called `pos` or `p`.

#### Use sensible names

* Spell your names correctly.
* Do not write the words `to` and `for` as `2` and `4`.
* All names and comments should be in American English.

#### Name functions and variables `like_this()`

Use lower case characters and underscores where you would put spaces in a normal sentence.

This style is preferred for functions and variables, because it is the most readable one (most similar to ordinary language) and functions and variables are the things we have most of.

Do not use any kind of Hungarian notation when naming variables and functions. Hungarian serves little purpose other than making the code less readable.

#### Name classes `LikeThis`

It is good to use a different standard for classes than variables, because it means that we can give temporary variables of a class good names:

```cpp
Circle circle;
```

If the class was called circle, the variable would have to be called something horrible like `a_circle` or `the_circle` or `tmp`.

#### Name member variables `_like_this`

Being able to quickly distinguish member variables from local variables is good for readability... and it also allows us to use the most natural syntax for getter and setter methods:

```cpp
auto circle() const -> Circle const& { return _circle; }
void set_circle(Circle const& circle) { _circle = circle; }
```

A single underscore is used as a prefix, because a prefix with letters in it (like `m_`) makes the code harder to read.

This _sentence can _be _easily read _even though _it _has _extra underscores.

But m_throw in m_some letters m_and it m_is m_not so m_easy m_anymore, m_kay.

Also, using underscores makes the member variables stand out more, since there could be other variables starting with m.

* Take care while naming member variables with a underscore
* The standard reserves all names using two or more underscores in a row and names starting with a underscore followed by a Big letter.
* It also reserves all names starting with a underscore in the global namespace / file scope.

```cpp
// Reserved:
int __age;
int __age__;
int _Age;

// Allowed (in any user class or namespace context, excluding `std`)
int _age;
```


#### Name macros ```LIKE_THIS```

It is good to have `#define` macro names really standing out, since macros can be devious traps when it comes to understanding the code. (Like when Microsoft redefines `GetText` to `GetTextA`.)

#### Name namespaces `like_this`

This is the most readable syntax, so we prefer this when we don't have any reason to do otherwise.

Use a single namespace identifier if parent namespaces do not define or declare anything new.

```cpp
// BAD
namespace Foo
{
    namespace Bar
    {
        namespace FooBar
	{
            ...
	}
    }
}

// GOOD
namespace Foo::Bar::FooBar
{
    ...
}
```

#### Name enums and enum values `LikeThis`

Enums are types, just as classes and structs and should follow the same naming convention.

Enums should use the `enum class` feature of C++11 to avoid exporting the enum names to the enclosing scope:

```cpp
enum class Align {Left, Right, Center};
```

Enums not using the `enum class` feature are required to prefix each value with the enum name followed by a underscore.
```cpp
enum CommonValues { CommonValues_Foo, CommonValues_Bar, CommonValues_FooBar };
```

#### Name source files `like_this.cxx`, and header files `like_this.hxx`

Again, this is the most readable format, so we choose that when we don't have a reason to do something else.

The `.h` files should be put in the same directory as the `.cpp` files, not in some special "include" directory, for easier navigation between the files.

#### Place only public header files in the `public` directory.

Never place source files in the `public` directory, as its only purpose is to provide an interface for your library.

Place only interface header files in the `public` directory, if a header files contains implementation details, keep it in the `private` directory.

#### Using types in code

Always prefer standarized types like `uint32_t` before `int` or `unsigned`.

Use `east const` instead of west const, this allows for better code readability as the only rule to remember is: `const` applies to the type on the left.

When appliciable use `const` types for local variables or class members.

Use `constexpr` for simple static class members or `const` instead. 
Try to avoid static mutable values, because this introduces a lot of problems in concurrent code.

#### Standard functions

Getter and setter functions should look like this.

```cpp
auto circle() const -> Circle const& { return _circle; }
void set_circle(Circle& circle) { _circle = circle; }
```

The getter is called `circle` rather than `get_circle`, since the `get_` prefix is superfluous. However, we use a `set_` prefix to emphasize that we are changing the state of the object.

#### Return types 

Prefer using trailing return types instead of the old syntax. This allows to focus on the function name and it's arguments instead of reading a multiline return type first, which might not even be used.

The exeption are `bool` and `void` return types, which also have 4 letters and are so common it is of no value to put them at the end.

```cpp
// OK, but not prefered
void bar_task();
bool is_foobar_done();

FooResult foo_task();

std::unique_ptr<WithAResultType, AndACustomDeleterType> get_future();

// Better
void bar_task();
bool is_foobar_done();

auto foo_task() -> FooResult;

auto get_future() -> std::unique_ptr<WithAResultType, AndACustomDeleterType>;
```

#### Const the world

Because C++ does implicitly allow to modify everything _(not like Rust)_ try to `const` eveything that is a good candidate. 

If you need to access something, __DO NOT EVER__ `const_cast` it away, search for another way to access the mutable version of that value or if you created the interface, change it to a mutable one. 

You can also always provide two method definitions in a class.

```cpp
class Object
{
    auto some_value() -> int&; // Can be only accessed on non-const Object's.
    auto some_value() const -> int const&; // Can be only accessed on const Object's.
};
```

#### C++ Attributes

Make use of the new C++ attributes like `[[nodiscard]]` and `[[maybe_unused]]`.

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

However because IceShard does quite heavily embrace modern C++ a lot of defines are replaced or also available as `constexpr` values. In such scenarios you don't need to mark parameters as `[[maybe_unused]]` if writing platform specific code _(unless it does not use platform specific SDK's)_

```cpp
void do_something_on_windows(int some_param)
{
    // if the expression turns to be false, the compiler still parses it's body and marks `some_param` as used.
    // This will not trigger an `unused parameter` warning or error.
    if constexpr(is_windows_build)
    {
        // `some_param` only used on windows builds
    }
}
```

### Braces and Scopes

Use braces to increase readability in nested scopes

Instead of

```cpp
// BAD
while (a)
    if (b)
        c;
```

Write

```cpp
while (a)
{
    if (b)
    {
        c;
    }
}
```

Only the innermost scope is allowed to omit its braces, but try to _(em-)_brace everything.

#### Fit matching braces on a single screen

The opening and closing of a brace should preferably fit on the same screen of code to increase readability.

Class and namespace definitions can of course cover more than one screen.

Function definitions can sometimes cover more than one screen -- if they are clearly structured -- but preferably they should fit on a single screen.

`while`, `for` and `if` statements should always fit on a single screen, since otherwise you have to scroll back and forth to understand the logic.

Use `continue`, `break` or even (gasp) `goto` to avoid deep nesting.

Code that is indented four or five times can be very hard to read. Often such indentation comes from a combination of loops and `if`-statements:

```cpp
// BAD
for (int i = 0; i < parent->num_children(); ++i)
{
    Child child = parent->child(i);
    if (child->is_cat_owner()) {
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

Excessive indentation can also come from error checking:

```cpp
// BAD
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

Local helper lambda functions is another good way of avoiding deep nesting.

#### The three bracing styles and when to use them

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

### Indentation and Spacing

#### Use tabs for indentation _(under review for IceShard)_

Tabs gives users more flexibility in controlling the indentation.

You should set your editor to display the tabs as four spaces. This provides a good compromise between readability and succinctness.

#### Use spaces to align columns _(problem imposed by tabs, under review)_

The start of a line should always be indented with tabs, but if you want to align some other column of code you should use spaces:

```cpp
void f()
{
	int some_var    = 1;
    int another_var = 2;
    int x           = 3;
}
```

This ensures that the columns line up even if a different tab setting is used. Even if we always view the source with four spaces for tabs, external viwers such as diff tools or github may use a different setting. We should ensure that the code always looks good.

#### No extra spaces at end of line

There should be no whitespace at the end of a line. Such invisible whitespace can lead to merge issues.

Empty lines are __not__ an exception. Empty lines may not contain any tabs or spaces. 

NOTE: There are plugins, settings and other means to see these extra spaces and even remove them on saving in various text editors.

#### Think about evaluation order when placing spaces

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

You can use a more terse or a more loose style if you want to, but make sure that the placement of spaces reflects the evaluation order of the expression. I.e. begin by removing spaces around operators that have a higher order of precedence. This is OK:

```cpp
z = x*y(7)*(3 + p[3]) - 8;
```

Because * has higher precedence than - and =. This is confusing and not OK:

```cpp
// BAD
z=x * y(7) * (3+p [3])-8;
```

#### Make lines reasonably long

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

#### General guidelines for spaces

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

#### Indent `#if` statements

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
_(Under review)_

```cpp
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
}
```

In visual studio go to **Tools > Text Editor > C/C++ > Tabs** and change `Indenting` from `Smart` to `Block` to prevent the default indenting.

### Comments

#### Use `//` for descriptive comments `/*` for disabling code

`//` comments are better for comments that you want to leave in the code, because they don't have any nesting problems, it is easy to see what is commented, etc.

`/*` is useful when you want to quickly disable a piece of code.

#### Do not leave disabled code in the source

Commenting out old bad code with `/* ... */` is useful and necessary.

It can be useful to leave the old commented out code in the source file *for a while*, while you check that the new code does not have any bugs, performance problems, etc. But once you are sure of that you should remove the commented out code from the file.

Having a lot of old, unused, commented out code in the source files makes them harder to read, because you constantly ask yourself why was this commented out, maybe the solution to my problem lies in this commented out code, etc. Source control already keeps a version history, we don't need to keep old code in comments.

#### Use comments as hints to the reader

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

#### Avoid boilerplate comments

The purpose of comments is to convey information. Avoid big cut-and-paste boilerplate comments in front of classes and functions. Make the comments succint and to the point. There is no point in repeating information in the comment that is already in the function header, like this:

```cpp
// BAD
// p1 a point
// p2 another point
// Returns the distance between p1 and p2
float distance(const Vector3 &p1, const Vector3 &p2);
```

You don't have to comment every single function in the interface. If the function's meaning is clear from its name, then adding a comment conveys no extra information. I.e... this is pointless:

```cpp
// BAD
// Returns the speed.
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

MoonEd does not use Doxygen, so avoid using Doxygen markup in your function comments. Instead write them in plain English. Note that we *used to* use Doxygen, so there is a fair ammount of Doxygen markup still left in the code base. This will be cleaned up over time.

Also, since we are not using Doxygen, avoid using `\\\` for comments, just use plain `\\` instead.

If you need to add markup to your comments to highlight specific words, you should use Markdown syntax.

#### Don't put high level documentation in source code comments

Source code comments are not and should not be the only kind of documentation. Source code comments are good for documenting details that are directly related to the code, such as reference documentation for an API.

Aside from detail documentation, systems also need high level documentation. The high level documentation should provide an overview of the system and an entry point for programmers who are new to the system. It should explain the different concepts that are used in the system and how they relate to each other, the goals of system and the different design choices that have been made.

High level documentation should not be put in source code comments. That makes it fragmented and hard to read. Instead, it should be created as an HTML document, where the user can read it as a single continuous text with nice fonts, illustrations, examples, etc.

#### Put interface documentation in the .h file

Put interface (function and class documentation) in the .h file. This makes it easier to find all the relevant interface documentation for someone browsing the .h files.

A drawback of this is that the .h files will become bigger and harder to grasp, but that is a price we are willing to pay.

### Design and Implementation Issues

#### Optimize wisely

All the code in the engine does not have to be super optimized. Code that only runs once-per-frame has very little impact on a game's performance. Do not spend effort on optimizing that code. Consider what are the heavy-duty number-crunching parts of the code and focus your efforts on them. Use the profiler as a guide to finding the parts of the code that matter.

Be very wary of sacrificing simplicity for code efficiency. Your code will most likely live for a long time and go through several rounds of optimization and debugging. Every time you add complexity you make future optimizations more difficult. Thus, an optimization that makes the code faster today may actually make it slower in the long run by preventing future optimizations. Always strive for the simplest possible code. Only add complexity when it is absolutely necessary.

Be aware that the rules of optimization have changed. Cycle counts matter less. Memory access patterns and parallelization matter more. Write your optimizations so that they touch as little memory as possible and as linearly as possible. Write the code so that it can be parallelized and moved to SPUs. Focus on data layouts and data transforms. Read up on data oriented design.

### C++11 Features

The IceShard engine is compiled using C\+\+20, and aims to use only features up to this standard, nothing above. The features listed below are the C\+\+17 features that are known to be working and that we recommend using.

#### `auto`

Using the `auto` type is recommended for declaring complex types such as `Array<Item>::const_iterator`. For simple standard types, such as `int` or `char *` it is usually clearer not to use `auto`.

#### `decltype()`

Use this if you need it.

#### Range based for loops

Use this instead of regular for loops where it makes the code simpler to read.

#### lambda functions

Use lambda functions whenever you need a small local helper function in a function.

```cpp
void format_markdown(const char *s)
{
	auto is_header = [](const char *line) {
    	return line[0] == '#';
    };

    ...
}
```

#### The `noexcept` keyword

As this is a game engine, we do not want expetions to be used at all, thus it's generally a rule of thumb to mark every engine function as `noexcept` and ensure no exceptions are thrown.

#### `stdint.h` types `int8_t`, `uint8_t`, `int32_t`, `uint32_t`, ...

We assume that any compiler compiling the IceShard project has sensible type sizes, i.e.:

* `char` = 8 bits
* `short` = 16 bits
* `int` = 32 bits

Still, for integer sizes other than 32 bit, the `stdint.h` types should be used as they are less ambiguous:

* Use: `int8_t`, `int16_t`, `uint64_t`, ...
* Rather than: `char`, `short`, `size_t`, `long long`, ...

Note that a lot of the code still uses `short` and `long` though. It has not yet been rewritten to use the new types.

For 32-bit integers, the `stdint.h` types are prefered:

* `int32_t` instead of `int`
* `uint32_t` instead of `unsigned`

When you are referring to a string or a buffer of raw data you should still use `char *` rather than `int8_t *`. Use `int8_t` when you want a small integer.

#### `override`, `final`

These keywords should be used to document the intent of virtual methods.

#### `enum class`

Most enums should be written to use `enum class` to avoid the leaky scope of regular enums.

#### `static_assert`

Use this everywhere possible to detect compile time errors.

### Miscellaneous Tidbits

#### `#pragma once`

Use `#pragma once` to avoid multiple header inclusion

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

#### (pointer, size)

When writing a function that takes a pointer and a size/count as parameters, following the standard of the standard C library (`memcpy`, etc) the pointer argument should be passed first and the size argument last. I.e.:

```cpp
void do_something(char *data, unsigned len);
```

An even better approach is to use the `core::data_view` as an argument type, which simillary to `std::string_view` holds a pointer to the data and it's size.

#### (type, name)

When writing functions that take a resource type and resource name as arguments, the type argument should precede the name argument, i.e. as in:

```cpp
bool can_get(ResourceID const& type, ResourceID const& name) const;
```
