---
layout: post
title: Basics of linear algebra for machine learning
date: 2020-01-28 20:47:00 +0800
category: notes
visible: False
tags: ML
ext-js: https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML
---
<!-- $$ \usepackage{amsmath} $$  -->
## Scalar, Vector, Matrices and Tensors

* **Scalars**: A single number, written in italics, lower-case $$ \alpha $$.
* **Vectors**: An array of numbers, written in lower-case and bold typeface, such as **_x_**.
* **Matrices** : 2-D array of numbers, wriiten in upper-case and bold typeface, such as **_A_**. Elements can be identifies using 
* **Tensors** : array with more than two axes. In the general case, an array of numbers arranged on a regular grid with a variable number of axes is known as a tensor.

## Transpose 
The transpose of a matrix is the mirror image of the matrix across a diagonal line.

$$ (\mathbf{A}^T)_{i,j} = \mathbf{A}_{j,i} $$

## Special matrices and vectors

**Diagonal** matrices have non-zero entries only along the main diagonal. Multiplying by a diagonal matrix is very computationally efficient:
$$ diag(\mathbf v) \dot x = v (element-wise multipy) x $$

**Eigenvalue decomposition**

**Singular value decomposition**
---
