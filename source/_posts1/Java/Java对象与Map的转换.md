---
title: Java对象与Map的转换

categories:
- Java

date: 2018-06-14
---

Map转Bean

## apache BeanUtils

```java
Map<String, Object> map = ImmutableMap.of("name", "tron");
LandlordDTO dto = new LandlordDTO();
Long time = System.currentTimeMillis();
for (int i = 0; i < 1000000; i++) {
    BeanUtils.populate(dto, map);
}
System.out.println("take time: " + (System.currentTimeMillis() - time));

// take time: 12787
```

## 阿里巴巴JSON方式

```java
Map<String, Object> map = ImmutableMap.of("name", "tron");
Long time = System.currentTimeMillis();
for (int i = 0; i < 1000000; i++) {
    JSON.parseObject(JSON.toJSONString(map), LandlordDTO.class);
}
System.out.println("take time:: " + (System.currentTimeMillis() - time));

// take time: 1247
```

## 使用Introspector进行转换

```java
Map<String, Object> map = ImmutableMap.of("name", "tron");
Long time = System.currentTimeMillis();
for (int i = 0; i < 1000000; i++) {
    Object obj = LandlordDTO.class.newInstance();
    BeanInfo beanInfo = Introspector.getBeanInfo(obj.getClass());
    PropertyDescriptor[] propertyDescriptors = beanInfo.getPropertyDescriptors();
    for (PropertyDescriptor property : propertyDescriptors) {
        Method setter = property.getWriteMethod();
        if (setter != null) {
            setter.invoke(obj, map.get(property.getName()));
        }
    }
}
System.out.println("take time: " + (System.currentTimeMillis() - time));

// take time: 1468
```

## 使用反射机制

```java
Map<String, Object> map = ImmutableMap.of("name", "tron");
Long time = System.currentTimeMillis();
for (int i = 0; i < 1000000; i++) {
    Object obj = LandlordDTO.class.newInstance();
    Field[] fields = obj.getClass().getDeclaredFields();
    for (Field field : fields) {
        int mod = field.getModifiers();
        if (Modifier.isStatic(mod) || Modifier.isFinal(mod)) {
            continue;
        }

        field.setAccessible(true);
        field.set(obj, map.get(field.getName()));
    }
}
System.out.println("take time: " + (System.currentTimeMillis() - time));

// take time: 195
```