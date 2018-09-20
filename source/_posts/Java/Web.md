---
title: Web

categories:
- Java

date: 2018-03-26 13:00:00
---

Java web 笔记

# Servlet的生命周期

Servlet是运行在Servlet容器中的，常用的tomcat、jboss、weblogic都是Servlet容器，其生命周期是由容器来管理。Servlet的生命周期通过java.servlet.Servlet接口中的init（）、service（）、和destroy（）方法表示。Servlet的生命周期有四个阶段：加载并实例化、初始化、请求处理、销毁。

1. 加载并实例化
    Servlet容器负责加载和实例化Servelt。当Servlet容器启动时，或者在容器检测到需要这个Servlet来响应第一个请求时，创建Servlet实例。当Servlet容器启动后，Servlet通过类加载器来加载Servlet类，加载完成后再new一个Servlet对象来完成实例化。
1. 初始化
    在Servlet实例化之后，容器将调用init（）方法，并传递实现ServletConfig接口的对象。在init（）方法中，Servlet可以部署描述符中读取配置参数，或者执行任何其他一次性活动。在Servlet的整个生命周期类，init（）方法只被调用一次。
1. 请求处理
    当Servlet初始化后，容器就可以准备处理客户机请求了。当容器收到对这一Servlet的请求，就调用Servlet的service（）方法，并把请求和响应对象作为参数传递。当并行的请求到来时，多个service（）方法能够同时运行在独立的线程中。通过分析ServletRequest或者HttpServletRequest对象，service（）方法处理用户的请求，并调用ServletResponse或者HttpServletResponse对象来响应。
1. 销毁
    一旦Servlet容器检测到一个Servlet要被卸载，这可能是因为要回收资源或者因为它正在被关闭，容器会在所有Servlet的service（）线程之后，调用Servlet的destroy（）方法。然后，Servlet就可以进行无用存储单元收集清理。这样Servlet对象就被销毁了。这四个阶段共同决定了Servlet的生命周期。

# Servlet的调用过程

1. 用户点击一个链接，指向一个servlet而不是一个静态页面。
1. web服务器接到这个请求后转发给容器。容器接着创建两个对象：HttpServletRequest和HttpServletResponse。
1. 容器根据请求中的URL找到相应的servlet，为这个请求创建一个线程，并把请求对象HtttpServletRequest和响应对象HttpServletResponse传递给这个servlet线程。
1. 线程接下来调用service()方法，根据请求的不同，service()方法调用doGet()和doPost()方法。
1. doGet()方法生成动态页面，并把这个页面塞到响应对象里。
1. service()方法结束，随之线程结束，容器把响应对象装换为一个HTTP相应，发送给客户，然后删除请求和响应对象。

# Servlet是线程安全的吗？

1. 当客户端第一次请求Servlet的时候,tomcat会根据web.xml配置文件实例化servlet。

1. 当又有一个客户端访问该servlet的时候，不会再实例化该servlet，也就是多个线程在使用这个实例。

1. JSP/Servlet容器默认是采用单实例多线程(这是造成线程安全的主因)方式处理多个请求的，这种默认以多线程方式执行的设计可大大降低对系统的资源需求，提高系统的并发量及响应时间。

1. Servlet本身是无状态的，一个无状态的Servlet是绝对线程安全的，无状态对象设计也是解决线程安全问题的一种有效手段。

1. servlet是否线程安全是由它的实现来决定的，如果它内部的属性或方法会被多个线程改变，它就是线程不安全的，反之，就是线程安全的。

# 如何控制Servlet的线程安全性？

1. 避免使用实例变量
1. 避免使用非线程安全的集合
1. 在多个Servlet中对某个外部对象(例如文件)的修改是务必加锁（Synchronized，或者ReentrantLock），互斥访问。
1. 属性的线程安全：ServletContext、HttpSession是线程安全的；ServletRequest是非线程安全的。

# JSP的执行原理

1. 客户端发起请求
1. jsp引擎把jsp页面解析为Servlet的java源文件，在tomcat中将源文件编译为class文件
1. 加载到内存中执行
1. 返回结果给客户端

# JSP和Servlet的区别

1. jsp的可读性强，容易维护，jsp在最后会编译成为servlet
1. servlet容易调试

# JSP 重定向或转发有什么区别

1. 重定向是客户端行为，转发是服务端行为。
1. 重定向时服务器产生2次请求，转发产生一次请求。
1. 重定向可以转发到项目以外的任何网址，转发只能在当前项目里面转发。
1. 重定向会导致request对象信息丢失，转发则不会。
1. 重定向url会改变，转发url不会改变。

# JSP的九大内置对象

1. page对象
    对象代表jsp这个实体本身，即当前页面有效，相当于java中的this
1. request对象
    接收客户端的http请求
1. response对象
    封装jsp产生的回应
1. session对象
    用于保存用户信息,跟踪用户行为
1. out对象
    向客户端输出数据,字节流
1. pageContext对象
    pageContext对象提供了对JSP页面内所有的对象及名字空间的访问,也就是说他可以访问到本页所在的SESSION,也可以取本页面所在的application的某一属性值,他相当于页面中所有功能的集大成者,它的本类名也叫pageContext。
1. application对象
    实现了用户间数据的共享,可存放全局变量
1. config对象
    Servlet的配置信息
1. exception对象
    代表运行时的异常.

# jsp七大动作

1. include
    包含动态页面
1. useBean
    使用javaBean
1. forward
    跳转页面
1. getProperty
    从对象中取出属性值
1. setProperty
    为对象设置属性值
1. param
    传递参数
1. plugin
    用于指定在客户端运行的插件

# JSP的三大指令

1. page指令
    指定页面编码`<%@ page language="java" contentType="text/html;charset=gbk" pageEncoding="gbk" %>`
    导入包`<%@ page import="java.util.*,java.text.*" %>`
1. include指令
    静态包含(统一编译):`<%@ include file="included.jsp"%>`
1. taglib

# session

1. 用于保存每个用户的专用信息
1. Session中的信息保存在Web服务器内容中
1. 每个客户端用户访问时，服务器都为每个用户分配一个唯一的会话ID
1. 生存期是用户持续请求时间再加上一段时间(一般是20分钟左右)

# session的几个主要方法

1. getId 获取当前的会话ID
1. getCreationTime 会话创建时间
1. setMaxInactiveInterval 设置会话的最大持续时间

# 为什么要在session少放对象

1. Session中的信息保存在Web服务器内容中

# application和session的区别

1. application全局变量
1. 只要服务器端不关闭该网站，application始终存在
1. session是会话变量
1. 当关闭这个网站的时候，session就结束了