---
title: Linux kernel development - Process scheduling
date: 2020-05-06 10:03
summary: Process scheduler - the kernel subsystem that puts processes to work.
category: notes
visible: True 
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

The timeslice is the numeric value represents how long a task can run until it is preempted.

timeslice too low -> high overhead switching tasks  
timeslice too long-> high latency

The default timeslice for many OSs is rather low - 10 ms for the sake of better interactive experience. Linux's CFS scheduler, however, does not assign timeslices to processes. Instead, it assigns processes a proportion of the processor. So the timeslice of a process is determined by:

+ the load of the system
+ nice value of the process

When can the newly runnable process preempt the running process:

+ it has consumed a smaller proportion of the processor than the running process

### The scheduling policy in action

Consider a system with 2 runnable processes: a video encoder and a text editor.

To achieve the following goals for the text editor for good interactive experience:

+ large amount of processor time
+ ability to preempt the encoder

what will OSs do?

+ most OSs: the text editor will be given a higher priority and larger timeslice
+ advanced OSs: same but it detects interactive processes and do it automatically
+ Linux: because the text editor is blocked most of the time, its consumed timeslice is low. Thus, it is more likely to preempt the video encoder even if they are at the same nice level.

## The Linux scheduling algorithm

___

### Scheduler classes

The Linux scheduler is modular, enabling different algorithms to schedule different types of processes. This modularity is called _scheduler classes_. Each scheduler class has a priority. The base scheduler which is defined in `kernel/sched.c` iterates all scheduler class in order of priority. Who runs next will be decided by  __the highest priority scheduler class with a runnable process__.


### Process scheduling in Unix systems

The CFS is for normal processes, called `SCHED_NORMAL` in Linux. As aforementioned, timeslices are absolute value determined by nice values. This leads to suboptimal switching behavior:

+ thread A with nice value +20 (lowest priority), thread B with high priority will be assigned 5ms (1/21), 100ms (20/21) processor time respectively.  
+ thread A and B both with nice values +20, both will be assigned 5ms which leads to a intensive context switch.

A second problem concerns relative nice values and the nice value to timeslice mapping.

|nice value|timeslice|
|----------|-------|
| 0| 100|
| 1| 95 |


|nice value|timeslice|
|----------|-------|
| 18| 10|
| 19| 5 |

### Fair scheduling

Some concepts:  
> _timeslice_： proportional to its weight divided by the total weight of all runnable threads.  
> _targeted latency_: the approximation of the "infinitely small" scheduling duration for the actual timeslice calculation.  
> _minimum granularity_: the floor of _targeted latency_, by default 1ms.  

The proportion of processor time that any process receives is determined only by the relative different in niceness. __CFS__ is called a _fair scheduler_ because it gives each process a fair share. But it is still not prefectly fair when the number of tasks is too high that the _minimum granularity_ is reached. The lower bound for unfairness is O(n).

## The Linux scheduling implementation
___

This part will be about CFS's actual implementation, which lives in `kernel/sched_fair.c`. Specifically, four components of CFS:

+ Time accounting
+ Process selection
+ The scheduler entry point
+ Sleeping and waking up

### Time accounting

All process scheduler must account for the time that a process runs.

#### The scheduler entity structure

```C
struct sched_entity {
    /* For load-balancing: */
    struct load_weight		load;
    unsigned long			runnable_weight;
    struct rb_node			run_node;
    struct list_head		group_node;
    unsigned int			on_rq;

    u64				exec_start;
    u64				sum_exec_runtime;
    u64				vruntime;
    u64				prev_sum_exec_runtime;

    u64				nr_migrations;

    struct sched_statistics		statistics;
    // ...
    // More config specific variables 
}
```
CFS keeps the account for the time that each process runs in a _scheduler entity structure_, `struct sched_entity`, defined in `<linux/sched.h>`. This structure is embedded in the _process descriptor_, `struct task_struct` as a member named `se`.

#### The virtual runtime

The `vruntime` variable stores the `virtual runtime` of a process which is the actual runtime normalized by the number of runnable processes. CFS uses it to account for hong long it has run and how much longer it ought to run.

The function `update_curr()`, defined in `kernel/sched/fair.c`, manages the accounting:

```C
/*
 * Update the current task's runtime statistics.
 */
static void update_curr(struct cfs_rq *cfs_rq)  
{
	struct sched_entity *curr = cfs_rq->curr;   // curr points to the current running entity on this runqueue
	u64 now = rq_clock_task(rq_of(cfs_rq));     // get the CPU runqueue to which this cfs_rq is attached and 
                                                // get its clock as now
	u64 delta_exec;

	if (unlikely(!curr))
		return;

	delta_exec = now - curr->exec_start;        // calculate the execution time of the current process
	if (unlikely((s64)delta_exec <= 0))
		return;

	curr->exec_start = now;

	schedstat_set(curr->statistics.exec_max,    // update maximum execution time
		      max(delta_exec, curr->statistics.exec_max));

	curr->sum_exec_runtime += delta_exec;       // update total execution time
	schedstat_add(cfs_rq->exec_clock, delta_exec);
                                                // update vruntime with weighted execution time
	curr->vruntime += calc_delta_fair(delta_exec, curr);
	update_min_vruntime(cfs_rq);

	if (entity_is_task(curr)) {                 // an entity is task if it doesn't "own" a runqueue
		struct task_struct *curtask = task_of(curr);

		trace_sched_stat_runtime(curtask, delta_exec, curr->vruntime);
		cgroup_account_cputime(curtask, delta_exec);
		account_group_exec_runtime(curtask, delta_exec);
	}

	account_cfs_rq_runtime(cfs_rq, delta_exec);
}
```

A few related structures:

+ `struct rq`： main, per-CPU runqueue data structure.
+ `struct cfs_rq`: CFS-related fields in a runqueue. It contains a pointer to the CPU runqueue to which it is attached.

`update_curr()` is invoked periodically by:

+ the system timer 
+ whenever a process becomes runnable or blocks, becoming unrunnable.

Therefore, `vruntime` accurately measures the runtime of a given process and indicates which process should run next.

### Process selection

Simple rule: Picks the process with the smallest `vruntime`.

However, how to implement this function effectively is not simple. CFS uses a `red-black tree` to manage the list of runnable processes so that the task with smallest `vruntime` can be found in O(1) time.

### Picking the next task

Finding the smallest node in a red-black tree takes O(logN). The function that performs the selection is `__pick_next_entity()` defined in `kernel/sched/fair.c`. 

```C
static struct sched_entity *__pick_next_entity(struct sched_entity *se)
{
	struct rb_node *next = rb_next(&se->run_node); // find the next node of the current running node

	if (!next)
		return NULL;

	return rb_entry(next, struct sched_entity, run_node);
}
```
This snippet differs from the version referenced by the book. I hope to go back later.

### Adding processes to the tree

When a process becomes runnable or is first created via `fork()`, we need to add it to the rbtree which is done by `enqueue_entity()`. It first executes `update_curr()` to update the runtime and other statistics and then invoke `__enqueue_entity()` to do the actual heavy lifting of inserting. 

### Removing processes from the tree

When a process blocks (becomes unrunnable) or terminates (cease to exist), CFS will remove it from the rb-tree. This is done by `dequeue_entity()`. First, runtime statistics are updated first, then the node is removed by calling a helper function `__dequeue_entity()`. Lastly, the `rb_leftmost` cache will be updated and the leftmost node will be updated if the removed process was the leftmost one. 

### The scheduler entry point

 The main entry point into the process scheduler is the function `schedule()` defined in `kernel/sched.c`. Its core is the invocation of `pick_next_task()` which goes through all scheduler classes and selects the highest prirority process in the highest priority class:

```C
static inline struct task_struct *
pick_next_task(struct rq *rq, struct task_struct *prev, struct rq_flags *rf)
{
	const struct sched_class *class;
	struct task_struct *p;

    // If all tasks are in the fair class, we can call that function diretly, but only if
    // the prev task wasn't of a higher scheduling class
    if (likely((prev->sched_class == &idle_sched_class ||
		    prev->sched_class == &fair_sched_class) &&
		   rq->nr_running == rq->cfs.h_nr_running)) {

		p = pick_next_task_fair(rq, prev, rf);
		if (unlikely(p == RETRY_TASK))
			goto restart;

		/* Assumes fair_sched_class->next == idle_sched_class */
		if (!p) {
			put_prev_task(rq, prev);
			p = pick_next_task_idle(rq);
		}

		return p;
	}

    // ...
    
    for_each_class(class) {
        p = class->pick_next_task(rq);
        if (p)
            return p;
    }

    BUG();
}
```

### Sleeping and Waking up

A task sleeps(blocks) while it is waiting for some event which can be:

+ a specified amount of time
+ data from a fil I/O
+ hardware event
+ obtaining a contended semaphore

The task marks itself as sleeping, puts itself on a wait queue, removes itself from the rbtree of runnable and calls `schedule` to select a new process to execute. Waking it up is the inverse: set as runnable, remove from the wait queue, add to rbtree.

There are two states associated with sleeping, `TASK_INTERRUPTIBLE` and `TASK_UNINTERRUPTIBLE`. They differ only in that in the `TASK_UN...` state ignore signals, whereas the other one wake up prematurely and respond to a signal if one is issued.

### Wait queues

A wait queue is a simple list of processes waiting for an event to occur. When the associated event occurs, the processes on the wait queue are awakened.

If the task go to sleep _after_ the condition becomes true, the task might sleep indefintely. Therefore, to avoid the race condition, the followings steps should be taken:
```C
/* ‘q’ is the wait queue we wish to sleep on */
    DEFINE_WAIT(wait);
    add_wait_queue(q, &wait); 
    while (!condition) { /* condition is the event that we are waiting for */
        prepare_to_wait(&q, &wait, TASK_INTERRUPTIBLE);
        if (signal_pending(current))
            /* handle signal */
            schedule();
        }
    finish_wait(&q, &wait);
```
1. creates a wait queue entry via the macro `DEFINE_WAIT()`  
2. Add itself to a wait queue via `add_wait_queue()`.  
3. calls `prepare_to_wait()` to change the process state to either `TASK_INTERRUPTIBLE` or `TASK_UNINTERRUPTIBLE`. this function also adds the task back to the queue if necessary.  
4. If the state is set to `TASK_INTERRUPTIBLE`, a signal wakes the process up which is called _spurious wake up_.  
5. When the task awakens, it checks whether the condition is true. If true, exits the loop. Otherwise, calls `schedule()`.  
6. Now that the condition is true, the task set itself to `TASK_RUNNING` and removes itself from the wait queue via `finish_wait()`.

### Waking up

Waking is handled by `wake_up()` which calls `try_to_wake_up()` that does the following:

1. sets the task's state to `TASK_RUNNING`.
2. calls `enqueue_task()` to add the task to the rbtree
3. sets `need_resched` if the new task's priority is higher than that of the current task.

> Note: There are spurious wake-ups. A task is awakened does not mean that its associated event has occured; sleeping should always be handled in a loop that ensures that the condition is satisfied.

![Sleeping and waking up]({static}/images/figure4_1.PNG)
figure: sleeping and waking up flowchart

## Preemption and context switch

Context switch is handled by `context_switch()` defined in `kernel/sched/core.c`. It is called by `schedule()` when a new process has been selected to run. It does:

1. calls `switch_mm()`, declared in `<asm/mmu_context.h>`, to switch the virtual memory mapping.
2. calls `switch_to()`, declared in `<asm/system.h>`, to switch the process state, which involves: 1, saving and restoring stack information and processor registers and any other architecture specific state that must be managed and restored on a per-processor basis.

