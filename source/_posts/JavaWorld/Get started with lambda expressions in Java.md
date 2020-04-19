---
title : Get started with lambda expressions in Java

categories:
- JavaWorld

date: 2020-04-17
---
Before Java SE 8, anonymous   classes were typically used to pass functionality to a method. This practice (实践) obfuscated (模糊化) source code, making it harder to understand. Java 8 eliminated (淘汰) this problem by introducing lambdas. This tutorial first introduces the lambda language feature, then provides a more detailed introducetion to functional programming with lambda expressions along with target types. You'll also learn how lambdas interact with scopes, local variables, the this and super keywords, and Java exceptions.

Note that code examples in this tutorial are compatible with JDK 12.

> Discovering types for yourself
> I won't introduce any non-lambda language features in this tutorial that you haven't perviously learned about, but I will demonstrate lambdas via types taht I haven't perviously discussed in this series. One example is the java.lang.Math class. I will introduce these types in future Java 101 tutorials. For now, I suggest reading the JDK 12 API documentation to learn more about them.

## Lambdas: A primer
A lambda expression (lambda) describes a block of code (an anonymous function) that can be passed to constructors or methods for subsequent execution. The constructor or method receviers the lambdas as an argument. Consider the following example:
```java
() -> System.out.println("Hello")
```

This example identifies a lambda for outputting a message to the standard output stream. From left to right, `()` identifies the lambda's formal parameter list (there are no parameters in the example) `->` indicates that the expression is a lambda, and `System.out.println("Hello")` is the code to be executed.

Lambdas simplify the use of functional interfaces, which are annotated interfaces that each declare one abstract method (although they can also declare any combination of default, static, and private methods.) For example, the standard class library provides a java.lang.Runnable interface with a single abstract `void run()` method. This functional interface's declaration appears below:
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

The class libaray annotates Runable with @FunctionalInterface, which is an instance of the java.lang.FunctionInterface annotation type. FuntcionalInterface is used to annotate those interfaces that are to be used in lambda contexts.

A lambda doesn't have an explicit interface type. Instead, the compiler uses the surrounding context to infer which functional interface to instantiate when a lambda is specified -- the lambda is bound to that interface. For example, suppose I specified the following code fragment, which passes the previous lambda as an argument to the java.lang.Thread class's `Thread(Runnable target)` constructor:
```java
new Thread(() -> System.out.println("Hello"));
```

The compiler determines that the lambda is being passed to `Thread(Runnable r)` because this is the only contructor that satisfies the lambda: Runnable is a functional interface, the lambda's emptry formal paramter list `()` matches `run()`'s emptry parameter list, and the return types `(void)` also agree. The lambda is bound to Runnable.

Listing 1 presents the source code to a small application that lets you play with this example.

### Listing 1. LambdaDemo
```java
public class LambdaDemo {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("Hello").start());
    }
}
```

Compile Listing 1 and run the application. You should oberve the following output:
```
hello
```

Lambdas can greatly simplify the amount of source code that you must write, and can also make source code much easier to understand. For example, without lambdas, you would probably specify Listing 2's more verbose code, which is based on an instance of an anonymous calss that implements Runnable.

### Listing 2. LambdaDemo
```java
public class LambdaDemo
{
   public static void main(String[] args)
   {
      Runnable r = new Runnable()
                   {
                      @Override
                      public void run()
                      {
                         System.out.println("Hello");
                      }
                   };
      new Thread(r).start();
   }
}
```

After compiling this source code, run the application. You'll discover the same output as  previously shown.

> Lambdas and the Streams API
> As well as simplifying source code, lambdas play an important role in Java's functionally-oriented Streams API. The describe units of functionality that are passed to various API methods.

## Java lambdas in depth
To use lambdas effectively, you must understand the syntax of lambda expression along with the notion of a target type. You also need to understand how lambdas interact with scopes, local variables, the this and super keywords, and exceptions. I'll cover all of these topics in the sections that follow.

> How lambdas are implemented
> Lambdas are implemented in terms of the Java virtual mechine's invokedynamic (动态类型语言) instruction (指令) and the java.lang.invoke API.

## Lambda syntax
Every lambda conforms to the following syntax:
```
( formal-parameter-list ) -> { expression-or-statements }
```

The formal-parameter-list is a comma-separated list of formal paramater, which must match the parameters of a funcitonal interface's single abstract method at runtime. If you omit their types, the compiler infers (推断) these types from the context in which the lambda is used. Consider the following examples:
```java
(double a, double b)// types explicitly specified
(a, b)              // types inferred by compiler
```

> Lambdas and var
> Starting with Java SE 11, you can repalce a type name with var. For example, you could specify `(var a, var b)`.

You must specify parentheses (括弧) for multiple or no formal parameters. However, you can omit the parentheses (although you don't have to) when specifying a single formal parameters. (This applies to the parameter name only -- parentheses  are required when the type is also specified.) Consider the follwoing addditional examples:
```java
x           // parentheses omitted due to single formal parameter
(double x)  // parentheses required because type is also present
()          // parentheses required when no formal parameters
(x, y)      // parentheses required because of multiple formal parameters
```

The formal-parameter-list if followed by a `->` token, which is followed by expression-or-statements -- an expression of a block of statements (either is known as the lambda's body). Unlike expression-based bodies, statement-based bodies must be placed between open and close brace characters(`{}`):
```java
(double radius) -> Math.PI * radius * radius
radius -> { return Math.PI * radius * radius; }
radius -> { System.out.println(radius); return Math.PI * radius * radius; }
```

The first example's expression-based lambda body doesn't have to be placed between braces. The second example converts the expression-based body to a stattement-based body, in which return must be specified to return the expressions's value. The final example demonstractes multiple statements and cannot be expressed without braces.

> Lambda bodies and semicolons
> Note the absence or presence of semicolons (分号, `;`) in the previous examples.