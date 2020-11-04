# Java类加载机制
## 类的加载时机
类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）7个阶段。

### 加载

### 验证
验证主要分为：文件格式验证、元数据验证、字节码验证、符号引用验证。
### 准备
准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在Java堆中。

变量value在准备阶段过后的初始值为0而不是123，因为这时候尚未开始执行任何Java方法。
```
public static int value=123；
```

如果类字段的字段属性表中存在ConstantValue属性，那在准备阶段变量value就会被初始化为ConstantValue属性所指定的值。在准备阶段就会将value值赋为123。
```
public static final int value=123；
```


### 解析
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

- 符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。

- 直接引用：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。

### 初始化
初始化阶段是执行类构造器`＜clinit＞（）`方法的过程。＜clinit＞（）方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问。

### 使用

## 类加载器的基本概念

类加载器（class loader）用来加载 Java 类到 Java 虚拟机中。一般来说，Java 虚拟机使用 Java 类的方式如下：Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class 文件）。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class类的一个实例。每个这样的实例用来表示一个 Java 类。通过此实例的 newInstance()方法就可以创建出该类的一个对象。实际的情况可能更加复杂，比如 Java 字节代码可能是通过工具动态生成的，也可能是通过网络下载的。

## java.lang.ClassLoader类介绍
java.lang.ClassLoader类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个 Java 类，即 java.lang.Class类的一个实例。除此之外，ClassLoader还负责加载 Java 应用所需的资源，如图像文件和配置文件等。不过本文只讨论其加载类的功能。为了完成加载类的这个职责，ClassLoader提供了一系列的方法，比较重要的方法如 表 1所示。关于这些方法的细节会在下面进行介绍。
表 1. ClassLoader 中与加载类相关的方法

|方法	|说明|
|-------|----|
|getParent()	|返回该类加载器的父类加载器。|
| loadClass(String name) |	加载名称为 name的类，返回的结果是 java.lang.Class类的实例。|
| findClass(String name)	| 查找名称为 name的类，返回的结果是 java.lang.Class类的实例。|
|findLoadedClass(String name)	| 查找名称为 name的已经被加载过的类，返回的结果是 java.lang.Class类的实例。|
|defineClass(String name, byte[] b, int off, int len) |	把字节数组 b中的内容转换成 Java 类，返回的结果是 java.lang.Class类的实例。这个方法被声明为 final的。|
|resolveClass(Class<?> c) |	链接指定的 Java 类。|

## 类加载器的树状组织结构

Java 中的类加载器大致可以分成两类，一类是系统提供的，另外一类则是由 Java 应用开发人员编写的。系统提供的类加载器主要有下面三个：

- 引导类加载器（bootstrap class loader）：它用来加载 Java 的核心库，是用原生代码来实现的，并不继承自java.lang.ClassLoader。

- 扩展类加载器（extensions class loader）：它用来加载 Java 的扩展库。Java虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。

- 系统类加载器（system class loader）：它根据 Java应用的类路径（CLASSPATH）来加载 Java类。一般来说，Java 应用的类都是由它来完成加载的。可以通过ClassLoader.getSystemClassLoader()来获取它。

除了系统提供的类加载器以外，开发人员可以通过继承 java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求。

## 双亲委派模型
如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。
![](Java%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/D3698D4D-147C-4014-B2FC-B8FB7C5D5646.png)

显而易见的好处就是Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类java.lang.Object，它存放在rt.jar之中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。







#Java虚拟机