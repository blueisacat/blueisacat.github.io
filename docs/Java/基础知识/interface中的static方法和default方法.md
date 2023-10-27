---
layout: default
title: interface中的static方法和default方法
parent: 基础知识
grand_parent: Java
---

### static方法

java8中为接口新增了一项功能:定义一个或者更多个静态方法。用法和普通的static方法一样。

示例:

```java
public interface InterfaceA {
    static void showStatic(){
        System.out.println("this is InterfaceA showStatic method");
    }
}
```

测试:

```java
public class Test {
    public static void main(String[] args){
        InterfaceA.show();
    }
}
```

输出:

```java
this is InterfaceA showStatic method
```

### default方法

在接口中,增加default方法,是为了既有的成千上万的java类库的类增加新的功能,且不必对这些类重新进行设计。比如,只需要在Collection接口中增加default Stream stream(),相应的Set和List接口以及它们的子类都包含此方法,不必为每个子类都重新copy这个方法。

示例:

```java
public interface InterfaceA {
    default void showDefault(){
        System.out.println("this is InterfaceA showDefault method");
    }
}
public class InterfaceAImpl implements InterfaceA {
}
```

测试:

```java
public class Test {
    public static void main(String[] args){
        new InterfaceAImpl().showDefault();
    }
}
```

输出:

```java
this is InterfaceA showDefault method
```

**如果接口中的默认方法不能满足某个实现类的需要,那么实现类可以覆盖默认方法。**

示例:

```java
public interface InterfaceA {
    default void showDefault(){
        System.out.println("this is InterfaceA showDefault method");
    }
}
public class InterfaceAImpl implements InterfaceA{
    @Override
    public void showDefault(){
        System.out.println("this is InterfaceAImpl showDefault method");
    }
}
```

测试:

```java
public class Test {
    public static void main(String[] args){
        new InterfaceAImpl().showDefault();
    }
}
```

输出:

```java
this is InterfaceAImpl showDefault method
```

**如果实现多个接口,且接口中拥有相同的default方法和static方法,实现类需要重写该方法**

示例:

```java
public interface InterfaceA {
    static void showStatic(){
        System.out.println("this is InterfaceA showStatic method");
    }
    
    default void showDefault(){
        System.out.println("this is InterfaceA showDefault method");
    }
}
public interface InterfaceB {
    static void showStatic(){
        System.out.println("this is InterfaceB showStatic method");
    }
    
    default void showDefault(){
        System.out.println("this is InterfaceB showDefault method");
    }
}
public class InterfaceAImpl implements InterfaceA,InterfaceB{
    @Override
    public void showDefault(){
        System.out.println("this is InterfaceAImpl showDefault method");
    }
}
```

输出:

```java
this is InterfaceAImpl showDefault method
```

### 反射调用default方法

在Mybatis源码中的MapperProxy.class中,具体使用如下:

```java
private Object invokeDefaultMethod(Object proxy,Method method,Object[] args) throws Throwable{
    final Constructor<MethodHandles.Lookup> constructor = MethodHandles.Lookup.class.getDeclaredConstructor(Class.class,int.class);
    if(!constructor.isAccessible()){
        constructor.setAccessible(true);
    }
    final Class<?> declaringClass = method.getDeclaringClass();
    return constructor.newInstance(declaringClass,MethodHandles.Lookup.PRIVATE).unreflectSpecial(method,declaringClass).bindTo(proxy).invokeWithArguments(args);
}
private boolean isDefaultMethod(Method method){
    return ((method.getModifiers() & (Modifier.ABSTRACT | Modifier.PUBLIC | Modifier.STATIC)) == Modifier.PUBLIC) && method.getDeclaringClass().isInterface();
}
```