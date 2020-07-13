---
layout: post
title:  A glance at the sorting function of C++
category: notes
date:   2020-07-10 05:26:00 +0800
visible: False
tags: C++
---

No matter how hard you try to implement a sorting function, it is very unlikely that its performance will be superior than the `sort` function provided by the `algorithm` library of C++ and the `qsort` provided by `stdlib` library of C. So, where do they do better?


# qsort

We first discuss it on a higher (more abstract) level and then go into details of implementation.  

## Core idea
The core of quick sort consists of 

+ Pick a median pivot from among first element, element in the middle, last element to avoid a pathological pivot that degrades the performance of quick sort (e.g. if the the smallest element is picked every time, quicksort degrades to its worst performance O(N^2)).
+ Swap the first, middle, last element such that the one with median value is in the middle.
+ __Collapse the Walls__ section. This is the main reason that makes the algorithm faster than most.
  + Initialize two pointers that point to the first and last element respectively. 
  + Move left pointer to point to an element larger than pivot
  + Move right pointer to point to an element smaller than pivot
  + If left pointer is on the left side of right pointer, swap their value and continue. 
    (regular so far)
  + Now there are two partitions, based on the size of both partitions: 1) if both are smaller than the threshold, pop pointers from the stack (if any) for next iteration of partition. 2) if only one partition is smaller than the threshold, move pointers for iteration on the small partition. 3) if both are larger than the threshold, move pointers for iteration on the smaller partition and save pointers for the larger partition on the stack.
  + Use insertion sort for every threshold (e.g. every 4 elements if the threshold is 4).

## Some details

### non-recursive implementation of qsort
A stack is used for storing partitions pointers to avoid expensive recursive function call if both partitions are not small. Small partitions (size is smaller than the threshold) are ignored first because insertion sort is more efficient for partitions below the threshold size.

```C++
#define STACK_SIZE             (CHAR_BIT * sizeof (size_t))
#define PUSH(low, high)        ((void) ((top->lo = (low)), (top->hi = (high)), ++top))
#define POP(low, high)         ((void) (--top, (low = top->lo), (high = top->hi)))
#define STACK_NOT_EMPTY        (stack < top)

/* Stack node declarations used to store unfulfilled partition obligations. */
typedef struct
  {
    char *lo;
    char *hi;
  } stack_node;
```

An efficient implementation of a stack. `top` refers to the pointer points to the top of the stack. Each node in the stack stores the low and high pointers of partitions still to be sorted. Each node is of size 16 bytes on a 32-bit machine. 

### choice of pivot

```C++
// simplified version
if (mid < low) swap(mid, low);
if (high < mid) {
    swap(mid, high);
    if (mid < low) {
        swap(mid, low);
    }
}
```

A straightfoward way to sort three numbers. A good pivot can eliminate certain extraneous comparisons.

# sort

`sort(first, last)` sorts the elements in the range [`first`, `last`) into ascending order where `first` and `second` are random access iterators for which `swap` is properly defined.



# References
[__qsort.c source code__](https://code.woboq.org/userspace/glibc/stdlib/qsort.c.html)  



