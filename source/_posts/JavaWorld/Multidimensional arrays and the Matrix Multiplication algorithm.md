---
title: Multidimensional arrays and the Matrix Multiplication algorithm

categories:
- JavaWorld

date: 2020-04-06 00:00:03
---
Data structures and algorithm in Java, Part 2 instroduced a variety of techniques for searching and sorting one-dimensional arrays, which are the simplest arrays. In this tutorial you'll explore multidimensional arrays. I'll show you the three ways to create multidimensional arrays, then you'll learn how to use the Matrix Multiplication algorithm to multiply elemtns in a two-dimensional array. I'll also introduce ragged arrays and you'll learn why they're popular for big data applications. Finally, we'll consider the question of whether an array is or is not a Java object.

This article sets you up for Part 4, which introduces searching and sorting with singly-linked lists.

## Mutidimentsional arrays
A multidimensional array associates each element in the array with multiple indexes. The most commonly used multidimensional array is the two-dimensional array, also known as a table or matrix. A two-dimensional array associates each of its elements with two indexes.

We can conceptualize a two-dimensional array as a rectanglar grid of elements divided into rows and colums. We use the (row, column) natation to identify an element, as shown in Figure 1.

![Figure 1. A conceptual view of a two-dimensional array reveals a grid of elements](001.jpg)

Because two-dimensional arrays are so commonly used, I'll focus on them. What you learn about two-dimensional arrays can be generalized to higher-dimensional ones.

## Creating two-dimensional arrays
There are three techniques for creating a two-dimensional array in Java:
- Using an initializer
- Using the keyword new
- Using the keyword new with an initializer

### Using an initializer to create a two-dimensional array
The initializer-only approach to creating a two-dimensional array has the following syntax:
```
'{' [rowInitializer (',' rowInitializer)*] '}'
```

rowinitializer has the following syntax:
```
'{' [expr (',' expr)*] '}'
```

This syntax states that a two-dimensional array is an optional, comma-separated list of row initializer appearing between open - and close - brace characters. Futhermore, each row initializer is an optional, comma-separated list of expressions appearing between oopen - and close - brace characters. Like one-dimensional ararys, all expressions must evaluate to compatible types.

Here's an example of a two-dimensional array:
```java
{ { 20.5, 30.6, 28.3 }, { -38.7, -18.3, -16.2 } }
```

This example creates a table with rows and three columns. Figure 2 presents a concetual (概念上的) view of this table along with a memory view that shows how Java lays out this (and every) table in memory.

![Figure 2. Conceptual and memory views of a two-dimensional array](002.jpg)

Figure 2 reveals that Java represents a two-dimensional array as a one-dimensional row array whose elements reference one-dimensional column arrays. The row index identifies the column array; the column index identifies the data item.

### Keyword new-only creation
The keyword new allocates memory for a two-dimensional array and returns its reference. This approach has the following syntax:
```
'new' type '[' int_expr1 ']' '['int_expr2 ']'
```

The syntax states that a two-dimensional array is a region of (positive) int_expr1 row elements and (positive) int_expr2 column elements that all share the same type. Furthermore, all elements are zeroed. Here's an example:
```java
new double[2][3] // Create a two-row-by-three-column table.
```

### Keyword new and initializer creation
The keyword new with an initializer approach has the following syntax:
```
'new' type '[' ']' [' ']' '{' [rowInitializer (',' rowInitializer)*] '}'
```

Where rowInitializer has the following syntax:
```java
'{' [expr (',' expr)*] '}'
```

The syntax blends the previous two examples. Because the number of elements can be determined from the comma-separated lists of expressions, you don't provide an int_expr between either pair of square brackets. Here is an exmaple:
```java
new double [][] { { 20.5, 30.6, 28.3 }, { -38.7, -18.3, -16.2 } }
```

## Two-dimensinal arrays and array variable
By itself, a newly-created two-dimensional array is useless. Its reference must be assigned to an array variable of a compatible type, either directly or via a method call. The follwoing syntaxes show how you would declare this variable:
```
type var_name '[' ']' '[' ']'
type '[' ']' '[' ']' var_name
```

Each syntax declares an array variable taht stores a reference to a two-dimensional array. It's perferred to palce the square brackets after type. Consider the follwoing examples:
```
double[][] temperatures1 = { { 20.5, 30.6, 28.3 }, { -38.7, -18.3, -16.2 } };
double[][] temperatures2 = new double[2][3];
double[][] temperatures3 = new double[][] { { 20.5, 30.6, 28.3 }, { -38.7, -18.3, -16.2 } };
```

Like one-dimensional array variables, a two-dimensional array variable is associated with a `.length`  property, which reutrns the length of the row array. For example, `temperatures1.length` returns 2. Each row element is also an array variable with a `.length` property, which returns the number of columns for the column array assigned to the row element. For example, `temperatures1[0].length` returns 3.

Given an array variable, you can access any element in a two-dimensional array by specifying an expression that agress with the following syntax:
```
array_var '[' row_index ']' '[' col_index ']'
```

Both indexes are positive ints that range from 0 to one less than the value returned from the respective `.length` properties. Consider the next two examples:
```java
double temp = temperatures1[0][1]; // Get value.
temperatures1[0][1] = 75.0;        // Set value.
```

The first example returns the value in the second column of the first row (30.6). The second example  repalces this value with 75.0.

If you spcify a negative idnex or an index that is greater than or equal to the value returned by the array variable's `.length` property, Java creates and throws an ArrayIndexOutOfBoundsException object.

## Multiplying two-dimensional arrays
Multiplying one matrix by another matrix is a comon operation in fields ranging from computer graphics, to economics, to the transportation industry. Developer usually use the Matrix Multiplication algorithm for this operation.

How does matrix mutiplication work? Let A represent a matrix with $m$ rows and $p$ columns. Similarly, let B represent a matrix with $p$ rows and $n$ columns. Multiply A by B to produce a maxtrix C, with $m$ rows and $n$ columns. Each cij entry in C is obtained by mutiplying all entries in A's ith row by corresponding entries in B's jth column, then adding the results. Figure 3 illustrates these operations.

![Figure 3. Each of A's rows if mutiplied (and andded with each of B's columns to produce an entry in C)](003.jpg)

> Left-matrix columns must equal right-matrix rows
> Matrix multiplcation requries that the number of columns (p) in the left matrix (A) equal the number of rows (p) in the right matrix (B). Otherwise, this algorithm won't work.

The following pseudocode expresses Matrix Multiplication in a 2-row-by-2-column and a 2-row-by-1-column table context. (Recall that I introduced pseudocode in Part 1.)

```
// ==      ==   == ==   ==                     ==
// | 10  30 |   | 5 |   | 10 x 5 + 30 x 7 (260) |
// |        | X |   | = |                       | 
// | 20  40 |   | 7 |   | 20 x 5 + 40 * 7 (380) | 
// ==      ==   == ==   ==                     ==

DECLARE INTEGER a[][] = [ 10, 30 ] [ 20, 40 ]
DECLARE INTEGER b[][] =  [ 5, 7 ]
DECLARE INTEGER m = 2 // Number of rows in left matrix (a)
DECLARE INTEGER p = 2 // Number of columns in left matrix (a)
                      // Number of rows in right matrix (b)
DECLARE INTEGER n = 1 // Number of columns in right matrix (b)
DECLARE INTEGER c[m][n] // c holds 2 rows by 1 columns
                        // All elements initialize to 0
FOR i = 0 TO m - 1
   FOR j = 0 TO n - 1
      FOR k = 0 TO p - 1
         c[i][j] = c[i][j] + a[i][k] * b[k][j]
      NEXT k
   NEXT j
NEXT i
END
```