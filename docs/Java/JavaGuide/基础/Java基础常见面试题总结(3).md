## 异常

**Java 异常类层次结构图概览**

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(3)_image_0.png)

在 Java 中，所有的异常都有一个共同的祖先 

- Exception

- Error

**Checked Exception**

比如下面这段 IO 操作的代码：

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(3)_image_1.png)

除了

**Unchecked Exception**

RuntimeException

- NullPointerException

- IllegalArgumentException

- NumberFormatException

- ArrayIndexOutOfBoundsException

- ClassCastException

- ArithmeticException

- SecurityException

- UnsupportedOperationException

- ......

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(3)_image_2.png)

- String getMessage()

- String toString()

- String getLocalizedMessage()

- void printStackTrace()

- try

- catch

- finally

代码示例：

```
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
```

输出：

```
Try to do something
Catch Exception -> RuntimeException
Finally
```

**注意：不要在 finally 语句块中使用 return!**

[jvm 官方文档](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.10.2.5)中有明确提到：

> If the 
> Saves the return value (if any) in a local variable.
> Executes a 
> Upon return from the 


代码示例：

```
public static void main(String[] args) {
    System.out.println(f(2));
}
public static int f(int value) {
    try {
        return value * value;
    } finally {
        if (value == 2) {
            return 0;
        }
    }
}
```

输出：

```
0
```

不一定的！在某些情况下，finally 中的代码不会被执行。

就比如说 finally 之前虚拟机被终止运行的话，finally 中的代码就不会被执行。

```
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
    // 终止当前正在运行的Java虚拟机
    System.exit(1);
} finally {
    System.out.println("Finally");
}
```

输出：

```
Try to do something
Catch Exception -> RuntimeException
```

另外，在以下 2 种特殊情况下，

1. 程序所在的线程死亡。

1. 关闭 CPU。

相关 issue：[https://github.com/Snailclimb/JavaGuide/issues/190](https://github.com/Snailclimb/JavaGuide/issues/190)。

🧗🏻 进阶一下：从字节码角度分析

1. 适用范围（资源的定义）： 任何实现 

1. 关闭资源和 finally 块的执行顺序： 在 

《Effective Java》中明确指出：

> 面对必须要关闭的资源，我们总是应该优先使用 


Java 中类似于

```
//读取文本文件的内容
Scanner scanner = null;
try {
    scanner = new Scanner(new File("D://read.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

使用 Java 7 之后的 

```
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

当然多个资源需要关闭的时候，使用 

通过使用分号分隔，可以在

```
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))) {
    int b;
    while ((b = bin.read()) != -1) {
        bout.write(b);
    }
}
catch (IOException e) {
    e.printStackTrace();
}
```

- 不要把异常定义为静态变量，因为这样会导致异常栈信息错乱。每次手动抛出异常，我们都需要手动 new 一个异常对象抛出。

- 抛出的异常信息一定要有意义。

- 建议抛出更加具体的异常比如字符串转换为数字格式错误的时候应该抛出

- 使用日志打印异常之后就不要再抛出异常了（两者不要同时存在一段代码逻辑中）。

- ......

**Java 泛型（Generics）**

编译器可以对泛型参数进行检测，并且通过泛型参数可以指定传入的对象类型。比如 

```
ArrayList<E> extends AbstractList<E>
```

并且，原生 

泛型一般有三种使用方式:

**1.泛型类**

```
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{
    private T key;
    public Generic(T key) {
        this.key = key;
    }
    public T getKey(){
        return key;
    }
}
```

如何实例化泛型类：

```
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

**2.泛型接口**

```
public interface Generator<T> {
    public T method();
}
```

实现泛型接口，不指定类型：

```
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}
```

实现泛型接口，指定类型：

```
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```

**3.泛型方法**

```
   public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```

使用：

```
// 创建不同类型数组：Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```

> 注意: 


- 自定义接口通用返回结果 

- 定义 

- 构建集合工具类（参考 

- ......

关于反射的详细解读，请看这篇文章 [Java 反射机制详解](/java/basis/reflection.html) 。

如果说大家研究过框架的底层原理或者咱们自己写过框架的话，一定对反射这个概念不陌生。反射之所以被称为框架的灵魂，主要是因为它赋予了我们在运行时分析类以及执行类中方法的能力。通过反射你可以获取任意一个类的所有属性和方法，你还可以调用这些方法和属性。

反射可以让我们的代码更加灵活、为各种框架提供开箱即用的功能提供了便利。

不过，反射让我们在运行时有了分析操作类的能力的同时，也增加了安全问题，比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）。另外，反射的性能也要稍差点，不过，对于框架来说实际是影响不大的。

相关阅读：[Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow) 。

像咱们平时大部分时候都是在写业务代码，很少会接触到直接使用反射机制的场景。但是！这并不代表反射没有用。相反，正是因为反射，你才能这么轻松地使用各种框架。像 Spring/Spring Boot、MyBatis 等等框架中都大量使用了反射机制。

**这些框架中也大量使用了动态代理，而动态代理的实现也依赖反射。**

比如下面是通过 JDK 实现动态代理的示例代码，其中就使用了反射类 

```
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * 代理类中的真实对象
     */
    private final Object target;
    public DebugInvocationHandler(Object target) {
        this.target = target;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after method " + method.getName());
        return result;
    }
}
```

另外，像 Java 中的一大利器 

为什么你使用 Spring 的时候 ，一个

这些都是因为你可以基于反射分析类，然后获取到类/属性/方法/方法的参数上的注解。你获取到注解之后，就可以做进一步的处理。

Annotation

注解本质是一个继承了

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
public interface Override extends Annotation{
}
```

JDK 提供了很多内置的注解（比如 

注解只有被解析之后才会生效，常见的解析方法有两种：

- 编译期直接扫描：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用

- 运行期通过反射处理：像框架中自带的注解(比如 Spring 框架的 

关于 SPI 的详细解读，请看这篇文章 [Java SPI 机制详解](/java/basis/spi.html) 。

SPI 即 Service Provider Interface ，字面意思就是：“服务提供者的接口”，我的理解是：专门提供给服务提供者或者扩展框架功能的开发者去使用的一个接口。

SPI 将服务接口和具体的服务实现分离开来，将服务调用方和服务实现者解耦，能够提升程序的扩展性、可维护性。修改或者替换服务实现并不需要修改调用方。

很多框架都使用了 Java 的 SPI 机制，比如：Spring 框架、数据库加载驱动、日志接口、以及 Dubbo 的扩展实现等等。

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(3)_image_3.png)

**那 SPI 和 API 有啥区别？**

说到 SPI 就不得不说一下 API 了，从广义上来说它们都属于接口，而且很容易混淆。下面先用一张图说明一下：

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(3)_image_4.png)

一般模块之间都是通过接口进行通讯，那我们在服务调用方和服务实现方（也称服务提供者）之间引入一个“接口”。

当实现方提供了接口和实现，我们可以通过调用实现方的接口从而拥有实现方给我们提供的能力，这就是 API ，这种接口和实现都是放在实现方的。

当接口存在于调用方这边时，就是 SPI ，由接口调用方确定接口规则，然后由不同的厂商去根据这个规则对这个接口进行实现，从而提供服务。

举个通俗易懂的例子：公司 H 是一家科技公司，新设计了一款芯片，然后现在需要量产了，而市面上有好几家芯片制造业公司，这个时候，只要 H 公司指定好了这芯片生产的标准（定义好了接口标准），那么这些合作的芯片公司（服务提供者）就按照标准交付自家特色的芯片（提供不同方案的实现，但是给出来的结果是一样的）。

通过 SPI 机制能够大大地提高接口设计的灵活性，但是 SPI 机制也存在一些缺点，比如：

- 需要遍历加载所有的实现类，不能做到按需加载，这样效率还是相对较低的。

- 当多个 

关于序列化和反序列化的详细解读，请看这篇文章 [Java 序列化详解](/java/basis/serialization.html) ，里面涉及到的知识点和面试题更全面。

如果我们需要持久化 Java 对象比如将 Java 对象保存在文件中，或者在网络传输 Java 对象，这些场景都需要用到序列化。

简单来说：

- 序列化：将数据结构或对象转换成二进制字节流的过程

- 反序列化：将在序列化过程中所生成的二进制字节流转换成数据结构或者对象的过程

对于 Java 这种面向对象编程语言来说，我们序列化的都是对象（Object）也就是实例化后的类(Class)，但是在 C++这种半面向对象的语言中，struct(结构体)定义的是数据结构类型，而 class 对应的是对象类型。

下面是序列化和反序列化常见应用场景：

- 对象在进行网络传输（比如远程方法调用 RPC 的时候）之前需要先被序列化，接收到序列化的对象之后需要再进行反序列化；

- 将对象存储到文件之前需要进行序列化，将对象从文件中读取出来需要进行反序列化；

- 将对象存储到数据库（如 Redis）之前需要用到序列化，将对象从缓存数据库中读取出来需要反序列化；

- 将对象存储到内存之前需要进行序列化，从内存中读取出来之后需要进行反序列化。

维基百科是如是介绍序列化的：

> **序列化**


综上：

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(3)_image_5.png)

**序列化协议对应于 TCP/IP 4 层模型的哪一层？**

我们知道网络通信的双方必须要采用和遵守相同的协议。TCP/IP 四层模型是下面这样的，序列化协议属于哪一层呢？

1. 应用层

1. 传输层

1. 网络层

1. 网络接口层

TCP/IP 四层模型

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(3)_image_6.png)

如上图所示，OSI 七层协议模型中，表示层做的事情主要就是对应用层的用户数据进行处理转换为二进制流。反过来的话，就是将二进制流转换成应用层的用户数据。这不就对应的是序列化和反序列化么？

因为，OSI 七层协议模型中的应用层、表示层和会话层对应的都是 TCP/IP 四层模型中的应用层，所以序列化协议属于 TCP/IP 协议应用层的一部分。

对于不想进行序列化的变量，使用 

transient

关于 

- transient

- transient

- static

JDK 自带的序列化方式一般不会用 ，因为序列化效率低并且存在安全问题。比较常用的序列化协议有 Hessian、Kryo、Protobuf、ProtoStuff，这些都是基于二进制的序列化协议。

像 JSON 和 XML 这种属于文本类序列化方式。虽然可读性比较好，但是性能较差，一般不会选择。

我们很少或者说几乎不会直接使用 JDK 自带的序列化方式，主要原因有下面这些原因：

- 不支持跨语言调用 : 如果调用的是其他语言开发的服务的时候就不支持了。

- 性能差：相比于其他序列化框架性能更低，主要原因是序列化之后的字节数组体积较大，导致传输成本加大。

- 存在安全问题：序列化和反序列化本身并不存在问题。但当输入的反序列化的数据可被用户控制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期的对象，在此过程中执行构造的任意代码。相关阅读：

关于 I/O 的详细解读，请看下面这几篇文章，里面涉及到的知识点和面试题更全面。

- Java IO 基础知识总结

- Java IO 设计模式总结

- Java IO 模型详解

IO 即 

Java IO 流的 40 多个类都是从如下 4 个抽象类基类中派生出来的。

- InputStream/Reader

- OutputStream/Writer

问题本质想问：

个人认为主要有两点原因：

- 字符流是由 Java 虚拟机将字节转换得到的，这个过程还算是比较耗时；

- 如果我们不知道编码类型的话，使用字节流的过程中很容易出现乱码问题。

参考答案：Java IO 设计模式总结

参考答案：Java IO 模型详解

**语法糖（Syntactic sugar）**

举个例子，Java 中的 

```
String[] strs = {"JavaGuide", "公众号：JavaGuide", "博客：
for (String s : strs) {
  	System.out.println(s);
}
```

不过，JVM 其实并不能识别语法糖，Java 语法糖要想被正确执行，需要先通过编译器进行解糖，也就是在程序编译阶段将其转换成 JVM 认识的基本语法。这也侧面说明，Java 中真正支持语法糖的是 Java 编译器而不是 JVM。如果你去看

Java 中最常用的语法糖主要有泛型、自动拆装箱、变长参数、枚举、内部类、增强 for 循环、try-with-resources 语法、lambda 表达式等。

关于这些语法糖的详细解读，请看这篇文章 [Java 语法糖详解](/java/basis/syntactic-sugar.html) 。