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

## Variable scope
In addition to type, scope is also an important characteristic of a variable. Scope establishes (确定) when a variable is created and destroyed and where a developer can access the variable within a program. The place in your program where you declare the variable determines its scope.

So far, I've discussed local variables, which hold temporary data that you use within a method. You declare local varialbes inside methods, and you can access them only from those methods. This means that you can retrieve (检索) only local variables `anInteger`, which you used in `IntegerTest.java`, and `aDouble`, which you used in `DoubleTest.java`, from the main method in which they were declared and nowhere (没有…的地方) else.

You can declare local variables within any method. The example code below declares a local variable in the `AlarmClock.snooze()` method:
```java
public class AlarmClock {
    public void snooze() {
        // Snooze time in millisecond = 5 secs
        long snoozeInterval = 5000;
        System.out.println("ZZZZZ for: " + snoozeInterval);
    }
}
```

You acn get to `snoozeInterval` only from the `snooze()` method, which is where you declared `snoozeInterval`, as shown here:
```java
public class AlarmClockTest{
    public static void main(String[] args) {
        AlarmClock aClock = new AlarmClock();
        aClock.snooze();    // This is still fine.
        // The next line of code is an ERROR.
        // You can't access snoozeInterval outside the snooze method.
        snoozeInterval = 10000;
    }
}
```

## Method parameters
A method parameter, which has a scope similar to a local variable, is another type of variable. Method parameters pass arguments into methods. When you declare the method, you specify its arguments in a parameter list. You pass the arguments when you call the method. Method parameters function similarly to local variables in that they lie within the scope of the method to which they are linked, and can be used throughout the method. However, unlike local variables, method parameters obtain (取得) a value from the caller when it calls a method. Here's a modification of the alarm clock that allows you to pass in the `snoozeInterval`.

```java
public class AlarmClock {
    public void sooze(long snoozeInterval) {
        System.out.println("ZZZZZ for: " + snoozeInterval);
    }
}

public class AlarmClockTest {
    public static void main(String[] args) {
        AlarmClock aClock = new AlarmClock();
        // Pass in the snooze interval when you call the method.
        aClock.snooze(10000); // Snooze for 100000 msecs.
    } 
}
```

## Member variables
Local variables are useful, but because the provice only temporary storage, their value is limited. Since their lifetimes span the length of the method in which they are declared, local variables compare to a notepad that appears every time you receive a telephone call, but disappears when you hang up. That setup can be useful for jotting (略记) down notes, but sometimes you need something more permanent, What's a programmer to do? Enter `member variables`.

## Variable scope an lifetime
Developers implement instance variable (实例变量) to contain data useful to a class. An instance variable differs from a local variable in the nature of its scope and its lifetime. The entire class makes up the scope of an instance variable, not the method in which it was declared. In other words, developers can access instance variables anywhere in the class. In addition, the lifetime of an instance variable does not depend on any particular method of the class; that is, its lifetime is the lifetime of the instance that contains it.

Instances are the actual objects that you create from the blueprint you design in the class definition. You declare instance variables in the class definition, affecting each instance you create from the blueprint. Each instance contains those instance variables, and data held within the variables can vary (不同) from instance to instance.

Consider the `AlaremClock` class. Passing the `snoozeInterval` into the `snooze()` method isn't a great design. Imagine having to type in a snooze interval on your alarm clock each time you fumbled for the snooze button. Instead, just give the whold alarm clock a `snoozeInterval`. You complete this with an instance variable in the `AlarmClock` class, as shown below:
```java
public class AlarmClock {
    // You declare snoozeInterval here. This makes it an instance variable.
    // You can also initialize it here.
    long m_snoozeInterval = 5000;   // Snooze time in millisecond = 5 secs.

    public void snooze() {
        // You can still get to m_snoozeInterval in an AlarmClock method 
        // because you are within the scope of the class.
        System.out.println(m_snooozeInterval);
    }
}
```

You can access instance variables almost anywhere within the class that declares them. To be technical about it, you declare the instance variable within the class scope, and you can retrieve it from almost anywhere within that scope. Practically speaking, you can access the variable anywhere between the first curly bracket (花括号) that starts the class and the closing bracket. Since you also declare methods within the class scope. they too can access the instance variables.

You can also access instance variables from outside the class, as long sa an instance exists, and you have a variable that references the instance. To retrieve an instance variable through an instance, you use the *dot*(`.`) operator together with the instance. That may not be the ideal way to access the variable, but for now, complete it this way for illustrative (说明的) purposes (目的):
```java
public class AlarmClockTest {
    public static void main(String[] args) {
        // Create two clocks. Each has its own m_snoozeInterval
        AlarmClock aClock1 = new AlarmClock();
        AlarmClock aClock2 = new AlarmClock();

        // Change aCLock2
        // You'll soon see that are much better ways to do this.
        aClock2.m_snoozedInterval = 100000;
        aClock1.snooze();   // Snooze with aClock1's interval
        aClock2.snooze();   // Snooze with aClock2's interval
    }
}
```

Try this program out, and you'll see that `aClock1` still ha its interval of 5000 while `aClock2` has an interval of 10000. Again, each instance has its own instance data.

Don't forget, the class definition is only a blueprint, so the instance variables don't actually exist until you create instances from the blueprint. Each instance of a class has it own copy of the instance variables, and the blueprint defines what those instance variables will be.

## Encapsulation
Encapsulation (封装) is one of the foundations of object-oriented programming. When using encapsulation, the user interacts (交互) with the type through the exposed (暴露的) behavior, not directly with the internal implementation. Through encapsulation, you hide the details of a type's implementation. In Java, encapsulation basically translates to this simple guideline: "Don't access your object's data directly; use its methods."

That is an elementary (简单的) idea, but it eases our lives as programmers. Imagine, for example, that you wanted to instruct a `Person` object to stand up. Without encapsulation, your commands could go something like this: "Well, I guess you'd need to tighten (绷紧) this muscle (肌肉) here at the front of the leg, loosen (放松) this muscle here at the back of the leg. Hmmm -- need to bend at (弯曲) the waist (腰部) too. Which muscles spark (触发) that movement? Need to tighten these, loosen those. Whoops! Forgot the other leg. Darn. Watch it -- don't top over..." You get the idea. With encapsulation, you would just need to invoke the `stanUp()` method, Pretty easy, yes?

Some advantages to encapsulation:
- Abstraction of detail: The user interacts with a type at a higher level. If you use the `standUp` method, you no longer need to know all the muscles required to initiate that motion.
- Isolation (隔离) from changes: Changes in internal implementation don't affect the users. If a person sprains (扭伤) an ankle (踝关节), and depends on a create for a while, the users still invoke only the `standUp()` method.
- Correctness (正确性): Users can't arbitraily ()任意 change the insiders of an object. The can only complete what you allow them to do in the methods you write.

Here's a short example in which encapsulation clearly hepls in a program's accuracy (准确度):
```java
// Bad -- doesn't use encapsulation
public class Person {
    int m_age;
}

public class PersonTest {
    public static void main(String[] args) {
        Person p = new Person();
        p.m_age = -5; // Hey -- how can someone be minus 5 years old?
    }
}

// Better -- uses encapsulation
public class Person {
    int m_age;

    public void setAge(int age) {
        // Check to make sure age is greater than 0. I'll talk more about
        // if statements at another time.
        if (age > 0) {
            m_age = age;
        }
    }
}

public class PersonTest {
    public static void main(String[] args) {
        Person p = new Person();
        p.setAge(-5);   // Won't have any effect now.
    }
}
```

Even that simple program shows how you can slip into trouble if you directly access the internal data of classes. The larger and more complex the program, the more important encapsulation becomes. Also remember that many programs start out small and then grow to last indefinitely (无限地). so it's essential that you design them correctly, right from the beginning. To apply encapsulation to `AlarmClock`, you can just create methods to manipulate (操纵) the snooze interval.

> A note about methods
> Methods can return values that the caller users. To return a value, declare a nonvoid return type, and use a `return` statement.

## Write the program
Okay -- you're ready to manipulate the snooze interval. You do this by adding get and set methods for the snooze interval. When you have an instance variable like `snoozeInterval`, you will regularly call the get and set methods `getSnoozeInterval()` and `setSnoozeInterval()`.

```java
public class AlarmClock {
    long m_snoozeInterval = 5000;    // Snooze time in millisecond

    // Set method for m_snoozeInterval.
    public void setSnoozeInterval(long snoozeInterval) {
        m_snoozeInterval = snoozeInterval;
    }

    // Get method for m_snoozeInterval.
    // Note that you are returning a value of type long here.
    public long getSnoozeInterval() {
        // Here's the line that returns the value.
        return m_snoozeInterval;
    }

    public void snooze() {
        // You can still get to m_snoozeInterval in an AlarmClock method
        // because you are within the scope of the class.
        System.out.println("ZZZZZ for: " + m_snoozeInterval);
    }
}
```

```java
public class AlarmClockTest {

    public static void main(String[] args) {
        // Create two clocks. Each has its own m_snoozeInterval.
        AlarmClock aClock1 = new AlarmClock();
        AlarmClock aClock2 = new AlarmClock();
        // Change aClock2. You use the set method.
        aClock2.setSnoozeInterval(10000);
        aClock1.snooze();    // Snooze with aClock1's interval.
        aClock2.snooze();    // Snooze with aClock2's interval.
    }
}
```

Defined now are tow methods to manipulate the snooze interval. One is used to get the snooze interval, and the other is used to set it. That may seem trivial (琐碎的), but then, `AlarmClock` is a trivial class.

## Conclusion
In this quick tutorial you've looked at how manipulate primitive types like `int` and `double`. You examined local variables, method parameters, and variable scope. You learned how to add data to classes using instance variables, and how that data is contained in each instance. Finally, you explored encapsulation and how it leads to better code.