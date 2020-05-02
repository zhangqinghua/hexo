---
title: Chapter 1.1 基础

categories:
- Interview

date: 2020-04-28 00:00:11
---
#### ==、equals、hashCode 的区别
-  == 是比较操作符
    - 基本数据类型：比较值
    - 引用类型（类、接口、数组）：比较的是他们在内存中的存放地址
-  equals 是 Object 的一个实例方法，比较两个对象的内容是否相同
-  hashCode 是 Object 的 native 方法（就是一个 Java 调用非 Java 代码的接口）, 获取对象的哈希值，用于确定该对象在哈希表中的索引位置，它实际上是一个 int 整数

#### int、char、long 各占多少字节数
- 1 字节： byte boolean
- 2 字节： short char
- 4 字节： int float
- 8 字节： long double

#### int 与 Integer 的区别
- 类型：int 是 Java 的一种基本数据类型，Integer 是 int 的包装类
- 存储：int 直接存储数据值。Integer 是对象的引用，当 new 一个 Integer 时，实际上是生成一个指针指向此对象
- 默认值：int 默认值是 0，Integer 默认值是 null，所以 Integer 必须实例化后才能使用
- 为什么需要包装类

#### final、finally、finalize 的区别
- final 可以用来修饰类，方法和变量（成员变量或局部变量）
    - 修饰类的时，表明该类不能被其他类所继承
    - 修饰方法时，方法锁定，以防止继承类对其进行更改
    - 修饰成员变量时表示常量，只能被赋值一次，赋值后其值不再改变
- 　finally 作为异常处理的一部分，它只能用在 try/catch 语句中，并且附带一个语句块，表示这段语句最终一定会被执行（不管有没有抛出异常），经常被用在需要释放资源的情况下
- finalize() 是在 Object 里定义的，也就是说每一个对象都有这么个方法。这个方法在 gc 启动，该对象被回收的时候被调用

## Object
#### Object 类的 equals 和 hashCode 方法重写，为什么

## String
#### 说一下 Java 中的 String 的理解

#### Spring、StringBuffer、StringBuilder 的区别
- 类型
    - String 字符串常量，是不可变的对象。因此在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象
    - StringBuffer、StringBuilder 是字符串变量
- 性能：
    - 大部分情况下，StringBuffer 优于 String：String s = a + "b" 生成多个对象
    - 特别情况下，String 优于 StringBuffer：String s = "a" + "b" 被 JVM 优化成 String s = "ab"
- 线程安全：StringBuffer 线程安全 StringBuilder 非线程安全

#### String 为什么要设计成不可变的

#### String 转换成 Integer 的方式以及原理
- integer.parseInt(string str) 方法调用 Integer 内部的 parseInt(string str,10) 方法,默认基数为 10，parseInt 内部首先判断字符串是否包含符号（- 或者 +），则对相应的 negative 和 limit 进行赋值，然后再循环字符串，对单个 char 进行数值计算 Character.digit(char ch, int radix)。在这个方法中，函数肯定进入到 0 - 9 字符的判断（相对于 string 转换到 int），否则会抛出异常，数字就是如上面进行拼接然后生成的 int 类型数值
```java
public static int parseInt(String s) throws NumberFormatException {
    //内部默认调用parseInt(String s, int radix)基数设置为10
    return parseInt(s,10);
}

public static int parseInt(String s, int radix)
                throws NumberFormatException
    {
        /*
         * WARNING: This method may be invoked early during VM initialization
         * before IntegerCache is initialized. Care must be taken to not use
         * the valueOf method.
         */
        //判断字符是否为null
        if (s == null) {
            throw new NumberFormatException("s == null");
        }
        //基数是否小于最小基数
        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }
        //基数是否大于最大基数
        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        int result = 0;
        //是否时负数
        boolean negative = false;
        //char字符数组下标和长度
        int i = 0, len = s.length();
        //限制
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;
        //判断字符长度是否大于0，否则抛出异常
        if (len > 0) {
            //第一个字符是否是符号
            char firstChar = s.charAt(0);
            //根据ascii码表看出加号(43)和负号(45)对应的
            //十进制数小于‘0’(48)的
            if (firstChar < '0') { // Possible leading "+" or "-"
                //是负号
                if (firstChar == '-') {
                    //负号属性设置为true
                    negative = true;
                    limit = Integer.MIN_VALUE;
                }
                //不是负号也不是加号则抛出异常
                else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);
                //如果有符号（加号或者减号）且字符串长度为1，则抛出异常
                if (len == 1) // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                //此方法为确定数字的的十进制值
                digit = Character.digit(s.charAt(i++),radix);
                //小于0，则为非数值字符串
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                //result第一次为0，第一次肯定为true
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                //result乘以基数（10）为得到位置
                //例如第一次的result为-1，第二次乘以10后为-10
                //下面再-=digit（例如：1）则得到-11
                //以此类推
                result *= radix;
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                //第一次result为0 -=digit则为负值的该digit
                result -= digit;
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        //根据上面得到的是否负数，返回相应的值
        return negative ? result : -result;
    }

public static int digit(int codePoint, int radix) {
        //基数必须再最大和最小基数之间
        if (radix < MIN_RADIX || radix > MAX_RADIX) {
            return -1;
        }
        if (codePoint < 128) {
            // Optimized for ASCII
            int result = -1;
            //字符在0-9字符之间
            if ('0' <= codePoint && codePoint <= '9') {
                result = codePoint - '0';
            }
            //字符在a-z之间
            else if ('a' <= codePoint && codePoint <= 'z') {
                result = 10 + (codePoint - 'a');
            }
            //字符在A-Z之间
            else if ('A' <= codePoint && codePoint <= 'Z') {
                result = 10 + (codePoint - 'A');
            }
            //通过判断result和基数大小，输出对应值
            //通过我们parseInt对应的基数值为10，
            //所以，只能在第一个判断（字符在0-9字符之间）
            //中得到result值 否则后续程序会抛出异常
            return result < radix ? result : -1;
        }
        return digitImpl(codePoint, radix);
    }
```

## 内部类
#### 什么是内部类
将一个类定义在另一个类里面或者一个方法里面，这样的类称为内部类

#### 有哪些内部类
- 成员内部类：成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）
- 局部内部类：局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内
- 匿名内部类：没有名字的内部类
- 静态内部类：指被声明为static的内部类，他可以不依赖内部类而实例，而通常的内部类需要实例化外部类，从而实例化。静态内部类不可以有与外部类有相同的类名

#### 内部类有什么用
- 每个内部类都能独立的继承一个接口的实现，所以无论外部类是否已经继承了某个(接口的)实现，对于内部类都没有影响。内部类使得多继承的解决方案变得完整
- 方便将存在一定逻辑关系的类组织在一起，又可以对外界隐藏。　　
- 方便编写事件驱动程序 　　
- 方便编写线程代码

#### 静态属性和静态方法是否可以被继承？是否可以被重写？原因是什么
- 父类的静态属性和方法可以被子类继承
    ```java
    public class One {
        // 静态属性和静态方法是否可以被继承？
        public static String one_1 = "one";
        public static void oneFn() {
            System.out.println("oneFn");
        }
    }

    public class Two extends One{
        // 空
    }

    public class MyTest {
        // 静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？
        public static void main(String[] args) {
            One one = new Two();
            one.oneFn();
            String one_1 = One.one_1;
            System.out.println("One.one_1>>>>>>>"+one_1);
            String one_12 = one.one_1;
            System.out.println("one.one_1>>>>>>>"+one_12);
        }
    }
    // 打印结果如下
    oneFn
    One.one_1>>>>>>>one
    one.one_1>>>>>>>one
    ```
- 当父类的引用指向子类时，使用对象调用静态方法或者静态变量，是调用的父类中的方法或者变量。并没有被子类改写。所以可以认为不可以被子类重写
    ```java
    public class One {
        // 静态属性和静态方法是否可以被重写？以及原因？
        public static String one_1 = "one";
        public static void oneFn() {
            System.out.println("oneFn");
        }
    }

    public class Two extends One {

        public static String one_1 = "two";

        public static void oneFn() {

            System.out.println("TwoFn");
        }
    }
    public class MyTest {
        // 静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？
        public static void main(String[] args) {
            One one = new Two();
            one.oneFn();
            String one_1 = One.one_1;
            System.out.println("One.one_1>>>>>>>"+one_1);
            String one_12 = one.one_1;
            System.out.println("one.one_1>>>>>>>"+one_12);
        }
    }
    // 打印结果如下
    // oneFn
    // One.one_1>>>>>>>one
    // one.one_1>>>>>>>one
    ```
- 原因：static 修饰函数/变量时，其实是全局函数/变量，它只是因为 Java 强调对象的要挂，它与任何类都没有关系。靠这个类的好处就是这个类的成员函数调用 static 方法不用带类名

#### 闭包和局部内部类的区别
- 局部内部类就像是方法里面的一个局部变量一样，是不能有 public、protected、private 以及 static 修饰符的
- 闭包（Closure）是一种能被调用的对象，它保存了创建它的作用域的信息。Java 并不能显式地支持闭包，但是在 Java 中，闭包可以通过“接口 + 内部类”来实现
    ```java
    class Food{
        public static final String name = "Food";
        private static int num = 20;
        public Food() {
            System.out.println("Delicious Food");
        }
    
        public Active getEat() {
            return new EatActive();
        }
        private class EatActive implements Active {
            @Override
            public void eat() {
                if (num == 0) {
                    System.out.println("吃货，已经吃没了");
                }
                num --;
                System.out.println("吃货，你吃了一份了");
            }
        }
    
        public void currentNum() {
            System.out.println("还剩:"+num+"份");
        }
    }
    
    interface Active{
        void eat();
    }
    ```


## Java 特性
#### Java 中三大特性是什么

#### 什么是封装

#### 什么是继承

#### 什么是多态

#### 谈谈对 Java 多态的理解
- 面向对象的三大基本特征是：封装、继承、多态
- 多态是指程序中定义的引用变量（即一个引用变量倒底会指向哪个类的实例对象）在编程时并不确定，而是在程序运行期间才确定
- 举例：有一个接口和 2 个实现类，当我们调用这个接口的方法时，并不知道调用的是哪个是实现类的方法，要在程序运行时才能确定

#### Java 中实现多态的机制是什么

#### 父类的静态方法能否被子类重写
- 首先来说，一个方法如果被static声明后，这个方法就和这个类的实例对象脱离了关系，应该用类名点方法调用
- 如果父类中存在一个被static修饰的public方法，在子类中也存在被static修饰的public方法，这两个方法名字和方法参数相同，那么声明父类引用指向子类对象时，调用这个方法，其实是调用的父类的方法

## 接口和抽象类
#### 什么是接口

#### 什么是抽象类

#### 接口有什么用

#### 抽象类有什么用

#### 接口和抽象类的相同点和不同点
- 相同点：抽象类和接口都不能直接实例化，如果要实例化，抽象类变量必须指向实现所有抽象方法的子类对象，接口变量必须指向实现所有接口方法的类对象
- 不同点：
    - 抽象类要被子类继承，接口要被类实现
    - 接口可继承接口，并可多继承接口，但类只能单根继承
    - 接口只能做方法申明，抽象类中可以做方法申明，也可以做方法实现
    - 接口里定义的变量只能是公共的静态的常量，抽象类中的变量是普通变量

#### 抽象类的意义
- 因为抽象类不能实例化对象，所以必须要有子类来实现它之后才能使用。这样就可以把一些具有相同属性和方法的组件进行抽象，这样更有利于代码和程序的维护
- 当又有一个具有相似的组件产生时，只需要实现该抽象类就可以获得该抽象类的那些属性和方法

#### 接口的应用场景
- 类与类之前需要特定的接口进行协调，而不在乎其如何实现
- 作为能够实现特定功能的标识存在，也可以是什么接口方法都没有的纯粹标识
- 要将一组类视为单一的类，而调用者只通过接口来与这组类发生联系。
- 需要实现特定的多项功能，而这些功能之间可能完全没有任何联系

#### 抽象类的应用场景
- 定义了一组接口，但又不想强迫每个实现类都必须实现所有的接口。可以用抽象类定义一组方法体，甚至可以是空方法体，然后由子类选择自己所感兴趣的方法来覆盖
- 些场合下，只靠纯粹的接口不能满足类与类之间的协调，还必需类中表示状态的变量来区别不同的关系。抽象类的中介作用可以很好地满足这一点
- 规范了一组相互协调的方法，其中一些方法是共同的，与状态无关的，可以共享的，无需子类分别实现；而另一些方法却需要各个子类根据自己特定的状态来实现特定的功能

#### 抽象类是否可以没有方法和属性
- 抽象类专用于派生出子类，子类必须实现抽象类所声明的抽象方法，否则，子类仍是抽象类
- 包含抽象方法的类一定是抽象类，但抽象类中的方法不一定是抽象方法
- 抽象类中可以没有抽象方法，但有抽象方法的一定是抽象类。所以，Java中抽象类里面可以没有抽象方法

## 泛型
#### 什么是泛型

#### 泛型有什么用

#### 泛型中 extends 和 super 的区别
- extends 的主要作用是设定类型通配符的上限
- super 与 extends 是完全相反的，其定义的是下界通配符
    List<? super Fruit> 也就是说 List 中存放的都是 Fruit 和它的父类的对象，比如 Food，Object

#### 说一下泛型原理，并举例说明

## 反射
#### 说说你对 Java 反射的理解

## 注解
#### 什么是注解

#### 注解有什么用

#### 怎么声明一个注解

#### 说说你对 Java 注解的理解

## 序列化
#### 什么是序列化，什么又是反序列化
序列化是指将对象转化成一个字节序列，便于存储。反序列化则是将序列化的字节序列还原。

#### 序列化有什么用
 序列化可以实现对象的"持久性”， 所谓持久性就是指对象的生命周期不取决于程序

#### 序列化的方式
总的来说有三种方式来序列化，它们分别是实现 Serializable 接口，实现 Externalizable 接口，实现 Serializable 接口并且添加 writeObject() 和 readObject() 方法。

- 实现 Serializable 接口（隐式序列化）
    这种是最简单的序列化方式，会自动序列化所有非static和 transient关键字修饰的成员变量
    ```java
    class Student implements Serializable{
        private String name;
        private int age;
        public static int QQ = 1234;
        private transient String address = "CHINA";
        
        Student(String name, int age ){
            this.name = name;
            this.age = age;
        }
        public String toString() {
            return "name: " + name + "\n"
                    +"age: " + age + "\n"
                    +"QQ: " + QQ + "\n" 
                    + "address: " + address;
                    
        }
        public void SetAge(int age) {
            this.age = age;
        }
    }
    public class SerializableDemo {
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            //创建可序列化对象
            System.out.println("原来的对象：");
            Student stu = new Student("Ming", 16);
            System.out.println(stu);
            //创建序列化输出流
            ByteArrayOutputStream buff = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(buff);
            //将序列化对象存入缓冲区
            out.writeObject(stu);
            //修改相关值
            Student.QQ = 6666; // 发现打印结果QQ的值被改变
            stu.SetAge(18);   //发现值没有被改变
            //从缓冲区取回被序列化的对象
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(buff.toByteArray()));
            Student newStu = (Student) in.readObject();
            System.out.println("序列化后取出的对象：");
            System.out.println(newStu);
            
        }
    }
    ```
- 实现 Externalizable 接口（显式序列化）
    Externalizable 接口继承自 Serializable, 我们在实现该接口时，必须实现 writeExternal() 和 readExternal() 方法，而且只能通过手动进行序列化，并且两个方法是自动调用的，因此，这个序列化过程是可控的，可以自己选择哪些部分序列化
    ```java
    public class Blip implements Externalizable{
        private int i ;
        private String s;
        public Blip() {}
        public Blip(String x, int a) {
            System.out.println("Blip (String x, int a)");
            s = x;
            i = a;
        }
        public String toString() {
            return s+i;
        }
        @Override
        public void writeExternal(ObjectOutput out) throws IOException {
            // TODO Auto-generated method stub
            System.out.println("Blip.writeExternal");
            out.writeObject(s);
            out.writeInt(i);
        }
        @Override
        public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
            // TODO Auto-generated method stub
            System.out.println("Blip.readExternal");
            s = (String)in.readObject();
            i = in.readInt();
        }
        public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
            System.out.println("Constructing objects");
            Blip b = new Blip("A Stirng", 47);
            System.out.println(b);
            ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream("F://Demo//file1.txt"));
            System.out.println("保存对象");
            o.writeObject(b);
            o.close();
            //获得对象
            System.out.println("获取对象");
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("F://Demo//file1.txt"));
            System.out.println("Recovering b");
            b = (Blip)in.readObject();
            System.out.println(b);
        }
    
    }
    ```
- 实现 Serializable 接口 + 添加 writeObject() 和 readObject() 方法（显 + 隐序列化）
    如果想将方式一和方式二的优点都用到的话，可以采用方式三， 先实现 Serializable 接口，并且添加 writeObject() 和 readObject() 方法。注意这里是添加，不是重写或者覆盖。但是添加的这两个方法必须有相应的格式
    ```java
    public class SerDemo implements Serializable{
        public transient int age = 23;
        public String name ;
        public SerDemo(){
            System.out.println("默认构造器。。。");
        }
        public SerDemo(String name) {
            this.name = name;
        }
        private  void writeObject(ObjectOutputStream stream) throws IOException {
            stream.defaultWriteObject();
            stream.writeInt(age);
        }
        private void readObject(ObjectInputStream stream) throws ClassNotFoundException, IOException {
            stream.defaultReadObject();
            age = stream.readInt();
        }
        
        public String toString() {
            return "年龄" + age + "  " + name; 
        }
        public static void main(String[] args) throws IOException, ClassNotFoundException {
            SerDemo stu = new SerDemo("Ming");
            ByteArrayOutputStream bout = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bout);
            out.writeObject(stu);
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bout.toByteArray()));
            SerDemo stu1 = (SerDemo) in.readObject();
            System.out.println(stu1);
        }
    }
    ```

#### 如何将一个 Java 对象序列化到文件里

## 编码
#### 什么是编码
编码是从一种形式或格式转换为另一种形式的过程也称为计算机编程语言的代码简称编码。计算机中存储信息的最小单元是一个字节，即 8 个 bit

#### 讲一下常见的编码方式
常见有的编码方式有ASCII、ISO8859-1、GB2312、GBK、UTF-8、UTF-16等：
- ASCII 码：共有 128 个，用一个字节的低 7 位表示
- ISO8859-1：在 ASCII 码的基础上涵盖了大多数西欧语言字符，仍然是单字节编码，它总共能表示 256 个字符
- GB2312：全称为《信息交换用汉字编码字符集基本集》，它是双字节编码，总的编码范围是A1~F7 A1~A9 符号区 B0~F7 汉字区
- GBK：数字交换用汉字编码字符集，它可能是单字节、双字节或者四字节编码，与GB2312编码兼容
- UTF-8：UTF-8采用一种变长技术，每个编码区域有不同的字码长度，不同的字符可以由1~6个字节组成。如果一个字节，最高位为 0，表示这是一个 ASCII 字符（00~7F）如果一个字节，以 11 开头，连续的 1 的个数暗示这个字符的字节数 
- UTF-16：具体定义了Unicode字符在计算机中的存取方法。采用2字节来表示Unicode转化格式，它是定长的表示方法，不论什么字符都可以用两个字节表示

#### UTF-8 编码中的中文占几个字节？
- 一个 UTF-8 数字占 1 个字节
- 一个 UTF-8 英文字母占 1 个字节
- 少数是汉字每个占用 3 个字节，多数占用 4 个字节