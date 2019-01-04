

# 升级Gradle3.0

> 业精于勤荒于嬉，行成于思毁于随

Android Gradle 3.0.0插件是一个大版本的升级，对于构建多个module带来了显著的性能提升。但同时也更新了DSL和APIS。 

升级Gradle3.0所带来的好处如下：

- 通过细粒度的任务图更好地实现多模块项目的并行性 
- Variant-aware dependency management （变体感知的依赖管理）
- 可以使用Gradle的新依赖项配置来限制哪些依赖项将其API泄漏到其他模块：implementation，api，compileOnly和runtimeOnly。从而增加构建速度。
- 使用 per-class dexing 加快增量构建速度。每个类被编译成一个独立的dex文件，并且只有这个类被修改后才会重新编译。

值得注意的是，Google打算在Android studio 3.0中引入D8作为原先Dex的升级版 。

## 升级步骤

升级Gradle版本。在gradle/wrapper/gradle-wrapper.properties

```java
distributionUrl=https\://services.gradle.org/distributions/gradle-4.4-all.zip
```



升级Android Gradle Plugin。在工程的build.gradle

```groovy
classpath 'com.android.tools.build:gradle:3.1.3'
```

### 新的依赖配置



| 新配置         | 弃用     |                                                              |
| -------------- | -------- | ------------------------------------------------------------ |
| implementation | compile  | 在编译时，该模块的依赖不会泄露给其他模块。只有在运行时其他模块才能获取依赖。 |
| api            | compile  | 不论是在编译时，还是在运行时，其他模块都可以获得这个依赖。   |
| compileOnly    | provided | 依赖项仅在编译时对模块可用，并且在编译或运行时对其消费者不可用。 此配置的行为类似于 `provided`（现在已弃用）。 |
| runtimeOnly    | apk      | 依赖项仅在运行时对模块及其消费者可用。 此配置的行为类似于 `apk`（现在已弃用）。 |
|                |          |                                                              |



## ledesmall升级Gradle3.0

### 问题



1. Gradle 3.0 API更新较多。例如，PrepareLibraryTask等类被移除掉


### 解决方式



把移除掉的类全部拿过来，放在本地。将gradle2.3的PrepareLibraryTask挪到本地。

注意问题

1. 在gradle2.3中的`PrepareLibraryTask`类中使用`@ParallelizableTask`注解。而在gradle3.0中该注解被删除，可以使用`WorkerExecutor`  API替换。
2. 在`PrepareLibraryTask`类中去掉useBuildCache相关代码



问题二

```groovy
groovy.lang.MissingPropertyException: Could not get unknown property 'libraries' for task ':webview:processDevelopDebugManifest' of type com.android.build.gradle.tasks.MergeManifests
```

我查看了gradle2.3和gradle3.1.3的源码。发现MergeManifests类做了很大改变。

```groovy
 @Override
    protected void hookPreDebugBuild() {
        super.hookPreDebugBuild()

        // If an app.A dependent by lib.B and both of them declare application@name in their
        // manifests, the `processManifest` task will raise a conflict error. To avoid this,
        // modify the lib.B manifest to remove the attributes before app.A `processManifest`
        // and restore it after the task finished.

        // processDebugManifest
        project.tasks.withType(MergeManifests.class).each {
            if (it.variantName.startsWith('release')) return

            if (it.hasProperty('providers')) {
                it.providers = []
                return
            }
		// 出问题代码定位到这一行
            hookProcessDebugManifest(it, it.libraries)
        }

        // processDebugAndroidTestManifest
        project.tasks.withType(ProcessTestManifest.class).each {
            if (it.variantName.startsWith('release')) return

            if (it.hasProperty('providers')) {
                it.providers = []
                return
            }

            hookProcessDebugManifest(it, it.libraries)
        }
    }
```

从抛出异常的信息来看，不能识别libraries这个属性。





## DataCollectionSdkForAndroid升级Gradle 3.0



问题一

> Cannot invoke method doLast() on null object

对这个task做判空操作。

问题二

> ```
> Error:Unable to resolve dependency for ':app@debug/compileClasspath':
>   Could not resolve project :library
> ```



使用变体的依赖关系解决方案，您不再需要使用特定于变体的配置（例如freeDebugImplementation）来获取本地模块依赖关系 - 插件会自动提供配置。 

```groovy
// implementation project(path: ":dcsdk",configuration:'debug')

implementation project(":dcsdk") // 用这种替换
```

问题三

> Resolving configuration 'provided' directly is not allowed

解决方案

设置setCanBeResolved(true)，刚开始并没有作用。后来发现，主项目是通过jar包的形式加载bytePlugin插件。虽然修改了代码，但是并没有打包到指定位置。汗:smile: 不知道为啥没有自动打包到指定位置。

问题四

> java.lang.RuntimeException:com.android.builder.dexing.DexArchiveMergerException:

包重复引用。将bytecodemonitor中的

```groovy
implementation fileTree(dir: 'libs', include: ['*.jar'])
```

修改为

```groovy
compileOnly fileTree(dir: 'libs', include: ['*.jar'])
```

## NTESMobileAnalysis 升级Gradle3.0

问题一

> Could not resolve all dependencies for configuration ':demo:debugAndroidTestCompileClasspath'.
>
> A problem occurred configuring project ':sdk'.
>
>  ABIs [mips64, armeabi, mips] are not supported for platform. Supported ABIs are [armeabi-v7a, arm64-v8a, x86, x86_64].

解决方案

最新的NDK r17版本，已经去掉了armeabi、mips、mips64的ABI支持。 

去掉abiFilters中的armeabi选项 

```groovy
ndk {
            abiFilters 'x86', 'x86_64',  'armeabi-v7a', 'arm64-v8a'
        }
```

问题二

>  Could not resolve all files for configuration ':demo:debugAndroidTestCompileClasspath'.
>
> More than one variant of project :sdk matches the consumer attributes:
>      - Configuration ':sdk:debugApiElements' variant android-aidl

解决方案

In the sample project's [app/gradle.build#L24](https://github.com/gmetal/sample-dependency-check-app/blob/ecd85a0d72c552b3c55822179c145ee1ac5a857e/app/build.gradle#L24) change

```
implementation project(':mylibrary')
```

To

```
implementation project(path: ':mylibrary', configuration: 'default')
```

Explicitly setting the configuration on the project reference resolves the error. Additional enhancements are being made to better support Android AAR dependencies.



问题三

```java
Caused by: org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration$ArtifactResolveException: Could not resolve all files for configuration ':demo:releaseUnitTestRuntimeClasspath'.
	at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration.rethrowFailure(DefaultConfiguration.java:918)
	at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration.access$1600(DefaultConfiguration.java:116)
	at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration$ConfigurationFileCollection.getFiles(DefaultConfiguration.java:892)
	at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration.getFiles(DefaultConfiguration.java:404)
	at org.gradle.api.internal.artifacts.configurations.DefaultConfiguration_Decorated.getFiles(Unknown Source)
	at org.gradle.api.file.FileCollection$getFiles.call(Unknown Source)
	at com.netease.tech.analysis.plugin.MobileAnalysisPlugin$_apply_closure2$_closure3.doCall(MobileAnalysisPlugin.groovy:53)
	at sun.reflect.GeneratedMethodAccessor607.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:93)
	at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:325)
	at org.codehaus.groovy.runtime.metaclass.ClosureMetaClass.invokeMethod(ClosureMetaClass.java:294)
	at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1022)
	at groovy.lang.Closure.call(Closure.java:414)
	at groovy.lang.Closure.call(Closure.java:430)
	at org.codehaus.groovy.runtime.DefaultGroovyMethods.each(DefaultGroovyMethods.java:2040)
	at org.codehaus.groovy.runtime.DefaultGroovyMethods.each(DefaultGroovyMethods.java:2025)
	at org.codehaus.groovy.runtime.DefaultGroovyMethods.each(DefaultGroovyMethods.java:2078)
	at org.codehaus.groovy.runtime.dgm$165.invoke(Unknown Source)
	at org.codehaus.groovy.runtime.callsite.PogoMetaMethodSite$PogoMetaMethodSiteNoUnwrapNoCoerce.invoke(PogoMetaMethodSite.java:251)
	at org.codehaus.groovy.runtime.callsite.PogoMetaMethodSite.call(PogoMetaMethodSite.java:71)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:125)
	at com.netease.tech.analysis.plugin.MobileAnalysisPlugin$_apply_closure2.doCall(MobileAnalysisPlugin.groovy:41)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:93)
	at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:325)
	at org.codehaus.groovy.runtime.metaclass.ClosureMetaClass.invokeMethod(ClosureMetaClass.java:294)
	at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1022)
	at groovy.lang.Closure.call(Closure.java:414)
	at org.gradle.listener.ClosureBackedMethodInvocationDispatch.dispatch(ClosureBackedMethodInvocationDispatch.java:40)
	at org.gradle.listener.ClosureBackedMethodInvocationDispatch.dispatch(ClosureBackedMethodInvocationDispatch.java:25)
	at org.gradle.internal.event.AbstractBroadcastDispatch.dispatch(AbstractBroadcastDispatch.java:42)
	at org.gradle.internal.event.BroadcastDispatch$SingletonDispatch.dispatch(BroadcastDispatch.java:230)
	at org.gradle.internal.event.BroadcastDispatch$SingletonDispatch.dispatch(BroadcastDispatch.java:149)
	at org.gradle.internal.event.AbstractBroadcastDispatch.dispatch(AbstractBroadcastDispatch.java:58)
	at org.gradle.internal.event.BroadcastDispatch$CompositeDispatch.dispatch(BroadcastDispatch.java:324)
	at org.gradle.internal.event.BroadcastDispatch$CompositeDispatch.dispatch(BroadcastDispatch.java:234)
	at org.gradle.internal.event.ListenerBroadcast.dispatch(ListenerBroadcast.java:140)
	at org.gradle.internal.event.ListenerBroadcast.dispatch(ListenerBroadcast.java:37)
	at org.gradle.internal.dispatch.ProxyDispatchAdapter$DispatchingInvocationHandler.invoke(ProxyDispatchAdapter.java:93)
	at com.sun.proxy.$Proxy27.afterEvaluate(Unknown Source)
	at org.gradle.configuration.project.LifecycleProjectEvaluator.notifyAfterEvaluate(LifecycleProjectEvaluator.java:76)
	... 85 more
```

解决方案

问题定位到自定义插件的代码

```java
if (conf.isCanBeResolved()) {
    f.addAll(conf.getFiles())
}
```

当调用conf.getFiles() 方法时，就会报错。报错的信息显示，`Could not resolve all files for configuration':demo:releaseUnitTestRuntimeClasspath' `。根据报错信息提示，应该是解析不了releaseUnitTestRuntimeClasspath配置的所有文件。



这个问题困扰了很久，最终临时把这些解析不了的task全部过滤掉，然后就可以正常运行了。



## 升级易龙智投项目

问题一

```java
A problem occurred configuring project ':app'.
> Failed to notify project evaluation listener.
   > Could not resolve all files for configuration ':app:apt'.
      > Could not resolve project :unicorn.
        Required by:
            project :app
         > Cannot choose between the following configurations of project :unicorn:
             - debugApiElements
             - debugRuntimeElements
             - releaseApiElements
             - releaseRuntimeElements
           All of them match the consumer attributes:
             - Configuration 'debugApiElements':
                 - Found com.android.build.api.attributes.BuildTypeAttr 'debug' but wasn't required.
                 - Found com.android.build.api.attributes.VariantAttr 'debug' but wasn't required.
                 - Found com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' but wasn't required.
                 - Found org.gradle.usage 'java-api' but wasn't required.
             - Configuration 'debugRuntimeElements':
                 - Found com.android.build.api.attributes.BuildTypeAttr 'debug' but wasn't required.
                 - Found com.android.build.api.attributes.VariantAttr 'debug' but wasn't required.
                 - Found com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' but wasn't required.
                 - Found org.gradle.usage 'java-runtime' but wasn't required.
             - Configuration 'releaseApiElements':
                 - Found com.android.build.api.attributes.BuildTypeAttr 'release' but wasn't required.
                 - Found com.android.build.api.attributes.VariantAttr 'release' but wasn't required.
                 - Found com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' but wasn't required.
                 - Found org.gradle.usage 'java-api' but wasn't required.
             - Configuration 'releaseRuntimeElements':
                 - Found com.android.build.api.attributes.BuildTypeAttr 'release' but wasn't required.
                 - Found com.android.build.api.attributes.VariantAttr 'release' but wasn't required.
                 - Found com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' but wasn't required.
                 - Found org.gradle.usage 'java-runtime' but wasn't required.
```

解决方案
使用Android 官方提供的annotationProcessor 去代替android-apt。
删除Android-apt的配置
project的build.gradle文件中删除
`classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'`
module的build.gradle文件中删除
`apply plugin: 'com.neenbedankt.android-apt'`
module的build.gradle文件中替换
```groovy
//apt 'com.jakewharton:butterknife-compiler:8.0.1'
annotationProcessor 'com.jakewharton:butterknife-compiler:8.0.1'
```
build.gradle文件中使用android-apt引入的依赖修改为使用annotationProcessor，修改前配置如下
dependencies {
  apt rootProject.ext.dependencies["dagger-compiler"]
}
修改后配置如下：

dependencies {
    annotationProcessor rootProject.ext.dependencies["dagger-compiler"]
}



问题二

```groovy
java.lang.RuntimeException: Manifest Tasks does not support the manifestOutputFile property any more, please use the manifestOutputDirectory instead.
```



问题三

```groovy
No signature of method: com.android.build.gradle.internal.scope.VariantScopeImpl.getMergeResourcesTask() is applicable for argument types: () values: []
  Possible solutions: getMergeJavaResourcesTask()

```
主要是下面这段代码引起的。这段代码用于固定资源id。
```groovy
afterEvaluate {
    for (variant in android.applicationVariants) {
        def scope = variant.getVariantData().getScope()
        String mergeTaskName = scope.getMergeResourcesTask().name
        def mergeTask = tasks.getByName(mergeTaskName)

        mergeTask.doLast {
            copy {
                int i=0
                from(project.getRootDir().getAbsolutePath() +'/app+stub/src/main/resstub/values') {
                    include 'public.xml'
                    rename 'public-stub.xml',
                            (i++ == 0? "public-stub.xml": "public-stub-${i}.xml")
                }
                into(mergeTask.outputDir.getAbsolutePath()+File.separator+"values")
            }
        }
    }
}
```

将代码改为

```groovy
project.afterEvaluate {
    if (project.plugins.hasPlugin("com.android.application")) {
        def android = project.extensions.getByName("android")
        android.applicationVariants.all {def variant ->
            def processResourcesTask = project.tasks.getByName("process${variant.name.capitalize()}Resources")
            if (processResourcesTask) {
                def aaptOptions = processResourcesTask.aaptOptions
//                def path = pro.getAbsolutePath()+File.separator+"values"+File.separator+"public-stub.xml"
                File publicTxtFile = project.rootProject.file('public.txt')
                //public文件存在，则应用，不存在则生成
                if (publicTxtFile.exists()) {
                    project.logger.error "${publicTxtFile} exists, apply it."
                    //aapt2添加--stable-ids参数应用
                    aaptOptions.additionalParameters("--stable-ids", "${publicTxtFile}")
                } else {
                    project.logger.error "${publicTxtFile} not exists, generate it."
                    //aapt2添加--emit-ids参数生成
                    aaptOptions.additionalParameters("--emit-ids", "${publicTxtFile}")
                }
            }
        }
    }
}
```

这种方式还需要测试。



问题四

```groovy
org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration$ArtifactResolveException: Could not resolve all files for configuration ':openaccount:compile'.
```



出问题代码

```groovy
if (compile.isCanBeResolved()) {
			f.addAll(compile.getFiles());
		}

		if (provided.isCanBeResolved()) {
			f.addAll(provided.getFiles());
		}

		if (debugCompile.isCanBeResolved()) {
			f.addAll(debugCompile.getFiles());
		}

		if (releaseProvided.isCanBeResolved()) {
			f.addAll(releaseProvided.getFiles());
		}

```



和APM中遇到的问题一样，暂时将异常抛出。

问题五

```groovy
org.gradle.api.InvalidUserDataException: Cannot change role of configuration ':app:compile' after it has been resolved.
```

解决方式

将setCanbeResolved(true)代码删掉



问题六

将下面代码

```groovy
Configuration compile = project.getConfigurations().getByName("compile");
```

修改为

```groovy
Configuration compile = project.getConfigurations().getByName("implementation");
```





```groovy
Error:All flavors must now belong to a named flavor dimension.
The flavor 'flavor_name' is not assigned to a flavor dimension.
```






