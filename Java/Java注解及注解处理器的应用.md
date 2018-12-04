# Java注解及注解处理器的应用

## 注解的基本知识



注解（也被称为元数据），为我们在代码中添加信息提供了一种形式化的方法，使得我们可以在稍后某个时刻可以非常方便的使用这些数据。 



为什么要使用注解 ？？ 

通过使用注解，我们可以将元数据保存在Java源代码中，并利用annotation API为自己的注解构造处理工具，，同时，注解还有使得代码更加干净易读，以及编译器类型检查等优点。 


Java中内置了三种标准注解，以及四种元注解。 

**标准注解** 

- @Override 表示当前的方法将覆盖父类中的方法。 
- @Deprecated 如果程序员使用了此注解，那么编译器会发出警告信息。被程序作者不推荐使用。 
- @SuppressWarnings 关闭不当的编译器警告信息。 

**元注解**  专门负责注解其他的注解 

- @Target  表示注解可以用于什么地方。可能的 `ElementType`参数为：CONSTRUCTOR(构造器)，FIELD(域)，LOACAL_VARIABLE(局部变量),METHOD（方法）,PACKAGE（包）,PARAMETER（参数）,TYPE（类，接口） 
- @Retention  表示需要在什么级别保存该注解信息。 `RetentionPolicy`参数：SOURCE（注解将被编译器丢弃），CLASS（注解在class文件中可以使用，但会被VM丢弃），RUNTIME（VM也在运行期间也保留注解，因此可以通过反射机制读取注解的信息）。 
- @Documented  将此注解包含在Javadoc中。 
- @Inherited  允许子类继承父类中的注解。 



### 注解的定义

注解可以通过 `@interface` 关键字进行定义。下面我们自定义一个注解，并详细讲述如何应用注解。



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface CustomAnnation{
    String field();
}
```



上面代码自定义了一个`CustomAnnation`注解，该注解只能用于类或接口（@Target(ElementType.TYPE)），并且它是一个编译时注解（@Retention(RetentionPolicy.CLASS)）。此外，它还有一个String类型的元素。



那么，如何使用注解呢？假设类Test需要使用`CustomAnnation`注解。



```java
@CustomAnnation(field="test")
public class Test{
    
}
```



## 注解处理器

注解处理器（Annotation Processor）是javac的一个工具，它用于在**编译时**扫描和处理注解。一个注解的注解处理器，以Java代码（或者编译过的字节码）作为输入，生成文件（通常是`.java`文件）作为输出。这具体的含义什么呢？你可以生成Java代码！这些生成的Java代码是在生成的.java文件中，所以你不能修改已经存在的Java类，例如向已有的类中添加方法。这些生成的Java文件，会同其他普通的手动编写的Java源代码一样被**javac**编译。



自定义一个注解处理器需要继承`AbstractProcessor`类。



```java
public class MyProcessor extends AbstractProcessor {

    @Override
    public synchronized void init(ProcessingEnvironment env){ }

    @Override
    public boolean process(Set<? extends TypeElement> annoations, RoundEnvironment env) { }

    @Override
    public Set<String> getSupportedAnnotationTypes() { }

    @Override
    public SourceVersion getSupportedSourceVersion() { }

}
```



- init(ProcessingEnvironment env) 方法：被注解处理工具调用，并输入`ProcessingEnviroment`参数。`ProcessingEnviroment`提供很多有用的工具类`Elements`, `Types`，`Messenger`和`Filer`
- process方法：所有处理注解的逻辑都是在这里调用，包括扫描，评估和处理注解。
- getSupportedAnnotationTypes方法：指定注解处理器是注册给那一个注解的，它是一个字符串的集合，意味着可以支持多个类型的注解，并且字符串是合法全名。
- getSupportedSourceVersion方法：指定使用的Java版本。



## 注册注解处理器

在写完自己的注解处理器后，需要将自定义注解处理器注册到javac中，这样自定义注解处理器才能生效。然而，如何注册呢？只需要将它打包成.jar文件之前，添加配置信息到META-INF/services路径下。

－－myprcessor.jar
－－－－com
－－－－－－example
－－－－－－－－MyProcessor.class
－－－－META-INF
－－－－－－services
－－－－－－－－javax.annotation.processing.Processor





当然，也可以直接使用谷歌提供的注册处理器库。

```groovy
implementation 'com.google.auto.service:auto-service:1.0-rc2'
```

使用方法很简单。直接在自定义注解处理器上添加注解即可。

```java
@AutoService(Processor.class)
public class MyProcessor extends AbstractProcessor {
	...
}
```



## 例子

下面将通过一个例子，来说明注解处理器是如何使用的。



