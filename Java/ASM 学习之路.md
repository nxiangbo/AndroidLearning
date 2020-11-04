# ASM 学习之路


[AOP的利器：ASM3.0介绍](https://www.ibm.com/developerworks/cn/java/j-lo-asm30/index.html)
[使用ASM对Java字节码打桩](http://www.dengshenyu.com/java/2017/07/02/java-asm.html)
[ASM官网](https://asm.ow2.io/)

可以阅读并翻译这篇文章[Developer Guide](https://asm.ow2.io/developer-guide.html#introduction)



## Java 字节码

在插桩之前，需要先了解Java字节码的基本知识。众所周知，Java代码是由javac编译器编译成.class文件，而.class文件中存储的是可被Java虚拟机执行的字节码。因此，我们有必要弄清Java字节码是什么？字节码是如何执行的？

Java字节码是Java虚拟机中的指令集。每一个指令是由单字节的**操作码**加上零或多个**操作数**组成。

指令大致可以分为以下几类，详情可以参考[wiki](https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings)

- 加载和存储（例如aload_0，istore）
- 算术运算和逻辑运算（Iadd，fcmpl）
- 类型转换（i2b，d2i）
- 对象创建和操作（new，putfield）
- 操作数栈操作（swap，dup2）
- 控制转移（ifeq，goto）
- 方法调用和返回（invokespecial，areturn）

​    通过`javap -c`命令可以查看.class文件的字节码。

## ASM字节码操作

ASM 是一个 Java 字节码操控框架。它能被用来动态生成类或者增强既有类的功能。ASM 可以直接产生二进制 class 文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为。Java class 被存储在严格格式定义的 .class 文件里，这些类文件拥有足够的元数据来解析类中的所有元素：类名称、方法、属性以及 Java 字节码（指令）。ASM 从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。



在插桩的过程中，我们可能知道需要插入哪些代码，但是不知道如何修改成ASM需要的字节码格式。ASMifier 可以帮我们生成相应的字节码
```java
$ javac TestInstrumented.java
$ java -cp .:asm-all-5.0.3.jar org.objectweb.asm.util.ASMifier TestInstrumented
```

