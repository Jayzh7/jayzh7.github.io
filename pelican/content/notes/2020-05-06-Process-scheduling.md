---
title: Linux kernel development - Process scheduling
date: 2020-05-06 10:03
summary: Process scheduler - the kernel subsystem that puts processes to work.
category: notes
visible: False
tags: Linux, Linux kernel
--- 

The process scheduler aims to best utilize processor time. Deciding which process(es) to run next, given a list of runnable processes, is the fundamental decision that the scheduler must take.

## Multitasking

There are two type of multitasking operating systems: _cooperative multitasking_ and _preemptive multitasking_.

Linux, like all Unix variants and most modern OSs, implements preemptive multitasking. In preemptive multitasking, the scheduler decides when a process is to cease running and a new process to begin running. The act of involuntary suspending a running process is called _preemption_. The time that a process runs before it is preempted is predetermined and is called the _timeslice_ of the process. The timeslice is dynamically calculated as a function of process behavior and system policy.

## (History of) Linux's process scheduler

From Linux's first version in 1991 to the 2.4 kernel series, the Linux scheduler was simple in design but scaled poorly in light of many runnable processes or many processors.

In response, Linux received a new schedular in the 2.5 kernel series. It scaled effortlessly as Linux supported large "iron" with tens if not hundreds of processors. However, it had several pathological failures related to latency-sensitive applications (e.g., interactive processes).

In 2.6 kernel series, the _Rotating Staircase Deadline_ scheduler was introduced. It introduced the concept of _fair scheduling_, borrowed from queuing theory. It inspired the eventual replacement in kernel version 2.6.23, the _Completely Fair Scheduler_, or _CFS_.

## Policy

### I/O-bound versus Porcessor-bound processes

Processes can be classified as:

+ _I/O-bound_ that spends much of its time submitting and waiting on I/O requests.
+ _processor-bound_ that spend much of its time executing code

These classifications are not mutually exclusive. Processes can exhibit both behaviors simultaneously.

There are two conflicting goals for the scheduling policy to satisfy:

+ low latency
+ high throughput

### Process priority

Linux kernel implements two separate priority ranges:

+ _nice_ value, from -20 to +19 with a default of 0. Higher value corresponds to lower priority - you are being "nice" to other processes. (use command `ps -el`, they are under column _NI_)
+ _real-time priority_, configurable, by default range from 0 to 99. Higher values corresponds to greater priority. All real-time processes are at a higher priority than normal processes. (use `ps -eo state,uid,pid,ppid,rtprio,time,comm`, they are under column _RTPRIO_)

### Timeslice