# Kotlin 学习笔记：基本语法和函数





## ![Kotlin-函数](images\Kotlin-函数.png)



```kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
	}
>>> println(getMnemonic(Color.BLUE))
Battle
```



```kotlin
fun main(args: Array<String>) {
	val name = if (args.size > 0) args[0] else "Kotlin"
	println("Hello, $name!")
}
```



## 函数

### 声明函数的语法

![](images\Kotlin-functiondefine.png)



Kotlin也有与Java相对应的main函数，定义方式如下

```kotlin
fun main(args: Array<String>) {
    println("hello Kotlin")
}
```





### 参数

- 命名参数



- 支持默认参数值



假如需要写一个在list列表中将元素用分隔符分割并打印以String类型打印的功能。

一般情况下，可以这样写。

```kotlin
fun <T>joinToString(collection: Collection<T>, 
                    separator: String, 
                    prefix: String, 
                    suffix: String): String{
    val result = StringBuilder(prefix)

    for((index, element) in collection.withIndex()) {
        if (index >0) result.append(separator)
        result.append(element)
    }
    result.append(suffix)
    return result.toString()
}
```



为了解决过度重载的问题，可以使用默认参数值。

```kotlin
fun <T>joinToString(collection: Collection<T>, 
                     separator: String=",", 
                     prefix: String="", 
                     suffix: String=""): String{
    val result = StringBuilder(prefix)

    for((index, element) in collection.withIndex()) {
        if (index >0) result.append(separator)
        result.append(element)
    }
    result.append(suffix)

    return result.toString()
}
```

可以使用以下方式调用

```kotlin
joinToString(list)
joinToString(list, ";")
```



### 顶层函数

在Java中经常会用到一些工具类，这些工具类中往往都是静态方法，没有类的实例对象，例如Jdk中的Collections类，以及我们自己定义的一些放在util包下的工具类。在Kotlin中，完全不需要这些类。Kotlin引入了顶层函数，可以替代静态类。

那么如何定义顶层函数呢？

创建一个Kotlin文件，然后将函数添加进去即可。以joinToString为例，

```kotlin
fun <T>joinToString(...): String{
  ....
}
```

等价于

```kotlin
/* Java */
package strings;
public class JoinKt {
	public static String joinToString(...) { ... }
}
```





### 扩展函数

定义：函数定义在类的外部，但可以像调用该类的成员函数一样调用

- 扩展函数的语法

接收者类型：要扩展类或接口的名称，放到要添加的函数前。

接收者对象：用来调用扩展函数的对象

![](images\Kotlin-extensionfunction.png)

- 扩展函数如何调用及优点

扩展函数可以在不更改原来类的情况下，对类的方法进行拓展。扩展函数可以像被扩展类的方法一样调用。

举例来讲，自定义一个String类的扩展函数lastChar，该函数可以取字符串的最后一个字符。

```kotlin
fun String.lastChar(): Char = get(length-1)
```



在Kotlin代码中调用

```kotlin
import com.nxiangbo.kotlin_learning.function.lastChar
"str".lastChar()
```

在Java代码中调用

```java
import com.nxiangbo.kotlin_learning.function.FunctionLearnKt;
public class FunctionTest {
	public void Test(){
		FunctionLearnKt.lastChar("str");
	}
}
```



- 扩展函数能做什么

在Kotlin标准库中，大量使用扩展函数。例如，String等





### 函数与集合



### 字符串和正则表达式





