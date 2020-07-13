---
title: Select, poll, and epoll on Linux
date: 2020-06-23 09:03
summary: 
category: notes
visible: True
tags: Linux
---

## Motivation

As a server, there are usually more than 1 connection to deal with. A lot of [file descriptors](https://en.wikipedia.org/wiki/File_descriptor)(representing the connection) need to be monitored and responded. Conventional multithread does not work well as it involves costly context switch.

A simple loop would be too slow and inefficient.
```C
for x in open_connections:
    if has_new_input(x):
        process_input(x)
```

Linux offered three ways to monitor the `fd`s: `select`, `poll`, `epoll`.

## Select & Poll

### select
`select` accepts a list of file descriptors to get information about, and returns the information about which `fd` to write or read. See source code on:  
- Github - [`select`](https://github.com/torvalds/linux/blob/v4.10/fs/select.c#L634-L656), [`do_select`](https://github.com/torvalds/linux/blob/v4.10/fs/select.c#L404-L542)
- Bootlin - [`select`](https://elixir.bootlin.com/linux/latest/source/fs/select.c#L722), [`do_select`](https://elixir.bootlin.com/linux/latest/source/fs/select.c#L728)
```C
// sys call for __select__
/********************
inp:  fd for input
outp: fd for output      
exp:  fd for exceptions                
tvp:  timeout value
*********************/
SYSCALL_DEFINE5(select, int, n, fd_set __user *, inp, fd_set __user *, outp, fd_set __user *, exp, struct timeval __user *, tvp)
```

[`fd_set`](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/posix_types.h#L26) is an array of unsigned long integer storing bit values indicating if there is data to write or read. The default size is 1024 bits which means it supports monitoring at most 1024 tasks at the same time. 

Cons of `select`:
1. the sets need to be re-initialized every time before being passed to `select` and cannot be reused. 
2. at most 1024 tasks (can be modified but leads to slow performance).
3. the cost of copying `fd`s from user space to kernel space is high.
4. requires iterating through the whole `fd` list after returning which is `O(n)` complexity.

### poll

Instead of using bitmaps, `poll` accepts an array of `pollfd` to monitor the specific tasks. This is especially useful when the `fd`s you want to monitor is sparse because you don't need to go through all the `fd`s every time but only iterate through the specific ones. Also, instead of having only 3 set of events like `select` (read, write, exception), the `events` variable in `pollfd` allows you to set events you'd like to monitor. See a comprehensive list of events [here](see https://elixir.bootlin.com/linux/latest/source/include/uapi/asm-generic/poll.h).  Source code can be found at:

- [Bootlin](https://elixir.bootlin.com/linux/latest/source), definition of [`ppoll`](https://elixir.bootlin.com/linux/latest/source/fs/select.c#L1081) and [`do_sys_poll`](https://elixir.bootlin.com/linux/latest/source/fs/select.c#L960).

```C
// structure of `pollfd`.
struct pollfd {
	int fd;
	short events;  // set events to monitor, can be POLLIN, POLLOUT, etc. 
	short revents; // receive events. Corresponding flags will be set to 1 if the event occurs.
                   // it needs to be reset after every read for next time use.
};

// sys call for __poll__
/*********************
ufds:  fds and events to monitor 
nfds:  number of fds to monitor
*********************/
SYSCALL_DEFINE5(ppoll, struct pollfd __user *, ufds, unsigned int, nfds, struct timespec __user *, tsp, const sigset_t __user *, sigmask,
size_t, sigsetsize)
```


Cons of `select`:  
3 & 4 of `poll`.

## epoll
Due to the cons of `select` and `poll`, they are less widely used.

> level-triggered: triggers when receive buffer is not empty and when send buffer is not full. High frequency.
> edge-triggered: triggers when an empty receive buffer received data and when a full send buffer cleared new space. Low frequency.

### calling procedure
0. related strcuts
```C
    typedef union epoll_data
    {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
    } epoll_data_t;

    struct epoll_event
    {
    uint32_t events;  /* Epoll events */
    epoll_data_t data;    /* User data variable */
    };
```
1. call `epoll_create` to create an `epoll` and return an id.
```C
    int epfd = epoll_create(10);
```
2. register `fd`s with `epoll_ctl`.
```C
    struct epoll_event ev;
    ev.data.fd = accept(sockfd, ...); // open a connection
    ev.events  = EPOLLIN; // register events type
    //              add fd       
    epoll_ctl(epfd, EPOLL_CTL_ADD, ev.data.fd, &ev);
```
3. call `epoll_wait` to wait for updates and iterate through all active events.
```C
    struct epoll_event events[5];
    while (1) {
        nfds = epoll_wait(epfd, events, 5, 10000);

        for (int i = 0; i < nfds; ++i) {
            read(events[i].data.fd, ...);
            process();
        }
    }
```

The `epoll_wait` will re-order the `fd`s such that active ones are at the beginning (this is efficient  due to the implementation of a red-black tree that allows fast O(lgN) insertion and deletion.)

### Why is epoll fast

In `epoll_create`, a R-B tree is created for storing sockets and a linked list for storing ready events. And `epoll_wait` only needs to check if the list is empty, if not, it blocks and waits and return otherwise.

So how is the linked list maintained? When we call `epoll_ctl`, aside from registering the socket with the RB-tree of the epoll instance, it also registers a callback function for processing interrupts. If the interrupt for the event arrives, the callback function will insert socket into the list. So whenever a socket receives data, the kernel copies the data from the Internet to the kernel and insert the socket into the list.

Due to more syscalls and more switches between kernel mode and user mode, `epoll` can be more costly than `select` when almost all sockets are active.

To conclude, `epoll` is more efficient in that:
1. Use mmap to avoid costly memory copy.
2. Use RB-tree for storing and re-ordering `fd`s to avoid iterating through all `fd`s.
3. Reduces `fd`s copy between user space and kernel space.
4. Use callback function to avoid performance degrade in high pressure scenarios.

## References
[Async IO on Linux: select, poll, and epoll](https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/)  
[深入理解 Epoll](https://zhuanlan.zhihu.com/p/93609693)