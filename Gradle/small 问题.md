尝试方法

方法一

获取依赖时，添加上productFlavor，会报错

方法二

将没有flavor的buildLib结果拷贝到有flavor的项目中，buildBundle会报错



获取依赖的这一步出现问题。Gradle 3.x获取依赖的方式与Gradle 2.x不同。



small中，遍历variants时会过滤掉非release的变体。在没有product flavor时，只有两个变体debug和release。过滤掉debug后，其实只有一个变体release。

现在，因为有多个product flavor，就会有多个release变体。总会出现这种类型的问题。

```
More than one variant of project :app+stub matches the consumer attributes:
```

根据错误信息，猜测出错原因可能和变体有关。

尝试解决方案

设置当前Flavor，在遍历变体时，只有等于当前变体时，才继续进行。组装成带有Flavor的变体，然后运行。

结果：还是出现问题

**在GitHub上也有人问这个问题，不过都没有给出解决方案。**

[AmbiguousVariantSelectionException问题链接](https://github.com/gradle/gradle/issues/5426)



是否可以考虑重写small？但是花费时间可能更多

尝试解决下面问题

捕获异常

```
Caused by: org.gradle.internal.component.AmbiguousVariantSelectionException: More than one variant of project :app+stub matches the consumer attributes:
  - Configuration ':app+stub:normalReleaseApiElements' variant android-aidl:
      - Found artifactType 'android-aidl' but wasn't required.
      - Required com.android.build.api.attributes.BuildTypeAttr 'release' and found compatible value 'release'.
      - Required com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' and found compatible value 'Aar'.
      - Found com.android.build.gradle.internal.dependency.VariantAttr 'normalRelease' but wasn't required.
      - Required normal 'normal' and found compatible value 'normal'.
      - Required org.gradle.usage 'java-api' and found compatible value 'java-api'.
  - Configuration ':app+stub:normalReleaseApiElements' variant android-classes:
      - Found artifactType 'android-classes' but wasn't required.
      - Required com.android.build.api.attributes.BuildTypeAttr 'release' and found compatible value 'release'.
      - Required com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' and found compatible value 'Aar'.
      - Found com.android.build.gradle.internal.dependency.VariantAttr 'normalRelease' but wasn't required.
      - Required normal 'normal' and found compatible value 'normal'.
      - Required org.gradle.usage 'java-api' and found compatible value 'java-api'.
  - Configuration ':app+stub:normalReleaseApiElements' variant android-manifest:
      - Found artifactType 'android-manifest' but wasn't required.
      - Required com.android.build.api.attributes.BuildTypeAttr 'release' and found compatible value 'release'.
      - Required com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' and found compatible value 'Aar'.
      - Found com.android.build.gradle.internal.dependency.VariantAttr 'normalRelease' but wasn't required.
      - Required normal 'normal' and found compatible value 'normal'.
      - Required org.gradle.usage 'java-api' and found compatible value 'java-api'.
  - Configuration ':app+stub:normalReleaseApiElements' variant android-renderscript:
      - Found artifactType 'android-renderscript' but wasn't required.
      - Required com.android.build.api.attributes.BuildTypeAttr 'release' and found compatible value 'release'.
      - Required com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' and found compatible value 'Aar'.
      - Found com.android.build.gradle.internal.dependency.VariantAttr 'normalRelease' but wasn't required.
      - Required normal 'normal' and found compatible value 'normal'.
      - Required org.gradle.usage 'java-api' and found compatible value 'java-api'.
  - Configuration ':app+stub:normalReleaseApiElements' variant jar:
      - Found artifactType 'jar' but wasn't required.
      - Required com.android.build.api.attributes.BuildTypeAttr 'release' and found compatible value 'release'.
      - Required com.android.build.gradle.internal.dependency.AndroidTypeAttr 'Aar' and found compatible value 'Aar'.
      - Found com.android.build.gradle.internal.dependency.VariantAttr 'normalRelease' but wasn't required.
      - Required normal 'normal' and found compatible value 'normal'.
      - Required org.gradle.usage 'java-api' and found compatible value 'java-api'.
        at org.gradle.api.internal.artifacts.transform.AttributeMatchingVariantSelector.doSelect(AttributeMatchingVariantSelector.java:70)
        at org.gradle.api.internal.artifacts.transform.AttributeMatchingVariantSelector.select(AttributeMatchingVariantSelector.java:55)
        at org.gradle.api.internal.artifacts.ivyservice.resolveengine.artifact.DefaultArtifactSet.select(DefaultArtifactSet.java:125)
        at org.gradle.api.internal.artifacts.ivyservice.resolveengine.artifact.DefaultVisitedArtifactResults.select(DefaultVisitedArtifactResults.java:48)
        at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.getSelectedArtifacts(DefaultLenientConfiguration.java:94)
        at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.getFirstLevelNodes(DefaultLenientConfiguration.java:173)
        at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.getFirstLevelModuleDependencies(DefaultLenientConfiguration.java:165)
        at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.getFirstLevelModuleDependencies(DefaultLenientConfiguration.java:282)
        at org.gradle.api.internal.artifacts.ivyservice.DefaultResolvedConfiguration.getFirstLevelModuleDependencies(DefaultResolvedConfiguration.java:67)
        at org.gradle.api.internal.artifacts.ivyservice.ErrorHandlingConfigurationResolver$ErrorHandlingResolvedConfiguration.getFirstLevelModuleDependencies(ErrorHandlingConfigurationResolver.java:280)
        at net.wequick.gradle.util.DependenciesUtils.getAllDependencies(DependenciesUtils.groovy:58)
        at net.wequick.gradle.util.DependenciesUtils$getAllDependencies$4.callStatic(Unknown Source)
        at net.wequick.gradle.util.DependenciesUtils.getAllDependencies(DependenciesUtils.groovy:53)
        at net.wequick.gradle.util.DependenciesUtils$getAllDependencies$3.callStatic(Unknown Source)
        at net.wequick.gradle.util.DependenciesUtils.getAllCompileDependencies(DependenciesUtils.groovy:39)
        at net.wequick.gradle.util.DependenciesUtils$getAllCompileDependencies$0.call(Unknown Source)
        at net.wequick.gradle.RootPlugin.createPrepareResourceAarTasks(RootPlugin.groovy:442)
        at net.wequick.gradle.RootPlugin$createPrepareResourceAarTasks.callStatic(Unknown Source)
        at net.wequick.gradle.RootPlugin.createPrepareTasks(RootPlugin.groovy:408)
        at org.gradle.internal.metaobject.BeanDynamicObject$GroovyObjectAdapter.invokeOpaqueMethod(BeanDynamicObject.java:579)
        at org.gradle.internal.metaobject.BeanDynamicObject$MetaClassAdapter.invokeMethod(BeanDynamicObject.java:506)
        at org.gradle.internal.metaobject.BeanDynamicObject.tryInvokeMethod(BeanDynamicObject.java:191)
        at org.gradle.internal.metaobject.ConfigureDelegate.invokeMethod(ConfigureDelegate.java:78)
        at net.wequick.gradle.RootPlugin$_createPrepareTasks_closure17$_closure67$_closure68$_closure69.doCall(RootPlugin.groovy:396)
        at org.gradle.api.internal.ClosureBackedAction.execute(ClosureBackedAction.java:71)
        at org.gradle.util.ConfigureUtil.configureTarget(ConfigureUtil.java:160)
        at org.gradle.util.ConfigureUtil.configure(ConfigureUtil.java:106)
        at org.gradle.util.ConfigureUtil$1.execute(ConfigureUtil.java:123)
        at org.gradle.api.internal.DefaultDomainObjectCollection.all(DefaultDomainObjectCollection.java:136)
        at org.gradle.api.internal.DefaultDomainObjectCollection.all(DefaultDomainObjectCollection.java:154)
        at org.gradle.api.DomainObjectCollection$all$1.call(Unknown Source)
        at net.wequick.gradle.RootPlugin$_createPrepareTasks_closure17$_closure67$_closure68.doCall(RootPlugin.groovy:393)
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
        at com.sun.proxy.$Proxy25.afterEvaluate(Unknown Source)
        at org.gradle.configuration.project.LifecycleProjectEvaluator.notifyAfterEvaluate(LifecycleProjectEvaluator.java:76)
        at org.gradle.configuration.project.LifecycleProjectEvaluator.doConfigure(LifecycleProjectEvaluator.java:70)
        at org.gradle.configuration.project.LifecycleProjectEvaluator.access$100(LifecycleProjectEvaluator.java:34)
        at org.gradle.configuration.project.LifecycleProjectEvaluator$ConfigureProject.run(LifecycleProjectEvaluator.java:110)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:336)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:328)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:199)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:110)
        at org.gradle.configuration.project.LifecycleProjectEvaluator.evaluate(LifecycleProjectEvaluator.java:50)
        at org.gradle.api.internal.project.DefaultProject.evaluate(DefaultProject.java:667)
        at org.gradle.api.internal.project.DefaultProject.evaluate(DefaultProject.java:136)
        at org.gradle.execution.TaskPathProjectEvaluator.configure(TaskPathProjectEvaluator.java:35)
        at org.gradle.execution.TaskPathProjectEvaluator.configureHierarchy(TaskPathProjectEvaluator.java:62)
        at org.gradle.configuration.DefaultBuildConfigurer.configure(DefaultBuildConfigurer.java:38)
        at org.gradle.initialization.DefaultGradleLauncher$ConfigureBuild.run(DefaultGradleLauncher.java:249)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:336)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:328)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:199)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:110)
        at org.gradle.initialization.DefaultGradleLauncher.configureBuild(DefaultGradleLauncher.java:167)
        at org.gradle.initialization.DefaultGradleLauncher.doBuildStages(DefaultGradleLauncher.java:126)
        at org.gradle.initialization.DefaultGradleLauncher.executeTasks(DefaultGradleLauncher.java:109)
        at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:78)
        at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:75)
        at org.gradle.internal.work.DefaultWorkerLeaseService.withLocks(DefaultWorkerLeaseService.java:152)
        at org.gradle.internal.invocation.GradleBuildController.doBuild(GradleBuildController.java:100)
        at org.gradle.internal.invocation.GradleBuildController.run(GradleBuildController.java:75)
        at org.gradle.tooling.internal.provider.ExecuteBuildActionRunner.run(ExecuteBuildActionRunner.java:28)
        at org.gradle.launcher.exec.ChainingBuildActionRunner.run(ChainingBuildActionRunner.java:35)
        at org.gradle.tooling.internal.provider.ValidatingBuildActionRunner.run(ValidatingBuildActionRunner.java:32)
        at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner$1.run(RunAsBuildOperationBuildActionRunner.java:43)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:336)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:328)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:199)
        at org.gradle.internal.progress.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:110)
        at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner.run(RunAsBuildOperationBuildActionRunner.java:40)
        at org.gradle.tooling.internal.provider.SubscribableBuildActionRunner.run(SubscribableBuildActionRunner.java:51)
        at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:49)
        at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:32)
        at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:39)
        at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:25)
        at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:80)
        at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:53)
        at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:57)
        at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:32)
        at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:36)
        at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:25)
        at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:43)
        at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:29)
        at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:64)
        at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:29)
        at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:59)
        at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:44)
        at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:45)
        at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:30)
        at org.gradle.launcher.daemon.server.exec.ExecuteBuild.doBuild(ExecuteBuild.java:67)
        at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.WatchForDisconnection.execute(WatchForDisconnection.java:37)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.ResetDeprecationLogger.execute(ResetDeprecationLogger.java:26)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.RequestStopIfSingleUsedDaemon.execute(RequestStopIfSingleUsedDaemon.java:34)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:74)
        at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:72)
        at org.gradle.util.Swapper.swap(Swapper.java:38)
        at org.gradle.launcher.daemon.server.exec.ForwardClientInput.execute(ForwardClientInput.java:72)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.LogAndCheckHealth.execute(LogAndCheckHealth.java:55)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.LogToClient.doBuild(LogToClient.java:62)
        at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.EstablishBuildEnvironment.doBuild(EstablishBuildEnvironment.java:82)
        at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.StartBuildOrRespondWithBusy$1.run(StartBuildOrRespondWithBusy.java:50)
        at org.gradle.launcher.daemon.server.DaemonStateCoordinator$1.run(DaemonStateCoordinator.java:295)

```



./gradlew :app:dependencies --configuration normalReleaseCompileClasspath

```java
normalReleaseCompileClasspath - Resolved configuration for compilation for variant: normalRelease
+--- project :app+stub
|    \--- com.android.support.constraint:constraint-layout:1.0.2
|         \--- com.android.support.constraint:constraint-layout-solver:1.0.2
+--- com.android.databinding:library:1.3.1
|    +--- com.android.support:support-v4:21.0.3 -> 25.1.0
|    |    +--- com.android.support:support-compat:25.1.0
|    |    |    \--- com.android.support:support-annotations:25.1.0
|    |    +--- com.android.support:support-media-compat:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    +--- com.android.support:support-core-utils:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    +--- com.android.support:support-core-ui:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    \--- com.android.support:support-fragment:25.1.0
|    |         +--- com.android.support:support-compat:25.1.0 (*)
|    |         +--- com.android.support:support-media-compat:25.1.0 (*)
|    |         +--- com.android.support:support-core-ui:25.1.0 (*)
|    |         \--- com.android.support:support-core-utils:25.1.0 (*)
|    \--- com.android.databinding:baseLibrary:2.3.0-dev -> 3.0.0
+--- com.android.databinding:baseLibrary:3.0.0
+--- com.android.databinding:adapters:1.3.1
|    +--- com.android.databinding:library:1.3 -> 1.3.1 (*)
|    \--- com.android.databinding:baseLibrary:2.3.0-dev -> 3.0.0
+--- com.android.support:appcompat-v7:23.2.1 -> 25.1.0
|    +--- com.android.support:support-annotations:25.1.0
|    +--- com.android.support:support-v4:25.1.0 (*)
|    +--- com.android.support:support-vector-drawable:25.1.0
|    |    +--- com.android.support:support-annotations:25.1.0
|    |    \--- com.android.support:support-compat:25.1.0 (*)
|    \--- com.android.support:animated-vector-drawable:25.1.0
|         \--- com.android.support:support-vector-drawable:25.1.0 (*)
+--- com.android.support:design:23.2.1 -> 25.1.0
|    +--- com.android.support:support-v4:25.1.0 (*)
|    +--- com.android.support:appcompat-v7:25.1.0 (*)
|    +--- com.android.support:recyclerview-v7:25.1.0
|    |    +--- com.android.support:support-annotations:25.1.0
|    |    +--- com.android.support:support-compat:25.1.0 (*)
|    |    \--- com.android.support:support-core-ui:25.1.0 (*)
|    \--- com.android.support:transition:25.1.0
|         +--- com.android.support:support-annotations:25.1.0
|         \--- com.android.support:support-v4:25.1.0 (*)
+--- project :small
\--- small.support:databinding:1.1.2

(*) - dependencies omitted (listed previously)

```

**./gradlew :app:dependencies --configuration implementation**

```java
implementation - Implementation only dependencies for 'main' sources. (n)
+--- unspecified (n)
+--- com.android.support:appcompat-v7:23.2.1 (n)
+--- com.android.support:design:23.2.1 (n)
+--- project small (n)
\--- small.support:databinding:1.1.2 (n)

(n) - Not resolved (configuration is not meant to be resolved)

```



在Gradle 2.x中，可以通过compile获得配置。然而，在Gradle3.x中，compile被弃用，获取得到的配置为null。

**Gradle 2.x**

**./gradlew :app:dependencies --configuration compile**

```java
compile - Classpath for compiling the main sources.
+--- com.android.support:appcompat-v7:25.1.0
|    +--- com.android.support:support-annotations:25.1.0
|    +--- com.android.support:support-v4:25.1.0
|    |    +--- com.android.support:support-compat:25.1.0
|    |    |    \--- com.android.support:support-annotations:25.1.0
|    |    +--- com.android.support:support-media-compat:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    +--- com.android.support:support-core-utils:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    +--- com.android.support:support-core-ui:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    \--- com.android.support:support-fragment:25.1.0
|    |         +--- com.android.support:support-compat:25.1.0 (*)
|    |         +--- com.android.support:support-media-compat:25.1.0 (*)
|    |         +--- com.android.support:support-core-ui:25.1.0 (*)
|    |         \--- com.android.support:support-core-utils:25.1.0 (*)
|    +--- com.android.support:support-vector-drawable:25.1.0
|    |    +--- com.android.support:support-annotations:25.1.0
|    |    \--- com.android.support:support-compat:25.1.0 (*)
|    \--- com.android.support:animated-vector-drawable:25.1.0
|         \--- com.android.support:support-vector-drawable:25.1.0 (*)
+--- com.android.support:design:25.1.0
|    +--- com.android.support:support-v4:25.1.0 (*)
|    +--- com.android.support:appcompat-v7:25.1.0 (*)
|    +--- com.android.support:recyclerview-v7:25.1.0
|    |    +--- com.android.support:support-annotations:25.1.0
|    |    +--- com.android.support:support-compat:25.1.0 (*)
|    |    \--- com.android.support:support-core-ui:25.1.0 (*)
|    \--- com.android.support:transition:25.1.0
|         +--- com.android.support:support-annotations:25.1.0
|         \--- com.android.support:support-v4:25.1.0 (*)
+--- project :small
|    \--- com.android.support:appcompat-v7:25.1.0 (*)
+--- project :small-databinding
+--- project :app+stub
|    +--- com.android.support.constraint:constraint-layout:1.0.2
|    |    \--- com.android.support.constraint:constraint-layout-solver:1.0.2
|    +--- project :vendor-aar
|    +--- com.jakewharton:butterknife:8.6.0
|    |    +--- com.jakewharton:butterknife-annotations:8.6.0
|    |    |    \--- com.android.support:support-annotations:25.1.0
|    |    +--- com.android.support:support-annotations:25.1.0
|    |    \--- com.android.support:support-compat:25.1.0 (*)
|    \--- com.android.support:appcompat-v7:25.1.0 (*)
+--- com.android.databinding:library:1.3.1
|    +--- com.android.support:support-v4:21.0.3 -> 25.1.0 (*)
|    \--- com.android.databinding:baseLibrary:2.3.0-dev -> 2.3.0
+--- com.android.databinding:baseLibrary:2.3.0
\--- com.android.databinding:adapters:1.3.1
     +--- com.android.databinding:library:1.3 -> 1.3.1 (*)
     \--- com.android.databinding:baseLibrary:2.3.0-dev -> 2.3.0

(*) - dependencies omitted (listed previously)


```

