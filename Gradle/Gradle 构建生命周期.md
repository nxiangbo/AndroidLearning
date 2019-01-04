# Gradle 构建声明周期

Gradle的核心是基于依赖的编程语言。这意味着你可以定义任务和任务之间的依赖关系。Gradle保证这些任务按其依赖项的顺序执行，并且每个任务只执行一次。这些任务可以抽象成一个有向无环图。Gradle会在执行任务之前构建完整的依赖关系图。这就是Gradle的核心思想。

你的构建脚本配置此依赖关系图。因此，他们严格来说是构建配置脚本。

## 构建阶段

Gradle构建过程有三个阶段。

### 初始化（Initialization）

Gradle可以构建一个和多个项目。在初始化阶段，Gradle会确定哪些项目参与构建，并且为这些项目创建一个Project实例。

### 配置（Configuration）

在这个阶段，会配置project对象。将执行构建的所有项目的构建脚本。也就是说，会执行每个项目的build.gradle文件。

### 执行（Execution）

Gradle确定要在执行期间创建和配置的任务子集。子集由传递给gradle命令和当前目录的任务名称参数确定。 Gradle然后执行每个选定的任务。

## Settings文件

除了build.gradle 文件外，Gradle定义了一个settings文件。settings文件由Gradle通过命名约定确定。该文件默认明为`settings.gradle`

`settings.gradle`是在初始化阶段执行。构建多个项目时，必须在根目录中有`settings.gradle`文件。因为在这个文件中定义了哪些项目参加构建。在构建Android项目时，我们会在根目录找到`settings.gradle`文件。除了定义包含的项目之外，您可能还需要将库添加到构建脚本类路径中。下面我们举一个简单的例子。

settings.gradle

```groovy
println 'This is executed during the initialization phase.'
```

build.gradle

```groovy
println 'This is executed during the configuration phase.'

task configured {
    println 'This is also executed during the configuration phase.'
}

task test {
    doLast {
        println 'This is executed during the execution phase.'
    }
}

task testBoth {
	doFirst {
	  println 'This is executed first during the execution phase.'
	}
	doLast {
	  println 'This is executed last during the execution phase.'
	}
	println 'This is executed during the configuration phase as well.'
}
```

运行结果

```groovy
$ gradle test testBoth
This is executed during the initialization phase.
This is executed during the configuration phase.
This is also executed during the configuration phase.
This is executed during the configuration phase as well.
:test
This is executed during the execution phase.
:testBoth
This is executed first during the execution phase.
This is executed last during the execution phase.

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed

```

对于构建脚本，属性访问和方法调用将委派给project对象。类似地，settings文件中的属性访问和方法调用被委托给setting对象.

构建多项目



## 响应构建脚本中的生命周期

随着构建在其生命周期中的进展，您的构建脚本可以接收通知。这些通知通常采用两种形式：您可以实现特定的侦听器接口，也可以在触发通知时提供执行闭包。下面的例子使用闭包。

### 项目评估（project evaluation）

您可以在评估项目之前和之后立即收到通知。

将测试任务添加到具有特定属性集的每个项目

```groovy
allprojects {
    afterEvaluate { project ->
        if (project.hasTests) {
            println "Adding test task to $project"
            project.task('test2') {
                doLast {
                    println "Running tests for $project"
                }
            }
        }
    }
}

allprojects {  

    ext.hasTests = true
}
```

输出结果

```groovy
$ gradle -q test2
Adding test task to root project 'gradle lifecycle'
Running tests for root project 'gradle lifecycle'

```



