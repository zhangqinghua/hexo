---
title: Java Map List 初始化

categories:
- Java

date: 2018-6-14
---

Map 初始化时赋值

# Map

1. 传统方法
    ```java
    Map<String, String> map = new HashMap<>();
    map.put("key1", "value1");
    map.put("key2", "value2");
    ```
1. 双括号初始化法
    ```java
    Map<String, String> map = new HashMap<string, String>(){{
            put("key1", "value1");
            put("key2", "value2");
            put("keyN", "valueN");
        }};
    ```
1. ImmutableMap方法
    ```java
    Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
    ```
1. JDK9
    ```java
    Map<String, Integer> left = Map.of("Hello", 1, "World", 2);
    ```

## List

1. 传统方法
    ```java
    List<String> list = new ArrayList<>();
    list.add("string1");
    list.add("string2");
    ```
1. 双括号初始化法
    ```java
    List<String> list = new ArrayList<String>(){{
            add("string1");
            add("string2");
            //some other add() code......
            add("stringN");
        }};
    ```
1. Arrays.asList
    ```java
    List<String> supplierNames = Arrays.asList("sup1", "sup2", "sup3");
    ```

1. JDK8
    ```java
    List<String> list = Stream.of("one", "two", "three").collect(Collectors.toList());
    ```

1. JDK9
    ```java
    List<String> list = List.of("one", "two", "three");
    ```