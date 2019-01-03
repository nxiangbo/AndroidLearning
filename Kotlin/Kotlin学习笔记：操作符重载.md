# Kotlin学习笔记：操作符重载

![](images\Kotlin-operator.png)

我们都知道Java是不支持操作符重载的。

Java不支持重载的原因，来自StackOverflow

> Java only allows arithmetic operations on elementary numeric types. It's a mixed blessing, because although it's convenient to define operators on other types (like complex numbers, vectors etc), there are always implementation-dependent idiosyncrasies. So operators don't always do what you expect them to do. By avoiding operator overloading, it's more transparent which function is called when. A wise design move in some people's eyes.

而Kotlin是支持操作符重载的。

- 算术操作符重载
- 比较操作符重载
- 集合相关的操作符重载



## 算术运算符重载

- 二元算术运算符

```kotlin
data class Point(val x: Int, val y: Int){ 
    operator fun plus(other: Point): Point{
		return Point(x + other.x, y + other.y)
	}
}

>>> val p1 = Point(10, 20)
>>> val p2 = Point(30, 40)
>>> println(p1 + p2)
Point(x=40, y=60)
```

| 表达式 | 函数名 |
| ------ | ------ |
| a * b  | times  |
| a / b  | div    |
| a % b  | mod    |
| a + b  | plus   |
| a - b  | minus  |



- 一元算术运算符

![](images\operator02.PNG)



## 重载比较运算符

- 等于运算符：equals 对应于 ==

- 比较运算符：compareTo

  在Kotlin中，比较运算符（>，<,<=,>=）

