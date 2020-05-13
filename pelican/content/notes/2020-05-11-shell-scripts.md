---
title: Shell scripts - notes on missing lectures
date: 2020-05-11 23:13 
summary: 
category: notes
visible: False
tags: Linux, Missing lecture
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

Shell scripts accept arguments from users.

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

Use the output of a command as a variable which can be done by `$(CMD)`.

```bash
# iterate the files in current directory
for file in $(ls)