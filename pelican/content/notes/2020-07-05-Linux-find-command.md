---
layout: post
title:  Linux Find Command 
category: notes
date:   2020-07-05 12:14:40 +0100
tags: Linux
status: published
comments: true
---

Apart from removing or copying files directly using regular expression, `find` offers more functionalities when searching for files.

# Command Options

|option| function |
|----|----|
|`-name` and `-iname` | find by name (ignore case) |
|`-mindepth` and `-maxdepth` | limit the search depth of directories |
|`-exec` | execute commands on the files found |
|`-not` | invert the match |
|`-inum` | find by inode number |
|`-perm` | specify file permissions, like `g=r`, or `755`|
|`-empty` | find empty files |
|`-type` | specify file `f`, directory `d`, socket files `s`,  |
|`-size` | find file by size `-100M`, less than 100M, `+100M`, more than 100M |
|`-mmin -n` | find files that are modified within `n` minutes | 
|`-mtime -n` | find files that are modified within `n` days (24 hour) |
|`-amin`, `-atime` | similiar to above (note that `+`, `-`, and no sign represents more than, within, and exactly)|
|

> File change time gets updated when the inode of the file changes

# Examples

## Find the top 5 big files
```shell
find . -type f -exec ls -s {} \; | sort -n -r | head -5
```

## Find all empty files in home directory and its sub-directories.
```shell
find ~ -type f -empty -maxdepth 1
```

## Find files which has read permission only to group
```shell
find . -perm g=r -type f
find . -perm 040 -type f
```

## Use more than 1 pair of {} in the same command

### add postfix for specific files
```shell
find -name "?abc" -exec mv {} {}.bkup \;
```

## Substitute space with underscore in the file name
```shell
find . -type f -iname "*.mp3" -exec rename "s/ /_/g" {} \;
```



## References

[_Mommy, I found it! — 15 Practical Linux Find Command Examples_](https://www.thegeekstuff.com/2009/03/15-practical-linux-find-command-examples/)  
[_Daddy, I found it!, 15 Awesome Linux Find Command Examples (Part2)_](https://www.thegeekstuff.com/2009/06/15-practical-unix-linux-find-command-examples-part-2/)