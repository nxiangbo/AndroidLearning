# Small的业务插件支持Flavor问题

small 编译插件的功能

1、解析插件的依赖，并将依赖的库写到small-build文件夹下

2、为资源分区（应该是修改aapt打包后的二进制资源文件）

3、将各个插件打包到asset目录下



## ./gradlew buildLib时生成结果不对



### 问题分析

通过断点调试，发现执行到**DependenciesUtils#getCompileConfiguration**方法时，`project.configurations["releaseCompileClasspath"]`为`null`。

因此，问题代码可以定位到**DependenciesUtils#getCompileConfiguration**

该代码片段的作用是获取Configuration对象。

在Gradle 3.x中，使用`project.configurations["releaseCompileClasspath"] `获得Configuration对象。

然而，当项目有ProductFlavor时，`project.configurations["releaseCompileClasspath"]`为`null`。

如果要正确获得Configuration对象，需要添加上Falvor，即`project.configurations['${ProductFlavor}ReleaseCompileClasspath']`

buildLib 任务是ResolveDependenciesTask类型的Task。而在ResolveDependenciesTask类中会调用**DependenciesUtils#getAllCompileDependencies**

-->**DependenciesUtils#getCompileConfiguration**



### 解决方式

获取依赖的过程中，区分业务插件是否有flavor。

```
if 含有flavor
	project.configurations[flavor+'ReleaseCompileClasspath']
else
	project.configurations['releaseCompileClasspath']
```



最后，**buildLib成功，生成结果正确**

### 最终解决方案



在buildLib 任务中，判断插件是否具有flavor，如果有flavor，则获取其Configuration时，添加flavor。即project.configurations['normalReleaseCompileClasspath']

## ./gradlew buildBundle 时出现UnknownTaskException



### 问题日志

```java
 org.gradle.api.UnknownTaskException: Task with path ':app+stub:smallJarNormalRelease' not found in project ':app.mine'.
        at org.gradle.api.internal.tasks.DefaultTaskContainer.getByPath(DefaultTaskContainer.java:200)
        at org.gradle.api.internal.tasks.DefaultTaskContainer.resolveTask(DefaultTaskContainer.java:182)
        at org.gradle.api.internal.tasks.DefaultTaskDependency.visitDependencies(DefaultTaskDependency.java:128)
        at org.gradle.api.internal.tasks.CachingTaskDependencyResolveContext$TaskGraphImpl.getNodeValues(CachingTaskDependencyResolveContext.java:90)
        at org.gradle.internal.graph.CachingDirectedGraphWalker$GraphWithEmpyEdges.getNodeValues(CachingDirectedGraphWalker.java:211)
        at org.gradle.internal.graph.CachingDirectedGraphWalker.doSearch(CachingDirectedGraphWalker.java:121)
        at org.gradle.internal.graph.CachingDirectedGraphWalker.findValues(CachingDirectedGraphWalker.java:73)
        at org.gradle.api.internal.tasks.CachingTaskDependencyResolveContext.doResolve(CachingTaskDependencyResolveContext.java:77)
        at org.gradle.api.internal.tasks.CachingTaskDependencyResolveContext.resolve(CachingTaskDependencyResolveContext.java:66)

```

### 分析问题



根据` Task with path ':app+stub:smallJarNormalRelease' not found in project ':app.mine'.`信息，app.mine项目的特定路径上的task没有找到。而只有这个插件添加了flavor normal和develop。所以，猜测在创建`:app+stub:smallJarXXX`类型的Task时，出现问题。



### 解决问题

判断插件是否**具有flavor的app**。如果有flavor，则执行:app+stub:smallJarRelease 任务。否则，逻辑不变。

最后，UnknownTaskException问题得到解决。然而，在打包插件的过程中，执行到app.mine时，又出现了问题。



## 打包具有flavor插件时，出现resources-release.ap_不存在问题



当插件有flavor时，resources-release.ap_路径不对

### 解决方式

具有flavor的业务插件的_ap文件与没有flavor的路径不同。

## 运行宿主出现问题  

buildBundle 成功后，运行宿主，出现问题

```
 Process: net.wequick.example.small, PID: 1423
    java.lang.IllegalStateException: java.lang.NoSuchMethodException: ensureStringBlocks []
        at net.wequick.small.util.ReflectAccelerator.mergeResources(ReflectAccelerator.java:578)
        at net.wequick.small.ApkBundleLauncher.postSetUp(ApkBundleLauncher.java:811)
        at net.wequick.small.Bundle.loadBundles(Bundle.java:789)
        at net.wequick.small.Bundle.loadBundles(Bundle.java:299)
        at net.wequick.small.Bundle.access$100(Bundle.java:69)
        at net.wequick.small.Bundle$LoadBundleThread.run(Bundle.java:744)
     Caused by: java.lang.NoSuchMethodException: ensureStringBlocks []
        at java.lang.Class.getMethod(Class.java:2068)
        at java.lang.Class.getDeclaredMethod(Class.java:2047)
        at net.wequick.small.util.ReflectAccelerator.mergeResources(ReflectAccelerator.java:487)
        at net.wequick.small.ApkBundleLauncher.postSetUp(ApkBundleLauncher.java:811) 
        at net.wequick.small.Bundle.loadBundles(Bundle.java:789) 
        at net.wequick.small.Bundle.loadBundles(Bundle.java:299) 
        at net.wequick.small.Bundle.access$100(Bundle.java:69) 
        at net.wequick.small.Bundle$LoadBundleThread.run(Bundle.java:744) 
```



## 插件找不到basicres类型资源问题

arsc的分离过程

 如果没有basicres



尝试方法1

将所有的basicres都拷贝到每个插件中

应该可以解决这个问题，但是资源会有重复。

还是需要弄清楚small插件是如何加载公共资源的。

应该和notIncludePackages没有关系

重点是

**如何加载公共资源**



1. 先阅读之前版本的small并调试

2. 然后再针对现在的问题进行解决



### 原先small

#### ./gradlew buildLib 过程

```java

遍历lib
//srcFile: D:\Code2\Small\Android\Sample\lib.utils\build\intermediates\symbols\release
// destFile: build-small

	将 class.jar .ap_ R.txt等文件拷贝到build-small目录
	


```

**拷贝这些文件有何用？**

继续查看buildBundle的过程

#### ./gradlew buildBundle过程

configureProject
configureReleaseVariant

将ap_文件解压，并拷贝到指定目录。

ap_文件：AM.xml文件，res/，resources.arsc文件











