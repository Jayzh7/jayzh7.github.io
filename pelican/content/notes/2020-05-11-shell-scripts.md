---
layout: post
published: true
title: Shell scripts - notes on missing lectures
date: 2020-05-11 23:13 
summary: 
category: notes
visible: False
tags: Linux
---

## Shell scripting

Shell scripting is optimized for performing shell related tasks.

String in bash can be defined with ' and " delimiters but they are not same. ' is used for literals while " can be used with variables. Variables inside " will be substituted with its value.

```bash
foo=bar
echo "$foo"
# prints bar
echo '$foo"
# prints $foo
```

Shell scripts accept arguments from users. Some special characters below. [A more comprehensive list](https://www.tldp.org/LDP/abs/html/special-chars.html)

+ `$0` - name of the script
+ `$1` to `$9` - arugments to the script.
+ `$@` - all the arguments
+ `$?` - return code of the previous command. 0 means OK, otherwise not.
+ `$$` - process id of the current script
+ `!!` - entire last command including arguments.
+ `$_` - last argument from the last command

Exit code can be used to conditionally execute commands using `&&` and `||`.

```bash
false || echo "Oops, fail"
# the second part will be executed

true || echo "will not be printed"
# the second part will not be executed

false && echo "will not be printed"

true && echo "things went well"
```

_Command substitution_: Using the output of a command as a variable can be done by `$(CMD)`. 
_Process substitution_: `<(CMD)` will execute `CMD` and place the output in a temporary file and substitute the `<(CMD)` with that file's name.

```bash
#!/bin/bash
# An example for showcasing the features

echo "Starting program at $(date)"
echo "Running program $0 with $# arguments with pid $$"

for file in $@; do
    grep foobar $file > /dev/null 2> /dev/null
    
    # when pattern is not found, grep return 1.
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

> `[[` and `[` are both used to evalute expressions. `[[` works only in Bash, Zsh, and Korn shell, and is more powerful; `[` and `test` are available in POSIX shells.  
`[[` is a keyword rather than a program like `test` and `[`. It is easier to use. Detailed explanation see [here](http://mywiki.wooledge.org/BashFAQ/031) and [here](https://stackoverflow.com/questions/669452/is-double-square-brackets-preferable-over-single-square-brackets-in-ba)

_shell globbing_ is a technique used to provide arguments of similar names to scripts.

+ Wildcards 
+ Curly braces `{}`

[Shell check](https://www.shellcheck.net/) is an online tool to check shell scripts.

### Shebang 
The shell knows how to execute the scripts by the __shebang__ line at the top of the script. A good practice is to write shebang lines using the `env` command that resolves where to find the commands like
```bash
#!/usr/bin/env bash
```


### Find files
`find` command will recursively search for files matching some criteria.

| options |          examples          |
|:-------:|:--------------------------:|
|  -name  |          file name         |
|  -iname | file name ignore case      |
|  -path  | specify file path          |
|  -type  |   d (directory) f (file)   |
|  -mtime |         modify time        |
|  -size  | +500k : >500k   -10M : < 10M ||

`find` can also perform actions over query results.

```bash
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \; 
# Convert all PNG files to JPG files
find . -name '*.png' -exec convert {} {.}.jpg \;
```

### find code
We can find code by `grep` which offers us a lot of options like

+ -C for getting context around the matching line
+ -v for inverting the match
+ -R for going into directories recursively and look for text files

Some alternatives to `grep` that add other features like multi-thread support like `ack`, `ag`, and `rg`.

### find shell commands
To find the commands you have executed a while ago. the `history` command can be of help. It prints the shell history to the standard output. Therefore, we can search for patterns using `grep`.

In most shells, it is also possible to type `Ctrl+R` to perform backwards search. After typing the command, you can enter a pattern and go backwards by typing `Ctrl+R` again.

A leading space can avoid commands from ending up in the history to protect sensitive information.