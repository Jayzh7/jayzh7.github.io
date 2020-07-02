---
title: Interview questions - programming language details
date: 2020-06-29 11:47
summary: 
category: Notes
visible: True
tags: CS
---

# C++

___

## Macros

Macro calls will be replaced with the processed copy of the body, which is called __expansion__ of the macro call. It is faster than function calls because it does not generate actual function calls.

Two types of macros
* Object-like macros
* function-like macros

Macros must be used with care. In the following example, the `++a`, `++b` expression will be evaluated multiple times in the macro causing unexpected results.

```C++
#define getrandom(min, max) \
    ((rand()%(int)(((max)+1)-(min)))+min)

a = 1;
b = 5;
getrandom(++a, ++b);
```



## inline functions

A function defined in the body of a class declaration is an inline function. A class's member function defined outside the class definition can also be declared inline by using `inline` keyword.

### Purpose
1. It serves as a compiler directive so that the compiler will substitute function call with the function code as __inline expansion__, thereby saving the overhead of a function call.

> When a function is called, the following steps are usually taken
> 1. argument values copied to the stack or special registers.
> 2. A return address is created and stored on the stack or to a register.
> 3. The program branches to the function.
> 4. A stack frame is setup for the local variables of the function
> 5. At the end of function, the stack frame is torn dowm.
> 6. The returned address is retrieved and the pc goes back to that address.

2. Help compiler for better optimization (calculations inside inline function may be reused by the body which is not possible for a function call).

3. More advantages provided by converting a function call to a block of code see [here](https://www.drdobbs.com/the-new-c-inline-functions/184401540).

### Disadvantages

By __inline expansion__, the size of program code gets larger, which can lead to more page faults and cache misses.

### Advantage of inline functions over macros

* type safety. Inline functions are subject to the same type checking as normal functions.
* Correct handling of arguments that have side effects. Inline functions evaluate the expressions supplied as arguments before the function body is entered.


# Memory in C
There are three types of segments:

- Code segment, `text`, where the compiled program sits in memory that is typically read-only.
- Uninitialized data segment, `bss`, where zero-initialized global and static variables are stored.
- Data segment, `data`, where initialized global and static variables are stored.
- Heap segment, `heap`, where dynamically allocated variables are stored.
- Stack segment, where local variables are stored.
```
high____________
    |           |
    |___________|
    |  stack    |
    |-----------|
    |    |      |
    |   \|/     |
    |           |
    |   /|\     |
    |    |      |
    |-----------|
    |   heap    |
    |___________|
    |   bss     |
    |___________|
    |   data    |
    |___________|
    |   text    |
low |___________|
```
# References
[The new C: inline functions | Dr Dobb's](https://www.drdobbs.com/the-new-c-inline-functions/184401540)
[Inline function - Wikipedia](https://en.wikipedia.org/wiki/Inline_function)