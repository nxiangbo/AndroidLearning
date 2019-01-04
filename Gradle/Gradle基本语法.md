# Gradle基本语法

我们知道，Gradle是基于Groovy语言的DSL。在编写Gradle脚本时，也遵从Groovy的基本语法。当然，也可以直接使用Java编写Gradle脚本。本文主要简单介绍Groovy的基本语法。

### 变量定义

Groovy 会根据对象的值来判断它的类型，不需要像Java那样显式的声明变量类型。

```groovy
def value = "Hello World"
```



### 循环



### 集合



### 闭包

**闭包**（英语：Closure），又称**词法闭包**（Lexical Closure）或**函数闭包**（function closures），是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。 

闭包是Groovy中非常重要的一个特性。Groovy中的闭包是一个开放的，匿名的代码块，可以接受参数，返回值并将其赋值给变量。



每一个闭包其实都是一个Closure对象。

### delegate，owner，this

查看Closure类的源码，可以发现闭包中有delegate、owner、thisObject三个成员变量，调用闭包没有的属性/方法时，会尝试在这三个变量上调用。一般情况下：

- this指向闭包外部的Object，指定义闭包的类。
- owner指向闭包外部的Object/Closure，指直接包含闭包的类或闭包。
- delegate默认和owner一致，指用于处理闭包属性/方法调用的第三方对象，可以修改。



委托策略

