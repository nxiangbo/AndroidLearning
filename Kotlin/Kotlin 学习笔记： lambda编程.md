# Kotlin 学习笔记： lambda编程

![](images\Kotlin-lambda.png)

## 什么是lambda表达式
“Lambda 表达式”(lambda expression)是一个匿名函数，Lambda表达式基于数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。
lambda表达式本质上是可以传递给其他函数的一段代码。lambda可以作为函数的参数和返回值。大大简化了代码。

lambda表达式的语法为
![](images\Kotlin-lambda01.PNG)



## 集合的函数式API

- filter

filter 可以移除集合中不满足条件的元素。

```kotlin
>>> val list = listOf(1, 2, 3, 4)
>>> list.filter { it % 2 == 0 }
[2, 4]
```



![](images\lambda-filter.PNG)



- map

map函数对集合中的每个元素应用给定的函数并把结果收集到一个集合中。

```kotlin
>>> val list = listOf(1, 2, 3, 4)
>>> list.map { it * it }
[1, 4, 9, 16]
```

![](images\lambda-map.PNG)



- all，any，find，count

这些主要用于对集合的判定

假如我们需要判定一群人中的年龄是否小于28岁

```kotlin
val canBeInClub27 = { p: Person -> p.age <= 27 }
```

**使用all判定是否所有人都小于28岁**

```kotlin
>>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
>>> println(people.all(canBeInClub27))
false
```

使用any判定其中的一些人小于28岁

```kotlin
>>> println(people.any(canBeInClub27))
true
```

使用count计算小于28岁的人的个数

```kotlin
>>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
>>> println(people.count(canBeInClub27))
1
```

使用find找到所有小于28岁的人信息

```kotlin
>>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
>>> println(people.find(canBeInClub27))
Person(name=Alice, age=27)
```

- groupBy

groupBy可以用于将列表转换成分组的map

```kotlin
>>> val people = listOf(Person("Alice", 31),
... Person("Bob", 29), Person("Carol", 31))
>>> println(people.groupBy { it.age })
```

![](images\lambda-groupby.PNG)

- flatmap和flatten

flatMap函数做两件事情：首先根据作为实参给定的函数对集合中的每个元素做变换（maps），然后把多个列表合并(flatten)成一个列表。

```kotlin
>>> val strings = listOf("abc", "def")
>>> println(strings.flatMap { it.toList() })
[a, b, c, d, e, f]
```

![](images\lambda-flatMap.PNG)

flatten可以将多个集合的元素合并到一个集合中。

```kotlin
>>>  val langs = listOf(listOf("js"), listOf("Java", "Kotlin"), listOf("C++", "PHP", "Python"))
>>> println(langs.flatten())
[js, Java, Kotlin, C++, PHP, Python]
```



## 序列

前面讲述了很多链式集合调用的例子，但是这些函数会及早的创建中间集合，也就是说每一步的中间结果都存储在一个临时列表里。所以，当直接使用链式集合的函数处理集合元素数量特别多的情况下，性能比较低。而序列可以避免创建临时中间变量。

举例来讲

有一个Person类，它有名字和年龄两个属性。

```kotlin
data class Person(val name: String, val age:Int)
```

要找出名字前缀是“J”的人的姓名

用链式集合函数

```kotlin
persons.map { person -> person.name }
            .filter { it.startsWith("J") }
            .toList()
```

编译代码，再将其反编译

```java
	//....
	Object element$iv$iv;
      while(var4.hasNext()) {
         element$iv$iv = var4.next();
         Person person = (Person)element$iv$iv;
         String var11 = person.getName();
         destination$iv$iv.add(var11);
      }
	//...	
```



用序列实现

```kotlin
 persons.asSequence()
            .map { person -> person.name }
            .filter { it.startsWith("J") }
            .toList()
```



反编译后代码

```java
List persons = CollectionsKt.listOf(new Person[]{new Person("Jack", 25), new Person("Cat", 29)}); 
SequencesKt.toList(SequencesKt.filter(SequencesKt.map(CollectionsKt.asSequence((Iterable)persons), (Function1)null.INSTANCE), (Function1)null.INSTANCE));
```

## 使用Java函数式接口

首先说函数式接口的定义：只有一个抽象方法的接口称为函数式接口，也就是SAM（单抽象方法）。

在Android中，最常用的接口就是OnClickListener了。现在我们以监听一个button的点击事件为例，来说明kotlin是如何使用Java函数式接口的。

用java编写监听button点击事件

```java
	btn.setOnClickListener(new View.OnClickListener() {
			@Override
			public void onClick(View v) {...}
		});
```



用kotlin编写button点击事件

```kotlin
btn.setOnClickListener{...}
```



我们还记得在函数文章中，我们用过匿名表达式。

```kotlin
  val vto = layout.viewTreeObserver
        vto.addOnGlobalLayoutListener(object: ViewTreeObserver.OnGlobalLayoutListener 		{
            @RequiresApi(Build.VERSION_CODES.JELLY_BEAN)
            override fun onGlobalLayout(){
                layout.viewTreeObserver.removeOnGlobalLayoutListener(this)
                val height = layout.measuredHeight
                val width = layout.measuredWidth
            }
        })
```

OnGlobalLayoutListener接口也只有一个方法，符合函数式接口的定义。那么，这个方法可以用lambda简化吗？答案是不可以。因为包含`layout.viewTreeObserver.removeOnGlobalLayoutListener(this)`语句。

在lambda 内部没有匿名对象那样的this。这意味着没有办法引用到lambda转换成的匿名类实例。

## 带接收者的lambda：with和apply

很多语言都有这样的语句：可以用他们对同一个对象执行多次操作，而不需要把对象的名称写出来。

with 和apply就是这样的语句。

- with

还是以一个例子开始：创建一个函数打印字母表。

一般情况下，都这样写

```kotlin
fun alphabet(): String {
	val result = StringBuilder()
	for (letter in 'A'..'Z') {
		result.append(letter)
	}
	result.append("\nNow I know the alphabet!")
	return result.toString()
} 
>>> println(alphabet())
ABCDEFGHIJKLMNOPQRSTUVWXYZ
Now I know the alphabet!
```

而用with，可以这样写

```kotlin
fun alphabet(): String {
	val result = StringBuilder()
	return with(result)
		{ for (letter in 'A'..'Z')
			{this.append(letter)}
			append("\nNow I know the alphabet!")
			this.toString()
		}
	}
```



- apply

apply几乎和with一模一样，唯一的区别是apply始终会返回作为实参传递给他的对象。

```kotlin
fun alphabet() = StringBuilder().apply{ 
    for (letter in 'A'..'Z') {
		append(letter)
	}
	append("\nNow I know the alphabet!")
}.toString()
```



## 高阶函数

高阶函数：以另一个函数作为参数或者返回值的函数。

- 函数类型

为了声明一个以lambda作为实参的函数，需要知道如何声明对应的形参类型。

语法：将函数参数类型放在括号中，紧接着是一个箭头和函数的返回类型

- 调用作为参数的函数类型

```kotlin
fun searchRepos(
        service: ZhihuService,
        query:String,
        onSuccess:(repos: List<Repo>) -> Unit,
        onError: (error: String) -> Unit) {
   //...
}

```



## 内联函数

