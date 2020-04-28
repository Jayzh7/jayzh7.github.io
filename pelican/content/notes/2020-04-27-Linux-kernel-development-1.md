---
title: Linux kernel development - Introduction
date: 2020-04-28 21:15
category: notes
visible: True
tag: Linux, Linux kernel
--- 

## Overview of OS and Kernels

### OS components

OS is considered the parts of the system responsible for basic use and administration which includes:

+ kernel and device drivers
+ bootloader
+ command shell
+ other user interface
+ basic file and system utilities

Basically stuff that a user will need with UI as the outermost part and kernel as the innermost part.

Typical components of a kernel are:

+ interrupt handler to service interrupt requests
+ a scheduler to share processor time among processes
+ a memory management system to manage process address spaces
+ system services like networking and IPC

### User application and kernel communication

**Kernel space**: a protected memory space with full access to hardware where kernel resides.  
**User space**: where applications can only see a subset of machine's available resources and can perform certain system functions, directly access hardware, access memory outside of that allotted them by the kernel.

Applications running on the system communicates with the kernel via _system calls_. This is the fundamental manner in which applications get work done.

An application typically calls functions in a library (like _C library_) that in turn rely on systems calls to instruct the kernel to do tasks

Kernel also manages the system's hardware by supporting _interrupts_. However, it can disable interrupts (either all or some) for synchronization. The interrupt handlers do not run in a process context but in a special _interrupt context_ which allows or intends to let it respond and exit quickly.

In Linux, we can generalize that each processor is doing exactly one of the three things at any given moment：

+ Executing user code in a process in user-space
+ Executing on behalf of a specfic process in kernel-space, in process context
+ Handling an interrupt in kernel-space, in interrupt context  
![Process state]({static}/images/figure1_1.png)  
Figure 1.1 Relationship between applications, the kernel, and hardware.

## Linux Versus Classic Unix Kernels

Common ancestry and same API, modern Unix kernels share various design traits. A Unix kernel is typically a monolithic (means single and large) static binary that runs in a single address space. Unix systems usually have a paged memory-management unit (MMU). Most Linux kernels also have it while some small embedded systems can also run without one.

> Monotlithic kernel Versus Microkernel  
Monolithic kernel: entirely as a single process running in a single address space. Thus, such kernels existon disk as single static binaries.  
Microkernel: functionality of the kernel is broken down into separate processes. Only servers _absolutely_ requiring such capabilities run in a privileged execution mode. Others run in user-space. All are separated into different address spaces in which direct function invocation is not possible. They communicate and invoke 'services' via an IPC - message passing. High fault tolerance due to the existance of multiple servers.  

Linux is a monilithic kernel while it borrows a lot from microkernel design:

+ modular design
+ kernel preemption
+ kernel threads
+ capability to dynamically load separate binaries (kernel modules) into kernel image.  

It has no performance-sapping features of microkernel designs. Everything runs in kernel mode with direct function invocation.  

Notable difference between Linux kernel and classic Unix kernel:

+ The Linux kernel is preemptive even for tasks executing in the kernel.
+ Linux does not differentiate between threads and normal processes. All processes are same while some (threads) happen to share resources. 
+ Linux provides object-oriented device model with device classes, hot-pluggable events, a user-space device filesystem(sysfs).

## Linux version

3 digits: major release, minor release, revision  
4 digits: 3 digits + stable version

An even minor release indicates a stable version.
