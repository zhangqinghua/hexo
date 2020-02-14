---
title: How to store data in Java Ojbects

categories:
- JavaWorld

date: 2020-02-15
---
Although the snooze (午睡) button is probably the most commoly used button on an alarm clock, even a simple `AlarmClock` class needs a few more features. For instance, you might want to control how long the alarm clock will stay in snooze mode. In order to add such a feature, you need to understand how Java controls data.

DeveLopers use variables in Java to hold data, with all variables having a data type and a name. The data type determines the values that a variable can hold. in this tutorial, you'll learn how integral (整体) types hold whole numbers, floating point types hold real numbers, and string types hold character strings. Then you'll get started with using instance varialbes in you Java classes.

## Varialbes and primitive (原始的) types
Called *primitive types*, integral and floating point types are the simplest data types in Java. The following program illustrates (举例) the integral type, which can hold both positive and negative whole numbers. This program also illustrates comments, which document your code but don't affect the program in any way.

```java
/**
 * This is also a comment. The complier ignores everything from 
 * the first /* until a "star slash" whith ends the comment.
 *
 * Here's the "star slash" that ends the comment. 
 */
public class IntegerTest {
    public static void main(String[] args) {
        // Here's the declaration of an int variable called anInteger,
        // which you given an initial value of 100.
        int anInteger = 100;            // Declare and initialize anInteger
        System.out.println(anInteger);  // Outputs 100
        
        // You can also do arithmetic with primitvie types, using the
        // standard arithmetic operators.
        anInteger = 100 + 100;           
        System.out.println(anInteger);  // Outputs 200
    }
}
```

Java also uses floating point types, which can hold real numbers, meaning numbers that include a decimal place. Here's an example program.

```java
public class DoubleTest {
    public static void main(String[] args) {
        // Here's the declaration of a double variable called aDouble.
        // You also give aDouble an initial value of 5.76.
        double aDouble = 5.76;          // Declare an initialize aDouble
        System.out.println(aDouble);    // Outputs 5.76

        // You can also do arithmetic with floating point types.
        aDouble = 5.76 + 1.45;
        System.out.println(aDouble);    // outputs 7.21
    }
}
```

Try running the programs above. Remember, you have to compile before you can run them.

```bash
javac *.java
java IntegerTest
java DoubleTest
```

Java uses four integral types and two floating point types, which both hold different ranges of numbers and take up varying amounts of storage space, as shown in the tables below.

|TYPE|CATE|SIZE(bits)|RANGE|
|:--|:--|:--|:--|
|Byte|Integral Type|8|-128 to 227|
|Short|Integral Type|16|-32,768 to 32,767|
|Int|Integral Type|32|-2,147,483,648 to 2,147,483,647|
|Long|Integral Type|64|-2^63 to 2^63-1|
|Float|Floating point types (IEEE 754 format)|32|+/-1.18  \* 10^-38 to +/-3.4 \* 10^38|
|Double|Floating point types (IEEE 754 format)|64|+/-2.23 \* 10^-308 to +/-1.8 \* 10^308|

A string type holds strings, and handles them differently from the way integral and floating point types handle numbers. The Java language includes a `String` class to represent strings. You declare a string using the type `String`, and initialize it with a quoted (引号) string, a sequence of characters contained within double quotes, as shown below. You can also combine two strings using the `+` operator.

```java
// Code fragment
// Declaration of variable s of type String.
// and initialization with quoted string "Hello".
String s = "Hello";

// Concatenation of string in s with quoted string " World"
String t = t + " World";
System.out.println(t); // Outputs Hello World
```