---
title: What is an array and how are arrays represented in Java

tag:
- Data structures and algorithms in Java

categories:
- JavaWorld

mathjax: true


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
The simplest kind of array has one dimension. A one-dimensional array associates each element with one index. One-dimensional arrays are used to store lists of data items. There are three techniques for creating one-dimensinal arrays in Java:
- Use only an initializer
- Use only keyword new
- Use keyword new with an initiazlier

### Creating a one-dimensinal array with only an initializer
Here's the syntax to create a one-dimensional array using just an initializer:
```java
{ 'J', 'a', 'v', 'a' }
```

The syntax that one-dimensional array is an optional, comma-separated (分开的) list of expressions appearing between open and close brace characters. Futhermore, all expressions must evaluate to compatible types. For example, in a two-element one-dimensional array of doubles, both elements might be of type double, or one element might be a double while the other element is a float or an integer type (such as int).

### Creating a one-dimensional array with the keyword new
The keyword new allocates memory for an array and returns its reference. Here's the syntax for this approach:
```java
new char[4]
```

The syntax states that a one-dimensional array is a region of (positive) int_exper elements that share the same type. furthermore, all elements are zeroed, and are interpreted as 0, 0L, 0F, 0.0, false, null.

### Creating a one-dimensional array with the new keyword and an initializer
Here's the syntax to create a one-dimensional array using the keyword new with an initializer. As you seee, it blends the syntax from the previous two approaches:
```java
new char[] { 'J', 'a', 'v', 'a' }
```

In this case, because the number of elements can be determined from the comma-separated list of expressions, it isn't necessary (or allowed) to provide an int_expr between the square brackets.

Something to note is that the syntax fro creating an array with only an initializer is no different in effect from the syntax using an initializer an a keyword. The initializer-only syntax is an example of syntactic sugar, which means syntax that make the language sweeter, or easier, to use.

## Array variables
By itself, a newly-created one-dimensional array is useless. Its reference must be assigned to an array variable of a compatibe type, either directly or via a method call. The following two lines of syntax how you would declare this variable.

Each syntax declares an array variable that stores a reference to a one-dimensional array. Although you can use either syntax, placing the square brackets after type is preferred (推荐).

Examples:
```java
char[] name1 = { 'J', 'a', 'v', 'a' };
char[] name2 = new char[4];
char[] name3 = new char[] { 'J', 'a', 'v', 'a' };
output(new char[] { 2, 3 }); // output({ 2, 3 }); results in a compiler error
static void output(char[] name)
{
   // ...
}
```

In the examples, name1, name2, name3 and name are array variables. The single pair of square brackets states that each stores references to one-dimensional arrays.

Keyword char indicates that each element must store a value of char type. However, you can specify a non-char value if Java can convert it to char. For example, `char[] chars = {'A', 10}` is legal because 10 is a samll enough positive in (meaning that it fits into the char range of 0 through 65535) to be converted to a char. In constact, `char[] chars = {'A', 80000}` would be illegal.

An array variable is associated with a `.length` property that returns the length of the associated one-dimensional array as a positive int; for example, `name1.length` return 4.

Given an array variable, you can access any element in a one-dimensional array by specifying an expression that argees with the following syntax:
```java
array_var '[' index ']'
```

Here, index is a positive int that ranges from 0 (Java arrays are zero-based) to one less than the vlaue returned from the `.length` property.

Example:
```java
char ch = names[0]; // Get value.
names[1] = 'A';     // Set value.
```

If you specify a negative index or an index that is greater than or equal to the value returned by the array variable's `.length` property, Java creates and throws a ArrayIndexOutOfBoundsException object.

## Algorithms for searching and sorting
It is a very common task to search one-dimensional arrays for specify data items, and there are a variety of algorithms for doing it. One of the most popular search algorithms is called Linear Search. Another option is Binary Search, which is usually more performant but also more demanding: in order to use Binary Search, the array's data items must first be sorted, or ordered. Although not very performant, Bubble Sort, Selction Sort, and Insertion Sort are all simple algorithms for sorting a one-dimensional array. Each works well enough for shorter arrays.

> Space complexity
> Each of the algorithms discussed in this section - Linear Search, Binary Search, Bubble Sort, Selection Sort, and Insertion Sort -- exhibits a $O(1)$ (constant) space complexity for variable storage.

## The Linear Search algorithm
Linear Search searches a one--dimensional array of $n$ data items for a specific one. It functions by comparing data items from the lowest index to the highest until it finds the specified data item, or until there are no more data items to compare.

The following pseudocode expresses Linear Search used for a one-dimensional array of integers:
```
DECLARE INTEGER i, srch = ...
DECLARE INTEGER x[] = [ ... ]
FOR i = 0 TO LENGTH(x) - 1
   IF x[i] EQ srch THEN
      PRINT "Found ", srch
      END
   END IF
NEXT i
PRINT "Not found", srch
END
```

Consider a one-dimensional unordered array of five integers [1, 4, 3, 2, 6], where integer 1 is located at index 0 and integer 6 is located at index 4. The pseudocode preforms the following tasks to find integer 3 in this array:
1. Compare the integer at index 0 (1) with 3.
1. Because there's no match, compare the integer at index 1 (4) with 3.
1. Because there's still no match, compare the integer at index 2 (3) with 3.
1. Because there's a match, print Found 3 and exit.

Liear Search has a time complexity of $O(n)$, which is pronounced Big Oh of $n$. For $n$ data items, this algorithm requires a maximum of $n$ comparisons. On average, it performs $\frac n2$ comparisons. Linear Search offers linear performance.

> Efficiency
> A downside of (负面) the Linear Search algorithm is that it is inefficient (效率低的). For an array of 4,000,000 data items, it would perform an average of 2,000,000 comparisons to find the specified item.

## Explore Linear Search
To let you experiment with Linear Search, I've created the LinearSearch Java application in Listing 1.

### Listing 1. A Java example with the Linear Search algorithm
```java
{
   public static void main(String[] args)
   {
      // Validate command line arguments count.

      if (args.length != 2)
      {
         System.err.println("usage: java LinearSearch integers integer");
         return;
      }

      // Read integers from first command-line argument. Return if integers 
      // could not be read.

      int[] ints = readIntegers(args[0]);
      if (ints == null)
         return;

      // Read search integer; NumberFormatException is thrown if the integer
      // isn't valid.

      int srchint = Integer.parseInt(args[1]);

      // Perform the search and output the result.

      System.out.println(srchint + (search(ints, srchint) ? " found"
                                                          : " not found"));
   }

   private static int[] readIntegers(String s)
   {
      String[] tokens = s.split(",");
      int[] integers = new int[tokens.length];
      for (int i = 0; i < tokens.length; i++)
         integers[i] = Integer.parseInt(tokens[i]);
      return integers;
   }

   private static boolean search(int[] x, int srchint)
   {
      for (int i = 0; i < x.length; i++)
         if (srchint == x[i])
            return true;

      return false;
   }
}
```

The LinearSearch application reads a comma-separated list of integers from its first command-line argument. It searchs the array for the integer identified by the second command-line argument, and outputs a found/not found message.

> Beware of the number format exception
> Specify digits and +/- sign characters only in each command-line argument. Otherwise, this application (and the subsequent search and sort applicaitons) will create and throw a NumberFormatException object.

To experiment with this application, start by compiling Listing 1:
```bash
javac LinearSearch.java
```

Next, run the resulting application as follows:
```bash
java LinearSearch "4,5,8" 5
```

You should observe the following output:
```
5 found
```

Run the resulting application a second time, as follows:
```bash
java LinearSearch "4,5,8" 15
```

You should observe the following output:
```
15 not found
```

## The Binary Search algorithm
The Binary Search algorithm searches an ordered one-dimensional array of $n$ data items for a specific data item. This algorithm consists of the following steps:   
1. Set low and high index variables to the indexes of the array's first and last data items, respectively (分别)
1. Terminate if the low index is greater than the high index. The serached-for data item is not in the array
1. Calculate the middle index by summing the low and high indexes and dividing the sum by 2
1. Compare the searched-for data item with the middle-indexed data item. Terminate if they are the same. The searched-for data item has been foud
1. If the searched-for data item is greater than the middle-indexed data item, set the low index to the middle index plus one and transfer execution to Step 2. Binary Search repeats the search in the upper half of the array
1. The searched-for data item must be smaller than the middle-indexed data item, so set the high index to the middle index minus on and transfer execution to Step 2. Binary Search repeats the earch in the lower half of the array

Here is pseudocode representing the Binary Search algorithm for a one-dimensional array of integers:
```
DECLARE INTEGER x[] = [ ... ]
DECLARE INTEGER loIndex = 0
DECLARE INTEGER hiIndex = LENGTH(x) - 1
DECLARE INTEGER midIndex, srch = ...
WHILE loIndex LE hiIndex
   midIndex = (loIndex + hiIndex) / 2
   IF srch GT x[midIndex] THEN
      loIndex = midIndex + 1
   ELSE
   IF srch LT x[midIndex] THEN
      hiIndex = midIndex - 1
   ELSE
      EXIT WHILE
   END IF
END WHILE
IF loIndex GT hiIndex THEN
   PRINT srch, " not found"
ELSE
   PRINT srch, " found"
END IF
END
```

Binary Search isn't hard to understand. For example, consider a one-dimensional ordered array of six integers [3, 4, 5, 6, 7, 8], where integer 3 is located at idnex 0 and integer 8 is located at index 5. The pseudocode does the following to find integer 6 in this array:
1. Obtain the low index (0) and high index (5)
1. Calculate the middle index: (0 + 5) / 2 = 2
1. Because the integer at index 2 (5) is less than 6, set the low index to 2 + 1 = 3
1. Calculate the middle index: (3 + 5) / 2 = 4
1. Because the integer at idnex 4 (7) is greater than 6, set the high idnex to 4 - 1 = 3
1. Calculate the middle index: (3 + 3) / 2 = 3
1. Becuase the integer at index 3 (6) equals 6, print 3 found and exit

Binary Search has a time complexity of $O(log_2n)$, which is pronounced Big Oh of log $n$ to the base 2. For $n$ data items, Binary Search required a maximum of $1 + log_2n$ comparisons, making this algorithm vastly more efficient than Linear Serach, in most cases. The algorithm offers logarithmic performance (for more about logarithmic performance, see Figure 3 int Part 1 of this series).

> When Linear Search outperforms Bineary Search
> Although Binary Search is typically more efficient than Linear Search, Binary Search isn't as efficient for short arrays. This was discovered by famous computer scientist Donald Knuth.

Listing 2 is a BinarySearch Java application that lets you experiment with Binary Search.

### Listing 2. A Java example with the Binary Search algorithm
```java
{
   public static void main(String[] args)
   {
      // Validate command line arguments count.

      if (args.length != 2)
      {
         System.err.println("usage: java BinarySearch integers integer");
         return;
      }

      // Read integers from first command-line argument. Return if integers 
      // could not be read.

      int[] ints = readIntegers(args[0]);
      if (ints == null)
         return;

      // Read search integer; NumberFormatException is thrown if the integer
      // isn't valid.

      int srchint = Integer.parseInt(args[1]);

      // Perform the search and output the result.

      System.out.println(srchint + (search(ints, srchint) ? " found"
                                                          : " not found"));
   }

   private static int[] readIntegers(String s)
   {
      String[] tokens = s.split(",");
      int[] integers = new int[tokens.length];
      for (int i = 0; i < tokens.length; i++)
         integers[i] = Integer.parseInt(tokens[i]);
      return integers;
   }

   private static boolean search(int[] x, int srchint)
   {
     int hiIndex = x.length - 1, loIndex = 0, midIndex;

      while (loIndex <= hiIndex)
      {
         midIndex = (loIndex + hiIndex) / 2;
         if (srchint > x[midIndex])
            loIndex = midIndex + 1;
         else
         if (srchint < x[midIndex])
            hiIndex = midIndex - 1;
         else
            return true;
      }

      return false;
   }
}
```

The BinarySearch application reads a comma-separated list of integers from its first command-line argument. It searchs the array for the integer identified by the second command-line argument, and outputs a found/not found message.

> A bug in Binary Search
> Joshua Bloch (author of Effictive Java) discovered a bug in the Binary Search algorithm, which can lead to a thrown instance of the ArrayIndexOutOfBoundsException class in Java. This bug manifests itself for arrays whose lengths are $2^{30}$ (roughly one billion) or greater.

Compile the code in Listing 2 as follows:
```bash
javac BinarySearch.java
```

Run the resulting application as follows:
```bash
java BinarySearch "4,5,8" 5
```

You should observe the following output:
```
15 not found
```

## The Bubble Sort algorithm
The Bubble Sort algorithm orders a one-dimensional array of $n$ data items into ascending or descending order. An outer loop makes $n-1$ passes over the array. Each pass uses an inner loop to exchange data items such that the next smallest (ascending) or largest (descending) data item "bubbles" towrads the beginning of the array.

The "Bubbling" action occurs in the inner loop, where each iteration compares the pass-numbered data item with each successive data item. If a successor data item is smaller (ascending sort) or larger (descending sort) than the pass-numbered data item, the successor data item is exchanged with the pass-numbered data item. 

Here is pseudocode representing  Bubble Sort in a one-dimensional array of integers/ascending sort context:
```
DECLARE INTEGER i, pass
DECLARE INTEGER x[] = [ ... ]
FOR pass = 0 TO LENGTH(x) - 2
   FOR i = LENGTH(x) - 1 DOWNTO pass + 1
      IF x[i] LT x[pass] THEN // switch to > for descending sort
         EXCHANGE x[i], x[pass]
      END IF
   NEXT i
NEXT pass
END
```

Bubble Sort is faily easy to understand. For example, consider a one-dimensional, unordered array of four integers: [18 16, 90, -3], where integer 18 is located at index 0 and integer -3 is located at index 3. When requested to sort this array into ascending order, Bubble Sort would execute as follow:
```
Pass 0               Pass 1               Pass 2
======               ======               ======
18  16  90  -3       -3  16  90  18       -3  16  90  18
^           ^            ^       ^                ^   ^
|           |            |       |                |   |
-------------            ---------                -----
-3  16  90  18       -3  16  90  18       -3  16  18  90
^       ^                ^   ^ 
|       |                |   |
---------                -----
-3  16  90  18       -3  16  90  18
^   ^
|   |
-----
-3  16  90  18
```

In terms of comparisons and also in terms of exchanges, Bubble Sort has a time complexity of $O(n^2)$, which is pronounced "Big Oh of $n$ squared". Bubble Sort offers quadratic performance, which isn't a problem for shorter-length arrays -- espicially when you consider that Bubble Sort is easy to code. (See Part 1 for more about quadratic performance.)

The BubbleSort Java application in Listing lets you experiment with Bubble Sort.

### Listing 3. A Java example with the Bubble Sort algorithm
```java
{
   public static void main(String[] args) {
      // Validate command line arguments count.
      if (args.length != 1) {
         System.err.println("usage: java BubbleSort integers");
         return;
      }

      // Read integers from first command-line argument. Return if integers could not be read.
      int[] ints = readInteger(args[0]);
      if (ints == null) {
         return;
      }

      // Output integer array's length and number of inversions statistics to standard outpou device.
      System.out.println("N = " + ints.length);
      int inversions = 0;
      for (int i = 0; i < ints.length - 1; i++) {
         for (int j = i + 1; j < ints.length; j++) {
            if (int[i] > int[j]) {
               inversions ++;
            }
         }
      } 
      System.out.println("I = " + inversions);

      // Output unsorted integer values to standard output, sort the array, and output sorted values to standard output.
      dum(ints);
      sort(ints);
      dump(intns);
   }

   private static int[] readIntegers(String s) {
      String[] tokens = s.split(",");
      int[] integers = new int[tokens.length];
      for (int i = 0; i < tokens.length; i++)
         integers[i] = Integer.parseInt(tokens[i]);
      return integers;
    }

   private static void dump(int[] a) {
      for (int i = 0; i < a.length; i++)
         System.out.print(a[i] + " ");
      System.out.print('\n');
   }

   private static void sort(int[] x) {
         for (int pass = 0; pass < x.length - 1; pass++) {
            for (int i = x.length -1; i > pass; i++) {
               if (x[i] < x[pass]>) {
                  int temp = x[i];
                  x[i] = x[pass];
                  x[pass] = temp;
               }
            }
         }
   }
}
```

BubbleSort reads a comma-separated list of integers from its command-line argument. It outputs the array length, calculates and outputs the number of inversions (larger items to the left of smaller items in the unsorted array), outputs the unsorted array, sorts the array, and outputs the sorted array. (Selection Sort and Insertion Sort, which I'll introduce next, behave similary.)

Compile Listing 3 as follows:
```bash
javac BubbleSort.java
```

Run the resulting application as follows:
```bash
java BubbleSort "18,16,90,-3"
```

You should observe the following output:
```
N = 4
I = 4
18 16 90 -3
-3 16 18 90
```

## The Selection Sort algorithm
The Selection Sort algorithm orders a one-dimensional array of $n$ data items into ascending order or descending order. An outer loop makes $n - 1$ passes over the array. Each pass uses an inner loop to find the next smallest (ascending sort) or lastest (descending sort) data item, which is exchanged with the pass-numbered data item.

Selection Sort assumes that the data item at the pass-numbered index is the smallest (ascending sort) or the largest (descending sort) of the remaining data items. It searches the rest of the array for a data item that's smaller/larger than this data item, and performs an exchange at the end of the search when a smaller/larger data item is found.

The following pseudocode expresses Selection Sort in a one-dimensinal array of integers/ascending sort context:
```
DECLARE INTEGER i, min, pass
DECLARE INTEGER x[] = [ ... ]
FOR pass = 0 TO LENGTH(x) - 2
   min = pass
   FOR i = pass + 1 TO LENGTH(x) - 1
      IF x[i] LT x[min] THEN
         min = i
      END IF
   NEXT i
   IF min NE pass THEN
      EXCHANGE x[min], x[pass]
   END IF
NEXT pass
END
```

Selection Sort is faily easy to understand. For example, consider a one-dimensional unordered array of four integers: [18, 16, 90, -3], where integer 18 is located at index 0  and integer -3 is located at index 3. When requested to sort this array into  ascending order, the Selection Sort pseudocode performs the sort as follows:
```
Pass 0                        Pass 1                        Pass 2
======                        ======                        ======
18  16  90  -3                -3  16  90  18                -3  16  90  18
^                                 ^                                 ^
|                                 |                                 |
min = 0                           min = 1                           min = 2

18  16  90  -3                -3  16  90  18                -3  16  90  18
    ^                                 ^                                 ^
    |                                 |                                 |
    16 < 18, min = 1                  90 > 16, min = 1                  18 < 90, min = 3
                                                                    ^   ^
18  16  90  -3                -3  16  90  18                        |   |
        ^                                 ^                         -----
        |                                 |                 -3  16  18  90
        90 > 16, min = 1                  18 > 16, min = 1

18  16  90  -3                               
            ^   
            |
            -3 < 16 min = 3
^           ^
|           |
------------- 
-3  16  90  18
```

Selection Sort has a time complexity of $O(n^2)$ comparisons and $O(n)$ exchanges. The algorithm offers quadratic performance in terms of comparisons and linear performance in terms of exchanges, which makes it somewhat more efficient than Bubble Sort.

Listing 4 shows the SelectionSort application in Java code.

### Listing 4. A Java example with the Selection Sort algorithm
```
public final class SelectionSort
{
   public static void main(String[] args)
   {
      // Validate command line arguments count.

      if (args.length != 1)
      {
         System.err.println("usage: java SelectionSort integers");
         return;
      }

      // Read integers from first command-line argument. Return if integers 
      // could not be read.

      int[] ints = readIntegers(args[0]);
      if (ints == null)
         return;

      // Output integer array's length and number of inversions statistics to
      // standard output device.

      System.out.println("N = " + ints.length);
      int inversions = 0;
      for (int i = 0; i < ints.length - 1; i++)
         for (int j = i + 1; j < ints.length; j++)
            if (ints[i] > ints[j])
               inversions++;
      System.out.println("I = " + inversions);

      // Output unsorted integer values to standard output, sort the array, 
      // and output sorted values to standard output.

      dump(ints);
      sort(ints);
      dump(ints);
   }

   static void dump(int[] a)
   {
      for (int i = 0; i < a.length; i++)
         System.out.print(a[i] + " ");
      System.out.print('\n');
   }

   static int[] readIntegers(String s)
   {
      String[] tokens = s.split(",");
      int[] integers = new int[tokens.length];
      for (int i = 0; i < tokens.length; i++)
         integers[i] = Integer.parseInt(tokens[i]);
      return integers;
    }

   static void sort(int[] x)
   {
      for (int pass = 0; pass < x.length - 1; pass++)
      {
         int min = pass;

         for (int i = pass + 1; i < x.length; i++)
            if (x[i] < x[min])
               min = i;

         if (min != pass)
         {
            int temp = x[min];
            x[min] = x[pass];
            x[pass] = temp;
         }
      }
   }
}
```

Compile Listing 4 as follows:
```bash
javac SelectionSort.java
```

Run the resulting application as follows:
```bash
java SelectionSort "18,16,90,-3"
```

You should oberve the following output:
```
N = 4
I = 4
18 16 90 -3
-3 16 18 90
```

## The Insertion Sort algorithm
The Insertion Sort algorithm orders a one-dimensional array of $n$ daata items into ascending order or descending order. An outer loop makes $n - 1$ passes over the array. Each pass selects the next data item to be inserted into the appropriate position. It uses an inner loop to find this position, shifting data items to make room.

Insertion Sort begins by dividing the data structure into sorted and unsorted sections. Initially, the sorted section contains the data item at index 0; hte unsorted section contains all other data ites. During the sort, each unsorted section data item is inserted into the proper position in the sorted section and the unsorted section shrinks (收缩) by on data item.

Here is pseudocode for the Insertion Sort algorithm in a one-dimensinal array of integers, where you are doing an ascending sort:
```
DECLARE INTEGER a, i, j
DECLARE INTEGER x[] = [ ... ]
FOR i = 1 TO LENGTH(x) - 1
   a = x[i]
   j = i
   WHILE j GT 0 AND x[j - 1] GT a
      x[j] = x[j - 1]
      j = j - 1
   END WHILE
   x[j] = a
NEXT i
END
```

Like Bubble Sort, Insertion Sort is fairly easy to understand. For example, consider a one-dimensional unordered array of four integers: [18, 16, 90, -3], where integer 18 is located at index 0 and integer -3 is located at index 3. When instructed to sort this array into ascending order, the algorithm performs the sort as follows:
```
i = 1                 i = 2                 i = 3
      =====                 =====                 =====
18 | 16   90   -3     16   18 | 90   -3     16   18   90 | -3     -3   16   18   90
     ^                          ^                          ^
     |                          |                          | 
     a,j                        a,j                        a,j
```

The sorted section appears on the left and initially consists of [18]. The unsorted section apprears on the right and initially consists of [16, 90, -3].

Insertion Sort has a time complexity of $O(n)$ comparisons for the bast case (data is already sorted or nearly sorted) and $O(n^2)$ for the average and worst cases. The algorithm offers linear (best case) or quadratic (average/worst case) performance.

Listing 5 shows the source code for the InsertionSort application.

### Listing 5. A Java example with the Insertion Sort algorithm
```java
{
   public static void main(String[] args)
   {
      // Validate command line arguments count.

      if (args.length != 1)
      {
         System.err.println("usage: java InsertionSort integers");
         return;
      }

      // Read integers from first command-line argument. Return if integers 
      // could not be read.

      int[] ints = readIntegers(args[0]);
      if (ints == null)
         return;

      // Output integer array's length and number of inversions statistics to
      // standard output device.

      System.out.println("N = " + ints.length);
      int inversions = 0;
      for (int i = 0; i < ints.length - 1; i++)
         for (int j = i + 1; j < ints.length; j++)
            if (ints[i] > ints[j])
               inversions++;
      System.out.println("I = " + inversions);

      // Output unsorted integer values to standard output, sort the array, 
      // and output sorted values to standard output.

      dump(ints);
      sort(ints);
      dump(ints);
   }

   static void dump(int[] a)
   {
      for (int i = 0; i < a.length; i++)
         System.out.print(a[i] + " ");
      System.out.print('\n');
   }

   static int[] readIntegers(String s)
   {
      String[] tokens = s.split(",");
      int[] integers = new int[tokens.length];
      for (int i = 0; i < tokens.length; i++)
         integers[i] = Integer.parseInt(tokens[i]);
      return integers;
    }

   static void sort(int[] x)
   {
      int j, a;

      // For all integer values except the leftmost value ...

      for (int i = 1; i < x.length; i++)
      {
         // Get integer value a.

         a = x[i];

         // Get index of a. This is the initial insert position, which is
         // used if a is larger than all values in the sorted section.

         j = i;

         // While values exist to the left of a's insert position and the
         // value immediately to the left of that insert position is
         // numerically greater than a's value ...

         while (j > 0 && x[j - 1] > a)
         {
            // Shift left value -- x[j - 1] -- one position to its right --
            // x[j].

            x[j] = x[j - 1];

            // Update insert position to shifted value's original position
            // (one position to the left).

            j--;
         }

         // Insert a at insert position (which is either the initial insert
         // position or the final insert position), where a is greater than
         // or equal to all values to its left.

         x[j] = a;
      }
   }
}
```

Compile Listing 5 as follows:
```bash
javac InsertionSort.java
```

Run the resulting appilcaiton as follows:
```bash
java InsertionSort "18,16,90,-3"
```

You should observe the following output:
```
N = 4
I = 4
18 16 90 -3
-3 16 18 90
```