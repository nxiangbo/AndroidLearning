# Gradle Project & Task

Gradle中的所有内容都基于两个基本概念：项目(Project)和任务(Task)。每个Gradle构建都是由一个或多个project组成。每个project都是有一个或者多个任务组成。任务之间具有依赖关系，保证了任务的执行顺序。任务代表构建执行的一些原子工作。这可能是编译某些类，创建JAR，生成Javadoc或将一些存档发布到存储库。



## Project

我们首先讲一下Project的概念。每个build.gradle 就对应一个project。构建中的每一个project，Gradle都会创建一个Project对象，并将这个对象与构建脚本相关联。简单的说，build.gradle是对一个Project对象的配置。以Android项目为例，在项目的根目录会有一个build.gradle文件，在每个模块的目录下也会有一个build.gradle 文件。

Project与build.gradle是一对一的关系。在构建初始化阶段，Gradle为每个参与构建的项目创建一个Project对象。



- 我们在构建脚本中调用的没有在构建脚本中定义的方法都委派给Project对象。
- 我们在在构建脚本中调用的没有在构建脚本中定义的属性都委派给Project对象。



| Name          | Type                                                         | Default Value                              |
| ------------- | ------------------------------------------------------------ | ------------------------------------------ |
| `project`     | [Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html) | The `Project` instance                     |
| `name`        | `String`                                                     | The name of the project directory.         |
| `path`        | `String`                                                     | The absolute path of the project.          |
| `description` | `String`                                                     | A description for the project.             |
| `projectDir`  | `File`                                                       | The directory containing the build script. |
| `buildDir`    | `File`                                                       | `*projectDir*/build`                       |
| `group`       | `Object`                                                     | `unspecified`                              |
| `version`     | `Object`                                                     | `unspecified`                              |
| `ant`         | [AntBuilder](https://docs.gradle.org/current/javadoc/org/gradle/api/AntBuilder.html) | An `AntBuilder` instance                   |

## Task

### 任务的定义

我们可以用以下方式创建task

方式一



```groovy
task hello {
    doLast {
         println 'Hello Gradle!'
    }
}
```

运行这个task，gradle hello

```groovy
> gradle -q hello
Hello Gradle!
```

方式二

```groovy
tasks.create('hello2') {
	doLast {
		println "Hello Gradle!"
	}
}
```

方式三

```groovy
task('hello3') {
	doLast {
		println "Hello Gradle!"
	}
}
```



### 任务的依赖

在使用任务时，可以使用dependsOn关键词声明任务与任务的依赖关系。

```groovy
task hello {
    doLast {
        println 'Hello world!'
    }
}
task intro(dependsOn: hello) {
    doLast {
        println "I'm Gradle"
    }
}
```

运行结果

```groovy
> gradle -q intro
Hello world!
I'm Gradle
```

### 添加任务的说明

我们还可以为任务添加说明。执行gradle任务时会显示此说明。

```groovy
task hello {
	description 'This is the Hello Task'
	doLast {
		println "Hello Gradle!"
	}
}
```

###  Configure by DAG

在Gradle构建生命周期中，Gradle有配置阶段和执行阶段。在配置阶段，Gradle明确了需要执行的所有任务。Gradle为我们提供了一个利用这些信息的钩子（hook）。举例来讲，我们可以为不同的任务，分配不同的变量值或者执行不同的代码逻辑。

```groovy
task distribution {
    doLast {
        println "We build the zip with version=$version"
    }
}

task release(dependsOn: 'distribution') {
    doLast {
        println 'We release now'
    }
}

gradle.taskGraph.whenReady {taskGraph ->
    if (taskGraph.hasTask(release)) {
        version = '1.0'
    } else {
        version = '1.0-SNAPSHOT'
    }
}
```

输出结果

```groovy
> gradle -q distribution
We build the zip with version=1.0-SNAPSHOT

> gradle -q release
We build the zip with version=1.0
We release now
```





### 构建脚本的外部依赖项

```groovy
import org.apache.commons.codec.binary.Base64

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'commons-codec', name: 'commons-codec', version: '1.2'
    }
}

task encode {
    doLast {
        def byte[] encodedString = new Base64().encode('hello world\n'.getBytes())
        println new String(encodedString)
    }
}
```

输出结果

```groovy
> gradle -q encode
aGVsbG8gd29ybGQK
```

### 任务的执行结果

| 任务结果   | 含义                             |
| ---------- | -------------------------------- |
| EXECUTED   | task的action已经执行             |
| UP-TO-DATE | task的输出没有改变，参考增量构建 |
| FROM-CACHE | 可以从之前的输出中得到任务的输出 |
| SKIPPED    | task没有执行它的action           |
| NO-SOURCE  | task不需要执行它的action         |



