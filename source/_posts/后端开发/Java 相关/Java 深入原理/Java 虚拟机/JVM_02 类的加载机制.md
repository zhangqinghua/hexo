---
title: JVM_02 类的加载机制

categories:
- Java 虚拟机

date: 2020-01-01 00:00:02
---
类的加载指的虚拟机将 .class 文件加载到内存中，并对数据进行校验，解析和初始化，最终形成可以被虚拟机直接使用的 Java 类型。

类的加载的最终产品是位于堆区中的 `Class` 对象，`Class` 对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。

![](https://img2020.cnblogs.com/blog/1846149/202004/1846149-20200401105701873-414824729.png)

类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM 规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了 .class 文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（`LinkageError` 错误），如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

加载 .class 文件的方式有：
1. 从本地系统中直接加载
1. 通过网络下载 .class 文件
1. 从 zip，jar 等归档文件中加载 .class 文件
1. 从专有数据库中提取 .class 文件
1. 将 Java 源文件动态编译为 .class 文件

## 类的加载
类的整个生命周期包括加载、验证、准备、解析、初始化、使用和卸载 7 个阶段，其中验证、准备、解析这 3 个部分统称为连接。

加载、验证、准备、初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持 Java 语言的运行时绑定（也成为动态绑定或晚期绑定）。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

> 解析阶段的顺序还得再研究研究。。。

![](https://images2015.cnblogs.com/blog/331425/201606/331425-20160621125943209-1443333281.png)

#### 加载
加载（查找并加载类的二进制数据）是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：
1. 通过一个类的全限定名来获取其定义的二进制字节流。
1. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
1. 在 Java 堆中生成一个代表这个类的 `java.lang.Class` 对象，作为对方法区中这些数据的访问入口。

相对于类加载的其他阶段而言，加载阶段（准确地说，是加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载。

加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在 Java 堆中也创建一个 `java.lang.Class` 类的对象，这样便可以通过该对象访问方法区中的这些数据。

#### 连接
连接阶段可以分成 3 个步骤：**验证**、**准备**、**解析**。

**验证**阶段用于确保被加载的类的正确性。

**验证**是连接阶段的第一步，这一阶段的目的是为了确保 `Class` 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成 4 个阶段的检验动作：
1. **文件格式验证**：验证字节流是否符合 `Class` 文件格式的规范；例如：是否以 `0xCAFEBABE` 开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
1. **元数据验证**：对字节码描述的信息进行语义分析（注意：对比 javac 编译阶段的语义分析），以保证其描述的信息符合 Java 语言规范的要求；例如：这个类是否有父类，除了 `java.lang.Object` 之外。
1. **字节码验证**：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
1. **符号引用验证**：确保解析动作能正确执行。

**验证**阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用 `-Xverify:none` 参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间（SpringBoot 项目测试 17s 可以缩短到 16s）。

**准备**阶段用于为类的静态变量分配内存，并将其初始化为默认值。

**准备**阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：
1. 这时候进行内存分配的仅包括类变量（`static`），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在 Java 堆中。
1. 这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、`null`、`false`等），而不是被在 Java 代码中被显式地赋予的值。

    假设一个类变量的定义为：`public static int value = 3`;

    那么变量 `value` 在准备阶段过后的初始值为 0，而不是 3，因为这时候尚未开始执行任何 Java 方法，而把 `value` 赋值为 3 的 `putstatic` 指令是在程序编译后，存放于类构造器 `<clinit>()` 方法之中的，所以把 `value` 赋值为 3 的动作将在初始化阶段才会执行。

1. 如果类字段的字段属性表中存在 ConstantValue 属性，即同时被 `final` 和 `static` 修饰，那么在准备阶段变量 `value` 就会被初始化为 ConstValue 属性所指定的值。

    假设上面的类变量value被定义为： `public static final int value = 3`。

    编译时 Javac 将会为 `value` 生成 ConstantValue 属性，在准备阶段虚拟机就会根据 ConstantValue 的设置将 `value` 赋值为 3。我们可以理解为 `static final` 常量在编译期就将其结果放入了调用它的类的常量池中。

**解析**阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，**解析**阶段会伴随着 JVM 执行完初始化之后再开始。

**解析**动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。符号引用就是一组符号来描述目标，可以是任何字面量。

#### 初始化
初始化就是为类的静态变量赋予正确的初始值，执行静态代码块。初始化阶段才真正开始执行类中定义的代码。

初始化会执行类构造器方法 `clinit()`，此方法不需要定义，由 javac 编译器自动收集类中的所有静态变量的赋值动作和静态代码块中的语句合并而来。如果类没有定义静态变量或静态代码块，则 javac 不生成 `clinit()`。

在 Java 中对类变量进行初始值设定有两种方式：
1. 声明类变量是指定初始值。
1. 使用静态代码块为类变量指定初始值。

#### 使用
程序之间的相互调用。

#### 卸载
即销毁一个对象，一般情况下中有垃圾回收器完成。代码层面的销毁只是将引用置为 null。

## 类的加载补充
#### 类加载的触发
1. 启动类、JVM 自动加载
1. 调用类的静态变量/方法
1. 创建类的实例
1. 动态加载（`Class.forName()` 或 `ClassLoader.loadClass()`）
1. 被加载的类的父类

> 通过数组定义来引用类，或者访问常量，不会触发此类的初始化。

#### 类加载的步骤
这里需要分情况来考虑。

类没有加载，父类没有加载：
1. 判断类还没被加载
1. 加载类（加载、验证、准备、解析）
1. 判断父类没有被加载，加载父类
1. 初始化类（赋值静态变量，执行静态代码块）

类没有加载，父类已经加载：
1. 判断子类还没被加载
1. 加载子类（加载、验证、准备、解析）
1. 初始化子类（赋值静态变量，执行静态代码块）

类已经加载：
1. 发现类已经加载了
1. 没有了

> 在使用类时，可能会碰到两种情况，一是类还没有加载到内存中，这时候需要走加载、验证、准备、解析、初始化阶段；二是类已经加载到内存中了，但是还没初始化，这是直接走初始化阶段就可以了。

#### 类加载的顺序
类加载的顺序或者调用顺序要分情况来讨论。

如果是调用静态变量/静态方法：
1. 加载父类的静态变量/静态代码块
    先递归地加载父类的静态变量/静态代码块。同一个类里的静态变量/静态代码块，按写代码的顺序加载。
1. 加载本类的静态变量/静态代码块
    这里的本类是指被调用的静态变量/静态方法所在的类。如果用子类调用父类的静态变量/静态方法，则本类是父类，而子类不加载。

如果是调用成员变量/成员方法：
1. 加载父类的静态变量/静态代码块
1. 加载本类的静态变量/静态代码块
1. 加载父类的成员变量/代码块
1. 加载父类的构造方法
1. 加载本类的成员变量/代码块
1. 加载本类的构造方法
1. 加载成员变量/成员方法

> 一个类的静态变量/静态代码块只加载一次，成员变量/代码块在每次创建实例时都加载。

#### 类的销毁
只能等到虚拟机关闭。？？

## 类的加载器
类的加载阶段（查找并加载类的二进制文件）放到了虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。这个动作的代码模块称为“类加载器”。

类加载器虽然说只用于一个类的加载动作，但是它在 Java 程序中起到的作用却远远不限于类加载阶段。对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在 Java 虚拟机中的唯一性。每一个类加载器都拥有一个独立的类名称空间。

看下面一个小例子：

```java
public class ClassLoaderTest {

    public static void main(String[] args) {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        System.out.println(loader);
        System.out.println(loader.getParent());
        System.out.println(loader.getParent().getParent());
    }
}
```

运行后，输出结果：

```
sun.misc.Launcher$AppClassLoader@64fef26a
sun.misc.Launcher$ExtClassLoader@1ddd40f3
null
```

从上面的结果可以看出，并没有获取到 `ExtClassLoader` 的父 Loader，原因是 Bootstrap Loader（引导类加载器）是用 C 语言实现的，找不到一个确定的返回父 Loader 的方式，于是就返回 `null`。

这几种类加载器的层次关系如下图所示：

![](https://images2015.cnblogs.com/blog/331425/201606/331425-20160621125944459-1013316302.jpg)

> 注意：这里父类加载器并不是通过继承关系来实现的，而是采用组合实现的。

站在 Java 虚拟机的角度来讲，只存在两种不同的类加载器：启动类加载器：它使用 C++ 实现（这里仅限于 Hotspot，也就是 JDK1.5 之后默认的虚拟机，有很多其他的虚拟机是用 Java 语言实现的），是虚拟机自身的一部分；所有其他的类加载器：这些类加载器都由 Java 语言实现，独立于虚拟机之外，并且全部继承自抽象类 `java.lang.ClassLoader`，这些类加载器需要由启动类加载器加载到内存中之后才能去加载其他的类。

站在 Java 开发人员的角度来看，类加载器可以大致划分为以下三类：
1. **启动类加载器**：Bootstrap ClassLoader，负责加载存放在 `JDK\jre\lib`（JDK代表JDK的安装目录，下同）下，或被 `-Xbootclasspath` 参数指定的路径中的，并且能被虚拟机识别的类库（如 rt.jar，所有的 java.* 开头的类均被 Bootstrap ClassLoader 加载）。启动类加载器是无法被 Java 程序直接引用的。
1. **扩展类加载器**：Extension ClassLoader，该加载器由 `sun.misc.Launcher$ExtClassLoader` 实现，它负责加载 `JDK\jre\lib\ext` 目录中，或者由 java.ext.dirs 系统变量指定的路径中的所有类库（如 javax.* 开头的类），开发者可以直接使用扩展类加载器。
1. **应用程序类加载器**：Application ClassLoader，该类加载器由 `sun.misc.Launcher$AppClassLoader` 来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入自定义的类加载器。因为 JVM 自带的 ClassLoader 只是懂得从本地文件系统加载标准的 Java class 文件，因此如果编写了自己的 ClassLoader，便可以做到如下几点：
1. 在执行非置信代码之前，自动验证数字签名。
1. 动态地创建符合用户特定需要的定制化构建类。
1. 从特定的场所取得java class，例如数据库中和网络中。

JVM类加载机制：
1. 全盘负责，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
1. 父类委托，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
1. 缓存机制，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效

## 双亲委派模型
双亲委派模型就是，如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

双亲委派模型意义：
1. 系统类防止内存中出现多份同样的字节码（例如用户就可以自定义一个一摸一样的 `String` 类，但是不会被加载）。
1. 保证 Java 程序安全稳定运行（确保 Java 的核心类不被修改）。

ClassLoader 源码分析：
```java
public Class<?> loadClass(String name)throws ClassNotFoundException {
    return loadClass(name, false);
}

protected synchronized Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException {
    // 首先判断该类型是否已经被加载
    Class c = findLoadedClass(name);
    if (c == null) {
        // 如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
        try {
            if (parent != null) {
                // 如果存在父类加载器，就委派给父类加载器加载
                c = parent.loadClass(name, false);
            } else {
                // 如果不存在父类加载器，就检查是否是由启动类加载器加载的类，通过调用本地方法native Class findBootstrapClass(String name)
                c = findBootstrapClass0(name);
            }
        } catch (ClassNotFoundException e) {
            // 如果父类加载器和启动类加载器都不能完成加载任务，才调用自身的加载功能
            c = findClass(name);
        }
    }
    if (resolve) {
        resolveClass(c);
    }
    return c;
}
```

## 自定义类加载器
通常情况下，我们都是直接使用系统类加载器。但是，有的时候，我们也需要自定义类加载器。比如应用是通过网络来传输 Java 类的字节码，为保证安全性，这些字节码经过了加密处理，这时系统类加载器就无法对其进行加载，这样则需要自定义类加载器来实现。自定义类加载器一般都是继承自 `ClassLoader` 类，从上面对 `loadClass` 方法来分析来看，我们只需要重写 `findClass` 方法即可。下面我们通过一个示例来演示自定义类加载器的流程：

```java
public class MyClassLoader extends ClassLoader {

    private String root;

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] loadClassData(String className) {
        String fileName = root + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
        try {
            InputStream ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args)  {

        MyClassLoader classLoader = new MyClassLoader();
        classLoader.setRoot("E:\\temp");

        Class<?> testClass = null;
        try {
            testClass = classLoader.loadClass("com.neo.classloader.Test2");
            Object object = testClass.newInstance();
            System.out.println(object.getClass().getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```

自定义类加载器的核心在于对字节码文件的获取，如果是加密的字节码则需要在该类中对文件进行解密。由于这里只是演示，我并未对 class 文件进行加密，因此没有解密的过程。这里有几点需要注意：

1. 这里传递的文件名需要是类的全限定性名称，即 `com.paddx.test.classloading.Test` 格式的，因为 defineClass 方法是按这种格式进行处理的。
1. 最好不要重写 `loadClass` 方法，因为这样容易破坏双亲委托模式。
1. 这类 `Test` 类本身可以被 `AppClassLoader` 类加载，因此我们不能把 `com/paddx/test/classloading/Test.class` 放在类路径下。否则，由于双亲委托机制的存在，会直接导致该类由 `AppClassLoader` 加载，而不会通过我们自定义类加载器来加载。

## 题外：关于符号引用和直接引用
在 JVM 中，类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载、验证、准备、解析、初始化、使用和卸载 7 个阶段。而解析阶段即是 JVM 将常量池内的符号引用替换为直接引用的过程。

符号引用是以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可，符号引用的目标不一定要加载到内存中。

在Java中，一个 Java 类将会编译成一个 class 文件。在编译时，Java 类并不知道所引用的类的实际地址，因此只能使用符号引用来代替。比如 `org.simple.People` 类引用了 `org.simple.Language` 类，在编译时 `People` 类并不知道 `Language` 类的实际内存地址，因此只能使用符号 `org.simple.Language`（假设是这个，当然实际中是由类似于 CONSTANT_Class_info 的常量来表示的）来表示 `Language` 类的地址。

直接引用是一个直接指向目标的指针，如果有了直接引用，那引用的目标必定已经被加载入内存中了。

## 符号引用如何解析
太复杂了...

## 内存分配

|变量类型|分配时间|销毁时间|基本数据类型|引用数据类型|
| :- |
|常量|验证|程序退出|
|静态变量|验证、初始化|程序退出||方法区|
|实例变量|创建实例|对象销毁||堆|
|局部变量|调用方法|方法结束|栈|堆|

## 问题
1. 什么是类的加载机制
.class 内存 方法区 class对象 访问入口

1. 类加载的有哪些步骤
加载、验证、准备、解析、初始化

1. 这些步骤的顺序哪些是固定的，哪些是不固定的，为什么不固定？
加载、验证、准备、初始化 > 解析
？？

1. 将一下有哪些类加载器
启动类加载器 扩展类加载器 应用程序类加载器

1. 如果实现自定义类加载器

1. 为什么需要自定义类加载器

1. 讲一下双亲委派机制
请求父加载器，再回来。

1. 为什么使用双亲委派机制
多份同样的字节码 篡改

1. 符号引用如何解析
??

1. 解析调用 与 分派调用
??