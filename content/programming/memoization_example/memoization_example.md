---
title: __IN PROGRESS__ Memoization in Python
description: The basics of memoization and how to use it in Python.
date: 2018-01-03
categories: [ "Python" ]
tags: [
    "python",
    "programming"
]
---

Memoization is a useful tool to speed up programs that are either recursive
or simply repetitive in nature. This post is an introduction in how to
use this technique (through decorators) in Python.

<!--more-->

### Introduction to Memoization
Imagine that you own a grocery store and that customers often ask you how much
your goods cost. Imagine also that you have a terrible memory so that whenever
a customer asks about an item, say a bag of apples, you must physically walk
from your desk all the way to the fruit aisle and check the price on the sign
next to the apples. Though this example does not make much sense because
in the real world us humans have memories, this task of checking prices would
quickly take up most of your workday given a sufficient number of customer inquiries.

Imagine instead that you now have a piece of paper on which you can write
the item the customer is asking about, the quantity they are asking about, where
the items are in the store, and any other details that may be necessary for
a shopper to know. Anytime you get a *new* request that you have not heard
before, you go fetch all of those
details and record them on your piece of paper. After the first time, for
subsequent inquiries about those goods, you already have all the information
you need and can quickly glance at your piece of paper and return the answer
to the customer. You now have much more time to deal with the other
aspects of your store!

This is exactly what memoization is! Put more formally, it records (or tables)
the results of some function call (in our example the looking up of product
details) given a set of input arguments to that function (what the product is,
how many the customer wants, etc.). Whatever amount of time the function took to
compute the answer the first time it was run (walking over to the product in
the store), it is reduced to simply looking up the result in memory (glancing at
the paper on your desk).

### Instructive Example: Euler Problem #15
To show how useful memoization can be even when used in simple tasks we will show how
memoization can be used to solve a problem on [Project Euler](https://projecteuler.net/).
If you have not heard of it, it is a great place to try your hand at math and programming
problems and will definitely improve your skills no matter your level of proficiency.

The problem is stated as follows:

```{color=black}
Starting in the top left corner of a 2×2 grid, and only being able to move
to the right and down, there are exactly 6 routes to the bottom right corner.

How many such routes are there through a 20×20 grid?
```
<center>
![alt text](https://projecteuler.net/project/images/p015.png)
</center>

How might you go about solving this problem? You could think about writing
a program that transverses all the possible paths of a given grid. While this
works in theory, if computing time is not a concern, in practice a more nuanced
solution is needed.
