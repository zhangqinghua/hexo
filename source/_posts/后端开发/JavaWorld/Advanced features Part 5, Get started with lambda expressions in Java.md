---
title : Advanced features Part 5, Get started with lambda expressions in Java

tag:
- Advanced Java language features

categories:
- JavaWorld

date: 2020-03-05 00:00:05
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
> Note the absence or presence of semicolons (分号, `;`) in the previous examples. In each case, the lambda body isn't terminated with a semicolon because the lambda isn't statement. However, within a statement-based lambda body, each statement must be terminated with a ssemicolon.

Listing 3 presents a simple application that demonstrates lambda syntax; note that this listing builds on the previous two code examples.

### Listing 3. LambdaDemo
```java
@FunctionalInterface
interface BinaryCalculator {
    double calculate(double value1, double value2);
}

@FunctionalInterface
interface UnaryCalculator {
    double calculate(double value);
}

public class LambdaDemo {
    public static void main(String[] args) {
        System.out.printf("18 + 36.5 = %f%n", calculate((double v1, double v2) -> v1 + v2, 18, 36.5));
        
        System.out.printf("89 / 2.9 = %f%n", calculate((v1, v2) -> v1 / v2, 89, 2.9));

        System.out.printf("-89 = %f%n", calculate(v -> -v, 89));
      
        System.out.printf("18 * 18 = %f%n", calculate((double v) -> v * v, 18));
    }

    static double calculate(BinaryCalculator calc, double v1, double v2) {
        return calc.calculate(v1, v2);
    }

    static double calculate(UnaryCalculator clac, double b) {
        return calc.calculate(b);
    }
}
```

Listing 3 first introduces the BinaryCalculator and UnaryCalculator functional interfaces whose `calculate()` methods perform calculations on two input arguments or on a single input argument, respectively. This listing also introduces a LambdaDemo calss whose `main()` method demonstrates these functional interfaces.

The functional interfaces are demonstrated in the `static double calculate(BinaryCalculator calc, double v1, double v2)` and `static double calculate(UnaryCalculator calc, double v)` methods. The lambdas pass code as data to these methods, which are received as BinaryCalculator or UnaryCalculator instances.

Compiling Listing 3 and run the application. You should observe the following output:
```
18 + 36.5 = 54.500000
89 / 2.9 = 30.689655
-89 = -89.000000
18 * 18 = 324.000000
```

## Target types
A lambda is associated with an implicit target type, which identifies the type of object to which a lambda is bound. The target type must be a functional interface that's inferred from the context, which limit lambdas to appearing in the following context:
- Variable declaration
- Assignment
- Return statement
- Array initializer
- Method or constructor arguments
- Lambda body
- Ternary (三元) conditional expression
- Cast expression

Listing 4 presents an application that demonstrates these target type contexts.

### Listing 4. LambdaDemo
```java 
public class LambdaDemo {
    public static void main(String[] args) {
        // Target type #1: variable declaration
        Runnable r = () -> {System.out.println("running");};
        r.run();

        // Target type #2: assignment
        r = () -> System.out.printnl("running");
        r.run();

        // Target type #3: return statement (in getFilter())
        File[] files = new File(".").listFiles(getFilter("txt"));
        for (int i = 0; i < files.length; i++) {
            System.out.println(files[i]);
        }

        // Target type #4: array initializer
        FileSystem fs = FileSystems.getDefault();
        final PathMatcher matchers[] = {
            (path) -> path.toString().endWith("txt"),
            (path) -> path.toString().endWith("java")
        };
        FileVistor<Path> visitor = new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attribs) {
                Path name = file.getFileName();
                for (int i = 0; i < matchers.length; i++) {
                    if (matchers[i].matches(name)) {
                        System.out.println("Found matched file: '%s'.%n", file);
                    }
                }
                return FileVisitResult.CONTINUE;
            }
        }
        Files.walkFileTree(Paths.get("."), visitor);

        // Target type #5: method or constructor arguments
        new Thread(() -> System.out.println("running").start());

        // Target type #6: lambda body (a nested lambda)
        Callable<Runnable> callable = () -> () -> System.out.println("called");
        callable.call().run();

        // Target type #7: ternary conditional expression
        boolean ascendingSort = false;
        Comparator<String> cmp = (ascendingSort) ? (s1, s2) -> s1.compareTo(s2)
                                                 : (s1, s2) -> s2.compareTo(s1);
                                                 
        List<String> cities = Arrays.asList("Washington", "london", "Rome", "Berlin", "Jerusalem", "Ottawa", "Sydney", "Moscow");
        Collections.sort(cities, cmp);
        for (int i = 0; i < cities.size(); i++) {
            System.out.println(cities.get(i));
        }

        // Target type #8: cast expression
        String user = AccessController.doPrivileged((PrivilegedAction<String>)() -> System.getProperty("user.name"));
        System.out.println(user);
    }

    static FileFilter getFilter(String ext) {
        return (pathname) -> pathname.toString().endWith(ext);
    }
}
```

The first example demonstrates a lombda in variable declaration context. it assigns lombda `() -> {System.out.println("running");}` to variable r of Runnable interface type. The second example is similar, but demonstrates a lambda in an assignment context (to previously declared variable r).

The third example demonstrates a lambda in a return statement context. It invokes the getFilter() method with a specified file extension arguement to return a java.io.FileFilter object. This object is passed to java.io.file's listFiles() method, which invokes the filter for each file, ignoring files that don't match extension.

The getFilter() method returns a FileFilter object expressed via a lambda. The compiler notes that the lambda satisfies this functional interafce's `boolean accept(File pathname)` method (both have a single parameter and the lambda body returns a Boolean value) and binds the lambda to FileFilter.

The fourth example demonstrates lambda usage in an array initializer context. Two java.nio.file.PathMatcher objects are created based on lambdas. Each PathMatcher object matches files based on criteria (标准) specified by its lambda's body. Here is the relevant code:
```java
final PathMatcher matchers[] = {
    (path) -> path.toString().endsWith("txt"),
    (path) -> path.toString().endsWith("java")
};
```

The PathMatcher functional interface provides a `boolean matches(Path path)` method that agrees with the lambda's parameter list and its body's Boolean return type. This method is subsequently called to determine a match (based on file extension) for each encountered file during a visit of the current directory and subdirectories.

The fifth example demonstrates a lambda in a Thread constructor context. The sixth example demonstrates a lambda in a lambda context, which shows that lambdas can be nested. The Seventh example demonstrates a lambda in a ternery conditional expression (`?:`) context: one of two lambdas is selected based on an ascending or descending sort.

The eighth (and final) example demonstrates a lambada in a cast expression context. The `() -> System.getProperty("user.name")` lambda is cast to `PrivilegedAction<String>` functional interface type. This cast addresses an ambiguity in the java.security.AccessController class, which declares the following mehtods:
```java
static <T> T doPrivileged(PrivilegedAction<T> action)
static <T> T doPrivileged(PrivilegedExceptionAction<T> action)
```

The problem is that each of interfaces PrivilegedAction and PrivilegedExceptionAction declares an identical `T run()` method. Because the compiler cannot figure out which interface is the target type, it reports an error in the absence of the cast.

Compile Listing 4 and run the application. You should observe the following output, which assumes that LambdaDemo.java is the only .java file in the current directory and that this directory contains no .txt files:
```
running
running
Found matched file: '.\LambdaDemo.java'.
running
called
Washington
Sydney
Rome
Ottawa
Moscow
London
Jerusalem
Berlin
jeffrey
```

## Lambdas and scopes
The term scope refers to that part of a program where a name is bound to a particular entiry (e.g., a variable). In another part of the program, the name may be bound to another enitty. A lambda body doesn't introduce a new scope. Instead, its scope is the enclosing scope.

## Lambdas an loal variables
A lambda body can define local variables. Because these variables are considered part of the enclosing scope, the compiler will report an error when it detects that the lambda body is redefinning a local variable. Listing 5 demonstrates this problem.

### Listing 5. LambdaDemo
```java
public class LambdaDemo {
    public static void main(String[] args) {
        int limit = 10;
        Runnable r = () -> {
            int limit = 5;
            for (int i = 0; i < limit; i++) {
                System.out.println(i);
            }
        }
    }
}
```

Because limit is already present in the enclosing scope (the main() method), the lambda body's redeinition of `limit (int limit =5;)` cause the compiler to report the following error message: error: variable limit is already defined in method main(String[] args).

> Lambda bodies and local variables
> Whether originating in a lambda body or in the enclosing scope, a local variable must be initialized before being used. Otherwise, the compiler will report an error.

A local varialbe or parameter that's defined outside a lambda body and referenced from the body must be markded final or considered effectively final (the variable cannot to assigned to after initialization). Attempting to modify an effectively final varialbe causes the compiler to report an error, as demonstrated in Listing 6.

### Listing 6. LambdaDemo
```java
public class LambdaDemo {
    public static void main(String[] args) {
        int limit = 10;
        Runable r = () -> {
            limit = 5;
            for (int i = 0; i < limit; i++) {
                System.out.println(i);
            }
        }
    }
}
```

limit is effectively final. The lambda body's attempt to modify this variable causes the compiler to report an error. It does so because a final/effectively final variable will need to hang around until the lambda executes, which may not happen until long after the code in which the variable was defined returns. Non-final/non-effectively final variable no longer exist.

## Lambdas and the 'this' and 'super' keywords
Any this or super reference that is used in a lambda body is regarded as being equivalent to its usage in the enclosing scope (bacause a lambda doesn't inctroduce a new scope). However, this isn't the case with anonymous classes, which Listing 7 demonstrates.

### Listing 7. LambdaDemo
```java
public class LambdaDemo {
    public static void main(String[] args) {
        new LambdaDemo().deWork();
    }

    public void doWork() {
        System.out.printf("this - %s%n", this);
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.printf("this = %s%n", this);
            }
        };

        new Thread(r).start();
        new Thread(() -> System.out.printf("this = %n%f", this)).start();
    }
}
```

Listing 7's main() method instantiates LambdaDemo and invokes the object's doWork() method to output the object's this reference, instantiate an anonymous class that implements Runnable, create a Thread object that executes this runnable when its thread is started, and create another Thread object whose thread executes a lambda when started.

Compile Listing 7 and run the application. You should observe something similar to the following output:
```
this = LambdaDemo@776ec8df
this = LambdaDemo$1@48766bb
this = LambdaDemo@776ec8df
```

The first line shows LambdaDemo's this reference, the second line shows a different this reference in the new Runnable scope, and the third output line shows the this reference in a lambda context. The third and first lines match because the lambda's scope is nested inside the doWrok() method; this has the same meaning throughout this method.

## Lambdas and exceptions
A lambda body isnot allowed to throw more exceptions than are specified in the throws clause of the functional interface method. If a lambda body throws an exception, the functional interface method's throws clause must declare the same exception type or its supertype. Consider Listing 8.

### Listing 8. LambdaDemo
```java
@FunctionalInterface
interface Work {
    void dosomething() thrwos IOException;
}

public class LambdaDemo {
    public static void main(String[] args) throws AWTException, IOException {
        Work work = () -> {throw new IOException();};
        work.doSomething();
        work = () -> {throw new AWTException("");};
    }
}
```

Listing 8 declares a Work functional interface whose doSomething() method is declared to throw java.io.IOException. The main() method assigns a lambda that throws IOException to work, which is okay becuase IOException is listed in doSomething()'s throws clause.

main() next assigns a lambda that throws java.awt.AWTException to work. However, the compiler doesn't allow this assignment bacuase AWTException isn't part of doSomething()'s throws caluse (and is certainly not a subtype of IOException).

## Predefined (预定义) functional interfaces
You might find yourself repeatedly creating similar functional interfaces. For example, you might carete a CheckConnection functional interface with a `boolean isConnected(Connection c)` method and a CheckAccout functional interface with a `boolean isPositiveBalance(Account acct)` method. This is wasteful.

THe previous examples expose the abstract concept of a predicate (a Boolean valued function). Anticipating such patterns. Oracle provides the java.util.function package of commonly-used functional interfaces. For example, this package's `Predicate<T>` functional interface can be used in place of CheckConnect and CheckAccount.

`Predicate<T>` provides a `boolean test(T t)` method that evaluates this predicate on its argument (t), returning true when t matches the predicate, and returning false otherwise. Notice that `test()` provides the same kind of parameter list as `isConnected()` and `isPositiveBalance()`. Also, notice that they all have the same return type (boolean).

The application source code in Listing 9 demonstrates `Predicate<T>`.

### Listing 9. LambdaDemo
```java
class Account
{
   private int id, balance;
   Account(int id, int balance)
   {
      this.balance = balance;
      this.id = id;
   }
   int getBalance()
   {
      return balance;
   }
   int getID()
   {
      return id;
   }
   void print()
   {
      System.out.printf("Account: [%d], Balance: [%d]%n", id, balance);
   }
}
public class LambdaDemo
{
   static List<Account> accounts;
   public static void main(String[] args)
   {
      accounts = new ArrayList<>();
      accounts.add(new Account(1000, 200));
      accounts.add(new Account(2000, -500));
      accounts.add(new Account(3000, 0));
      accounts.add(new Account(4000, -80));
      accounts.add(new Account(5000, 1000));
      // Print all accounts
      printAccounts(account -> true);
      System.out.println();
      // Print all accounts with negative balances.
      printAccounts(account -> account.getBalance() < 0);
      System.out.println();
      // Print all accounts whose id is greater than 2000 and less than 5000.
      printAccounts(account -> account.getID() > 2000 &&
                               account.getID() < 5000);
   }
   static void printAccounts(Predicate<Account> tester)
   {
      for (Account account: accounts)
         if (tester.test(account))
            account.print();
   }
}
```

Listing 9 creates an array-based list of accounts with positive, zero, and regative balances. It then demonstrates `Predicate<T>` by invoking `pringAccount()` with lambdas for printing out all accounts, only those accounts with negative balances, and only those accounts whose IDs are greater than 2000 and less than 5000.

Consider lambda expresison `account -> true`. The compiler verifies that the lambda matches `Predicate<T>`'s `boolean test(T)` method, which it does--the lambda presents a single parameter (account) and its body always returns a Boolean value (true). For this lambda, `test()` is implemented to execute `return true`.

Compiling Listing 9 and run the application. You should observe the following output:
```
Account: [1000], Balance: [200]
Account: [2000], Balance: [-500]
Account: [3000], Balance: [0]
Account: [4000], Balance: [-80]
Account: [5000], Balance: [1000]
Account: [2000], Balance: [-500]
Account: [4000], Balance: [-80]
Account: [3000], Balance: [0]
Account: [4000], Balance: [-80]
```

`Predicate<T>` is just one of java.util.function's various predefinded functional interfaces. Another example is `Consumer<T>`, which represents an operation that accepts a single argument and returns no result. Unlike `Predicate<T>`, `Consumer<T>` is expected to operate via side-effects. In other words, it modifies its argument in some way.

`Comsumer<T>`'s `void accept(T t)` method executes an operation on its argument (t). When appearing in the context of this functional interface, a lambda must conform to the `accept()` method's solitary parameter and return type. Listing 10 presents an example that demonstrates `Comsumer<T>` along with `Predicate<T>`.

### Listing 10. LambdaDemo
```java
@Data   
class Account {
    private int id, balance;

    void print() {
        System.out.printf("Account: [%d], Balance: [%d]%n", id, balance);
    }
}

public class LambdaDemo {
    static List<Account> accounts;

    public static void main(String[] args) {
        accounts = new ArrayList<>();
        accounts.add(new Account(1000, 200));
        accounts.add(new Account(2000, -500));
        accounts.add(new Account(3000, 0));
        accounts.add(new Account(4000, -80));
        accounts.add(new Account(5000, 1000));
    
        // Deposit enough money in accounts with negative balances so that they
        // end up with zero balances (and are no longer overdrawn).
        adjustAccount(account -> account.getBalance() < 0, 
                      account -> account.deposit(-account.getBalance()));
    }

    static void adjustAccounts(Prediate<Account> tester, Comsumer<Account> adjuster) {
        for (Account acount : accounts) {
            if (tester.test(account)) {
                adjuster.accept(account);
                account.print();
            }

        }
    }
}
```

Listing 10 continues on from the previous example by introducing an `adjustAccounts()` method that addresses overdrawn accounts by depositing enouth money to give them zero balances. `adjustAccounts()` takes two lambda arguments, which must comform to `Predicate<T>`'s and `Comsumer<T>`'s abstract method parameter lists and return types.

The compiler determines that the lambda arguments passed to `adjustAccounts()` are correct. The `test()` method is implemented to take an `Account acount` parameter and execute `return account.getBalance() < 0;`. Similarly, `accept()` is implemented to take the same parameter and execute `account.deposit(account.getBalance());`.

Compile Listing 10 and run the application. You should observe the following output:
```
Account: [2000], Balance: [0]
Account: [4000], Balance: [0]
```

> Primitive specializations of predefined functional interfaces
> java.util.function includes primitive specializations of various functional interfaces. For example, DoubleConsumer is a primitive specialization of Consumer. Each primitive specialization funcitonal interface exists for performance reasons, to avoid unnecessary object creation and method calls when the inputs or outputs are primitive type-based values.

## In conclusion
In this tutorial I've introduced you to programming with lambda expressions. I started with a high-level overview, then offered in-depth introductions to the core features and techniques associated with lambdas: target type, scopes, local variables, the this and super keyword, and exceptions.

While lambda hava done much to simplify and modernize Java programming, in some cases their usage still results in unnecessary clutter.

