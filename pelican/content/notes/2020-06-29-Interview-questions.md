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

## Memory alignment

```
struct {
    int a[5];
    double b;
    char c;
};
```
The size of the above struct is 24 rather than 20 due to memory alignment. The alignment is based on its largest member which is `double` (size 8) in this case. Therefore, its size should be multiples of 8 (size of double).

## Size of objects

An object of an empty class has a size of 1. If it has virtual functions, it takes some extra space to store a virtual pointer (8 bytes for 64-bit systems). Static variable, member functions, constructor and destructors are not owned by the object therefore occupying no space for objects.

## Inheritance and derived class

The derived class inherit all contents from the base class, some of which may not be useful leading to data redundancy. Therefore, it must be used with care to avoid heavy cost of copying, constructing and destroying of a derived class object.

There are three types of inheritance:
+ public inheritance: public and protected members from base class remain the same in the derived class, private members from base class is not inherited.
+ private inheritance: public and protected members from base class go to private in the derived class. 
+ protected inheritance: public and protected members from base class go to protected.

Protected members are accessible to friends and member functions in a derived class.

## Override, overwrite, and overload

## singleton

There is only one instance of the class. The instance can only be accessed by the static pointer obtained by calling a public member function.

```C++
Class CSingleton {
private:
    CSingleton() {
        // constructor
    }
    static CSingleton *m_pInstance;
public:
    CSingleton* getInstance() {
        if (!m_pInstance)
            m_pInstance = new CSingleton();

        return m_pInstance;
    }
}

// initialize the static instance to NULL
CSingleton* CSingleton::m_pInstance = NULL;
```

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

## Virtual functions
Every class that uses virtual functions is given a unique virtual table. The table is a static array that the compiler sets up at compile time. It contains one entry for each virtual function that can be called by objects of the class.

![vtable]({static}/images/VTable.gif)

+ Each object has a hidden virtual pointer that points to the virtual table.
+ Each entry of the table is filled with the most-derived function that the class of the object can call.

## Early binding and late binding
Early binding means the compiler knows the address of the function to call at compile time. 

For some programs, it is not possible to know which function to call at compile time, only until runtime. One way to do this is to use function pointers. 
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

# Linux

## inode

__inode__ (index node) is a data structure in Unix-like file systems. It is used for describing a file-system object such as a file or a directory. Each inode stores disk block(a block usually contains a fixed number(power of 2) of sectors which are commonly 4K bytes) locations of the object's data, and attributes like file type, previlige, ownership, time (create, modify, access), etc.

There is a map that maps the file name to inode index, with which the inode can be found in the inode array. Note that the inode does not contain file name because the general use case is to identify the inode by file name.

## Thread vs Process

In Linux, `fork()` creates new processes. These new processes are called child processes and they intially share all the segments like `text`, `stack`, `heap`, etc until child tries to make changes to them.

# Database

## Pessimistic and Optimistic locking
### Optimistic locking
When you read a record, take note of the 'version number'(could also be timestamp, entire state of the row itself, checksums, etc) and check that the version has not been changed before writing back.

The transaction will be aborted if the record is dirty (different version).

It is the most appilcable to __high-volumn systems__ and __three-tier architectures__ where connection is not necessary to be kept.

### Pssimistic locking
Lock the record for exclusive use until transaction completes. It has better integrity but is vulnerable to __Deadlocks__.

# Compiler 


# References
[The new C: inline functions | Dr Dobb's](https://www.drdobbs.com/the-new-c-inline-functions/184401540)  
[Inline function - Wikipedia](https://en.wikipedia.org/wiki/Inline_function)  
[inode - Wikipedia](https://en.wikipedia.org/wiki/Inode)  
[Optimistic vs. Pessimistic locking - Stackoverflow](https://stackoverflow.com/questions/129329/optimistic-vs-pessimistic-locking)  
[What are Linux Processes, Threads, Light Weight Processes, and Process State - THE GEEK STUFF](https://www.thegeekstuff.com/2013/11/linux-process-and-threads/#:~:text=Linux%20Threads%20vs%20Light%20Weight,known%20as%20multi%2Dthreaded%20process.&text=Each%20thread%20is%20viewed%20by,different%20from%20other%20normal%20processes.)  