---
title: Linux kernel development - Process Management
date: 2020-05-03 16:02
summary: Process and its related concepts like threads, how the Linux kernel manages each process - how they are enumerated within the kernel, how they are created ...
category: notes
visible: True
tag: Linux, Linux kernel
--- 

## The process

___
A _process_ is a program in the midst of execution. They are, however, more than just executing program code (_text_ section in Unix). They also include a set of resources such as open files and pending signals, a memory address space, ...

Threads of execution, aka _threads_, are obejcts of activity within the process. Each process has:

+ unique program counter
+ process stack
+ set of processor registers.

A process begin its life when it is created usually by a `fork()` system call which creates a new process by duplicating an existing one. The _parent_ (process that calls `fork()`) resumes execution after `fork()` and the _child_ (new process) starts execution at the same place where the call to `fork()` returns. The `fork()` returns from the kernel twice: once in the parent process and also in the newborn child.

After a fork, a process is desirable to execute a new, different program. The `exec()` family of function calls create a new address space and loads a new program into it.

Finally, a program exits via the `exit()` system call. This function terminates the process and free all its resources.

> Process is also called task from the kernel's point of view.

## Process descriptor and task structure

___
The list of processes are stored in a circular doubly linked list called the _task list_. Each element in it is a _process descriptor_ of type `struct task_struct`.

### Allocating the process descriptor

The `task_struct` structure is allocated via the _slab allocator_ to provide object reuse and cache coloring. It used to be stored at the end of kernel stack of each process and the location can be calculated via _stack pointer_ so that no register is needed to store the location. Now, it is created dynamically and a new structure, `struct thread_info` lives at the bottom or top of the stack depending on how the stack grows. See figure 3.2. The `thread_info` contains a pointer to the `task_struct`.

![Process kernel stack]({static}/images/figure3_2.PNG)  
Figure 3.2 The process descriptor and kernel stack

### Storing the process descriptor

Each process is identified by a unique _process identification_ value or _PID_ which is a numerical value represented by the opaque type `pid_t`. Due to backward compatibility, the default maximum is 32,768 (16 bit) which is enough for a desktop system but not for large servers. Moreover, the lower the value, the sooner the values will wrap around which destroy the useful notion that higher values indicate later-run processes. It can be modified via `/proc/sys/kernel/pid_max`.

> An opaque type is a data type whose physical representation is unknown or irrelevant.

Inside the kernel, tasks (processes) are typically referenced directly by a pointer to their `task_struct` structure. In fact, most kernel code works directly with it. It is, therefore, useful to quickly look up the process descriptor of the currently executing task via the `current` macro. How this is done is architecture-dependent. Some save it in a register, some with few registers, on the other hand, look it up by the pointer stored in the `thread_info` as figure 3.2 shows.

### Process State

Each process on the system is in exactly one of five states.

+ `TASK_RUNNING` - it is either currently running or on a running queue waiting to run. This is the only possible state for a process executing in user-space.
+ `TASK_INTERRUPTABLE` - it is sleeping (blocked), waiting for some condition to exist (resource, cpu). When the condition exists, the kernel sets its state to `TASK_RUNNING`.
+ `TASK_UNINTERRUPTABLE` - identical to `TASK_INTERRUPTABLE` except that it does not wake up and become runnable if it receives a signal. 
+ `TASK_TRACED` - it is being _traced_ by another process such as a debugger via _ptrace_.
+ `TASK_STOPPED` -  process execution has stopped; the task is not running nor is it eligible to run. It occurs after the task receives the `SIGSTOP`, `SIGTSTP`, `SIGTTIN`, or `SIGTTOU` or any signal while it is being debugged.

![Process kernel stack]({static}/images/figure3_3.png)  
Figure 3.3 Flowchart of process states.

### Process context

When a program executes a system call or trigger an exception, it enters kernel-space. The kernel will be "executing on behalf of the process" and is in _process context_ in which the `current` macro is still valid.

### Process family tree

All processes are descendants of the `init` process whose PID is 1. The kernel starts `init` in the last step of the boot process. The `init` process reads the system _initscripts_ and executes more program, eventually completing the boot process.

Every process has only one parent, 0 or more children. Each `task_struct` has a pointer to the parent's `task_struct` named `parent`, and a list of children named `children`. Process iteration can be done using macros like `for_each_process()` which iterates over the entire task list.

## Process creation

___
Unix takes the unusual approach of creating processes in two steps: `fork()` and `exec()`. 

 `fork()` creates a child process that is a copy of the current task. It differs from its parent in its PID, PPID, certain resources and statistics such as pending signals.

`exec()` loads a new executable into the address space and begins executing it.

### Copy-on-write

Traditionally, upon `fork()`, all resources owned by the parent are duplicated and the copy is given to the child which is inefficient due to too much (mostly unnecessary) copy. Copy-on-write (COW) is a technique to delay or even prevent copying of the data. Instead, the parent and child can share a single copy if it is only read by the child. When it is written to by the child, a duplicate will be made and the child receives a unique copy.

In the common case that a process executes a new executable immediately after forking, the technique prevents the copying and speed up the process creation and execution.

### forking

Linux implements `fork` via the `clone()` system call. This call takes a series of flags (introduced later this chapter). The `clone()`, in turn, calls `do_fork()`. The bulk of the work in forking is handled by `do_fork()` which is defined in __`kernel/fork.c`__. This function calls `copy_process()` and then starts process running. Interesting work done by `copy_process()`:

0, check if the parameters passed in are valid, exit otherwise.
1, it calls `dup_task_struct()`, which creates a new kernel stack, `thread_info` structure, and `task_struct` for the new process. The new values are identical to those of the current task.  
2, check resources limits  
3, members of the process descriptor are cleared or set to initial values.  
4, state set to `TASK_UNINTERRUPTIBLE` to ensure that it does not run.  
5, it calls `copy_flags()` to update flags.  
6, it calls `alloc_pid()` to assign an available PID to the new task.  
7,  depending on the flags passed to `clone()`, it either duplicates or shares resources like signal handler, process address space, and namespace.  
8, cleans up and return a pointer to the new child.

### `vfork()`

The `vfork()` system call has the same effect as `fork()` except that __the page table entries of the parent are not copied__. Instead, the child executes as the sole thread in the parent's address space and the parent is blocked until the child calls `exec()` or exits. It used to offers more optimization when the COW was not used in `fork()`.

## The Linux implementation of threads

___
Threads within the same program share memory address space, open files and other resources. They enable _concurrent programming_ and true _parallelism_ on multiple processor systems.

Linux implements are threads as standard process. A thread is merely a process that shares certain resources with other processes. To some other OS like Microsoft Windows, threads are an abstraction to provide a ligher, quicker execution unit.

### Creating threads

Specific flags indicating shared resources will be passed into the `clone()` system call when creating threads.

```C
clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0);
```

It means shared address space, filesystem resources, file descriptors, and signal handlers.

In contrast, a normal process `fork()` will be

```C
clone(SIGCHLD, 0);
```

and `vfork()` will be

```C
clone(CLONE_VFORK | CLONE_VM | SIGCHLD, 0);
```

More `clone()` related flags are defined in `<linux/sched.h>`.

### Kernel threads

Kernel also performs some operations in the background which is done via _kernel threads_ - standard processes that exist solely in kernel-space. __Kernel threads do not have an address space.__ Their `mm` pointer is `NULL`.

What do kernel threads do? Mostly notably the _flush_ tasks and _ksoftirqd_ task. More details to be followed later.

## Process termination

___

The process termination occurs when the process calls the `exit()` system call, either explicitly when it is ready to terminate or implicitly on return from the main subroutine (C compiler places a call to `exit()` after `main()` returns). A process can also terminate involuntarily when it receives a signal or exception it cannot handle or ignore. Regardless, the work is done by `do_exit()` defined in `kernel/exit.c`.

1, it sets the `PF_EXITING` flag in the `flags` memeber of the `task_struct`.  
2, it calls `del_timer_sync()` to remove any kernel timers. -- note: not found in the source code of 5.6.8
3, it calls `acct_update_integrals()` to write out accounting information.  
4, it calls `exit_mm()` to release the `mm_struct` held by this process. If the address space is not shared, the kernel then destroys it.  
5, it calls `exit_sem()` (also `exit_shm` shared memory?). If the process is queued waiting for an IPC semaphore, it is dequeued here.  
6, it calls `exit_files()` and `exit_fs()` to decrement the usage count of objects related to file descriptors and filesystem data. If either usage counts reach 0, the object is destroyed.  
7, it sets the tasks's exit code inside the `task_struct` to the code passed into the `exit()` -- note: it is before step 4 in 5.6.8.  
8, it calls `exit_notify()` to send signals to the task's parent, reparents any of the task's children to another thread in their thread group or the init process and sets the task's exit state to `EXIT_ZOMBIE`.  
9, it calls `schedule()` to switch to a new process. Because the process is not schedulable anymore, this is the last code the task will ever execute. `do_exit()` never returns. --note: the last line of code of `do_exit()` is actually `do_task_dead()` in which the `__schedule()` is called. A `BUG();` follows the `__schedule(false);`.  

> side note: the unlikely macros that appear frequently in the source code are hint to compiler that the branch is not likely to be taken so that a lot misprediction as well as the flush can be avoided.

### Removing the process descriptor

So far, all objects owned solely by the task were freed. The task is not runnable and is in the `EXIT_ZOMBIE` state. The only memory it occupies is its kernel stack, the `thread_info` and `task_struct`. After its parent or the kernel is notified, the remaining memroy will be freed and returned to the system by `release_task()` defined in `kernel/exit.c` which does the following:

1, it calls `__exit_signal()`, which calls `__unhash_process()`, which in turn calls `detach_pid()` to remove the process from the pidhash and remove the process from the task list.  
2, `__exit_signal()` releases any remaining resources used by the process and finalize statistics and bookkeeping.  
3, if the task was the last member of a thread group, and the leader is a zombie, then the zombie leader's parent will be notified.  
4, `release_task()` calls `put_task_struct()` to free pages containing the process's kernel stack and `thread_info`, and deallocate the slab cache containing the `task_struct`.  

### The dilemma of the parentless task

If a parent exits before its children, some mechanism must exist to _reparent_ any child tasks to a new process, otherwise parentless terminated processes would forever remain zombies.

The solution is to reparent a task's children on exit:

1 another process in the current thread group  
2 the first ancestor process which prctl'd itself as a child_subreaper for its children (like a service manager).  
3 the `init` process in our pid namespace.  

This is done in the `find_new_reaper()` (trace stack: `forget_original_parent()` <- `exit_notify()` <- `do_exit()`)

> side note: some macros like RCU_INIT_POINTER are defined in a construct like `do {} while (0)` for several reasons: 1, it can include multiple statements; 2, it can be followed by a semicolon; 3 it can be used within an `if` statement.

(This paragraph is kinda confusing)  
The rational behind having both a child list and a ptraced list: when a task's parent exits, it must be reparented along with its siblings. In early version kernels, this is done by a loop over _every_ process in the system to look for children. By adding a ptraced list, a list of a process's children is kept to avoid the wild search.
