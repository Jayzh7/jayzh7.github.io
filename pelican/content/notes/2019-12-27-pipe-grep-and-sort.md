---
layout: post
title:  The use of cat in combination with other linux commands
category: notes
date:   2019-12-27 21:26:40 +0100
tags: Linux
status: published
comments: true
---

## Intro

When searching for something in a file, we may want to look into a specific location of a file. The most straightforward way for me to handle this would be `vim` it and then `pg down`. It becomes less realistic when the file size grows larger.

Linux provides us with several commands (editor) to suit different needs. For instance,  `cat`, `vim`, `grep`, etc.

## cat
`cat` is short for "**concatenate**". It allows us to create one or more files, view contents, concatenate files, and redirect output to terminals or files.

### view contents
`cat` will direct all contents of a file to std output (therefore can be combined with pipe to see only part of it).
```bash
$ cat /etc/os-release 
NAME="Scientific Linux"
VERSION="7.6 (Nitrogen)"
...

$ cat file1 file2
contents of file1
contents of file2
```

### create a new file
Input from terminal can also be redirected into a file with `cat`. Finish input by `ctrl+d`.
```bash
$ cat > text
input here
```

### redirect to other commands
If the contents of a file is too much to fit into the terminal and overwhelms scrolling, we can redirect the output to other commands (e.g. `less`, `more`) for better viewing.


#### less and more

`less` can be used for openning a file for interactive reading, scrolling (both forward and backward), and searching.

##### go forward, go backward
`space` or `enter`, `b`
##### forward search, backward search  
`/word`, `?word`  
##### next match, previous match  
`n`, `N`  
##### display text from line 10  
`more +10 text.txt`  
##### prompt you to continue reading by space  
`more -d text.txt`  

#### grep
`grep` can be used for searching a pattern in a document and presenting the result in a customized format. Regular expression is supported.

| Option |                          Function                         |
|--------|:---------------------------------------------------------:|
| -v     | Shows all the lines that do not match the searched string |
| -c     | Displays only the count of matching lines                 |
| -n     | Shows the matching line and its number                    |
| -i     | Match both (upper and lower) case                         |
| -l     | Shows just the name of the file with the string           |

#### sort 
As the name suggests, it sorts the contents of a file alphabetically.

| Option |                     Function                    |
|--------|:-----------------------------------------------:|
| -r     | Reverses  sorting                               |
| -n     | Sorts numerically                               |
| -f     | Case insensitive sorting                        |

### useful options of cat

``` bash
$ cat -n /etc/passwd  # display line numbers in file
$ cat -e test  # display $ at the end of line
    hello everyone, how do you do?$
    $
    Hey, I am fine.$
    How is  your training going on?$
    $
```

## Use std output with redirection operators
Note the use of one and two redirection operators.
```bash
$ cat test > test1  # contents in test will overwrite contents in test1 
$ cat test >> test1 # contents in test will be appended into test1
``` 


## examples

``` bash
$ cat -n /etc/os-release | less +10  
    10  BUG_REPORT_URL="mailto:scientific-linux-devel@listserv.fnal.gov"
    11  
    12  REDHAT_BUGZILLA_PRODUCT="Scientific Linux 7"
    13  REDHAT_BUGZILLA_PRODUCT_VERSION=7.6
    14  REDHAT_SUPPORT_PRODUCT="Scientific Linux"
    15  REDHAT_SUPPORT_PRODUCT_VERSION="7.6"
$ cat > text
dbac  
cba
ba
a(ctrl+d to end input)
$ cat text | grep a
```
db<span style="color:red">a</span>c  
cb<span style="color:red">a</span>  
b<span style="color:red">a</span>  
<span style="color:red">a</span>  
``` bash
$ cat text | sort | grep a
```
<span style="color:red">a</span>  
b<span style="color:red">a</span>  
cb<span style="color:red">a</span>  
db<span style="color:red">a</span>c  


## Summary
`cat` is powerful especially when combined with other commands as well as pipe.

## References

[_Pipe, Grep and Sort Command in Linux/Unix with Examples_](https://www.guru99.com/linux-pipe-grep.html)  
[_The Difference Between more, less And most Commands_](https://www.ostechnix.com/the-difference-between-more-less-and-most-commands/)  
[_13 Basic Cat Command Examples in Linux_](https://www.tecmint.com/13-basic-cat-command-examples-in-linux/)