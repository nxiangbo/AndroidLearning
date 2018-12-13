# Kotlin学习笔记：类型系统

![](E:\AndroidLearning\Kotlin\images\Kotlin-type.png)



## Null 类型处理

Kotlin类型系统解决了空指针问题。解决方式是将运行时的错误转变成编译时错误。

在Kotlin中，默认情况下，变量不能存储null引用，否则编译时报错。

```kotlin
fun strLen(s: String) = s.length
strLen(null) //编译不通过
```



- 可null操作符（?）

若需要变量为null，可以使用**？**操作符。一旦有一个可null类型的值，对它的操作也会受限制。

不能把它赋值给非null类型的对象

```kotlin
val str:String? = null
val str2:String = str // 编译不通过
```

也不能把可null类型的对象传给拥有非null类型参数的函数。

```kotlin
val str: String? = null
strLen(str) // 编译不通过
```

如果一定要调用strLen方法，需要将方法进行null处理

```kotlin
fun strLen2(s:String?):Int = if (s!=null) s.length else 0
val str: String? = null
strLen(str) // 正常运行，返回0
```

- 安全调用运算符（?.）

安全调用运算符允许把一次null检查和一次方法调用合并成一个操作。

例如

```kotlin
str?.toUpperCase()
```

等价于

```kotlin
if (str != null) str.toUpperCase() else null
```

- Elvis运算符(?:)

elvis 接收两个运算数，如果第一个运算数不为null，则返回第一个运算数；如果为null，则返回第二个运算数。

![](E:\AndroidLearning\Kotlin\images\type-elvis.PNG)



- 安全转换(as?)

![](E:\AndroidLearning\Kotlin\images\type-as.PNG)

- 非null断言（!!）

![](E:\AndroidLearning\Kotlin\images\type-gantanhao.PNG)



- let函数

let函数可以用于处理可null表达式，它允许对表达式进行求值，并判断它是否为null。

举例

有一个发送邮件的函数

```kotlin
fun sendEmailTo(email: String) { /*...*/ }
```

调用发送邮件函数。由于email可以为null，编译不能通过。

```kotlin
>>> val email: String? = ...
>>> sendEmailTo(email)
ERROR: Type mismatch: inferred type is String? but String was expected
```

可以进行null判断

```kotlin
if (email != null) sendEmailTo(email)
```

也可以使用let函数

```kotlin
email?.let { email -> sendEmailTo(email) }
```

![](E:\AndroidLearning\Kotlin\images\type.PNG)



## 基本类型

与Java不同，Kotlin不区分基本数据类型和引用类型。在大多数情况下，Int类型会被编译为java的int类型。在一些情况下，比如作为泛型的类型参数时，会被编译为Java的Integer类型。

对应到Java基本类型的类型完整列表如下：

整数类型—Byte, Short, Int, Long
浮点数类型—Float, Double
字符类型—Char
布尔类型—Boolean



- Any：Any类型是所有类型的父类，类似于Java的Object
- Unit：类似于Java的void。
- Nothing：没有任何值，只有被当做返回值使用或者当做泛型函数返回值的类型参数

## 集合

Kotlin使用的是Java的集合框架。因此，本质上每一个Kotlin集合接口都对应着一个Java集合接口的实例。

与Java不同的是，Kotlin的集合分为两种，一种是只读集合，一种是可变集合。

![](E:\AndroidLearning\Kotlin\images\type-collection.PNG)

集合创建函数列表

![](E:\AndroidLearning\Kotlin\images\type-collection2.PNG)



