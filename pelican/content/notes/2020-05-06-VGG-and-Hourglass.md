---
layout: post
published: true
title: VGG and hourglass
date: 2020-05-06 10:51
summary: Two common architectures for face alignment. What are their differences?
category: notes
visible: True
tags: ML
---

## VGGNet

___

VGG16 is a convolutional neural network model proposed by researchers from University of Oxford in the paper “Very Deep Convolutional Networks for Large-Scale Image Recognition” in 201. It was ranked 2nd in classification. 1st in localization in ILSVRC'14.

![VGG architecture](https://neurohive.io/wp-content/uploads/2018/11/vgg16-1-e1542731207177.png) <!--({static}/images/vgg16-1.png)  -->
figure 1: VGG architecture. Image credit: [Neurohive](https://neurohive.io/en/popular-networks/vgg16/)

### Characteristics

+ small filters, deeper networks at that time (jumped from 8 layer to 19).

+ only 3x3 conv stride 1, pad 1, and 2x2 max pooling stide 2.

&nbsp;&nbsp;why smaller filters?

> + stack of small filters can have the same effect as one large filter. For example, 3 layers of 3x3 conv with stride 1 and pad 1 is the same as a layer of 7x7.  
> + deeper, more non-linearities  
> + fewer parameters  

### Drawbacks

+ painfully slow to train (depth leads to more parameters)
+ high memory usage (the number of channels goes up a lot in the feedforward process)

## Hourglass

___

The Hourglass is also a convolutional network architecture originally proposed for human pose estimation by researcher from UMich in [_Stacked Hourglass Networks for Human Pose Estimation_](https://arxiv.org/pdf/1603.06937.pdf).

![Hourglass architecture]({static}/images/hourglass.png)  
figure 2: a single "hourglass" module. Each box in the figure corresponds to a residual module

### Characteristics

The hourglass has a symmetric distribution of capacity between its bottom-up processing and top-down processing which is kind of a conv-deconv and encoder-decoder architecture.

It does not use unpooling or deconv layers for top-down processing. Instead, it uses simple nearest neighbor upsampling and skip connections for top-down processing.

It does repeated bottom-up, top-down inference by stacking multiple hourglasses.

## Reference

[_VGG16 – Convolutional Network for Classification and Detection_](https://neurohive.io/en/popular-networks/vgg16/)  
[_CS231n: Convolutional Neural Networks for Visual Recognition_](https://cs231n.github.io/)