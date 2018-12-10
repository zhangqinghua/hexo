---
title: Java8 Lambda

categories:
- Java

date: 2018-05-24
---

Java8 的一个大亮点是引入 Lambda 表达式，使用它设计的代码会更加简洁。当开发者在编写 Lambda 表达式时，也会随之被编译成一个函数式接口。

## 基础语法

Java8 中引入了一个新得操作符 `->` ，该操作符称为箭头操作符或 Lambda 操作符。箭头操作符将 Lambda 表达式拆分为两部分：
- 左侧：Lambda 表达式得参数列表
- 右侧：Lambda 表达式中所需执行得功能，即 Lambda 体 

1. 语法格式一：无参数，无返回值
    ```java
    () -> System.out.println("HelloWorld")
    ```
1. 语法格式二
    有一个参数，并且无返回值。
    ```java
    (x) -> System.out.println(x)
    ```
1. 语法格式三
    若只有一个参数，小括号可以省略不写。
    ```java
    x -> System.out.println(x)
    ```
1. 语法格式四
    有两个以上的参数，有返回值，并且 Lambda 体中有多条语句。
```java
Comparator<integer> com = (x, y) -> {
    System.out.println("函数式接口");
    return Integer.compare(x, y);
};
```
1. 语法格式五
    若 Lambda 体中只有一条语句，`return` 和大括号都可以省略不写。
    ```java
    Comparator<Integer> com = (x, y) -> Integer.compare(x, y)
    ```
1. 语法格式六
    Lambda 表达式得参数列表得数据类型可以省略不写，因为 JVM 编译器通过上下文推断出数据类型，即**类型推断**。
    ```java
    (Integer x, Integer y) -> Integer.compare(x, y)
    ```

## 函数式接口

接口中只有一个抽象方法的接口，称为函数式接口。可以使用`@FunctionInterface`修饰检查是否函数式接口。

```java
@FunctionInterface
public interface MyFun {
    Integer getValue(Integer num);
}
```

## 实例

1. `Hello World`
    首先在`main`方法的上面声明了一个接口`HelloWorld`，在`main`方法中实现了这个接口，随后调用了接口的唯一方法。 
    ```java
    public class LambdaHelloWorld {
        interface HelloWorld {
            String hello(String name);
        }

        public static void main(String[] args) {       
            HelloWorld helloWorld = (String name) -> { return "Hello " + name; };
            System.out.println(helloWorld.hello("Joe"));
        }
    }
    ```
1. 访问局部变量和成员变量
    在这个例子中演示了如何在 Lambda 表达式中访问局部变量和成员变量，请注意`Runnable`接口的用法。 
    ```java
    public class LambdaVariableAccess {
        public String wildAnimal = "Lion";

        public static void main(String[] arg) {
            new LambdaVariableAccess().lambdaExpression();
        }

        public void lambdaExpression() {
            String domesticAnimal = "Dog";

            new Thread (() -> {
                System.out.println("Class Level: " + this.wildAnimal);
                System.out.println("Method Level: " + domesticAnimal);
            }).start();       
        }
    }
    ```
1. 方法传递
    这个例子比前面的更高级一些。对于`Circle`接口，有两个不同的实现。这些实现本身作为参数传递到了另外一个方法中。
    ```java
    public class LambdaFunctionArgument {
        interface Circle {
            double get(double radius);
        }

        public double circleOperation(double radius, Circle c) {
            return c.get(radius);
        }

        public static void main(String args[]) {
            LambdaFunctionArgument reference = new LambdaFunctionArgument();
            Circle circleArea = (r) - >Math.PI * r * r;
            Circle circleCircumference = (r) - >2 * Math.PI * r;

            double area = reference.circleOperation(10, circleArea);
            double circumference = reference.circleOperation(10, circleCircumference);

            System.out.println("Area: " + area + " . Circumference: " + circumference);
        }
    }
    ```
1. Lambda 表达式的初始化
    ```java
    import java.util.concurrent.Callable;

    public class LambdaInitialization {
        public static void main(String args[]) throws Exception {
            Callable[] animals = new Callable[] { 
                () - >"Lion",
                () - >"Crocodile"
            };
            System.out.println(animals[0].call());
        }
    }
    ```
1. Lambda 表达式进行排序
    这个例子的关键是将方法的引用传递给了另一个方法来进行调用。`Comparator`接口的具体实现作为参数传递给了`Arrays.sort`方法。 
    ```java
    public class LambdaExpressionSort {
        public static void main(String[] ar) {
            Animal[] animalArr = {
                new Animal("Lion"),
                new Animal("Crocodile"),
                new Animal("Tiger"),
                new Animal("Elephant")
            };

            System.out.println("Before Sort: " + Arrays.toString(animalArr));
            Arrays.sort(animalArr, Animal: :animalCompare);
            System.out.println("After Sort: " + Arrays.toString(animalArr));
        }
    }

    class Animal {
        String name;

        Animal(String name) {
            this.name = name;
        }

        public static int animalCompare(Animal a1, Animal a2) {
            return a1.name.compareTo(a2.name);
        }

        public String toString() {
            return name;
        }
    }
    ```
1. 条件判断（`Predicates`）和 Lambda 表达式
    件判断和 Lambda 之间结合的非常好。我们可以使用 Lambda 来实现`Predicate`接口，并且将其传递给具体的方法，作为方法的判断条件。 
    ```java
    public class LambdaPredicate {

        public static int add(List numList, Predicate predicate) {
            int sum = 0;
            for (int number: numList) {
                if (predicate.test(number)) {
                    sum += number;
                }
            }
            return sum;
        }

        public static void main(String args[]) {

            List numList = new ArrayList();

            numList.add(new Integer(10));
            numList.add(new Integer(20));
            numList.add(new Integer(30));
            numList.add(new Integer(40));
            numList.add(new Integer(50));

            System.out.println("Add Everything: " + add(numList, n - >true));
            System.out.println("Add Nothing: " + add(numList, n - >false));
            System.out.println("Add Less Than 25: " + add(numList, n - >n < 25));
            System.out.println("Add 3 Multiples: " + add(numList, n - >n % 3 == 0));

            // 校验租客数据
            if (!CollectionUtils.exists(dto.getRentalRenters(), r -> {
                RentalRenterDTO renter = (RentalRenterDTO) r;
                if (renter.getRenterType() == RenterType.major) {
                    dto.setRenterName(renter.getName());
                    dto.setRenterMobile(renter.getMobile());
                    dto.setRenterSex(renter.getSex());
                }
                return renter.getRenterType() == RenterType.major;
            })) throw new ServiceException("租客数据非法");
        }
    }
    ```

1. 方法内部方法
    ```java
    // JDK7
    class SubFunction {
        private String drawTribleX(){

            // *** move trible(t) inside drawTribleX() ***
            class Trible {
                private String trible(String t){
                    return t + t + t;
                }
            }

            return new Trible().trible("X");
        }
        public static void main(String[] args){
            SubFunction o = new SubFunction();
            System.out.println(o.drawTribleX());
        }
    }

    // JDK8
    Function<String, String> trible = s -> s+s+s;
    System.out.println(trible.apply("X"));           // prints XXX
    ```