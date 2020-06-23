---
title: red black tree
date: 2020-06-21 22:22
summary: 
category: notes
visible: False
tags: Data Structure
---

## Properties

1. Each node is either black or red.
2. All leaves are black. (root is as well black, but less important for analysis).
3. If a node is red, its children are black.
4. Every path from a node to its descendant NIL nodes goes through the same number of black nodes. (Every perfect binary tree that consists of only black nodes are red-black tree)

The constraints enforce: path to furthest leaf is no more than twice as long as the path to the nearest leaf, i.e., h < 2 lg(n+1).
