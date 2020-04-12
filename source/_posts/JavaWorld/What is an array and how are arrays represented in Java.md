---
title: What is an array and how are arrays represented in Java

categories:
- JavaWorld

date: 2020-04-06 00:00:02
---
An array is a fundamental data strecture category, and a building block for more complex data structures. In this second tutorial in the data structures and algorithms series, you will learn how arrays are understood and used in Java programming. I'll start with the concept of an array and how arrays are represented in the Java language. I'll then introduce you to one-dimensional arrays and the tree ways that you can use them in your Java programs. Finally, we'll explore five algorithms used to search and sort one-dimentional arrays: Linear Search, Binary Search, Bubble Sort, Selection Sort, and Insertion Srot.

Note that this tutorial builds on Data Structures and algorithms, Part 1, which introduced the theoretical side of data structures and the algorithms associated with them. That tutorial includes an in-depth discussion of algorithms and how to use space and time complexity factors to evaluate and select the most efficient algorithm for your Java program. We'll get much more hands-on in this turorial, because I assume you've already read Part 1.

## What is an array?
An array is a sequence of elements where each element is associated with at least on index. An element is a group of memory locations that store a single data item. An index is a nonnegative integer, which is this case is used to uniquely identify an element. This relationship is simmilar to how a box number uniquely identifies a house on a given street.

The number of indexes associated with any element is the array's dimension. In this article, we'll be talking about one-dimensional arrays. The next article in this series introduces multi-dimensional arrays.

Java supports arrays. Each element ocuupies the same number of bytes, and the exact (精确) number depends on the types of the element's data item. Furthermore, all elements share the same type.

> Java arrays are not resizable
> Java Arrays hava a fixed size, You cannot change an array's size after creating it. Instead, if you needed to change an array's size, you would create another array of the desired size an copy all desired elements from the original array to the new one.

## One-dimensional arrays