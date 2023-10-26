## 面向对象基础

两者的主要区别在于解决问题的方式不同：

- 面向过程把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题。

- 面向对象会先抽象出对象，然后用对象执行方法的方式解决问题。

另外，面向对象开发的程序一般更易维护、易复用、易扩展。

相关 issue : [面向过程：面向过程性能比面向对象高？？](https://github.com/Snailclimb/JavaGuide/issues/431) 。

下面是一个求圆的面积和周长的示例，简单分别展示了面向对象和面向过程两种不同的解决方案。

**面向对象**

```
public class Circle {
    // 定义圆的半径
    private double radius;
    // 构造函数
    public Circle(double radius) {
        this.radius = radius;
    }
    // 计算圆的面积
    public double getArea() {
        return Math.PI * radius * radius;
    }
    // 计算圆的周长
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
    public static void main(String[] args) {
        // 创建一个半径为3的圆
        Circle circle = new Circle(3.0);
        // 输出圆的面积和周长
        System.out.println("圆的面积为：" + circle.getArea());
        System.out.println("圆的周长为：" + circle.getPerimeter());
    }
}
```

我们定义了一个 

**面向过程**

```
public class Main {
    public static void main(String[] args) {
        // 定义圆的半径
        double radius = 3.0;
        // 计算圆的面积和周长
        double area = Math.PI * radius * radius;
        double perimeter = 2 * Math.PI * radius;
        // 输出圆的面积和周长
        System.out.println("圆的面积为：" + area);
        System.out.println("圆的周长为：" + perimeter);
    }
}
```

我们直接定义了圆的半径，并使用该半径直接计算出圆的面积和周长。

new 运算符，new 创建对象实例（对象实例在堆内存中），对象引用指向对象实例（对象引用存放在栈内存中）。

- 一个对象引用可以指向 0 个或 1 个对象（一根绳子可以不系气球，也可以系一个气球）；

- 一个对象可以有 n 个引用指向它（可以用 n 条绳子系住一个气球）。

- 对象的相等一般比较的是内存中存放的内容是否相等。

- 引用相等一般比较的是他们指向的内存地址是否相等。

这里举一个例子：

```
String str1 = "hello";
String str2 = new String("hello");
String str3 = "hello";
// 使用 == 比较字符串的引用相等
System.out.println(str1 == str2);
System.out.println(str1 == str3);
// 使用 equals 方法比较字符串的相等
System.out.println(str1.equals(str2));
System.out.println(str1.equals(str3));
```

输出结果：

```
false
true
true
true
```

从上面的代码输出结果可以看出：

- str1

- str1

构造方法是一种特殊的方法，主要作用是完成对象的初始化工作。

如果一个类没有声明构造方法，也可以执行！因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法。如果我们自己添加了类的构造方法（无论是否有参），Java 就不会添加默认的无参数的构造方法了。

我们一直在不知不觉地使用构造方法，这也是为什么我们在创建对象的时候后面要加一个括号（因为要调用无参的构造方法）。如果我们重载了有参的构造方法，记得都要把无参的构造方法也写出来（无论是否用到），因为这可以帮助我们在创建对象的时候少踩坑。

构造方法特点如下：

- 名字与类名相同。

- 没有返回值，但不能用 void 声明构造函数。

- 生成类的对象时自动执行，无需调用。

构造方法不能被 override（重写）,但是可以 overload（重载）,所以你可以看到一个类中有多个构造函数的情况。

封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。就好像我们看不到挂在墙上的空调的内部的零件信息（也就是属性），但是可以通过遥控器（方法）来控制空调。如果属性不想被外界访问，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。就好像如果没有空调遥控器，那么我们就无法操控空凋制冷，空调本身就没有意义了（当然现在还有很多其他方法 ，这里只是为了举例子）。

```
public class Student {
    private int id;//id属性私有化
    private String name;//name属性私有化
    //获取id的方法
    public int getId() {
        return id;
    }
    //设置id的方法
    public void setId(int id) {
        this.id = id;
    }
    //获取name的方法
    public String getName() {
        return name;
    }
    //设置name的方法
    public void setName(String name) {
        this.name = name;
    }
}
```

不同类型的对象，相互之间经常有一定数量的共同点。例如，小明同学、小红同学、小李同学，都共享学生的特性（班级、学号等）。同时，每一个对象还定义了额外的特性使得他们与众不同。例如小明的数学比较好，小红的性格惹人喜爱；小李的力气比较大。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

**关于继承如下 3 点请记住：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，

1. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。

1. 子类可以用自己的方式实现父类的方法。（以后介绍）。

多态，顾名思义，表示一个对象具有多种的状态，具体表现为父类的引用指向子类的实例。

**多态的特点:**

- 对象类型和引用类型之间具有继承（类）/实现（接口）的关系；

- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定；

- 多态不能调用“只在子类存在但在父类不存在”的方法；

- 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法。

**共同点**

- 都不能被实例化。

- 都可以包含抽象方法。

- 都可以有默认实现的方法（Java 8 可以用 

**区别**

- 接口主要用于对类的行为进行约束，你实现了某个接口就具有了对应的行为。抽象类主要用于代码复用，强调的是所属关系。

- 一个类只能继承一个类，但是可以实现多个接口。

- 接口中的成员变量只能是 

关于深拷贝和浅拷贝区别，我这里先给结论：

- 浅拷贝：浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址，也就是说拷贝对象和原对象共用同一个内部对象。

- 深拷贝：深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。

上面的结论没有完全理解的话也没关系，我们来看一个具体的案例！

浅拷贝的示例代码如下，我们这里实现了 

clone()

```
public class Address implements Cloneable{
    private String name;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
public class Person implements Cloneable {
    private Address address;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Person clone() {
        try {
            Person person = (Person) super.clone();
            return person;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

测试：

```
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// true
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出， 

这里我们简单对 

```
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone();
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

测试：

```
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// false
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出，虽然 

**那什么是引用拷贝呢？**

我专门画了一张图来描述浅拷贝、深拷贝、引用拷贝：

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_0.png)

Object 类是一个特殊的类，是所有类的父类。它主要提供了以下 11 个方法：

```
/**
 * native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
 */
public final native Class<?> getClass()
/**
 * native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
 */
public native int hashCode()
/**
 * 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
 */
public boolean equals(Object obj)
/**
 * naitive 方法，用于创建并返回当前对象的一份拷贝。
 */
protected native Object clone() throws CloneNotSupportedException
/**
 * 返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
 */
public String toString()
/**
 * native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
 */
public final native void notify()
/**
 * native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
 */
public final native void notifyAll()
/**
 * native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
 */
public final native void wait(long timeout) throws InterruptedException
/**
 * 多了 nanos 参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 毫秒。。
 */
public final void wait(long timeout, int nanos) throws InterruptedException
/**
 * 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
 */
public final void wait() throws InterruptedException
/**
 * 实例被垃圾回收器回收的时候触发的操作
 */
protected void finalize() throws Throwable { }
```

**==**

- 对于基本数据类型来说，

- 对于引用数据类型来说，

> 因为 Java 只有值传递，所以，对于 == 来说，不管是比较基本数据类型，还是引用数据类型的变量，其本质比较的都是值，只是引用类型变量存的值是对象的地址。


**equals()**

Object

```
public boolean equals(Object obj) {
     return (this == obj);
}
```

equals()

- 类没有重写 

- 类重写了 

举个例子（这里只是为了举例。实际上，你按照下面这种写法的话，像 IDEA 这种比较智能的 IDE 都会提示你将 

```
String a = new String("ab"); // a 为一个引用
String b = new String("ab"); // b为另一个引用,对象的内容一样
String aa = "ab"; // 放在常量池中
String bb = "ab"; // 从常量池中查找
System.out.println(aa == bb);// true
System.out.println(a == b);// false
System.out.println(a.equals(b));// true
System.out.println(42 == 42.0);// true
```

String

当创建 

String

```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

hashCode()

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_1.png)

hashCode()

> ⚠️ 注意：该方法在 
> 
> 


```
public native int hashCode();
```

散列表存储的是键值对(key-value)，它的特点是：

我们以“

下面这段内容摘自我的 Java 启蒙书《Head First Java》:

> 当你把对象加入 


其实， 

**那为什么 JDK 还要同时提供这两个方法呢？**

这是因为在一些容器（比如 

我们在前面也提到了添加元素进

**那为什么不只提供 **

这是因为两个对象的

**那为什么两个对象有相同的 **

因为 

总结下来就是：

- 如果两个对象的

- 如果两个对象的

- 如果两个对象的

相信大家看了我前面对 

因为两个相等的对象的 

如果重写 

**思考**

**总结**

- equals

- 两个对象有相同的 

更多关于 [Java hashCode() 和 equals()的若干问题解答open in new window](https://www.cnblogs.com/skywang12345/p/3324958.html)

**可变性**

String

StringBuilder

```
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
  	//...
}
```

**线程安全性**

String

**性能**

每次对 

**对于三者使用的总结：**

1. 操作少量的数据: 适用 

1. 单线程操作字符串缓冲区下操作大量数据: 适用 

1. 多线程操作字符串缓冲区下操作大量数据: 适用 

String

```
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[];
	//...
}
```

> 🐛 修正：我们知道被 
> 
> String
> 保存字符串的数组被 
> String


相关阅读：[如何理解 String 类型值的不可变？ - 知乎提问](https://www.zhihu.com/question/20618891/answer/114125846)

补充（来自[issue 675](https://github.com/Snailclimb/JavaGuide/issues/675)）：在 Java 9 之后，

```
public final class String implements java.io.Serializable,Comparable<String>, CharSequence {
    // @Stable 注解表示变量最多被修改一次，称为“稳定的”。
    @Stable
    private final byte[] value;
}

abstract class AbstractStringBuilder implements Appendable, CharSequence {
    byte[] value;

}

```

[](#字符串拼接用-还是-stringbuilder)**Java 9 为何要将 **

新版的 

JDK 官方就说了绝大部分字符串对象只包含 

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_2.png)

如果字符串中包含的汉字超过 Latin-1 可表示范围内的字符，

这是官方的介绍：[https://openjdk.java.net/jeps/254](https://openjdk.java.net/jeps/254) 。[](#字符串拼接用-还是-stringbuilder)

### 字符串拼接用“+” 还是 StringBuilder?

Java 语言本身并不支持运算符重载，“+”和“+=”是专门为 String 类重载过的运算符，也是 Java 中仅有的两个重载过的运算符。

```
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

上面的代码对应的字节码如下：

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_3.png)

可以看出，字符串对象通过“+”的字符串拼接方式，实际上是通过 

不过，在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：

```
String[] arr = {"he", "llo", "world"};
String s = "";
for (int i = 0; i < arr.length; i++) {
    s += arr[i];
}
System.out.println(s);
```

StringBuilder

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_4.png)

如果直接使用 

```
String[] arr = {"he", "llo", "world"};
StringBuilder s = new StringBuilder();
for (String value : arr) {
    s.append(value);
}
System.out.println(s);
```

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_5.png)

如果你使用 IDEA 的话，IDEA 自带的代码检查机制也会提示你修改代码。

不过，使用 “+” 进行字符串拼接会产生大量的临时对象的问题在 JDK9 中得到了解决。在 JDK9 当中，字符串相加 “+” 改为了用动态方法 [JEP 280](https://openjdk.org/jeps/280) 提出的，这也意味着 JDK 9 之后，你可以放心使用“+” 进行字符串拼接了。关于这部分改进的详细介绍，推荐阅读这篇文章：还在无脑用 [StringBuilder？来重温一下字符串拼接吧](https://juejin.cn/post/7182872058743750715) 。

String

**字符串常量池**

```
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String aa = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String bb = "ab";
System.out.println(aa==bb);// true
```

更多关于字符串常量池的介绍可以看一下 [Java 内存区域详解](https://javaguide.cn/java/jvm/memory-area.html) 这篇文章。

会创建 1 或 2 个字符串对象。

1、如果字符串常量池中不存在字符串对象“abc”的引用，那么会在堆中创建 2 个字符串对象“abc”。

示例代码（JDK 1.8）：

```
String s1 = new String("abc");
```

对应的字节码：

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_6.png)

ldc

2、如果字符串常量池中已存在字符串对象“abc”的引用，则只会在堆中创建 1 个字符串对象“abc”。

示例代码（JDK 1.8）：

```
// 字符串常量池中已存在字符串对象“abc”的引用
String s1 = "abc";
// 下面这段代码只会在堆中创建 1 个字符串对象“abc”
String s2 = new String("abc");
```

对应的字节码：

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_7.png)

这里就不对上面的字节码进行详细注释了，7 这个位置的 

String.intern()

- 如果字符串常量池中保存了对应的字符串对象的引用，就直接返回该引用。

- 如果字符串常量池中没有保存了对应的字符串对象的引用，那就在常量池中创建一个指向该字符串对象的引用并返回。

示例代码（JDK 1.8） :

```
// 在堆中创建字符串对象”Java“
// 将字符串对象”Java“的引用保存在字符串常量池中
String s1 = "Java";
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s2 = s1.intern();
// 会在堆中在单独创建一个字符串对象
String s3 = new String("Java");
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s4 = s3.intern();
// s1 和 s2 指向的是堆中的同一个对象
System.out.println(s1 == s2); // true
// s3 和 s4 指向的是堆中不同的对象
System.out.println(s3 == s4); // false
// s1 和 s4 指向的是堆中的同一个对象
System.out.println(s1 == s4); //true
```

先来看字符串不加 

```
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";
String str4 = str1 + str2;
String str5 = "string";
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

> **注意**


![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_8.png)

**对于编译期可以确定值的字符串，也就是常量字符串 ，jvm 会将其存入字符串常量池。并且，字符串常量拼接得到的字符串常量在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。**

在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做 

![](../../../assets/images/Java/JavaGuide/基础/attachments/Java基础常见面试题总结(2)_image_9.png)

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( 

- final

- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 

```
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 

不过，字符串使用 

示例代码：

```
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true
```

被 

如果 ，编译器在运行时才能知道其确切值的话，就无法对其优化。

示例代码（

```
final String str1 = "str";
final String str2 = getStr();
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 在堆上创建的新的对象
System.out.println(c == d);// false
public static String getStr() {
      return "ing";
}
```

- 深入解析 String#intern：

- R 大（RednaxelaFX）关于常量折叠的回答：