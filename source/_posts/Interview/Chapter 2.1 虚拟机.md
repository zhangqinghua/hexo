---
title: Chapter 2.1 虚拟机

categories:
- Interview

date: 2020-04-28 00:00:21
---
#### 什么是虚拟机（JVM）

## 类加载机制
#### 类指的是什么
类就是指我们的 Java 源代码通过编译之后的 class 文件

#### 类的来源的来源
- 本地磁盘
- 网络下载的 .class 文件
- war、jar 下加载的 .class 文件
- 从专门的数据库中读取的 .class 文件
- 将 Java 源码文件动态编译成的 .class 文件
    - 动态代理，运行期间生成 .class 文件 
    - jsp 转换成为 servlet，会被编译成为 .class 文件

#### 通过什么加载
类加载器

#### 加载到哪里去
JVM

#### 什么是类加载器
类加载器就是根据指定全限定名称将 .class 文件加载到 JVM 内存，转为 Class 对象。如果站在 JVM 的角度来看，只存在两种类加载器:
- 启动类加载器（Bootstrap ClassLoader）：由C++语言实现（针对HotSpot）,负责将存放在 JAVA_HOME\lib 目录或-Xbootclasspath参数指定的路径中的类库加载到内存中。
- 其他类加载器：由Java语言实现，继承自抽象类ClassLoader。如：
    - 扩展类加载器（Extension ClassLoader）：负责加载 JAVA_HOME\lib\ext 目录或 java.ext.dirs 系统变量指定的路径中的所有类库。
    - 应用程序类加载器（Application ClassLoader），负责加载用户类路径（classpath）上的指定类库，我们可以直接使用这个类加载器。一般情况，如果我们没有自定义类加载器默认就是用这个加载器。

#### 什么是双亲委派模式
Java 的类加载使用双亲委派模式，即一个类加载器在加载类时，先把这个请求委托给自己的父类加载器去执行，如果父类加载器还存在父类加载器，就继续向上委托，直到顶层的启动类加载器。如果父类加载器能够完成类加载，就成功返回，如果父类加载器无法完成加载，那么子加载器才会尝试自己去加载。

![](https://upload-images.jianshu.io/upload_images/2154124-d2f7f6206935de2b?imageMogr2/auto-orient/strip%7CimageView2/2)

#### 双亲委派模式有什么好处 
这种双亲委派模式的好处，一个可以避免类的重复加载，另外也避免了 Java 的核心 API 被篡改。

#### JVM，垃圾回收机制，内存划分等

####  jvm性能调优都做了什么