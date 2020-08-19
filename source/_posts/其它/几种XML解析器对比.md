---
title: 几种XML解析器对比

categories:
- 其它

date: 2020-08-17
---

## 介绍
#### DOM
DOM 是用与平台和语言无关的方式表示 XML 文档的官方 W3C 标准。DOM 是以层次结构组织的节点或信息片断的集合。这个层次结构允许开发人员在树中寻找特定信息。分析该结构通常需要加载整个文档和构造层次结构，然后才能做任何工作。由于它是基于信息层次的，因而 DOM 被认为是基于树或基于对象的。DOM 以及广义的基于树的处理具有几个优点。首先，由于树在内存中是持久的，因此可以修改它以便应用程序能对数据和结构作出更改。它还可以在任何时候在树中上下导航，而不是像 SAX 那样是一次性的处理。DOM 使用起来也要简单得多。 

#### SAX
SAX 处理的优点非常类似于流媒体的优点。分析能够立即开始，而不是等待所有的数据被处理。而且，由于应用程序只是在读取数据时检查数据，因此不需要将数据存储在内存中。这对于大型文档来说是个巨大的优点。事实上，应用程序甚至不必解析整个文档；它可以在某个条件得到满足时停止解析。一般来说，SAX 还比它的替代者 DOM 快许多。 

#### JDOM
JDOM 的目的是成为 Java 特定文档模型，它简化与 XML 的交互并且比使用 DOM 实现更快。由于是第一个 Java 特定模型，JDOM 一直得到大力推广和促进。正在考虑通过“Java 规范请求 JSR-102 ”将它最终用作“ Java 标准扩展”。从 2000年 初就已经开始了 JDOM 开发。 

JDOM 与 DOM 主要有两方面不同。首先，JDOM 仅使用具体类而不使用接口。这在某些方面简化了 API，但是也限制了灵活性。第二，API 大量使用了 `Collections` 类，简化了那些已经熟悉这些类的 Java 开发者的使用。 

JDOM 文档声明其目的是“使用 20%（或更少）的精力解决 80%（或更多）Java/XML 问题”（根据学习曲线假定为 20%）。JDOM 对于大多数 Java/XML 应用程序来说当然是有用的，并且大多数开发者发现 API 比 DOM 容易理解得多。JDOM 还包括对程序行为的相当广泛检查以防止用户做任何在 XML 中无意义的事。然而，它仍需要您充分理解 XML 以便做一些超出基本的工作（或者甚至理解某些情况下的错误）。这也许是比学习 DOM 或 JDOM 接口都更有意义的工作。 

JDOM 自身不包含解析器。它通常使用 SAX2 解析器来解析和验证输入 XML 文档（尽管它还可以将以前构造的 DOM 表示作为输入）。它包含一些转换器以将 JDOM 表示输出成 SAX2 事件流、DOM 模型或 XML 文本文档。JDOM 是在 Apache 许可证变体下发布的开放源码。 

#### DOM4J
虽然 DOM4J 代表了完全独立的开发结果，但最初，它是 JDOM 的一种智能分支。它合并了许多超出基本 XML 文档表示的功能，包括集成的 XPath 支持、XML Schema 支持以及用于大文档或流化文档的基于事件的处理。它还提供了构建文档表示的选项，它通过 DOM4J API 和标准 DOM 接口具有并行访问功能。从 2000 下半年开始，它就一直处于开发之中。 

为支持所有这些功能，DOM4J 使用接口和抽象基本类方法。DOM4J 大量使用了 API 中的 `Collections` 类，但是在许多情况下，它还提供一些替代方法以允许更好的性能或更直接的编码方法。直接好处是，虽然 DOM4J 付出了更复杂的 API 的代价，但是它提供了比 JDOM 大得多的灵活性。 

在添加灵活性、XPath 集成和对大文档处理的目标时，DOM4J 的目标与 JDOM 是一样的：针对 Java 开发者的易用性和直观操作。它还致力于成为比 JDOM 更完整的解决方案，实现在本质上处理所有 Java/XML 问题的目标。在完成该目标时，它比 JDOM 更少强调防止不正确的应用程序行为。 

DOM4J 是一个非常非常优秀的 Java XML API，具有性能优异、功能强大和极端易用使用的特点，同时它也是一个开放源代码的软件。如今你可以看到越来越多的 Java 软件都在使用 DOM4J 来读写 XML，特别值得一提的是连 Sun 的 JAXM 也在用 DOM4J。

## 引入依赖
DOM 是 Java 自带的 XML 解析器，无需引入。

引入 SAX 依赖，大小约 30KB：

```xml
<dependency>
    <groupId>sax</groupId>
    <artifactId>sax</artifactId>
    <version>2.0.1</version>
</dependency>
```

引入 JDOM 依赖，大小约 160KB：

```xml
<dependency>
    <groupId>jdom</groupId>
    <artifactId>jdom</artifactId>
    <version>1.1</version>
</dependency>
```

引入 dom4j 依赖，大小约 946KB：

```xml
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
</dependency>
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.1.6</version>
</dependency>
```

## 使用
#### 生成 XML
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<resources>
    <product name="QQ">
        <account id="123456789">
            <nickname>qq-account-1</nickname>
            <password>qwe123qwe123</password>
            <level>56</level>
        </account>
        <account id="987654321">
            <nickname>qq-account-2</nickname>
            <password>ios9ios9ios9</password>
            <level>12</level>
        </account>
    </product>
    <product name="Netease">
        <account id="tom">
            <password>pwdOfTom</password>
            <capacity>1024</capacity>
        </account>
        <account name="Jim">
            <password>pwdOfJim</password>
            <capacity>2560</capacity>
        </account>
    </product>
</resources>
```

我们分别使用 4 种解析器对上面 XML 文本内容进行生成，查看它们之间的效率。

这是 DOM 的生成代码，较为复杂：
```java
//建立DocumentBuilderFactor，用于获得DocumentBuilder对象：
DocumentBuilderFactory bbf = DocumentBuilderFactory.newInstance();

// 获取构建对象
DocumentBuilder builder = bbf.newDocumentBuilder();

// 解析路径得到文档对象
Document doc = builder.parse(new File(filePath));

// 获取根元素
Element resources = doc.getDocumentElement();

Element product1 = doc.createElement("product");
product1.setAttribute("name", "QQ");
resources.appendChild(product1);

Element account1 = doc.createElement("account");
account1.setAttribute("id", "123456789");
product1.appendChild(account1);

Element nickname1 = doc.createElement("nickname");
nickname1.setTextContent("qq-account-1");
Element password1 = doc.createElement("password");
password1.setTextContent("qwe123qwe123");
Element level1 = doc.createElement("level");
level1.setTextContent("56");
account1.appendChild(nickname1);
account1.appendChild(password1);
account1.appendChild(level1);

Element account2 = doc.createElement("account");
account2.setAttribute("id", "123456789");
product1.appendChild(account2);

Element nickname2 = doc.createElement("nickname");
nickname2.setTextContent("qq-account-1");
Element password2 = doc.createElement("password");
password2.setTextContent("qwe123qwe123");
Element level2 = doc.createElement("level");
level2.setTextContent("56");
account2.appendChild(nickname2);
account2.appendChild(password2);
account2.appendChild(level2);


Element product2 = doc.createElement("product");
product2.setAttribute("name", "Netease");
resources.appendChild(product2);

Element account3 = doc.createElement("account");
account3.setAttribute("id", "tom");
product2.appendChild(account3);

Element password3 = doc.createElement("password");
password3.setTextContent("pwdOfTom");
Element capacity3 = doc.createElement("capacity");
capacity3.setTextContent("1024");
account3.appendChild(password3);
account3.appendChild(capacity3);

Element account4 = doc.createElement("account");
account4.setAttribute("id", "tom");
product2.appendChild(account4);

Element password4 = doc.createElement("password");
password4.setTextContent("pwdOfTom");
Element capacity4 = doc.createElement("capacity");
capacity4.setTextContent("1024");
account4.appendChild(password4);
account4.appendChild(capacity4);
```

这是 SAX 的生成代码，也较为复杂。

```java
// 创建一个SAX转换工厂
SAXTransformerFactory factory = (SAXTransformerFactory) SAXTransformerFactory.newInstance();

// 创建一个TransformerHandler实例
TransformerHandler handler = factory.newTransformerHandler();

// 创建一个handler转换器
Transformer transformer = handler.getTransformer();
transformer.setOutputProperty(OutputKeys.INDENT, "yes");
transformer.setOutputProperty(OutputKeys.ENCODING, "utf-8");

// 创建一个Result实例连接到XML文件
Result result = new StreamResult(new FileOutputStream(new File(filePath)));
handler.setResult(result);

AttributesImpl attr = new AttributesImpl();
handler.startDocument();
handler.startElement("", "", "resources", attr);
attr.clear();

attr.addAttribute("", "", "name", "", "QQ");
handler.startElement("", "", "product", attr);
attr.clear();

attr.addAttribute("", "", "id", "", "123456789");
handler.startElement("", "", "account", attr);
attr.clear();

handler.startElement("", "", "nickname", attr);
handler.characters("qq-account-1".toCharArray(), 0, "qq-account-1".length());
attr.clear();
handler.endElement("", "", "nickname");

handler.startElement("", "", "password", attr);
handler.characters("qwe123qwe123".toCharArray(), 0, "qwe123qwe123".length());
attr.clear();
handler.endElement("", "", "password");

handler.startElement("", "", "level", attr);
handler.characters("56".toCharArray(), 0, "56".length());
attr.clear();
handler.endElement("", "", "level");
handler.endElement("", "", "account");

attr.addAttribute("", "", "id", "", "987654321");
handler.startElement("", "", "account", attr);
attr.clear();

handler.startElement("", "", "nickname", attr);
handler.characters("qq-account-1".toCharArray(), 0, "qq-account-2".length());
attr.clear();
handler.endElement("", "", "nickname");

handler.startElement("", "", "password", attr);
handler.characters("qwe123qwe123".toCharArray(), 0, "ios9ios9ios9".length());
attr.clear();
handler.endElement("", "", "password");

handler.startElement("", "", "level", attr);
handler.characters("56".toCharArray(), 0, "12".length());
attr.clear();
handler.endElement("", "", "level");
handler.endElement("", "", "account");
handler.endElement("", "", "product");

attr.addAttribute("", "", "name", "", "Netease");
handler.startElement("", "", "product", attr);
attr.clear();

attr.addAttribute("", "", "id", "", "tom");
handler.startElement("", "", "account", attr);
attr.clear();
handler.startElement("", "", "password", attr);
handler.characters("pwdOfTom".toCharArray(), 0, "pwdOfTom".length());
attr.clear();
handler.endElement("", "", "password");
handler.startElement("", "", "capacity", attr);
handler.characters("1024".toCharArray(), 0, "1024".length());
attr.clear();
handler.endElement("", "", "capacity");
handler.endElement("", "", "account");

attr.addAttribute("", "", "id", "", "Jim");
handler.startElement("", "", "account", attr);
attr.clear();
handler.startElement("", "", "password", attr);
handler.characters("pwdOfJim".toCharArray(), 0, "pwdOfJim".length());
attr.clear();
handler.endElement("", "", "password");
handler.startElement("", "", "capacity", attr);
handler.characters("2560".toCharArray(), 0, "2560".length());
attr.clear();
handler.endElement("", "", "capacity");
handler.endElement("", "", "account");
handler.endElement("", "", "product");
handler.endElement("", "", "resources");

// 关闭doc对象
handler.endDocument();
```

这是 JDOM 的生成代码，较上面抽象了许多复杂的过程，看起来精简了很多。

```java
Document document = new Document();

Element resources = new Element("resources");
document.addContent(resources);

Element product1 = new Element("product");
resources.addContent(product1);
product1.setAttribute("name", "QQ");

Element account1 = new Element("account");
account1.setAttribute("id", "123456789");

Element nickname1 = new Element("nickname");
nickname1.setText("qq-account-1");
Element password1 = new Element("password");
password1.setText("qwe123qwe123");
Element level1 = new Element("level");
level1.setText("56");
account1.addContent(nickname1);
account1.addContent(password1);
account1.addContent(level1);
product1.addContent(account1);

Element account2 = new Element("account");
account2.setAttribute("id", "987654321");

Element nickname2 = new Element("nickname");
nickname2.setText("qq-account-2");
Element password2 = new Element("password");
password2.setText("ios9ios9ios9");
Element level2 = new Element("level");
level2.setText("12");
account2.addContent(nickname2);
account2.addContent(password2);
account2.addContent(level2);
product1.addContent(account2);

Element product2 = new Element("product");
product2.setAttribute("name", "QQ");
resources.addContent(product2);

Element account3 = new Element("account");
account3.setAttribute("id", "tom");

Element password3 = new Element("password");
password3.setText("pwdOfTom");
Element capacity3 = new Element("capacity");
capacity3.setText("1024");
account3.addContent(password3);
account3.addContent(capacity3);

Element account4 = new Element("account");
account4.setAttribute("id", "Jim");

Element password4 = new Element("password");
password4.setText("qwe123qwe123");
Element capacity4 = new Element("capacity");
capacity4.setText("1024");
account4.addContent(password4);
account4.addContent(capacity4);

product2.addContent(account3);
product2.addContent(account4);
```

这是 DOM4J 的生成代码，无论是易读性还是简洁性都比上面的好很多：

```java
Document document = DocumentHelper.createDocument();
Element resources = document.addElement("resources");
Element product1 = resources.addElement("product");
product1.addAttribute("name", "QQ");
Element account1 = product1.addElement("account");
account1.addAttribute("id", "123456789");
account1.addElement("nickname", "qq-account-1");
account1.addElement("password", "qwe123qwe123");
account1.addElement("level", "56");

Element account2 = product1.addElement("account");
account2.addAttribute("id", "987654321");
account2.addElement("nickname", "qq-account-2");
account2.addElement("password", "ios9ios9ios9");
account2.addElement("level", "12");

Element product2 = resources.addElement("product");
product2.addAttribute("name", "Netease");
Element account3 = product2.addElement("account");
account3.addAttribute("id", "tom");
account3.addElement("password", "qwe123qwe123");
account3.addElement("capacity", "1024");

Element account4 = product2.addElement("account");
account4.addAttribute("id", "Jim");
account4.addElement("password", "ios9ios9ios9");
account4.addElement("capacity", "2048");
```

#### 获取属性值
这里对比的是不同的解析器对读取元素属性的表达能力。

DOM 代码：
```java
NodeList products = resources.getElementsByTagName("product");
NodeList accounts = products.item(0).getChildNodes();
Node account = accounts.item(0);
Node id = account.getAttributes().getNamedItem("id");
id.getNodeValue();
```

SAX 代码，SAX 需要实行解析逻辑才能使用，极其不友好：
```java
private static class Test1Handler extends DefaultHandler {
    // 0未进入 1进入 2离开
    private int product1status;
    // 0未进入 1进入 2离开
    private int account1status;
    // 是否结束
    private boolean isfinish;

    private String id;


    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        super.startElement(uri, localName, qName, attributes);
        if (isfinish) return;

        if (qName.equals("product") && product1status == 0) {
            product1status = 1;
            return;
        }

        if (qName.equals("account") && product1status == 1 && account1status == 0) {
            account1status = 1;

            System.out.println("------------------------------------");
            System.out.println("id: " + attributes.getValue("id"));
            System.out.println("------------------------------------");
            id = attributes.getValue("id");

            isfinish = true;
            return;
        }
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        super.characters(ch, start, length);
        if (isfinish) return;

        if (product1status == 1 && account1status == 1) {
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        super.endElement(uri, localName, qName);
        if (isfinish) return;

        if (qName.equals("product") && product1status == 1) {
            product1status = 2;
            return;
        }
        if (qName.equals("account") && product1status == 1) {
            account1status = 2;
            return;
        }
    }

    public void result() {
        System.out.println("id is : " + id);
    }

    public static void main(String[] args) {
        // 1. 得到SAX解析工厂
        SAXParserFactory saxParserFactory = SAXParserFactory.newInstance();

        // 2. 让工厂生产一个sax解析器
        SAXParser newSAXParser = saxParserFactory.newSAXParser();

        // 3. 传入输入流和handler，解析
        Test1Handler handler = new Test1Handler();
        newSAXParser.parse(is, handler);
        is.close();

        // 4. 得到结果
        handler.result();
    }
}
```

JDOM 代码：
```java
List products = resource.getChildren();
List accounts = ((Element) products.get(0)).getChildren();
Element account = (Element) accounts.get(0);

Attribute id = account.getAttribute("id");
id.getValue();
```

DOM4J 代码：
```java
List products = resources.elements("product");
List accounts = ((Element) products.get(0)).elements("account");
Element account = (Element) accounts.get(0);

Attribute id = account.attribute("id");
id.getValue();
```

#### 设置属性值
这里比较不同解析器设置属性值写法。

JAX 过于复杂，排除在外。

DOM 写法：

```java
NodeList products = resources.getElementsByTagName("product");
NodeList accounts = products.item(0).getChildNodes();
Node account = accounts.item(0);
Node id = account.getAttributes().getNamedItem("id");
id.setNodeValue("123");
```

JDOM 写法：

```java
List products = resource.getChildren();
List accounts = ((Element) products.get(0)).getChildren();
Element account = (Element) accounts.get(0);

Attribute id = account.getAttribute("id");
id.setValue("123");
```

DOM4J 写法：

```java
List products = resources.elements("product");
List accounts = ((Element) products.get(0)).elements("account");
Element account = (Element) accounts.get(0);

Attribute id = account.attribute("id");
id.setValue("123");
```

可以看出，DOM、JDOM、DOM4J 的差别不大，DOM4J 相对而言更符合人的逻辑。

#### 删除属性
这里测试的不同的解析器如何删除元素。

同样的，JAX 排除掉。

DOM 写法：

```java
NodeList products = resources.getElementsByTagName("product");
NodeList accounts = products.item(0).getChildNodes();
Node account = accounts.item(0);
account.getAttributes().removeNamedItem("id");
```

JDOM 写法：

```java
List products = resource.getChildren();
List accounts = ((Element) products.get(0)).getChildren();
Element account = (Element) accounts.get(0);

account.removeAttribute("id");
```

DOM4J 写法：

```java
List products = resources.elements("product");
List accounts = ((Element) products.get(0)).elements("account");
Element account = (Element) accounts.get(0);

Attribute id = account.attribute("id");
account.remove(id);
```

#### 获取节点值
这里测试不同解析器如何获取元素节点的值。

同样的，JAX 排除在外。

DOM 代码：

```java
NodeList products = resources.getElementsByTagName("product");
NodeList accounts = products.item(0).getChildNodes();
Node account = accounts.item(0);
Node nickname = account.getChildNodes().item(0);
nickname.getTextContent();
```

JDOM 代码：

```java
List products = resource.getChildren();
List accounts = ((Element) products.get(0)).getChildren();
Element account = (Element) accounts.get(0);

Element nickname = account.getChild("nickname");
nickname.getText();
```

DOM4J 代码：

```java
List products = resources.elements("product");
List accounts = ((Element) products.get(0)).elements("account");

Element account = (Element) accounts.get(0);
Element nickname = account.element("nickname");
nickname.getText();
```

#### 设置节点值
这里测试的是不同解析器如果设置元素节点的值。

JAX 排除在外。

DOM 代码：

```java
NodeList products = resources.getElementsByTagName("product");
NodeList accounts = products.item(0).getChildNodes();
Node account = accounts.item(0);
Node nickname = account.getChildNodes().item(0);
nickname.setTextContent("123");
```

JDOM 代码：

```java
List products = resource.getChildren();
List accounts = ((Element) products.get(0)).getChildren();
Element account = (Element) accounts.get(0);

Element nickname = account.getChild("nickname");
nickname.setText("123");
```

DOM4J 代码：

```java
List products = resources.elements("product");
List accounts = ((Element) products.get(0)).elements("account");
Element account = (Element) accounts.get(0);

Element nickname = account.element("nickname");
nickname.setText("123");
```

#### 删除节点值
这里测试不同解析器如何删除元素节点的值。

同样的，JAX 排除在外。

DOM 代码：

```java
NodeList products = resources.getElementsByTagName("product");
NodeList accounts = products.item(0).getChildNodes();
Node account = accounts.item(0);
Node nickname = account.getChildNodes().item(0);
account.removeChild(nickname);
```

JDOM 代码：

```java
List products = resource.getChildren();
List accounts = ((Element) products.get(0)).getChildren();
Element account = (Element) accounts.get(0);

account.removeChild("nickname");
```

DOM4J 代码：

```java
List products = resources.elements("product");
List accounts = ((Element) products.get(0)).elements("account");
Element account = (Element) accounts.get(0);

Element nickname = account.element("nickname");
account.remove(nickname);
```

## 综合对比

|解析器|体积|开发效率|创建性能|查询性能|修改性能|
| :- |
|SAX|30KB|极低|6238/s|4526/s|-|
|DOM|0KB|较低|2928/s|8928571/s|6578947/s|
|JDOM|160KB|低|491642/s|17241379/s|8064516/s
|DOM4J|946KB|中等|674308/s|6896551/s|6060606/s|

## 结论
1. DOM4J 性能最好，连 Sun 的 JAXM 也在用 DOM4J。目前许多开源项目中大量采用 DOM4J，例如大名鼎鼎的 Hibernate 也用 DOM4J 来读取 XML 配置文件。如果不考虑可移植性，那就采用DOM4J。
1. JDOM 和 DOM 在性能测试时表现不佳，在测试 10M 文档时内存溢出。在小文档情况下还值得考虑使用 DOM 和 JDOM。虽然 JDOM 的开发者已经说明他们期望在正式发行版前专注性能问题，但是从性能观点来看，它确实没有值得推荐之处。另外，DOM 仍是一个非常好的选择。DOM 实现广泛应用于多种编程语言。它还是许多其它与XML相关的标准的基础，因为它正式获得 W3C 推荐（与基于非标准的 Java 模型相对），所以在某些类型的项目中可能也需要它（如在 JavaScript 中使用 DOM）。 
1. SAX 表现较好，这要依赖于它特定的解析方式－事件驱动。一个 SAX 检测即将到来的XML流，但并没有载入到内存（当然当 XML 流被读入时，会有部分文档暂时隐藏在内存中）。
