---
title: Advanced features Part 2, Exception handling basics

tag:
- Advanced Java language features

categories:
- JavaWorld

date: 2020-03-05 00:00:02
---
Nested (嵌套) classes are classes that are declared as members of others classes or scopes. Nested classed is one way to better organize your code. For example, say you have a non-nested class (also known as a top-level class) that stores objects in a resizable array, followed by an iterator class that returns each object. Rather than pollute the top-level class's namespace, you could declare the iterator class as a member of the rsizable array collection class. This works because the two are closely related.

In Java, nested classes are categoried as either static member classes or inner classes. Inner classes are non-static member classes, local classes, or anonymous (匿名) classes. In this tutorial you'll learn how to work with static member classes and the three types of inner classes in your Java code.

#### test

## Static classes in Java
Fromally (形式上) known as static member classes, these are nested classes that you declare at the same level as these other static entities, using the `static` keyword. Here's an example of a static member class declaration:
```java
class C {
    static int f;

    static void m() {}

    static {
        f = 2;
    }

    static class D {
        // members
    }
}
```

This example introduces top-level class `C` with static field `f`, static method `m()`, a static initializer, and static member cladd `D`. Notice that `D` is a member of `C`. The static field `f`, static method `m()`, and the static initializer are also members of `C`. Since all of these elements belong to class `C`, it is known as the enclosing class (封闭类). Class `D` is known as the enclosed class.

### Enclosure and access rules
Although it is enclosed (围绕), a static member class cannot access the enclosing class's instance fields and invoke its instance methods. However, it can access the enclosing class's static fields and invoke its static methods, even those members that are declared `private`. To demonstrate (演示), Listing 1 declares an `EnclosingClass` with a nested `SMClass`. 

### Listing 1. Declaring a static member class
```java
class EnclosingClass {
    private static String s;

    private static void m1() {
        System.out.println(s);
    }

    static void m2() {
        SMClass.accessEnclosingClass();
    }

    static class SMClass {
        
        static void accessEnclosingClass() {
            s = "Called from SMClass's accessEnclosingClas() method";
            m1();
        }

        void accessEnclosingClass2() {
            m2();
        }

    }
}
```

Listing 1 declares a top-level class named EnclosingClass with class field s, class method `m1()` and `m2()`, and static member class SMClass. SMClass declares class method `accessEnclosingClass()` and instance method `accessEnclosingClass2()`. Note the following:
1. `m2()`'s invocation (调用) of SMClass's `accessEnclosingClass()` method requires the `SMClass.` prefix because `accessEnclosingClass()` is declared `static`
1. `accessEnclosingClass()` is able to access EnclosingClass's s field and call its `m1()` method, even though both have been declared `private`.

Listing 2 presents the source code to an SMCDemo application class that demonstrates how to invoke SMClass's `accessEnclosingClass()` method. It also demonstrates how to instantiate SMClass and invoke its `accessEnclosingClass2()` instance method.

### Listing 2. Invoking a static member class's methods
```java
public class SMCDemo {

    public static void main(String[] args) {
        EnclosingClass.SMClass.accessEncolsingClass();
        EnclsoingClass.SMClass smc = new EnclosingClass.SMClass();
        smc.accessEnclosingClass2();
    }
}
```

As shown in Listing 2, if you want to invoke a top-level class's method from within an enclosed class, you must prefix the enclosed class's name with the name of its enclosing class. Likewise, in order to instanctiate an enclosed class you must prefix the name of that class with the name of its enclosing class. You can then invoke the instance method in the normal maner.

Compile Listing 1 and 2 as follows:
```bash
javac *.java
```

When you compile an enclosing class that contains a static member class, the compiler creates a class file for the static member class whose name consists of its enclosing class's name, dollar-sign character, and the static member class's name. In this case, compiling results is:
```
SMCDemo.class
EnclosingClass.class
EnclosingClass$SMCClass.class
```

Run the application as follows:

```bash
java SMCDemo
```

You should observe the following output:
```
Called from SMClass's accessEnclosingClass() method
Called from SMClass's accessEnclosingClass() method
```

### Example: Static classes and Java 2D
Java's standard class library is a runtime library of class files, which store compiled classes and other reference types. The library includes numerous (许多) examples of static member classes, some of which are found in the Java 2D geometric shape classes located in the `java.awt.geom` package.

The `Ellipse2D` class found in `java.awt.geom` describes an ellipe (椭圆), which is defined by a framing (框架、骨骼) rectangle in terms of (从…方面) an (x, y) upper-left corner along with width and heigh extends. The following code fragment shows that this class's architecture (架构、建筑物) is based on Float and Double static member classes, which both subclass `Ellipse2D`:
```java
public abstract class Ellipse2D extends RectangularShape {
    
    public static class Float extends Ellipse2D implements Serializable {

        public float x, y, widht, heigh;

        public Float() {}

        public Float(float x, float y, float w, float h) {
            setFrame(x, y, w, h);
        }

        public double getX() {
            return (double)x;
        }

        // additional instance methods
    }

    public static class Double extends Ellipse2D implements Serializable {
        
        public double x, y, width, height;
    
        public Double() {}

        public Double(double x, double y, double w, double h) {
            setFrame(x, y, w, h);
        }

        public double getX() {
            return x;
        }

        // additional instance methods
    }

    public boolean contains(double x, double y) {
        // ...
    }

    // additional instance methods shared by Float, Double, and other Ellipse2D subclass.
}
```

The `Float` and `Double` classes extend `Ellipse2D`, providing floating-point and double precision floating-point Ellipse2D implementations.

## Inner classes, tpye 1: Non-static member classes
Consider this example:
```java
class C {
    int f;

    void m() {}

    C() {
        f = 2;
    }

    class D {
        // members
    }
}
```

Here, we introduce top-level class C with instance field f, instance method m(), a construtor, and non-static member class D. All of these entities are members of class C, which encloses them. However, unlike in the previous example, these instance entities are associated with instances of C and not with C class itself.

Each instance of the non-static member class is implicitly associated with an isntance of its enclosing class. The non-static member class's instance methods can call the enclosing class's instance methods and access its instance fields. To demonstrate this access, Listing 3 decalres an EnclosingClass with a nested NSMClass.

#### Listing 3. Declare an enclosing class with a nested non-static member class
```java
class EnclosingClass {
    private String s;

    private void m() {
        System.out.println(s);
    }

    class NSMClass {
        void accessEnclosingClass() {
            s = "Called from NSMClass's accessEnclosingClass() method";
            m();
        }
    }
}
```

Listing 3 declares a top-level class nemed EnclosingClass with instance field s, instance method m(), and non-static member class NSMClass. Futhermore, NSMClass declares instance method `accessEnclosingClass()`.

Because `accessEnclosingClass()` in non-static, NSMClass must be instantiated before this method can be called. This instantiation must take place via an instance of EnclosingClass, as shown in Listing 4.

#### Listing 4. NSMCDemo.java
```java
public class NSMDemo {
    public static void main(String[] args) {
        EnclosingClass ec = new EnclosingClass();
        ec.new NSMClass().accessEnclosingClass();
    }
}
```

Listing 4's `main()` method first instantiates EnclosingClass and saves its reference in local variable ec. The `main()` method then uses the EnclosingClass reference as a prefix to the `new` operator, in order to instantiate NSMClass. The NSMClass reference is then used to call `accessEnclosingClass()` method.

> Should I use `new` with a reference to the enclosing class?
> Prefixing `new` with a reference to the enclosing ia rare (罕见的). Instead, you will typically call an enclosed  class's constructor from within a constructor or an instance method of its enclosing class.

Compile Listing 3 and 4 as follows:
```bash
java *.java
```

When you compile an enclosing class that contains a non-static member class, the compiler creates a class file for the non-static member class whose name consists of its enclosing class's name, a dollar-sign character, and the non-static member class's name. In this case, compiling results in：
```
NSMDemo.class
EnclosingClass.class
EnclisingClass$NSMCClass.class
```

Run the application as follows:
```bash
java NSMDemo
```

You should observe the following output:
```
Called from NSMClass's accessEnclosingClass() method
```

> When (and how) to qualify (限定) `this`
> An enclosed class's code can obtain (得到) a reference to its enclosing-class instance by qualifying reserved word (保留字) `this` with the enclosing class's name and the member access operator `.`. For example, if code within `accessEnclosingClass()` needed to obtain a reference to its `EnclosingClass` instance, it would spcify `EnclosingClass.this`. Because the compiler generates code to accomplish this task, spcifying this prefix is rare.

#### Example: Non-static member classes in HashMap
The standard class library includes non-static member as well as static member classes. For this example, we'll look at the HashMap class, which is part of the Java Collections Framework in the java.util package. HashMap, which describes as has table-based implementation of a map, includes several non-static member classes.

For example, the KeySet non-static member class describes a set-based view of the keys contained in the map. The following code fragment relates the enclosed KeySet class to its HashMap enclosing class:
```java
public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
    // various members

    final class KeySet extends AbstractSet<K> {
        // various members
    }

    // various members
}
```

The `<K, V>` and `<K>` syntaxes are examples of generics, a suite of related language features that help the compiler enforce type safety, these syntaxes help the compiler enforce the type of key ojbects that can be stored in the map and in the keyset, and the type of value objects that can be stored in the map.

HashMap provides a `KeySet()` method that instantiates KeySet when necessary and returns this instance or a cached instance. Here's the complete method:
```java
public Set<K> keyset() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}
```

Notice that the enclosed class's (KeySet's) constructor is called from within the enclosing class's (HashSet's) `keyset()` instance method. This illustrates a common practice (做法) especially because prefixing the `new` operator with an enclosing class reference is rare.

## Inner classes, type 2: Local classes
It's occasionally (偶然) helpful to declare a class in a block, such as a method body or sub-block. For example, you might declare a class that describes an iterator in a mthod that returns an instance of this class. Such classes are known as local classes because (as with local variables) they are local to the methods in which they are declared. Here is example:
```java
interface I {

}

class C {
    // or even static I m()
    I m() {
        class D implements I {
            // members
        }
        return new D();
    }
}
```

Top-level class C declares instance method `m()`, which returns an instance of local class D, which is declared in this method. Notice that `m()`'s return type is interface I, which D implements. The interface is necessary because giving `m()` return type D would result in a compiler error -- D isn't accessible outside of `m()`' body.

> Illegal access modifiers (修饰语) in local class declaration
> The compiler will report an error if a local class declaration contains any of the access modifiers `private`, `public`, or `protected`; or the modifier `static`.

A local class can be associated with an instance of its enclosing class, but only when used in a non-static context. Also, a local class can be declared anywhere that a local variable can be declared, and has the same scope as a local variable. It can access the surrounding scope's local variables and parameters, which must be declared `final`. Consider 5.

### Listing 5. Declaring a local class within an enclosing class instance method
```java
class EnclosingClass {
    
    void m(final int x) {
        final int y = x * 3;

        class LClass {
            int m = x;
            int n = y;
        }

        LClass lc = new LClass();
        System.out.println(lc.m);
        System.out.println(lc.n);
    }
}
```

Listing 5 declares EnclosingClass with instance method `m()`, declaring a local class named LClass. LClass declares a pair of instance fields (m and n). When LClass is instantiated, the instance fields are initialized to the values of final parameter x and final local variable y, as shown in Listing 6.

### Listing 6. A local class declares and initializes a pair of instance fields
```java
public class LCDemo {

    public static void main(String[] args) {
        EnclosingClass ec = new EnclosingClass();
        ec.m(5);
    }
}
```

Listing 6's `main()` method first instantiates EnclosingClass. It then invokes `m(5)` on this instance. The called `m()` method multiplies this argument by 3, instantiates LClass, whose `<init>()` method assigns the argument and the tripled value to its pair of instance fields and outputs LClass's instance fields. (Note that in this case the local class uses the `<init>()` method instead of a constructor to interact with its instance fields.)

Compile Listing 5 and 6 as follows:
```bash
javac *.java
```

When you compile a class whose method contains a local class, the compiler creates a class file for the local class whose name consists of its enclosing class's name, a dollar-sign character, a 1-base integer, and the local class's name. In this case, compiling results:
```
LCDemo.class
EnclosingClass.class
EnclosingClass$1LClass.class
```

### A note about local class name
When generating a name for a local class's file, the compiler adds an integer to the generated name. This integer is probobly generated to distinguish (区分) a local class's class file from a non-static member clas's class file. If two local classes have the same name, the compiler increments the integer to avoid conflicts. Consider the following example:
```java
public class EnclosingClass
{
    public void m1()
    {
       class LClass
       {
       }
    }

    public void m2()
    {
       class LClass
       {
       }
    }

    public void m3()
    {
       class LClass2
       {
       }
    }
}
```

EnclosingClass declares three instance methods that each declare a local calss. The first two methods generate two different local classes with the same name. The first two generate two different local classes with the same name. The compiler generates the following class files:
```
EnclosingClass$1LClass.class
EnclosingClass$1LClass2.class
EnclosingClass$2LClass.class
EnclosingClass.class
```

Run the application as follow:
```bash
java LCDemo
```

You should observe the following output:
```
5
15
```

### Example: Using local classes in regular expressions
The standard class library includes example of local class usage. For example, the Matcher class, in java.util.regex, providers a results() method that returns a stream of match results. This method declares a MatchResultIterator class for iterating over these results:
```java
public Stream<MatchResult> results()
{
   class MatchResultIterator implements Iterator<MatchResult>
   {
      // members
   }
   return StreamSupport.stream(Spliterators.spliteratorUnknownSize(new MatchResultIterator(),
                                                                   Spliterator.ORDERED |
                                                                   Spliterator.NONNULL),
                               false);
}
```

Note the instantiation of MatchResultIterator() following the class declaration. Don't worry about parts of the code that you don't understand; instead, think about the usefulness in being able to declare classes in the appropriate (恰当的) scopes (such as method body) to better organize your code.

## Inner classes, type 3: Anonymous classes
Static member classes, non-static member classes, and local classes have names. In constrast (相反), anonymous classes are unnamed nested classes. You introduce them in the context of expressions that invoke the new operator and the name of either a base class or an interface that is implemented by the anonmous class:
```java
// subclass the base class

abstract class Base
{
   // members
}

class A
{
   void m()
   {
      Base b = new Base()
               {
                 // members
               };
   }
}

// implement the interface

interface I
{
   // members
}

class B
{
   void m()
   {
      I i = new I()
            {
               // members
            };
   }
}
```

The first example demonstractes an anonymous class extending a base class. Expression new Base() is followed by a pair of brace characters that signify the anonymous class. The second example demonstrates an anonymous class inplementing an interface. Expression new I() is followed by a pair of brace characters that signify the anonymous class.

> Constructing anonymous class instances
> An anonymous class cannot have an explicit constructor because a constructor must be named after the class and anonymous classes are unnamed. Instead, you can use an object initilization block ({}) as a constructor.

Anonymous classes are useful for expressing functionality that's passed to a method as its argument as its argument. For example, consider a method for sorting an array of integer. You want to sort the array in ascending or descending order, based on compairisons between pairs of array elements. You might duplicate the sorting code, with one version using the less than (<) operator for one order, and the other version using the greater than (>) operator for the opposite order. Alternatively, as shown below, you could design the sorting code to invoke a comparsion method, then pass an object constaing this method as an argument to the sorting method.

### Listing 7. Using an anonymous class to pass functionality as a method argument
```java
public abstract class Comparer
{
   public abstract int compare(int x, int y);
}
```

The compare() method is invoked with two integer array elements and returns one of three valudes: a negative value if x is less than y, 0 if both values are the same, and a positive value is x is greater than y. Listing 8 presents an application whose sort() method invokes compare() to preform the compairsions.

### Listing 8. Sorting an array of integers with the Bubble Sort algorithm
```java
public class ACDemo
{
   public static void main(String[] args)
   {
      int[] a = { 10, 30, 5, 0, -2, 100, -9 };
      dump(a);
      sort(a, new Comparer()
                  {
                     public int compare(int x, int y)
                     {
                        return x - y;
                     }
                  });
      dump(a);
      int[] b = { 10, 30, 5, 0, -2, 100, -9 };
      sort(b, new Comparer()
                  {
                     public int compare(int x, int y)
                     {
                        return y - x;
                     }
                  });
      dump(b);
   }

   static void dump(int[] x)
   {
      for (int i = 0; i < x.length; i++)
         System.out.print(x[i] + " ");
      System.out.println();
   }

   static void sort(int[] x, Comparer c)
   {
      for (int pass = 0; pass < x.length - 1; pass++)
         for (int i = x.length - 1; i > pass; i--)
            if (c.compare(x[i], x[pass]) < 0)
            {
               int temp = x[i];
               x[i] = x[pass];
               x[pass] = temp;
            }
   }
}
```

The main() method reveals two calls to ites companion sort() method, which sorts an array of integers via the Bubble Sort algorithm. Each call receives an integer array as its first argument, and a reference to an object created from an anonymous Comparer subclass as its second argument. The first call achieves an ascending order sort by substracting y from x; the second call achieves a desceding order sort by subtracting x from y.

> Migrating from anonymous classes to lambdas
> As you can see in Listing 8, passing anonymous calss-based functionality can lead to varbose syntax. Startring with Java 8, you can use lambdas for more concise code.

Compile Listing 7 and 8 as follows:
```
javac *.java
```

When you compile a class whose method contains an anouymous class, the compiler creates a class file for the anonymous class whose name consits of its enclosing class's name, a dollar-sign character, and an integer that uniquely identifies the anonymous class. In this case, compiling results:
```
ACDemo.class.
ACDemo$1.class
ACDemo$2.class 
```

Run the application as follows:
```bash
java ACDemo
```

You should obeserve the following output:
```

10 30 5 0 -2 100 -9
-9 -2 0 5 10 30 100
100 30 10 5 0 -2 -9
```

### Example: Using anonymous classes with an AWT event hanlder
Anonymous classes can be used with many packages in the standard calss library. For this example, we'll use an anonymous class as an event handler in the abstract Windowing Toolkit or Swing Windowing Toolkit. The following code fragment registers an event handler with Swing's JButton class, which is located in the javax.swing package. JButton describes a button that performs an action (in the case printing a message) when clicked.

```java
JButton btnClose = new JButton("close");
btnClose.addActionListener(new ActionListener()
                               {
                                  public void actionPerformed(ActionEvent ae)
                                  {
                                     System.out.println("close button clicked");
                                  }
                               });
```

The first line instantiates JButton, passing close as the button label to JButton's constructor. The second line registers an action listener with the button. The action listener's actionPerformed() method is invoked whenever the button is clicked. The object passed to addActionListener() is instantiated from an anonymous class that implements the java.awt.event.ActionListener interface.

## Conclusion
Java's nesting capabilities help you organize non-top-level reference types. For top-level reference types, Java provides packages.