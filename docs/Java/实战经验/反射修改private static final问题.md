---
layout: default
title: 反射修改private static final问题
parent: 实战经验
grand_parent: Java
---

在日常应用中，常常会利用java的反射，在运行时将需要修改的常量强制更改成我们所需要的值。

对于private static final的私有静态常量，需要特别注意。

举个栗子：

```
class Bean{  
    private static final Integer INT_VALUE = 100;  
}
```

```
System.out.println(Bean.INT_VALUE);  
//获取Bean类的INT_VALUE字段  
Field field = Bean.class.getField("INT_VALUE");  
//将字段的访问权限设为true：即去除private修饰符的影响  
field.setAccessible(true);  
/*去除final修饰符的影响，将字段设为可修改的*/  
Field modifiersField = Field.class.getDeclaredField("modifiers");  
modifiersField.setAccessible(true);  
modifiersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);  
//把字段值设为200  
field.set(null, 200);  
System.out.println(Bean.INT_VALUE); 
```

以上代码输出的结果是：

> 100200


说明用反射私有静态常量成功了。

那么，这种方式是万能的么？答案是否定的，可以通过下一个栗子来说明，将Integer换成基本类型int。

```
class Bean{  
    private static final int INT_VALUE = 100;//把类型由Integer改成了int  
}
```

在其他代码都不变的情况下，代码输出的结果竟然变成了诡异的：

> 100100


而且在调试的过程中发现，在第二次输出的时候，内存中的Bean.INT_VALUE是已经变成了200，但System.out.println(Bean.INT_VALUE)输出的结果却依然时诡异的100？！
又试了其他几种类型，发现这种貌似失效的情会发生在int、long、boolean以及String这些基本类型上，而如果把类型改成Integer、Long、Boolean这种包装类型，或者其他诸如Date、Object都不会出现失效的情况。
对于基本类型的静态常量，JAVA在编译的时候就会把代码中对此常量中引用的地方替换成相应常量值。


```
if( index > maxFormatRecordsIndex   ){  
    index  =  maxFormatRecordsIndex ;  
}
```

这段代码在编译的时候已经被java自动优化成这样的：

```
if( index > 100){  
    index = 100;  
} 
```

所以在INT_VALUE是int类型的时候

```
System.out.println(Bean.INT_VALUE);  
//编译时会被优化成下面这样：  
System.out.println(100); 
```

所以，自然，无论怎么修改Boolean.INT_VALUE，System.out.println(Bean.INT_VALUE)都还是会依然固执地输出100了。

这本身是JVM的优化代码提高运行效率的一个行为，但是就会导致我们在用反射改变此常量值时出现类似不生效的错觉。