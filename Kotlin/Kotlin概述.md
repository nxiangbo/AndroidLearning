# Kotlin学习笔记：概述



![](/Users/nxiangbo/Documents/AndroidLearning/Kotlin/images/Kotlin.png)

kotlin是一门面向对象的语言。可以用作Android和服务器端开发。它有以下特点

- 面向对象
- 静态类型语言

Kotlin虽是静态类型语言，但是Kotlin编译器提供了类型推导的功能，部分情况下可以根据上下文自动判断类型

- 函数式编程

头等函数：函数是一等公民，可以作为变量和参数传递；

不可变性：不可变对象在创建之后不可变化

无副作用：使用的纯函数。在输入相同情况下，会产生相同的结果

- 与Java互操作

与Java的类和方法相互调用，可以最大程度依赖Java库，如Kotlin完全基于Java的集合库

- 交互式shell

```kotlin
>>> println("hello world")
hello world
```



- 开源免费

Kotlin GitHub地址：https://github.com/JetBrains/kotlin

## 

此外，kotlin有很多地方和Java很像，因此可以与Java相互对照的学习。Java提供了javac编译器，kotlin也为kotlin代码提供了kotlinc编译器。kotlinc会将Kotlin代码编译成.class文件。然后，由JVM解释字节码，运行程序。

如果是Android开发者，Android Studio在Tools->Kotlin中提供了字节码查看工具等。当然，也可以使用javap命令查看字节码。通过字节码可以更好的理解Kotlin是如何运行的。



