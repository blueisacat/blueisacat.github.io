# 一、Lombok是实现了什么功能？

Lombok是一款在java代码编译阶段改变代码的插件。比如生成getter和setter方法等，能够简化开发一些特定模板式代码。

# 二、Lombok是怎么实现的？

Lombok是基于APT、AST实现的，那么什么是APT、AST呢？

> APT是Lombok代码注入的处理时机AST是Lombok代码注入的核心方法


## 2.1 什么是APT？

APT即为Annotation Processing Tool，它是javac的一个工具，中文意思为编译时注解处理器。APT可以用来在编译时扫描和处理注解。通过APT可以获取到注解和被注解对象的相关信息，在拿到这些信息后我们可以根据需求来自动的生成一些代码，省去了手动编写。注意，获取注解及生成代码都是在代码编译时候完成的，相比反射在运行时处理注解大大提高了程序性能。APT的核心是AbstractProcessor类。

## 2.2 什么是AST？

AST即为Abstract Syntax Tree，中文意思为抽象语法树。AST是javac编译器阶段对源代码进行词法语法分析之后，语义分析之前进行的操作。用一个树形的结构表示源代码，源代码的每个元素映射到树上的节点。

# 三、代码实现

## 3.1 pom依赖

```
<dependencies>
    <!-- 自动生成META-INF/services/javax.annotation.processing.Processor文件 -->
    <dependency>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service</artifactId>
    </dependency>
    <!-- com.sun.tools -->
    <dependency>
        <groupId>com.sun</groupId>
        <artifactId>tools</artifactId>
        <version>1.8</version>
        <scope>system</scope>
        <systemPath>${java.home}/../lib/tools.jar</systemPath>
        <optional>true</optional>
    </dependency>
</dependencies>
```

## 3.1 自定义注解

```
// 只能标注在方法上
@Target({ElementType.METHOD})
// 只在源码期生效
@Retention(RetentionPolicy.SOURCE)
public @interface HelloWorld {
}
```

## 3.2 自定义注解处理器

```
@SupportedAnnotationTypes("cn.blueisacat.annotation.HelloWorld")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
@AutoService(Processor.class)
public class HelloWorldProcessor extends AbstractProcessor {
    /**
     * Messager接口提供注解处理器用来报告错误消息、警告和其他通知的方式
     * 它不是注解处理器开发者的日志工具，而是用来写一些信息给使用此注解器的第三方开发者的
     * 注意：我们应该对在处理过程中可能发生的异常进行捕获，通过Messager接口提供的方法通知用户（在官方文档中描述了消息的不同级别。非常重要的是Kind.ERROR）。
     * 此外，使用带有Element参数的方法连接到出错的元素，
     * 用户可以直接点击错误信息跳到出错源文件的相应行。
     * 如果你在process()中抛出一个异常，那么运行注解处理器的JVM将会崩溃（就像其他Java应用一样），
     * 这样用户会从javac中得到一个非常难懂出错信息
     */
    private Messager messager;
    /**
     * 实现Filer接口的对象，用于创建文件、类和辅助文件。
     * 使用Filer你可以创建文件
     * Filer中提供了一系列方法,可以用来创建class、java、resources文件
     * filer.createClassFile()[创建一个新的类文件，并返回一个对象以允许写入它]
     * filer.createResource() [创建一个新的源文件，并返回一个对象以允许写入它]
     * filer.createSourceFile() [创建一个用于写入操作的新辅助资源文件，并为它返回一个文件对象]
     */
    private Filer filer;
    /**
     * 用来处理Element的工具类
     * Elements接口的对象，用于操作元素的工具类。
     */
    private JavacElements elementUtils;
    /**
     * 用来处理TypeMirror的工具类
     * 实现Types接口的对象，用于操作类型的工具类。
     */
    private Types typeUtils;
    /**
     * 这个依赖需要将${JAVA_HOME}/lib/tools.jar 添加到项目的classpath,IDE默认不加载这个依赖
     */
    private JavacTrees trees;
    /**
     * 这个依赖需要将${JAVA_HOME}/lib/tools.jar 添加到项目的classpath,IDE默认不加载这个依赖
     * TreeMaker创建语法树节点的所有方法，创建时会为创建出来的JCTree设置pos字段，
     * 所以必须用上下文相关的TreeMaker对象来创建语法树节点，而不能直接new语法树节点。
     */
    private TreeMaker treeMaker;
    private Names names;
    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        messager = processingEnv.getMessager();
        filer = processingEnv.getFiler();
        elementUtils = (JavacElements) processingEnv.getElementUtils();
        typeUtils = processingEnv.getTypeUtils();
        this.trees = JavacTrees.instance(processingEnv);
        Context context = ((JavacProcessingEnvironment) processingEnv).getContext();
        this.treeMaker = TreeMaker.instance(context);
        this.names = Names.instance(context);
    }
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        // 通过注解获取到方法
        for (Element element : roundEnv.getElementsAnnotatedWith(HelloWorld.class)) {
            JCTree.JCMethodDecl jcMethodDecl = (JCTree.JCMethodDecl) elementUtils.getTree(element);
            treeMaker.pos = jcMethodDecl.pos;
            // 通过JCTree构建AST
            jcMethodDecl.body = treeMaker.Block(0, List.of(
                    treeMaker.Exec(
                            treeMaker.Apply(
                                    List.nil(),
                                    treeMaker.Select(
                                            treeMaker.Select(
                                                    treeMaker.Ident(elementUtils.getName("System")),
                                                    elementUtils.getName("out")
                                            ),
                                            elementUtils.getName("println")
                                    ),
                                    List.of(treeMaker.Literal("Hello World"))
                            )
                    ),
                    jcMethodDecl.body
            ));
        }
        return true;
    }
}
```

## 3.4 测试

将注解处理器打包，并在其他项目中引用，在方法上添加@HelloWorld注解。

```
public class Test {
    @HelloWorld
    public static void main(String[] args) {
        System.out.println("this is main");
    }
    
}
```

编译后，反编译查看代码。

```
public class Test {
    public static void main(String[] args) {
        System.out.println("Hello World");
        System.out.println("this is main");
    }
    
}
```

可以看到编译时已经自动插入了相应代码。

如果想要插入更多复杂的代码，那么需要深入了解下JCTree是如何修改语法树的。

# 四、如何使用JCTree修改语法树

## JCTree的介绍

JCTree是语法树元素的基类，包含一个重要的字段pos，该字段用于指明当前语法树节点（JCTree）在语法树中的位置，因此我们不能直接用new关键字来创建语法树节点，即使创建了也没有意义。此外，结合访问者模式，将数据结构与数据的处理进行解耦，部分源码如下：

```
public abstract class JCTree implements Tree, Cloneable, DiagnosticPosition {
    public int pos = -1;
    ...
    public abstract void accept(JCTree.Visitor visitor);
    ...
}
```

我们可以看到JCTree是一个抽象类，这里重点介绍几个JCTree的子类

1. JCStatement：声明语法树节点，常见的子类如下

- JCBlock：语句块语法树节点

- JCReturn：return语句语法树节点

- JCClassDecl：类定义语法树节点

- JCVariableDecl：字段/变量定义语法树节点

1. JCMethodDecl：方法定义语法树节点

1. JCModifiers：访问标志语法树节点

1. JCExpression：表达式语法树节点，常见的子类如下

- JCAssign：赋值语句语法树节点

- JCIdent：标识符语法树节点，可以是变量，类型，关键字等等

## TreeMaker介绍

TreeMaker用于创建一系列的语法树节点，我们上面说了创建JCTree不能直接使用new关键字来创建，所以Java为我们提供了一个工具，就是TreeMaker，它会在创建时为我们创建的JCTree对象设置pos字段，所以必须使用上下文相关的TreeMaker对象来创建语法树节点。

接下来着重介绍一下常用的几个方法。

### TreeMaker.Modifiers

**TreeMaker.Modifiers**

```
public JCModifiers Modifiers(long flags) {
    return Modifiers(flags, List.< JCAnnotation >nil());}
public JCModifiers Modifiers(long flags,
    List<JCAnnotation> annotations) {
        JCModifiers tree = new JCModifiers(flags, annotations);
        boolean noFlags = (flags & (Flags.ModifierFlags | Flags.ANNOTATION)) == 0;
        tree.pos = (noFlags && annotations.isEmpty()) ? Position.NOPOS : pos;
        return tree;
}
```

1. flags：访问标志

1. annotations：注解列表

其中flags可以使用枚举类com.sun.tools.javac.code.Flags来表示，例如我们可以这样用，就生成了下面的访问标志了。

> treeMaker.Modifiers(Flags.PUBLIC + Flags.STATIC + Flags.FINAL);public static final


### TreeMaker.ClassDef

**TreeMaker.ClassDef**

```
public JCClassDecl ClassDef(JCModifiers mods,
    Name name,
    List<JCTypeParameter> typarams,
    JCExpression extending,
    List<JCExpression> implementing,
    List<JCTree> defs) {
        JCClassDecl tree = new JCClassDecl(mods,
                                     name,
                                     typarams,
                                     extending,
                                     implementing,
                                     defs,
                                     null);
        tree.pos = pos;
        return tree;
}
```

1. mods：访问标志，可以通过TreeMaker.Modifiers来创建

1. name：类名

1. typarams：泛型参数列表

1. extending：父类

1. implementing：实现的接口

1. defs：类定义的详细语句，包括字段、方法的定义等等

### TreeMaker.MethodDef

**TreeMaker.MethodDef**

```
public JCMethodDecl MethodDef(JCModifiers mods,
    Name name,
    JCExpression restype,
    List<JCTypeParameter> typarams,
    List<JCVariableDecl> params,
    List<JCExpression> thrown,
    JCBlock body,
    JCExpression defaultValue) {
        JCMethodDecl tree = new JCMethodDecl(mods,
                                       name,
                                       restype,
                                       typarams,
                                       params,
                                       thrown,
                                       body,
                                       defaultValue,
                                       null);
        tree.pos = pos;
        return tree;}
public JCMethodDecl MethodDef(MethodSymbol m,
    Type mtype,
    JCBlock body) {
        return (JCMethodDecl)
            new JCMethodDecl(
                Modifiers(m.flags(), Annotations(m.getAnnotationMirrors())),
                m.name,
                Type(mtype.getReturnType()),
                TypeParams(mtype.getTypeArguments()),
                Params(mtype.getParameterTypes(), m),
                Types(mtype.getThrownTypes()),
                body,
                null,
                m).setPos(pos).setType(mtype);
}
```

1. mods：访问标志

1. name：方法名

1. restype：返回类型

1. typarams：泛型参数列表

1. params：参数列表

1. thrown：异常声明列表

1. body：方法体

1. defaultValue：默认方法（可能是interface中的哪个default）

1. m：方法符号

1. mtype：方法类型。包含多种类型，泛型参数类型、方法参数类型、异常参数类型、返回参数类型。

> 返回类型restype填写null或者treeMaker.TypeIdent(TypeTag.VOID)都代表返回void类型


### TreeMaker.VarDef

**TreeMaker.VarDef**

```
public JCVariableDecl VarDef(JCModifiers mods,
    Name name,
    JCExpression vartype,
    JCExpression init) {
        JCVariableDecl tree = new JCVariableDecl(mods, name, vartype, init, null);
        tree.pos = pos;
        return tree;}
public JCVariableDecl VarDef(VarSymbol v,
    JCExpression init) {
        return (JCVariableDecl)
            new JCVariableDecl(
                Modifiers(v.flags(), Annotations(v.getAnnotationMirrors())),
                v.name,
                Type(v.type),
                init,
                v).setPos(pos).setType(v.type);
}
```

1. mods：访问标志

1. name：参数名称

1. vartype：类型

1. init：初始化语句

1. v：变量符号

### TreeMaker.Ident

**TreeMaker.Ident**

```
public JCIdent Ident(Name name) {
        JCIdent tree = new JCIdent(name, null);
        tree.pos = pos;
        return tree;}
public JCIdent Ident(Symbol sym) {
        return (JCIdent)new JCIdent((sym.name != names.empty)
                                ? sym.name
                                : sym.flatName(), sym)
            .setPos(pos)
            .setType(sym.type);}
public JCExpression Ident(JCVariableDecl param) {
        return Ident(param.sym);
}
```

### TreeMaker.Return

**TreeMaker.Return**

```
public JCReturn Return(JCExpression expr) {
        JCReturn tree = new JCReturn(expr);
        tree.pos = pos;
        return tree;
}
```

### TreeMaker.Select

**TreeMaker.Select**

```
public JCFieldAccess Select(JCExpression selected,
    Name selector) 
{
        JCFieldAccess tree = new JCFieldAccess(selected, selector, null);
        tree.pos = pos;
        return tree;}
public JCExpression Select(JCExpression base,
    Symbol sym) {
        return new JCFieldAccess(base, sym.name, sym).setPos(pos).setType(sym.type);
}
```

1. selected：.运算符左边的表达式

1. selector：.运算符右边的表达式

> TreeMaker.Select(treeMaker.Ident(names.fromString("this")), names.fromString("name"));this.name


### TreeMaker.NewClass

**TreeMaker.NewClass**

```
public JCNewClass NewClass(JCExpression encl,
    List<JCExpression> typeargs,
    JCExpression clazz,
    List<JCExpression> args,
    JCClassDecl def) {
        JCNewClass tree = new JCNewClass(encl, typeargs, clazz, args, def);
        tree.pos = pos;
        return tree;
}
```

1. encl：不太明白此参数的含义，我看很多例子中此参数都设置为null

1. typeargs：参数类型列表

1. clazz：待创建对象的类型

1. args：参数列表

1. def：类定义

### TreeMaker.Apply

**TreeMaker.Apply**

```
public JCMethodInvocation Apply(List<JCExpression> typeargs,
    JCExpression fn,
    List<JCExpression> args) {
        JCMethodInvocation tree = new JCMethodInvocation(typeargs, fn, args);
        tree.pos = pos;
        return tree;
}
```

1. typeargs：参数类型列表

1. fn：调用语句

1. args：参数列表

### TreeMaker.Assign

**TreeMaker.Assign**

```
public JCAssign Assign(JCExpression lhs,
    JCExpression rhs) {
        JCAssign tree = new JCAssign(lhs, rhs);
        tree.pos = pos;
        return tree;
}
```

1. lhs：赋值语句左边表达式

1. rhs：赋值语句右边表达式

### TreeMaker.Exec

**TreeMaker.Exec**

```
public JCExpressionStatement Exec(JCExpression expr) {
        JCExpressionStatement tree = new JCExpressionStatement(expr);
        tree.pos = pos;
        return tree;
}
```

> TreeMaker.Apply以及TreeMaker.Assign就需要外面包一层TreeMaker.Exec来获得一个JCExpressionStatement


### TreeMaker.Block

**TreeMaker.Block**

```
public JCBlock Block(long flags,
    List<JCStatement> stats) {
        JCBlock tree = new JCBlock(flags, stats);
        tree.pos = pos;
        return tree;
}
```

1. flags：访问标志

1. stats：语句列表

## com.sun.tools.javac.util.List介绍

在我们操作抽象语法树的时候，有时会涉及到关于List的操作，但是这个List不是我们经常使用的 

```
public class List<A> extends AbstractCollection<A> implements java.util.List<A> {
    public A head;
    public List<A> tail;
    private static final List<?> EMPTY_LIST = new List<Object>((Object)null, (List)null) {
        public List<Object> setTail(List<Object> var1) {
            throw new UnsupportedOperationException();
        }
        public boolean isEmpty() {
            return true;
        }
    };
    List(A head, List<A> tail) {
        this.tail = tail;
        this.head = head;
    }
    public static <A> List<A> nil() {
        return EMPTY_LIST;
    }
    public List<A> prepend(A var1) {
        return new List(var1, this);
    }
    public List<A> append(A var1) {
        return of(var1).prependList(this);
    }
    public static <A> List<A> of(A var0) {
        return new List(var0, nil());
    }
    public static <A> List<A> of(A var0, A var1) {
        return new List(var0, of(var1));
    }
    public static <A> List<A> of(A var0, A var1, A var2) {
        return new List(var0, of(var1, var2));
    }
    public static <A> List<A> of(A var0, A var1, A var2, A... var3) {
        return new List(var0, new List(var1, new List(var2, from(var3))));
    }
    ...
}
```

## com.sun.tools.javac.util.ListBuffer介绍

由于 

```
public class ListBuffer<A> extends AbstractQueue<A> {
    public static <T> ListBuffer<T> of(T x) {
        ListBuffer<T> lb = new ListBuffer<T>();
        lb.add(x);
        return lb;
    }
    /** The list of elements of this buffer.
     */
    private List<A> elems;
    /** A pointer pointing to the last element of 'elems' containing data,
     *  or null if the list is empty.
     */
    private List<A> last;
    /** The number of element in this buffer.
     */
    private int count;
    /** Has a list been created from this buffer yet?
     */
    private boolean shared;
    /** Create a new initially empty list buffer.
     */
    public ListBuffer() {
        clear();
    }
    /** Append an element to buffer.
     */
    public ListBuffer<A> append(A x) {
        x.getClass(); // null check
        if (shared) copy();
        List<A> newLast = List.<A>of(x);
        if (last != null) {
            last.tail = newLast;
            last = newLast;
        } else {
            elems = last = newLast;
        }
        count++;
        return this;
    }
    ........
}
```

## com.sun.tools.javac.util.Names介绍

这个是为我们创建名称的一个工具类，无论是类、方法、参数的名称都需要通过此类来创建。它里面经常被使用到的一个方法就是 

```
Names names  = new Names()
names. fromString("setName");
```

## 举例说明

### 变量相关

在类中我们经常操作的参数就是变量，那么如何使用抽象语法树的特性为我们操作变量呢？接下来我们就将一些对于变量的一些操作。

#### 生成变量

例如生成 

```
// 生成参数 例如：private String age;
treeMaker.VarDef(
        treeMaker.Modifiers(Flags.PRIVATE), 
        names.fromString("age"), 
        treeMaker.Ident(
            names.fromString("String")
        ), 
        null
);
```

#### 变量赋值

例如我们想生成 

```
// private String name = "BuXueWuShu"
treeMaker.VarDef(
        treeMaker.Modifiers(Flags.PRIVATE),
        names.fromString("name"),
        treeMaker.Ident(
            names.fromString("String")
        ),
        treeMaker.Literal("BuXueWuShu")
)
```

#### 字面量相加

例如我们生成 

```
// add = "a"+"b"
treeMaker.Exec(
        treeMaker.Assign(
            treeMaker.Ident(
                names.fromString("add")
            ),
            treeMaker.Binary(
                JCTree.Tag.PLUS,
                treeMaker.Literal("a"),
                treeMaker.Literal("b")
            )
        )
)
```

#### +=语法

例如我们想生成 

```
// add+="test"
treeMaker.Exec(
        treeMaker.Assignop(
            JCTree.Tag.PLUS_ASG, 
            treeMaker.Ident(
                names.fromString("add")
            ), 
            treeMaker.Literal("test")
        )
)
```

#### ++语法

例如想生成 

```
treeMaker.Exec(
        treeMaker.Unary(
            JCTree.Tag.PREINC,
            treeMaker.Ident(
                names.fromString("i")
            )
        )
)
```

### 方法相关

我们对于变量进行了操作，那么基本上都是要生成方法的，那么如何对方法进行生成和操作呢？我们接下来演示一下关于方法相关的操作方法。

#### 无参无返回值

我们可以利用上面讲到的 

```
/*
    无参无返回值的方法生成
    public void test(){
    }
 */
JCTree.JCMethodDecl test = treeMaker.MethodDef(
        treeMaker.Modifiers(Flags.PUBLIC), // 方法限定值
        names.fromString("test"), // 方法名
        treeMaker.Type(new Type.JCVoidType()), // 返回类型
        com.sun.tools.javac.util.List.nil(),
        com.sun.tools.javac.util.List.nil(),
        com.sun.tools.javac.util.List.nil(),
        testBody,       // 方法体
        null
);
```

#### 有参无返回值

我们可以利用上面讲到的 

```
/*
    无参无返回值的方法生成
    public void test2(String name){
        name = "xxxx";
    }
 */
 
// 生成入参
JCTree.JCVariableDecl param = treeMaker.VarDef(
        treeMaker.Modifiers(Flags.PARAMETER), 
        names.fromString("name"),
        treeMaker.Ident(
            names.fromString("String")
        ), 
        null
);
com.sun.tools.javac.util.List<JCTree.JCVariableDecl> parameters = com.sun.tools.javac.util.List.of(param);
JCTree.JCMethodDecl test2 = treeMaker.MethodDef(
        treeMaker.Modifiers(Flags.PUBLIC), // 方法限定值
        names.fromString("test2"), // 方法名
        treeMaker.Type(new Type.JCVoidType()), // 返回类型
        com.sun.tools.javac.util.List.nil(),
        parameters, // 入参
        com.sun.tools.javac.util.List.nil(),
        testBody2,
        null
);
```

#### 有参有返回值

```
/*
    有参有返回值
    public String test3(String name){
       return name;
    }
 */
// 生成入参
JCTree.JCVariableDecl param3 = treeMaker.VarDef(
        treeMaker.Modifiers(Flags.PARAMETER), 
        names.fromString("name"),
        treeMaker.Ident(
            names.fromString("String")
        ), 
        null
);
com.sun.tools.javac.util.List<JCTree.JCVariableDecl> parameters3 = com.sun.tools.javac.util.List.of(param3);
JCTree.JCMethodDecl test3 = treeMaker.MethodDef(
        treeMaker.Modifiers(Flags.PUBLIC), // 方法限定值
        names.fromString("test4"), // 方法名
        treeMaker.Ident(
            names.fromString("String")
        ), // 返回类型
        com.sun.tools.javac.util.List.nil(),
        parameters3, // 入参
        com.sun.tools.javac.util.List.nil(),
        testBody3,
        null
);
```

#### 方法调用（无参）

```
JCTree.JCExpressionStatement exec = treeMaker.Exec(
        treeMaker.Apply(
                com.sun.tools.javac.util.List.nil(),
                treeMaker.Select(
                        treeMaker.Ident(
                            names.fromString("combatJCTreeMain")
                        ), // . 左边的内容
                        names.fromString("test") // . 右边的内容
                ),
                com.sun.tools.javac.util.List.nil()
        )
);
```

#### 方法调用（有参）

```
// 创建一个方法调用 combatJCTreeMain.test2("hello world!");
JCTree.JCExpressionStatement exec2 = treeMaker.Exec(
        treeMaker.Apply(
                com.sun.tools.javac.util.List.nil(),
                treeMaker.Select(
                        treeMaker.Ident(
                            names.fromString("combatJCTreeMain")
                        ), // . 左边的内容
                        names.fromString("test2") // . 右边的内容
                ),
                com.sun.tools.javac.util.List.of(
                    treeMaker.Literal("hello world!")
                ) // 方法中的内容
        )
);
```

### new对象

```
// 创建一个new语句 CombatJCTreeMain combatJCTreeMain = new CombatJCTreeMain();
JCTree.JCNewClass combatJCTreeMain = treeMaker.NewClass(
        null,
        com.sun.tools.javac.util.List.nil(),
        treeMaker.Ident(
            names.fromString("CombatJCTreeMain")
        ),
        com.sun.tools.javac.util.List.nil(),
        null
);
JCTree.JCVariableDecl jcVariableDecl1 = treeMaker.VarDef(
        treeMaker.Modifiers(Flags.PARAMETER),
        names.fromString("combatJCTreeMain"),
        treeMaker.Ident(
            names.fromString("CombatJCTreeMain")
        ),
        combatJCTreeMain
);
```

### if语句

```
/*
    创建一个if语句
    if("BuXueWuShu".equals(name)){
        add = "a" + "b";
    }else{
        add += "test";
    }
 */// "BuXueWuShu".equals(name)
JCTree.JCMethodInvocation apply = treeMaker.Apply(
        com.sun.tools.javac.util.List.nil(),
        treeMaker.Select(
                treeMaker.Literal("BuXueWuShu"), // . 左边的内容
                names.fromString("equals") // . 右边的内容
        ),
        com.sun.tools.javac.util.List.of(
            treeMaker.Ident(
                names.fromString("name")
            )
        )
);
//  add = "a" + "b"
JCTree.JCExpressionStatement exec3 = treeMaker.Exec(
    treeMaker.Assign(
        treeMaker.Ident(
            names.fromString("add")
        ), 
        treeMaker.Binary(
            JCTree.Tag.PLUS, 
            treeMaker.Literal("a"), 
            treeMaker.Literal("b")
        )
    )
);
//  add += "test"
JCTree.JCExpressionStatement exec1 = treeMaker.Exec(
    treeMaker.Assignop(
        JCTree.Tag.PLUS_ASG, 
        treeMaker.Ident(
            names.fromString("add")
        ), 
        treeMaker.Literal("test")
    )
);
JCTree.JCIf anIf = treeMaker.If(
        apply, // if语句里面的判断语句
        exec3, // 条件成立的语句
        exec1  // 条件不成立的语句
);
```