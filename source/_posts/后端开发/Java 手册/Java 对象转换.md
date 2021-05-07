---
title: Java 对象转换

categories:
- 后端开发
- Java 手册

date: 2021-05-07
---
## InputStream 转化为 File
#### JDK 原生提供
```java
// 测试OK
InputStream is = new InputStream("/test/tst.png");
OutputStream os = null;
try {
   os = new FileOutputStream(new File("/test/tst.png"));
   int len = 0;
   byte[] buffer = new byte[8192];

   while ((len = is.read(buffer)) != -1) {
      os.write(buffer, 0, len);
   }
} finally {
   os.close();
   is.close();
}
```

#### Apache Common 提供
```java
// 测试OK
InputStream initialStream = FileUtils.openInputStream(new File("src/main/resources/sample.txt"));
File targetFile = new File("src/main/resources/targetFile.tmp");
// 此方法会清空 initialStream 数据
FileUtils.copyInputStreamToFile(initialStream, targetFile);
```

#### Google Guava 提供
```java
// 测试OK
InputStream initialStream = new FileInputStream(new File("src/main/resources/sample.txt"));
byte[] buffer = new byte[initialStream.available()];
initialStream.read(buffer);
File targetFile = new File("src/main/resources/targetFile.tmp");
Files.write(buffer, targetFile);
```

#### HTTP 处理
如果输入流链接到正在进行的数据流上，如来自正在进行的链接的HTTP响应，此时可能无法一次读取整个流。这种情况下，我们需要确保一直读取到流的尽头。

```java
File targetFile = new File("src/main/resources/targetFile.tmp");
try(InputStream initialStream = new FileInputStream(new File("src/main/resources/sample.txt"));
    OutputStream outStream = new FileOutputStream(targetFile)) {
    byte[] buffer = new byte[8 * 1024];
    int bytesRead;
    while ((bytesRead = initialStream.read(buffer)) != -1) {
        outStream.write(buffer, 0, bytesRead);
    }
} catch (Exception e) {
    e.printStackTrace();
}
```

## InputStream 转化为 String
#### JDK 原生提供
**方法一**
```java
byte[] bytes = new byte[0];
bytes = new byte[inputStream.available()];
inputStream.read(bytes);
String str = new String(bytes);
```

**方法二**
```java
String result = new BufferedReader(new InputStreamReader(inputStream)).lines()
                                                                      .collect(Collectors.joining(System.lineSeparator()));
```

**方法三**
```java
String result = new BufferedReader(new InputStreamReader(inputStream)).lines()
                                                                      .parallel()
                                                                      .collect(Collectors.joining(System.lineSeparator()));
```

**方法四**
```java
Scanner s = new Scanner(inputStream).useDelimiter("\\A");
String str = s.hasNext() ? s.next() : "";
```

**方法五**
```java
String resource = new Scanner(inputStream).useDelimiter("\\Z").next();
```

**方法六**
```java
StringBuilder sb = new StringBuilder();
String line;

BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
while ((line = br.readLine()) != null) {
    sb.append(line);
}
String str = sb.toString();
return str;
```

**方法七**
```java
// 测试不OK
ByteArrayOutputStream result = new ByteArrayOutputStream();
byte[] buffer = new byte[1024];
int length;
while ((length = inputStream.read(buffer)) != -1) {
    result.write(buffer, 0, length);
}
String str = result.toString(StandardCharsets.UTF_8.name());
return str;
```

**方法八**
```java
BufferedInputStream bis = new BufferedInputStream(inputStream);
ByteArrayOutputStream buf = new ByteArrayOutputStream();
int result = bis.read();
while(result != -1) {
    buf.write((byte) result);
    result = bis.read();
}
String str = buf.toString();
return str;
```

#### Apache Common 提供
**方法九**
```java
StringWriter writer = new StringWriter();
IOUtils.copy(inputStream, writer, StandardCharsets.UTF_8.name());
String str = writer.toString();
```

**方法十**
```java
String str = IOUtils.toString(inputStream, "utf-8");
```

#### Google Guava 提供
**方法十一**
```java
String str = CharStreams.toString(new InputStreamReader(inputStream, StandardCharsets.UTF_8));
```

**方法十二**
```java
String str = new String(ByteStreams.toByteArray(inputStream));
```

#### 性能排行
针对一个2MB的文件的输入流，多次执行测试如下(单位是毫秒)：
1. 方法七: 21
1. 方法九: 31
1. 方法一: 36
1. 方法十二: 36
1. 方法六: 40
1. 方法三: 66
1. 方法二: 87
1. 方法四: 101
1. 方法十: 111
1. 方法八: 107
1. 方法五: 178
1. 方法十一: 236

从上述结果来看，方法七和方法九更好一些，而方法五和方法十一会更差一些。

## File 转换为 InputStream
```java
new FileInputStream(file);
```

## String 转化为 InputStream
#### JDK 原生提供
```java
// 测试OK
InputStream is = new ByteArrayInputStream(str.getBytes());
```

#### Apache Common 提供
```java
// 测试OK
InputStream targetStream = IOUtils.toInputStream(str, StandardCharsets.UTF_8.name());
```

#### Google Guava 提供
```java
// 测试OK
InputStream targetStream = new ReaderInputStream(CharSource.wrap(str).openStream(), StandardCharsets.UTF_8.name());
```

> InputStream > String > > File 这个流程怎么也走不通。待有时间解决。