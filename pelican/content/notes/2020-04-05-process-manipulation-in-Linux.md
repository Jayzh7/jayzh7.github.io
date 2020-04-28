---
layout: post
published: true
title: Process manipulation in Linux
date: 2020-04-05 13:55:00 +0100
category: notes
tags: Linux
comments: true
summary: Process monitoring and manipulation commands in Linux
ext-js: https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML
---
<!-- $$ \usepackage{amsmath} $$  -->
## Intro
In this blog, we will talk about process monitoring commands (`ps` and `top`) and process manipulation (kill, pause, suspend, etc.)  in Linux.

## Process Monitoring
### ps - process snapshot
To monitor the number of running processes on the machine the user is logged into, a general purpose process status command `ps` can be used. 

Formally speaking, 
> ___ps___ displays information about a selection of active processes. If you want a repetitive update of the selection and the displayed information, use ___top___ instead.  

By default (no arguments provided), process ID (**PID**), terminal associated with the process (**TTY**), the cumulated CPU time in hh:mm:ss format (**TIME**), and executable name (**CMD**) of **the current shell** are displayed.

#### command line options

`ps` accepts several kinds of options:
1. UNIX options, can be grouped, must be preceded by a dash.
2. BSD options, can be grouped, ___no dash___. Process state included in the display. Processes on other terminals are also included.
3. GNU long options, preceded by two dashes.

we will mainly focus on the UNIX options as an foundation.


| Option | functionality |
| ------ | ------------- |
 `-A` or `-e` | List all processes
 `-f` | full-format listing
 `-x` | current user owned process only
 `-u user_id` | processes owned by specified user
 `-t tty_name` | select process by terminal
 `-o format` | display in user defined format


* display top running processes by memory usage
```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head
```


### top  
As mentioned before, `top` can be used to monitor the running system in real-time, i.e., regularly update the displayed information. It can also display system summary information, of which order, type, size can be configured as per user's perference.
#### operation
`top` command often produces a long output which usually streches out a window. To slide, motion keys (`up`, `down`, `left`, `right`), `PgUp`, `PgDn`, `Home`, `End` work as they usually do.

#### command line options
As always, you can rely on `-h` for help. Other useful options include:
1. Exit after specific repetition. By default, it keeps refreshing until you press `q`.
```bash
    top -n 10
```
2. Only display processes belong to a specific user.
```bash
    top -u user_id
```
3. Highlight running processes color by pressing `z`.
4. Show absolute path of processes by pressing `c`.
5. Kill a running process while keeping the command running by pressing `k`.
6. Sort the processes by CPU utilisation by pressing `shift`+`p`.
7. see [manual](http://man7.org/linux/man-pages/man1/top.1.html) for more options and explanation of the summary info.  


## Process manipulation
> A process refers to a program in execution; it's a running instance of a program.
Each process has its unique process ID (PID) and its parent process ID (PPID).
### Types of processes
* Foreground processes (interactive processes).
* Background processes (non-interactive processes).  
or
* Parent process - create other processes at runtime
* Child process - get created by other processes.

Init process is the mother of all processes on the system with a PID of 1. It adopts all orphaned processes. The PID of a process can be obtained by command `pidof process_name`.

### States of processes
A linux process state transition graph. Image credit goes to [Tecmint](https://www.tecmint.com/linux-process-management/).
![Process state]({static}/images/ProcessState.png)

### Creation of a process
* Use system functions - simple but inefficient and insecure
* Use fork() and exec() - more flexibility, speed, and security.

### Background jobs
To start a process in the background (non-interactive), `&` symbol makes the command precedes it run in the background. (BTW, `&&` concatenates two sequential operations).
>If a command is terminated by the control operator &, the shell executes the command in the background in a subshell. The shell does not wait for the command to finish, and the return status is 0.

Once you've started a job in the foreground, you can also send it to the background by suspending it first using `Ctrl+Z`, and continue it in the background by `bg` command. To bring it foregound againn, use command `fg %job_id`, the job id in which `job_id` can be found by using `jobs`. It is worth noting that the process will still output to the terminal even it is in the background and it is going to be messy if you try to type a command when the output is frequent (your input will be mixed with the process output).

```bash
qsh@ghildes> sleep 100
(Ctrl+Z)
[1]+  Stopped                 sleep 100
qsh@ghildes> jobs
[1]+  Stopped                 sleep 100
qsh@ghildes> bg %1 
[1]+ sleep 100 &
qsh@ghildes> fg %1
sleep 100
```

If only one job in the bg/fg, job id can be omitted.  

Delete a suspended process by `kill` command followed by job id. Delete a process running in the foreground can be done by a simple `Ctrl+C`. Also, process can be killed by providing its PID.
```bash
qsh@ghildes> xclock &
[1] 31176
qsh@ghildes> kill -9 31176
[1]+  Killed                  xclock
```
```bash
qsh@ghildes> xclock &
[1] 32012
qsh@ghildes> kill %1
[1]+  Terminated              xclock
```
## Reference  
[_ps(1) - Linux manual page_](http://man7.org/linux/man-pages/man1/ps.1.html)  
[_top(1) - Linux manual page_](http://man7.org/linux/man-pages/man1/top.1.html)
[_Manipulating processes in Linux_](http://www.physics.smu.edu/coan/linux/processes.html)  
[_top command in Linux with Examples_](https://www.geeksforgeeks.org/top-command-in-linux-with-examples/)  
[_30 Useful ‘ps Command’ Examples for Linux Process Monitoring_](https://www.tecmint.com/ps-command-examples-for-linux-process-monitoring/)
---
