---
title: Editor-vim (notes of a MIT lecture)
date: 2020-05-09 22:30
summary: about vim
category: notes
visible: True
tags: Linux
---

## Philosophy of Vim

Vim is a modal editor: it has different modes for inserting text and manipulating text. Vim avoids the use of mouse because it's too slow; Vim even avoids using the arrow keys because it requires too much movement.

The end result is an editor that can match the speed at which you think.

## Modal editing

Vim's design is based on the idea that programmers spend time on not only writing, but also reading, navigating, and modifying. This leads to its multiple modes:

| mode        | key to enter | functions |
| ----        | ------------ | --------- |
| __normal__  | `Esc`        |for moving around a file and making edits |
| __insert__  | `i`          |for inserting text |
| __replace__ | `R`          |for replacing text |
| __visual__  | `V`          |for selecting blocks of text |
| __command-line__| `Ctrl+v` |for running a command |

## Vim's interface is a programming language

___
Key strokes are commands. And these commands __compose__. This enables efficient movement and edits.

### Movement

Movements in Vim are also called "nouns", because they refer to chunks of text.

+ basic movement: `hjkl` (left, down, up, right)
+ words: `w` (next word), `b` (beginning of word), `e` (end of word)
+ lines: `0` (beginning of line), `^` (first non-blank character), `$` (end of line)
+ screen: `H` (top of screen), `M` (middle of screen), `L` (bottom of screen)
+ scroll: `Ctrl-u` (up), `Ctrl-d` (down)
+ file: `gg` (beginning of file), `G` (end of file)
+ Line numbers: `:{number}` or `{number}G`
+ find: `f{ch}`, `t{ch}`, `F{ch}`, `T{ch}`  
find/to forward/backward {ch} on the current line  
`,` and `;` for navigating matches forward and backward
+ search: `/{regex}`, `n` or `N` for navigating matches

### Selection
Visual modes:

+ Visual
+ Visual line
+ Visual block

combine with movement to make selection

### Edits

+ append: `a`
+ insert: `i` add something
+ insert new line below/above: `o`/`O`
+ delete {motion}: `d{motion}`, e.g. `dw` delete word, `d$` delete to end of line  
`d0` delete to beginning of line, `dd` delete line
+ change(replace) {motion}: `c{motion}`, like delete
+ delete character: `x`, equal to `dl`
+ delete until end of line and insert: `C`
+ visual mode + manipulation  
select text, `d`/`x` to delete, `y` to yank, or `s` to replace (equal to `di`)
+ undo and redo: `u`, `Ctrl+r`
+ cut, yank(copy), paste: `d`, `y`, `p`
+ ...

### Counts

Combine nouns and verbs with a count, which will perform a given action a number of times.

+ `3w` move 3 words forward
+ `5j` move 5 lines down
+ `7dw` delete 7 words

### Modifiers

+ `ci(` changes the contents inside the current pair of parentheses
+ `ci[` changes the contents inside the current pair of square brackets
+ `da'` deletes a single quoted string, including the surrounding single quotes

### A lot more to learn

## Reference
[_Editors (Vim)_](https://missing.csail.mit.edu/2020/editors/)  
[_Vim cheatsheet_](https://devhints.io/vim)