> 本文整理完善自下面这两篇优秀的文章：
> 
> 


阅读过 JUC 源码的同学，一定会发现很多并发工具类都调用了一个叫做 Unsafe 的类。

那这个类主要是用来干什么的呢？有什么使用场景呢？这篇文章就带你搞清楚！

Unsafe

另外，

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_0.png)

**为什么要使用本地方法呢？**

1. 需要用到 Java 中不具备的依赖于操作系统的特性，Java 在实现跨平台的同时要实现对底层的控制，需要借助其他语言发挥作用。

1. 对于其他语言已经完成的一些现成功能，可以使用 Java 直接调用。

1. 程序对时间敏感或对性能要求非常高时，有必要使用更加底层的语言，例如 C/C++甚至是汇编。

在 JUC 包的很多并发工具类在实现并发机制时，都调用了本地方法，通过它们打破了 Java 运行时的界限，能够接触到操作系统底层的某些功能。对于同一本地方法，不同的操作系统可能会通过不同的方式来实现，但是对于使用者来说是透明的，最终都会得到相同的结果。

sun.misc.Unsafe

```
public final class Unsafe {
  // 单例对象
  private static final Unsafe theUnsafe;
  ......
  private Unsafe() {
  }
  @CallerSensitive
  public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    // 仅在引导类加载器`BootstrapClassLoader`加载时才合法
    if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
      throw new SecurityException("Unsafe");
    } else {
      return theUnsafe;
    }
  }
}
```

Unsafe

```
Exception in thread "main" java.lang.SecurityException: Unsafe
 at sun.misc.Unsafe.getUnsafe(Unsafe.java:90)
 at com.cn.test.GetUnsafeTest.main(GetUnsafeTest.java:12)
```

**为什么 **

这是因为在

**为什么要对 Unsafe 类进行这么谨慎的使用限制呢?**

Unsafe

**如若想使用 **

这里介绍两个可行的方案。

1、利用反射获得 Unsafe 类中已经实例化完成的单例对象 

```
private static Unsafe reflectGetUnsafe() {
    try {
      Field field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      return (Unsafe) field.get(null);
    } catch (Exception e) {
      log.error(e.getMessage(), e);
      return null;
    }
}
```

2、从

```
java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径
```

概括的来说，

1. 内存操作

1. 内存屏障

1. 对象操作

1. 数据操作

1. CAS 操作

1. 线程调度

1. Class 操作

1. 系统信息

如果你是一个写过 C 或者 C++ 的程序员，一定对内存操作不会陌生，而在 Java 中是不允许直接对内存进行操作的，对象内存的分配和回收都是由 JVM 自己实现的。但是在 

```
//分配新的本地空间
public native long allocateMemory(long bytes);
//重新调整内存空间的大小
public native long reallocateMemory(long address, long bytes);
//将内存设置为指定值
public native void setMemory(Object o, long offset, long bytes, byte value);
//内存拷贝
public native void copyMemory(Object srcBase, long srcOffset,Object destBase, long destOffset,long bytes);
//清除内存
public native void freeMemory(long address);
```

使用下面的代码进行测试：

```
private void memoryTest() {
    int size = 4;
    long addr = unsafe.allocateMemory(size);
    long addr3 = unsafe.reallocateMemory(addr, size * 2);
    System.out.println("addr: "+addr);
    System.out.println("addr3: "+addr3);
    try {
        unsafe.setMemory(null,addr ,size,(byte)1);
        for (int i = 0; i < 2; i++) {
            unsafe.copyMemory(null,addr,null,addr3+size*i,4);
        }
        System.out.println(unsafe.getInt(addr));
        System.out.println(unsafe.getLong(addr3));
    }finally {
        unsafe.freeMemory(addr);
        unsafe.freeMemory(addr3);
    }
}
```

先看结果输出：

```
addr: 2433733895744
addr3: 2433733894944
16843009
72340172838076673
```

分析一下运行结果，首先使用

你可以通过下图理解这个过程：

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_1.png)

在代码中调用

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_2.png)

拷贝完成后，使用

需要注意，通过这种方式分配的内存属于 堆外内存 ，是无法进行垃圾回收的，需要我们把这些内存当做一种资源去手动调用

**为什么要使用堆外内存？**

- 对垃圾回收停顿的改善。由于堆外内存是直接受操作系统管理而不是 JVM，所以当我们使用堆外内存时，即可保持较小的堆内内存规模。从而在 GC 时减少回收停顿对于应用的影响。

- 提升程序 I/O 操作的性能。通常在 I/O 通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存。

DirectByteBuffer

下图为 

```
DirectByteBuffer(int cap) {                   // package-private
    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);
    long base = 0;
    try {
        // 分配内存并返回基地址
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    // 内存初始化
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    // 跟踪 DirectByteBuffer 对象的垃圾回收，以实现堆外内存释放
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

在介绍内存屏障前，需要知道编译器和 CPU 会在保证程序输出结果一致的情况下，会对代码进行重排序，从指令优化角度提升性能。而指令重排序可能会带来一个不好的结果，导致 CPU 的高速缓存和内存中数据的不一致，而内存屏障（

在硬件层面上，内存屏障是 CPU 为了防止代码进行重排序而提供的指令，不同的硬件平台上实现内存屏障的方法可能并不相同。在 Java8 中，引入了 3 个内存屏障的函数，它屏蔽了操作系统底层的差异，允许在代码中定义、并统一由 JVM 来生成内存屏障指令，来实现内存屏障的功能。

Unsafe

```
//内存屏障，禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前
public native void loadFence();
//内存屏障，禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前
public native void storeFence();
//内存屏障，禁止load、store操作重排序
public native void fullFence();
```

内存屏障可以看做对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作。以

看到这估计很多小伙伴们会想到

```
@Getter
class ChangeThread implements Runnable{
    /**volatile**/ boolean flag=false;
    @Override
    public void run() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("subThread change flag to:" + flag);
        flag = true;
    }
}
```

在主线程的

```
public static void main(String[] args){
    ChangeThread changeThread = new ChangeThread();
    new Thread(changeThread).start();
    while (true) {
        boolean flag = changeThread.isFlag();
        unsafe.loadFence(); //加入读内存屏障
        if (flag){
            System.out.println("detected flag changed");
            break;
        }
    }
    System.out.println("main thread end");
}
```

运行结果：

```
subThread change flag to:false
detected flag changed
main thread end
```

而如果删掉上面代码中的

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_3.png)

了解 Java 内存模型（

在 Java 8 中引入了一种锁的新机制——

为了解决这个问题，

```
public boolean validate(long stamp) {
   U.loadFence();
   return (stamp & SBITS) == (state & SBITS);
}
```

**对象属性**

对象成员属性的内存偏移量获取，以及字段属性值的修改，在上面的例子中我们已经测试过了。除了前面的

```
//在对象的指定偏移地址获取一个对象引用
public native Object getObject(Object o, long offset);
//在对象指定偏移地址写入一个对象引用
public native void putObject(Object o, long offset, Object x);
```

除了对象属性的普通读写外，

```
//在对象的指定偏移地址处读取一个int值，支持volatile load语义
public native int getIntVolatile(Object o, long offset);
//在对象指定偏移地址处写入一个int，支持volatile store语义
public native void putIntVolatile(Object o, long offset, int x);
```

相对于普通读写来说，

有序写入的方法有以下三个：

```
public native void putOrderedObject(Object o, long offset, Object x);
public native void putOrderedInt(Object o, long offset, int x);
public native void putOrderedLong(Object o, long offset, long x);
```

有序写入的成本相对

- Load

- Store

顺序写入与

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_4.png)

在有序写入方法中，使用的是

综上所述，在上面的三类写入方法中，在写入效率方面，按照

**对象实例化**

使用 

```
@Data
public class A {
    private int b;
    public A(){
        this.b =1;
    }
}
```

分别基于构造函数、反射以及 

```
public void objTest() throws Exception{
    A a1=new A();
    System.out.println(a1.getB());
    A a2 = A.class.newInstance();
    System.out.println(a2.getB());
    A a3= (A) unsafe.allocateInstance(A.class);
    System.out.println(a3.getB());
}
```

打印结果分别为 1、1、0，说明通过

- 常规对象实例化方式：我们通常所用到的创建对象的方式，从本质上来讲，都是通过 new 机制来实现对象的创建。但是，new 机制有个特点就是当类只提供有参的构造函数且无显示声明无参构造函数时，则必须使用有参构造函数进行对象构造，而使用有参构造函数时，必须传递相应个数的参数才能完成对象实例化。

- 非常规的实例化方式：而 Unsafe 中提供 allocateInstance 方法，仅通过 Class 对象就可以创建此类的实例对象，而且不需要调用其构造函数、初始化代码、JVM 安全检查等。它抑制修饰符检测，也就是即使构造器是 private 修饰的也能通过此方法实例化，只需提类对象即可创建相应的对象。由于这种特性，allocateInstance 在 java.lang.invoke、Objenesis（提供绕过类构造器的对象生成方式）、Gson（反序列化时用到）中都有相应的应用。

arrayBaseOffset

```
//返回数组中第一个元素的偏移地址
public native int arrayBaseOffset(Class<?> arrayClass);
//返回数组中一个元素占用的大小
public native int arrayIndexScale(Class<?> arrayClass);
```

这两个与数据操作相关的方法，在 

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_5.png)

这部分主要为 CAS 相关操作的方法。

```
/**
	*  CAS
  * @param o         包含要修改field的对象
  * @param offset    对象中某field的偏移量
  * @param expected  期望值
  * @param update    更新值
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);
public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);
public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

**什么是 CAS?**

在 JUC 包的并发工具类中大量地使用了 CAS 操作，像在前面介绍

```
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);
```

参数中

```
private volatile int a;
public static void main(String[] args){
    CasTest casTest=new CasTest();
    new Thread(()->{
        for (int i = 1; i < 5; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
    new Thread(()->{
        for (int i = 5 ; i <10 ; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
}
private void increment(int x){
    while (true){
        try {
            long fieldOffset = unsafe.objectFieldOffset(CasTest.class.getDeclaredField("a"));
            if (unsafe.compareAndSwapInt(this,fieldOffset,x-1,x))
                break;
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
}
```

运行代码会依次输出：

```
1 2 3 4 5 6 7 8 9
```

在上面的例子中，使用两个线程去修改

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_6.png)

需要注意的是，在调用

Unsafe

```
//取消阻塞线程
public native void unpark(Object thread);
//阻塞线程
public native void park(boolean isAbsolute, long time);
//获得对象锁（可重入锁）
@Deprecated
public native void monitorEnter(Object o);
//释放对象锁
@Deprecated
public native void monitorExit(Object o);
//尝试获取对象锁
@Deprecated
public native boolean tryMonitorEnter(Object o);
```

方法 

此外，

```
//获得对象锁
@Deprecated
public native void monitorEnter(Object var1);
//释放对象锁
@Deprecated
public native void monitorExit(Object var1);
//尝试获得对象锁
@Deprecated
public native boolean tryMonitorEnter(Object var1);
```

monitorEnter

Java 锁和同步器框架的核心类 

```
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    UNSAFE.park(false, 0L);
    setBlocker(t, null);
}
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

LockSupport

```
public static void main(String[] args) {
    Thread mainThread = Thread.currentThread();
    new Thread(()->{
        try {
            TimeUnit.SECONDS.sleep(5);
            System.out.println("subThread try to unpark mainThread");
            unsafe.unpark(mainThread);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
    System.out.println("park main mainThread");
    unsafe.park(false,0L);
    System.out.println("unpark mainThread success");
}
```

程序输出为：

```
park main mainThread
subThread try to unpark mainThread
unpark mainThread success
```

程序运行的流程也比较容易看懂，子线程开始运行后先进行睡眠，确保主线程能够调用

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_7.png)

Unsafe

**静态属性读取相关的方法**

```
//获取静态属性的偏移量
public native long staticFieldOffset(Field f);
//获取静态属性的对象指针
public native Object staticFieldBase(Field f);
//判断类是否需要实例化（用于获取类的静态属性前进行检测）
public native boolean shouldBeInitialized(Class<?> c);
```

创建一个包含静态属性的类，进行测试：

```
@Data
public class User {
    public static String name="Hydra";
    int age;
}
private void staticTest() throws Exception {
    User user=new User();
    System.out.println(unsafe.shouldBeInitialized(User.class));
    Field sexField = User.class.getDeclaredField("name");
    long fieldOffset = unsafe.staticFieldOffset(sexField);
    Object fieldBase = unsafe.staticFieldBase(sexField);
    Object object = unsafe.getObject(fieldBase, fieldOffset);
    System.out.println(object);
}
```

运行结果：

```
falseHydra
```

在 

在上面的代码中首先创建一个

```
truenull
```

**使用**

```
public native Class<?> defineClass(String name, byte[] b, int off, int len, ClassLoader loader,ProtectionDomain protectionDomain);
```

在实际使用过程中，可以只传入字节数组、起始字节的下标以及读取的字节长度，默认情况下，类加载器（

```
private static void defineTest() {
    String fileName="F:\\workspace\\unsafe-test\\target\\classes\\com\\cn\\model\\User.class";
    File file = new File(fileName);
    try(FileInputStream fis = new FileInputStream(file)) {
        byte[] content=new byte[(int)file.length()];
        fis.read(content);
        Class clazz = unsafe.defineClass(null, content, 0, content.length, null, null);
        Object o = clazz.newInstance();
        Object age = clazz.getMethod("getAge").invoke(o, null);
        System.out.println(age);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

在上面的代码中，首先读取了一个

![](../../../../assets/images/Java/JavaGuide/基础/重点知识点/attachments/Java%20魔法类%20Unsafe%20详解_image_8.png)

除了

```
public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
```

使用该方法可以用来动态的创建一个匿名类，在

Lambda 表达式实现需要依赖 

这部分包含两个获取系统相关信息的方法。

```
//返回系统指针的大小。返回值为4（32位系统）或 8（64位系统）。
public native int addressSize();
//内存页的大小，此值为2的幂次方。
public native int pageSize();
```

这两个方法的应用场景比较少，在

在本文中，我们首先介绍了 