# Kotlin学习笔记：注解和反射

![](E:\AndroidLearning\Kotlin\images\kotlin-annotation and reflect.png)



## 注解

自定义注解

```kotlin
annotation class MyAnnotation{
    
}
```



元注解

- [`@Target`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html) 指定可以用该注解标注的元素的可能的类型（类、函数、属性、表达式等）；
- [`@Retention`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html) 指定该注解是否存储在编译后的 class 文件中，以及它在运行时能否通过反射可见 （默认都是 true）；
- [`@Repeatable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html) 允许在单个元素上多次使用相同的该注解；
- [`@MustBeDocumented`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html) 指定该注解是公有 API 的一部分，并且应该包含在生成的 API 文档中显示的类或方法的签名中。

### 注解使用处目标

当对属性或主构造函数参数进行标注时，从相应的 Kotlin 元素生成的 Java 元素会有多个，因此在生成的 Java 字节码中该注解有多个可能位置 。如果要指定精确地指定应该如何生成该注解，请使用以下语法：

```kotlin
class Example(@field:Ann val foo,    // 标注 Java 字段
              @get:Ann val bar,      // 标注 Java getter
              @param:Ann val quux)   // 标注 Java 构造函数参数
```



支持的使用处目标的完整列表为：

- `file`；
- `property`（具有此目标的注解对 Java 不可见）；
- `field`；
- `get`（属性 getter）；
- `set`（属性 setter）；
- `receiver`（扩展函数或属性的接收者参数）；
- `param`（构造函数参数）；
- `setparam`（属性 setter 参数）；
- `delegate`（为委托属性存储其委托实例的字段）。

## 反射

反射，简单点说，就是可以在运行时动态的访问对象属性和方法的方式



### Kotlin 反射API

![](E:\AndroidLearning\Kotlin\images\kotlin-reflect01.PNG)

- 类引用（KClass）

最基本的反射功能是获取 Kotlin 类的运行时引用。要获取对静态已知的 Kotlin 类的引用，可以使用 *类字面值* 语法：

```kotlin
val c = MyClass::class
```

Kotlin类引用和Java类引用不同。如果要获得Java类引用，需要改成`MyClass::class.java`。



### 可调用引用

函数、属性以及构造函数的引用，除了作为自省程序结构外， 还可以用于调用或者用作[函数类型](https://www.kotlincn.net/docs/reference/lambdas.html#%E5%87%BD%E6%95%B0%E7%B1%BB%E5%9E%8B)的实例。

所有可调用引用的公共超类型是 [`KCallable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-callable/index.html)， 其中 `R` 是返回值类型，对于属性是属性类型，对于构造函数是所构造类型。

### 函数引用

当我们有一个命名函数声明如下：

```kotlin
fun isOdd(x: Int) = x % 2 != 0
```

我们可以很容易地直接调用它（`isOdd(5)`），但是我们也可以将其作为一个函数类型的值，例如将其传给另一个函数。为此，我们使用 `::` 操作符：

```kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd))
```

这里 `::isOdd` 是函数类型 `(Int) -> Boolean` 的一个值。

函数引用属于 [`KFunction`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-function/index.html) 的子类型之一，取决于参数个数，例如 `KFunction3<T1, T2, T3, R>`。



### 属性引用

要把属性作为 Kotlin中 的一等对象来访问，我们也可以使用 `::` 运算符：

```java
val x = 1

fun main() {
    println(::x.get())
    println(::x.name) 
}
```

表达式 `::x` 求值为 `KProperty<Int>` 类型的属性对象，它允许我们使用 `get()` 读取它的值，或者使用 `name`属性来获取属性名。更多信息请参见[关于 `KProperty` 类的文档](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html)。



### 构造函数引用

构造函数可以像方法和属性那样引用。他们可以用于期待这样的函数类型对象的任何地方：它与该构造函数接受相同参数并且返回相应类型的对象。 通过使用 `::` 操作符并添加类名来引用构造函数。考虑下面的函数， 它期待一个无参并返回 `Foo` 类型的函数参数：

```
class Foo

fun function(factory: () -> Foo) {
    val x: Foo = factory()
}
```

使用 `::Foo`，类 Foo 的零参数构造函数，我们可以这样简单地调用它：

```kotlin
function(::Foo)
```

构造函数的可调用引用的类型也是 [`KFunction`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-function/index.html) 的子类型之一 ，取决于其参数个数。