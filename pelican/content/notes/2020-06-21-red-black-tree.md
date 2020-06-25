---
title: red black tree
date: 2020-06-21 22:22
summary: 
category: notes
visible: True
tags: Data Structure
ext-js: https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML
---

## Properties

1. Each node is either black or red.
2. All leaves are black. (root is as well black, but less important for analysis).
3. If a node is red, its children are black.
4. Every path from a node to its descendant NIL nodes goes through the same number of black nodes. (Every perfect binary tree that consists of only black nodes are red-black tree)

The constraints enforce: path to furthest leaf is no more than twice as long as the path to the nearest leaf, i.e., h < 2 lg(n+1). Thus, the height is still O(logN).

Proof:

- When red children are combined with their black parents, the RB tree becomes a 2-4 tree (each node has 2, 3, or 4 children).
- Define the height of the 2-4 tree to be $$ h\` $$. We then have $$ 2^{h\`} \leq # nodes (N) \leq 4^{h\`}.
- $$ h\` \geq \frac{h}{2} $$

=> h < 2log(N+1).

## Insertion

Always insert a red node.

| case no. | father | uncle | type | operations |
| -------- | ------ | ----- | ---- | ---------- |
| 1        | black  | -     | -    | -          |
| 2        | red    | red   | -    | color reverse (CR) |
| 3        | red    | black | left-left | right rotate + CR |
| 4        | red    | black | right-right | left rotate + CR|
| 5        | red    | black | left-right | left rotate + right rotate + CR|
| 6        | red    | black | right-left | right rotate + left rotate + CR|

If the father is black, inserting a red node will not violate the rules. If father and uncle are both red, reverse the color would do the trick. So father and uncle will turn black and the new one will be red.

Case 3 and case 4 are symmetric. Do a left/right rotate and it will like case 2.
Case 5 and case 6 are symmetric. Do a left/right rotate and it will degrade to be like case 3 and 4.

Actual cases and code to follow.