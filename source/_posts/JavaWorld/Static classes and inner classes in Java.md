---
title: Static classes and inner classes in Java

categories:
- JavaWorld

date: 2020-03-05
---
Nested (嵌套) classes are classes that are declared as members of others classes or scopes. Nested classed is one way to better organize your code. For example, say you have a non-nested class (also known as a top-level class) that stores objects in a resizable array, followed by an iterator class that returns each object. Rather than pollute the top-level class's namespace, you could declare the iterator class as a member of the rsizable array collection class. This works because the two are closely related.

In Java, nested classes are categoried as either static member classes or inner classes. Inner classes are non-static member classes, local classes, or anonymous (匿名) classes. In this tutorial you'll learn how to work with static member classes and the three types of inner classes in your Java code.

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

This example introduces top-level class `C` with static field `f`, static method `m()`, a static initializer, and static member cladd `D`. Notice that `D` is a member of `C`. The static field `f`, static method `m()`, and the static initializer are also members of `C`. Since all of these elements belong to class `C`, it is known as the enclosing class (封闭类). Class `D` is known as the enclosed calss.

### Enclosure and access rules
Although it is enclosed (围绕), a static member class cannot access the enclosing class's instance fields and invoke its instance methods. However, it can access the enclosing class's static fields and invoke its static methods, even those members that are declared `private`. To demonstrate (演示), Listing 1 declares an `EnclosingClass` with a nested `SMClass`. 

#### Listing 1. Declaring a static member class
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

Listing 1 declares a top-level class named `EnclosingClass` with class field `s`, class method `m1()` and `m2()`, and static member class `SMClass`. `SMClass` declares class method `accessEnclosingClass()` and instance method `accessEnclosingClass2()`. Note the following:
1. `m2()`'s invocation (调用) of `SMClass`'s `accessEnclosingClass()` method requires the `SMClass.` prefix because `accessEnclosingClass()` is declared `static`
1. `accessEnclosingClass()` is able to access `EnclosingClass`'s `s` field and call its `m1()` method, even though both have been declared `private`.

Listing 2 presents the source code to an `SMCDemo` application class that demonstrates how to invoke `SMClass`'s `accessEnclosingClass()` method. It also demonstrates how to instantiate `SMClass` and invoke its `accessEnclosingClass2()` instance method.

#### Listing 2. Invoking a static member class's methods
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

When you compile an enclosing class that contains a static member class, the compiler creates a class file for the static member class whose name consists of its enclosing class's name, dollar-sign character, and the static member class's name. In this case, compiling results in EnclosingClass.class and EnclosingClass$SMCClass.class.

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

#### Listing 3. Declare an enclosing class with a nested non-static member class

#### Listing 4. NSMCDemo.java

### Example: Non-static member classes in HashMap

## Inner classes, type 2: Local classes

## Example: Using local classes in regular expressions

## Inner classes, type 3: Anonymous classes

## Example: Using anonymous classes with an AWT event hanlder

## Conclusion