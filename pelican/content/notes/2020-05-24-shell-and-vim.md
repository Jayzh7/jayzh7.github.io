---
layout: post
published: true
title: How do shell and Vim coexist?
date: 2020-05-24 22:08
summary: It is often the case that we want to modify the code according to the output of a command. How do we keep them both at the same time? 
category: notes
visible: False
tags: Linux 
---

There are two common commands to achieve this, `tmux` and `screen`.

## tmux
__tmux__ : terminal multiplexer
>__tmux__ is a program that runs in a terminal and allows multiple other terminal programs to be run inside it.

__tmux__ can:

+ protect running programs on a remote server from connection drops
+ allows programs on a remote server to be accessed from multiple local PCs.
+ work with multiple programs and shells together in one terminal

### related concepts

tmux keeps all its states in a single main process, called the tmux server. 

### Usage

#### create a session
A new session is created using the `new-session` command - `new` for short:
```bash
$ tmux new
$ tmux new -smysession # This creates a new session called mysession
```
The tmux server starts upon the creation of the first session. The new session will have one window with a single pane containing the shell.

#### The prefix key

All keys that control tmux have a prefix `Ctrl+b`. Once the prefix is presses, tmux waits for another key to execute corresponding commands.

> help key: `Ctrl+b ?` enters view mode to show text

#### Commands and flags
___
#### enter the command prompt
`Ctrl+b :` Commands can be entered at the prompt similar to at the shell. Output will either be shown for a short period in the status line, or switch the active pane into view mode.

#### attach and detach

Enter `Ctrl+b d` inside a session to detach tmux. After detaching, the session will exit and detach message will be printed.
In the bash, execute `tmux attach` to attach the most recent detached session or specify the session name by using `-t` flag: 

`tmux attach -tmysession`.

#### list all sessions
```bash
$ tmux ls
0: 2 windows (created Sun May 24 22:26:44 2020)
```

#### kill tmux entirely
When there are no session, windows, or panes inside tmux, the server will exit. To kill it manually, enter this at the command prompt:
```bash
:kill-server
```
#### create a new window
A new window can be created in an attached session with `Ctrl+b c`. It can also be done at the command prompt.
```bash
:neww -dnmynewwindow # -d does not make the new window the current window,
                     # -n to specify the window name
```

#### split the window

+ split horizontally: `Ctrl+b %`
+ split vertically: `Ctrl+b "`

#### change the current window

+ change to window #: `Ctrl+b #`

#### change the active pane

+ `Ctrl+b arrow-key` to go to the pane in the specified direction.
+ `Ctrl+b q` print the pane number first and press the number key to switch to that pane.
+ `Ctrl+b o` move to the next pane by pane number.
