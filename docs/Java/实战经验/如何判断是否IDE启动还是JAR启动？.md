---
layout: default
title: 如何判断是否IDE启动还是JAR启动？
parent: 实战经验
---

### 判断逻辑

1. 首先找到启动类

1. 通过getResources()方法判断，因在IDE中getResource()并不能获取到资源文件。

### 代码实现

```java
@SneakyThrows
public static String getProtocol() {
    //根据main方法，获取到启动类
    Class mainClazz = null;
    StackTraceElement[] stackTraceElements = new RuntimeException().getStackTrace();
    for (StackTraceElement stackTraceElement : stackTraceElements) {
        if ("main".equals(stackTraceElement.getMethodName())) {
            mainClazz = Class.forName(stackTraceElement.getClassName());
        }
    }
    //获取资源的协议类型，IDE中为file，JAR中为jar
    return mainClazz.getResource("").getProtocol();
}
```