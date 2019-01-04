# Gradle 学习

Gradle是一个基于[Apache Ant](https://zh.wikipedia.org/wiki/Apache_Ant)和[Apache Maven](https://zh.wikipedia.org/wiki/Apache_Maven)概念的项目自动化构建工具。它使用一种基于[Groovy](https://zh.wikipedia.org/wiki/Groovy)的[特定领域语言](https://zh.wikipedia.org/wiki/%E7%89%B9%E5%AE%9A%E9%A2%86%E5%9F%9F%E8%AF%AD%E8%A8%80)（DSL, domain-specific language ）来声明项目设置，而不是传统的XML。

gradle是一个自动化工具。配置gradle时，使用的是Groovy语言。Groovy的语法与Java类似。



Gradle可以构建多个project，一个`build.gradle`代表一个project，每个project包含多个task。每个task里面可以包含多个action。

构建过程分为三步：

- **初始化阶段：**创建 Project 对象，如果有多个build.gradle，也会创建多个project.
- **配置阶段：**在这个阶段，会执行所有的编译脚本，同时还会创建project的所有的task，为后一个阶段做准备。
- **执行阶段：**在这个阶段，gradle 会根据传入的参数决定如何执行这些task,真正action的执行代码就在这里.

## Groovy语法

在一些情况下，可能需要我们自己定义task。而Gradle其实使用的是groovy语言，因此，在定义task之前需要先了解groovy语法。而groovy与Scala、Kotlin语言一样，都是一种JVM语言。

 ### 闭包

**闭包**（英语：Closure），又称**词法闭包**（Lexical Closure）或**函数闭包**（function closures），是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。 

使用闭包的groovy代码

```groovy
def hash = ["name":"Andy", "age":25]
	hash.each{ key, value ->
 		println "${key} : ${value}"
	}
```

没用闭包的Java代码

```java
Map<String, String>map = new HashMap<String, String>();
map.put("name", "Andy");
map.put("age","20");
               
for(Iterator iter = map.entrySet().iterator(); iter.hasNext();){
 	Map.Entry entry = (Map.Entry)iter.next();
 	System.out.println(entry.getKey() + " : " + 	entry.getValue());
}
```

虽然在迭代上使用闭包的机会最多，但闭包确实还有其他用途。因为闭包是一个代码块，所以能够作为参数进行传递（Groovy 中的函数或方法不能这样做）。 

### 类

Groovy 中可以像在 Java 代码中一样定义类。不过不需要使用 `public` 修改符，而且还可以省略方法参数的类型。 

例如，我们可以创建一个User类，它有姓名和年龄属性。

```groovy
class User{
    def name
    def age
}
```

与java不同的是，groovy会自动生成User类的getter和setter方法。



### Groovy in Gradle

在配置文件中的`apply plugin: 'com.android.application'`转化为groovy语法为：

```groovy
project.apply([plugin: 'com.android.application'])
```

apply()是Project类的方法，参数是map集合。

build.gradle文件中的依赖配置。

```groovy
dependencies {
	compile 'com.google.code.gson:gson:2.3'
}
```

转化为groovy语法为：

```groovy
project.dependencies({
	add('compile', 'com.google.code.gson:gson:2.3', {
		// Configuration statements
	})
})
```

## 构建Android项目

![1532662243210](C:\Users\NIUXIA~1\AppData\Local\Temp\1532662243210.png)

Gradle 和 Android 插件可以完成以下方面的构建配置： 

- buildTypes:在模块级 `build.gradle` 文件的 `android {}` 代码块内部创建和配置buildTypes。 

```groovy
android {
    ...
    defaultConfig {...}
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

        debug {
            applicationIdSuffix ".debug"
        }

        /**
         * The 'initWith' property allows you to copy configurations from other build types,
         * so you don't have to configure one from the beginning. You can then configure
         * just the settings you want to change. The following line initializes
         * 'jnidebug' using the debug build type, and changes only the
         * applicationIdSuffix and versionNameSuffix settings.
         */

        jnidebug {

            // This copies the debuggable attribute and debug signing configurations.
            initWith debug

            applicationIdSuffix ".jnidebug"
            jniDebuggable true
        }
    }
}
```

- productFlavors:与配置buildTypes类似，将配置添加到`productFlavors {} `代码块中。

```groovy
android {
    ...
    defaultConfig {...}
    buildTypes {...}
    productFlavors {
        demo {
            applicationIdSuffix ".demo"
            versionNameSuffix "-demo"
        }
        full {
            applicationIdSuffix ".full"
            versionNameSuffix "-full"
        }
    }
}
```

Gradle 创建的Build Variants数量等于每个productFlavors中的Flavor数量与配置的buildTypes数量的乘积。 

- dependencies:

```groovy
dependencies {
    // The 'compile' configuration tells Gradle to add the dependency to the
    // compilation classpath and include it in the final package.

    // Dependency on the "mylibrary" module from this project
    compile project(":mylibrary")

    // 远程二进制依赖
    compile 'com.android.support:appcompat-v7:27.1.1'

    // Local binary dependency
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

## Gradle 与Android Gradle Plugin

Gradle作为一个异常强大的构建工具，为了满足不同平台的需求，比如：Java平台有Java构建逻辑，Android平台有Android构建逻辑。Gradle务必是要支持自定义构建的，这个功能正是由Gradle Plugin提供，而对应于Android工程的构建逻辑就是由Android Gradle Plugin实现的了。  

## 自定义Gradle插件

自定义gradle插件时，既可以使用groovy语言，也可以使用Java或者Kotlin语言。一般我都是使用groovy语言。



自定义插件的目录结构如下：

```
plugin
└── src
	└── main
	|	├── groovy
	|	|	└── packagename
	|	|		   └── CustomPlugin.groovy
	|	|				
	|	└── resources
	|		└── META-INF
	|			└── gradle-plugins
	|                |-- customplugin.properties
	|_ build.gradle
```

由上述插件结构可知，自定义插件大致可分为三个部分。groovy目录，resources目录和build.gradle配置文件。

### build.gradle

```groovy
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    implementation gradleApi()
    implementation localGroovy()
}
```

### META-INF/gradle-plugins 

在META-INF/gradle-plugins 目录下添加一个属性文件，主要用来使得Gradle找到该插件。

```groovy
implementation-class=com.nxiangbo.plugin.main.CustomPlugin
```

### Groovy 目录

为了创建Gradle插件，我们必须创建一个实现`Plugin`接口的类。当我们将自定义的插件应用到项目时，Gradle就会创建这个类的实例，并调用这个类的`apply()`方法。而project作为该方法的参数，因此插件可以使用project的配置。下面我们在groovy目录下创建CustomPlugin.groovy。

```groovy
 class CustomPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        def extension = project.extensions.create("customPlugin", CustomExtension)
        project.task("demo"){
            project.afterEvaluate{
       
             		println("versionName=${extension.versionName}   versionCode=${extension.versionCode}")
            }
        }


    }
}
```

```groovy
class CustomExtension {
    String versionName
    String versionCode

}
```

### 发布插件

只需要在build.gradle文件中添加uploadArchives task就可以将插件发布到本地。

```groovy
afterEvaluate { project ->
    uploadArchives {
        repositories {
            mavenDeployer {
                repository(url: uri('D:/repo'))
            }
        }
    }
}
```



### 应用插件

在需要使用该插件的项目中的build.gradle 文件中，添加如下代码

```groovy
apply plugin: 'customplugin'

customPlugin {
    versionName='version'
    versionCode='1.1.0'
}
```

## Gradle插件调试



首先，在Android Studio 中设置Edit Configurations -> +号 -> Remote -> 填写Host和端口号

![gradle调试](E:\笔记\images\gradle调试.png)



然后，运行`gradle :app:clean -Dorg.gradle.debug=true  --no-daemon `

最后，点击调试按钮即可。![gradle调试02](E:\笔记\images\gradle调试02.png)





