# Android Gradle Plugin 源码分析

本文是根据gradle3.1.2源码进行分析。Android Gradle Plugin本质上是一个gradle插件，肯定遵循自定义插件的结构，即继承Plugin类的源码，resources目录结构等。为简便起见，以下都将Android Gradle Plugin简称为AGP。

在分析源码之前，我们需要先下载AGP的源码。我们在此链接手动下载AGP3.1.2的源码。

我们在创建一个Android新项目时，项目模块的build.gradle文件中的第一行往往都是`apply plugin: 'com.android.application'`这行代码。这行代码会调用`com.android.application`插件的apply方法。

我们打开AGP源码的resources目录下的`com.android.application.properties`文件。其内容为`implementation-class=com.android.build.gradle.AppPlugin`

![AGP插件的配置](E:\笔记\images\agp-resources.PNG)

因此，我们从AppPlugin类的apply方法开始分析。



![apply](images\apply.png)



```java
public class AppPlugin extends BasePlugin<AppExtensionImpl> {
    ...
    @Override
    public void apply(@NonNull Project project) {
        super.apply(project);
    }
}
```

AppPlugin#apply方法很简单，只是调用父类BasePlugin#apply的方法。我们继续查看BasePlugin#apply。

```java
 @Override
    public void apply(@NonNull Project project) {
        // We run by default in headless mode, so the JVM doesn't steal focus.
        System.setProperty("java.awt.headless", "true");

        this.project = project;
        this.projectOptions = new ProjectOptions(project);

        project.getPluginManager().apply(AndroidBasePlugin.class);

        checkConfigureOnDemandGradleVersionCompat();
        checkPathForErrors();
        checkModulesForErrors();

        PluginInitializer.initialize(project);
        ProfilerInitializer.init(project, projectOptions);
        threadRecorder = ThreadRecorder.get();

        ProcessProfileWriter.getProject(project.getPath())
                .setAndroidPluginVersion(Version.ANDROID_GRADLE_PLUGIN_VERSION)
                .setAndroidPlugin(getAnalyticsPluginType())
                .setPluginGeneration(GradleBuildProject.PluginGeneration.FIRST)
                .setOptions(AnalyticsUtil.toProto(projectOptions));

        BuildableArtifactImpl.Companion.disableResolution();
        if (!projectOptions.get(BooleanOption.ENABLE_NEW_DSL_AND_API)) {
            TaskInputHelper.enableBypass();

            threadRecorder.record(
                    ExecutionType.BASE_PLUGIN_PROJECT_CONFIGURE,
                    project.getPath(),
                    null,
                    this::configureProject);

            threadRecorder.record(
                    ExecutionType.BASE_PLUGIN_PROJECT_BASE_EXTENSION_CREATION,
                    project.getPath(),
                    null,
                    this::configureExtension);

            threadRecorder.record(
                    ExecutionType.BASE_PLUGIN_PROJECT_TASKS_CREATION,
                    project.getPath(),
                    null,
                    this::createTasks);
        } else {
           // 代码省略
        }
    }
```

上述代码中，依次执行configureProject，configureExtension，createTasks方法。

下面我们将依次分析这三个方法。

BasePlugin#configureProject

```java
 private void configureProject() {
        final Gradle gradle = project.getGradle();

        extraModelInfo = new ExtraModelInfo(project.getPath(), projectOptions, project.getLogger());
     // 检查Gradle版本，如果Gradle版本低于所要求的最低版本，
     // 则抛出异常。gradle3.1.2要求gradle版本最低为4.4
        checkGradleVersion(project, getLogger(), projectOptions);

        sdkHandler = new SdkHandler(project, getLogger());
	// 省略代码

        androidBuilder =
                new AndroidBuilder(
                        project == project.getRootProject() ? project.getName() : project.getPath(),
                        creator,
                        new GradleProcessExecutor(project),
                        new GradleJavaProcessExecutor(project),
                        extraModelInfo.getSyncIssueHandler(),
                        extraModelInfo.getMessageReceiver(),
                        getLogger(),
                        isVerbose());
        dataBindingBuilder = new DataBindingBuilder();
        dataBindingBuilder.setPrintMachineReadableOutput(
                SyncOptions.getErrorFormatMode(projectOptions) == ErrorFormatMode.MACHINE_PARSABLE);
		// 检查是否有被移除的配置选项
        if (projectOptions.hasRemovedOptions()) {
            androidBuilder
                    .getIssueReporter()
                    .reportWarning(Type.GENERIC, projectOptions.getRemovedOptionsErrorMessage());
        }
		// 检查是否使用被弃用的配置选项
        if (projectOptions.hasDeprecatedOptions()) {
            extraModelInfo
                    .getDeprecationReporter()
                    .reportDeprecatedOptions(projectOptions.getDeprecatedOptions());
        }

        // Apply the Java plugin
        project.getPlugins().apply(JavaBasePlugin.class);

        project.getTasks()
                .getByName("assemble")
                .setDescription(
                        "Assembles all variants of all applications and secondary packages.");

        // call back on execution. This is called after the whole build is done (not
        // after the current project is done).
        // This is will be called for each (android) projects though, so this should support
        // being called 2+ times.
        gradle.addBuildListener(
                new BuildListener() {
                    @Override
                    public void buildStarted(@NonNull Gradle gradle) {
                        TaskInputHelper.enableBypass();
                        BuildableArtifactImpl.Companion.disableResolution();
                    }

                    @Override
                    public void settingsEvaluated(@NonNull Settings settings) {}

                    @Override
                    public void projectsLoaded(@NonNull Gradle gradle) {}

                    @Override
                    public void projectsEvaluated(@NonNull Gradle gradle) {}

                    @Override
                    public void buildFinished(@NonNull BuildResult buildResult) {
                        // Do not run buildFinished for included project in composite build.
                        if (buildResult.getGradle().getParent() != null) {
                            return;
                        }
                        sdkHandler.unload();
                        threadRecorder.record(
                                ExecutionType.BASE_PLUGIN_BUILD_FINISHED,
                                project.getPath(),
                                null,
                                () -> {
                                    WorkerActionServiceRegistry.INSTANCE
                                            .shutdownAllRegisteredServices(
                                                    ForkJoinPool.commonPool());
                                    PreDexCache.getCache()
                                            .clear(
                                                    FileUtils.join(
                                                            project.getRootProject().getBuildDir(),
                                                            FD_INTERMEDIATES,
                                                            "dex-cache",
                                                            "cache.xml"),
                                                    getLogger());
                                    Main.clearInternTables();
                                });
                    }
                });

        gradle.getTaskGraph()
                .addTaskExecutionGraphListener(
                        taskGraph -> {
                            TaskInputHelper.disableBypass();
                            Aapt2DaemonManagerService.registerAaptService(
                                    Objects.requireNonNull(androidBuilder.getTargetInfo())
                                            .getBuildTools(),
                                    loggerWrapper,
                                    WorkerActionServiceRegistry.INSTANCE);

                            for (Task task : taskGraph.getAllTasks()) {
                                if (task instanceof TransformTask) {
                                    Transform transform = ((TransformTask) task).getTransform();
                                    if (transform instanceof DexTransform) {
                                        PreDexCache.getCache()
                                                .load(
                                                        FileUtils.join(
                                                                project.getRootProject()
                                                                        .getBuildDir(),
                                                                FD_INTERMEDIATES,
                                                                "dex-cache",
                                                                "cache.xml"));
                                        break;
                                    }
                                }
                            }
                        });

        createLintClasspathConfiguration(project);
    }
```

从上面代码可以看出，该方法前面主要做一些gradle版本检查，以及配置选项是否被弃用等等，诸如此类的工作。



BasePlugin#configureExtension

```java
 private void configureExtension() {
        ObjectFactory objectFactory = project.getObjects();
        final NamedDomainObjectContainer<BuildType> buildTypeContainer =
                project.container(
                        BuildType.class,
                        new BuildTypeFactory(
                                objectFactory,
                                project,
                                extraModelInfo.getSyncIssueHandler(),
                                extraModelInfo.getDeprecationReporter()));
        final NamedDomainObjectContainer<ProductFlavor> productFlavorContainer =
                project.container(
                        ProductFlavor.class,
                        new ProductFlavorFactory(
                                objectFactory,
                                project,
                                extraModelInfo.getDeprecationReporter(),
                                project.getLogger()));
        final NamedDomainObjectContainer<SigningConfig> signingConfigContainer =
                project.container(
                        SigningConfig.class,
                        new SigningConfigFactory(
                                objectFactory,
                                GradleKeystoreHelper.getDefaultDebugKeystoreLocation()));

        final NamedDomainObjectContainer<BaseVariantOutput> buildOutputs =
                project.container(BaseVariantOutput.class);

        project.getExtensions().add("buildOutputs", buildOutputs);

        sourceSetManager = createSourceSetManager();

        extension =
                createExtension(
                        project,
                        projectOptions,
                        androidBuilder,
                        sdkHandler,
                        buildTypeContainer,
                        productFlavorContainer,
                        signingConfigContainer,
                        buildOutputs,
                        sourceSetManager,
                        extraModelInfo);

        ndkHandler =
                new NdkHandler(
                        project.getRootDir(),
                        null, /* compileSkdVersion, this will be set in afterEvaluate */
                        "gcc",
                        "" /*toolchainVersion*/,
                        false /* useUnifiedHeaders */);


        @Nullable
        FileCache buildCache = BuildCacheUtils.createBuildCacheIfEnabled(project, projectOptions);

        GlobalScope globalScope =
                new GlobalScope(
                        project,
                        projectOptions,
                        androidBuilder,
                        extension,
                        sdkHandler,
                        ndkHandler,
                        registry,
                        buildCache);

        variantFactory = createVariantFactory(globalScope, androidBuilder, extension);

        taskManager =
                createTaskManager(
                        globalScope,
                        project,
                        projectOptions,
                        androidBuilder,
                        dataBindingBuilder,
                        extension,
                        sdkHandler,
                        ndkHandler,
                        registry,
                        threadRecorder);

        variantManager =
                new VariantManager(
                        globalScope,
                        project,
                        projectOptions,
                        androidBuilder,
                        extension,
                        variantFactory,
                        taskManager,
                        sourceSetManager,
                        threadRecorder);

        registerModels(registry, globalScope, variantManager, extension, extraModelInfo);

        // map the whenObjectAdded callbacks on the containers.
        signingConfigContainer.whenObjectAdded(variantManager::addSigningConfig);

        buildTypeContainer.whenObjectAdded(
                buildType -> {
                    SigningConfig signingConfig =
                            signingConfigContainer.findByName(BuilderConstants.DEBUG);
                    buildType.init(signingConfig);
                    variantManager.addBuildType(buildType);
                });

        productFlavorContainer.whenObjectAdded(variantManager::addProductFlavor);

        // map whenObjectRemoved on the containers to throw an exception.
        signingConfigContainer.whenObjectRemoved(
                new UnsupportedAction("Removing signingConfigs is not supported."));
        buildTypeContainer.whenObjectRemoved(
                new UnsupportedAction("Removing build types is not supported."));
        productFlavorContainer.whenObjectRemoved(
                new UnsupportedAction("Removing product flavors is not supported."));

        // create default Objects, signingConfig first as its used by the BuildTypes.
        variantFactory.createDefaultComponents(
                buildTypeContainer, productFlavorContainer, signingConfigContainer);
    }
```

该方法主要是创建Extension，并且创建相应的TaskManager和VariantManager。AppPlugin对应的Extension是AppExtension，对应的TaskManager是ApplicationTaskManager。

然后依次添加signingConfig，buildType和productFlavor。主要通过调用Variant Manager的addSigningConfig，addBuildType以及addProductFlavor方法。



BasePlugin#createTasks

```java
private void createTasks() {
        threadRecorder.record(
                ExecutionType.TASK_MANAGER_CREATE_TASKS,
                project.getPath(),
                null,
                () -> taskManager.createTasksBeforeEvaluate());

        project.afterEvaluate(
                project ->
                        threadRecorder.record(
                                ExecutionType.BASE_PLUGIN_CREATE_ANDROID_TASKS,
                                project.getPath(),
                                null,
                                () -> createAndroidTasks(false)));
    }

```



BasePlugin#createAndroidTasks

```java
@VisibleForTesting
    final void createAndroidTasks(boolean force) {
        // Make sure unit tests set the required fields.
        checkState(extension.getBuildToolsRevision() != null,
                "buildToolsVersion is not specified.");
        checkState(extension.getCompileSdkVersion() != null, "compileSdkVersion is not specified.");

        ndkHandler.setCompileSdkVersion(extension.getCompileSdkVersion());

        // get current plugins and look for the default Java plugin.
        if (project.getPlugins().hasPlugin(JavaPlugin.class)) {
            throw new BadPluginException(
                    "The 'java' plugin has been applied, but it is not compatible with the Android plugins.");
        }

        boolean targetSetupSuccess = ensureTargetSetup();
        sdkHandler.ensurePlatformToolsIsInstalledWarnOnFailure(
                extraModelInfo.getSyncIssueHandler());
        // Stop trying to configure the project if the SDK is not ready.
        // Sync issues will already have been collected at this point in sync.
        if (!targetSetupSuccess) {
            project.getLogger()
                    .warn("Aborting configuration as SDK is missing components in sync mode.");
            return;
        }

        // don't do anything if the project was not initialized.
        // Unless TEST_SDK_DIR is set in which case this is unit tests and we don't return.
        // This is because project don't get evaluated in the unit test setup.
        // See AppPluginDslTest
        if (!force
                && (!project.getState().getExecuted() || project.getState().getFailure() != null)
                && SdkHandler.sTestSdkFolder == null) {
            return;
        }

        if (hasCreatedTasks) {
            return;
        }
        hasCreatedTasks = true;

        extension.disableWrite();

        taskManager.configureCustomLintChecks();

        ProcessProfileWriter.getProject(project.getPath())
                .setCompileSdk(extension.getCompileSdkVersion())
                .setBuildToolsVersion(extension.getBuildToolsRevision().toString())
                .setSplits(AnalyticsUtil.toProto(extension.getSplits()));

        String kotlinPluginVersion = getKotlinPluginVersion();
        if (kotlinPluginVersion != null) {
            ProcessProfileWriter.getProject(project.getPath())
                    .setKotlinPluginVersion(kotlinPluginVersion);
        }

        // setup SDK repositories.
        sdkHandler.addLocalRepositories(project);

        threadRecorder.record(
                ExecutionType.VARIANT_MANAGER_CREATE_ANDROID_TASKS,
                project.getPath(),
                null,
                () -> {
                    variantManager.createAndroidTasks();
                    ApiObjectFactory apiObjectFactory =
                            new ApiObjectFactory(
                                    androidBuilder,
                                    extension,
                                    variantFactory,
                                    project.getObjects());
                    for (VariantScope variantScope : variantManager.getVariantScopes()) {
                        BaseVariantData variantData = variantScope.getVariantData();
                        apiObjectFactory.create(variantData);
                    }

                    // Make sure no SourceSets were added through the DSL without being properly configured
                    // Only do it if we are not restricting to a single variant (with Instant
                    // Run or we can find extra source set
                    if (projectOptions.get(StringOption.IDE_RESTRICT_VARIANT_NAME) == null) {
                        sourceSetManager.checkForUnconfiguredSourceSets();
                    }

                    // must run this after scopes are created so that we can configure kotlin
                    // kapt tasks
                    taskManager.addDataBindingDependenciesIfNecessary(
                            extension.getDataBinding(), variantManager.getVariantScopes());
                });

        // create the global lint task that depends on all the variants
        taskManager.configureGlobalLintTask(variantManager.getVariantScopes());

        // Create and read external native build JSON files depending on what's happening right
        // now.
        //
        // CREATE PHASE:
        // Creates JSONs by shelling out to external build system when:
        //   - Any one of AndroidProject.PROPERTY_INVOKED_FROM_IDE,
        //      AndroidProject.PROPERTY_BUILD_MODEL_ONLY_ADVANCED,
        //      AndroidProject.PROPERTY_BUILD_MODEL_ONLY,
        //      AndroidProject.PROPERTY_REFRESH_EXTERNAL_NATIVE_MODEL are set.
        //   - *and* AndroidProject.PROPERTY_REFRESH_EXTERNAL_NATIVE_MODEL is set
        //      or JSON files don't exist or are out-of-date.
        // Create phase may cause ProcessException (from cmake.exe for example)
        //
        // READ PHASE:
        // Reads and deserializes JSONs when:
        //   - Any one of AndroidProject.PROPERTY_INVOKED_FROM_IDE,
        //      AndroidProject.PROPERTY_BUILD_MODEL_ONLY_ADVANCED,
        //      AndroidProject.PROPERTY_BUILD_MODEL_ONLY,
        //      AndroidProject.PROPERTY_REFRESH_EXTERNAL_NATIVE_MODEL are set.
        // Read phase may produce IOException if the file can't be read for standard IO reasons.
        // Read phase may produce JsonSyntaxException in the case that the content of the file is
        // corrupt.
        boolean forceRegeneration =
                projectOptions.get(BooleanOption.IDE_REFRESH_EXTERNAL_NATIVE_MODEL);

        checkSplitConfiguration();
        if (ExternalNativeBuildTaskUtils.shouldRegenerateOutOfDateJsons(projectOptions)) {
            threadRecorder.record(
                    ExecutionType.VARIANT_MANAGER_EXTERNAL_NATIVE_CONFIG_VALUES,
                    project.getPath(),
                    null,
                    () -> {
                        for (VariantScope variantScope : variantManager.getVariantScopes()) {
                            ExternalNativeJsonGenerator generator =
                                    variantScope.getExternalNativeJsonGenerator();
                            if (generator != null) {
                                // This will generate any out-of-date or non-existent JSONs.
                                // When refreshExternalNativeModel() is true it will also
                                // force update all JSONs.
                                generator.build(forceRegeneration);
                            }
                        }
                    });
        }
        BuildableArtifactImpl.Companion.enableResolution();
    }


```

VariantManager#createAndroidTasks



```java
public void createAndroidTasks() {
        variantFactory.validateModel(this);
        variantFactory.preVariantWork(project);

        if (variantScopes.isEmpty()) {
            recorder.record(
                    ExecutionType.VARIANT_MANAGER_CREATE_VARIANTS,
                    project.getPath(),
                    null /*variantName*/,
                    this::populateVariantDataList);
        }

        // Create top level test tasks.
        recorder.record(
                ExecutionType.VARIANT_MANAGER_CREATE_TESTS_TASKS,
                project.getPath(),
                null /*variantName*/,
                () -> taskManager.createTopLevelTestTasks(!productFlavors.isEmpty()));



        for (final VariantScope variantScope : variantScopes) {
            recorder.record(
                    ExecutionType.VARIANT_MANAGER_CREATE_TASKS_FOR_VARIANT,
                    project.getPath(),
                    variantScope.getFullVariantName(),
                    () -> createTasksForVariantData(variantScope));
        }

        taskManager.createReportTasks(variantScopes);
    }
```

该方法是创建变体和任务的入口。首先，判断variantScopes变量是否为空。variantScopes变量是VariantScope的List。而VariantScope是存放特定变体的数据。如果variantScopes为空，则会调用populateVariantDataList方法。populateVariantDataList方法用于创建所有的变体。现在我们查看一下populateVariantDataList方法如何创建变体的。

VariantManager#populateVariantDataList

```java
public void populateVariantDataList() {
        List<String> flavorDimensionList = extension.getFlavorDimensionList();

        if (productFlavors.isEmpty()) {
            configureDependencies();
            createVariantDataForProductFlavors(Collections.emptyList());
        } else {
            // ensure that there is always a dimension
            if (flavorDimensionList == null || flavorDimensionList.isEmpty()) {
                androidBuilder
                        .getIssueReporter()
                        .reportError(
                                EvalIssueReporter.Type.UNNAMED_FLAVOR_DIMENSION,
                                "All flavors must now belong to a named flavor dimension. "
                                        + "Learn more at "
                                        + "https://d.android.com/r/tools/flavorDimensions-missing-error-message.html");
            } else if (flavorDimensionList.size() == 1) {
                // if there's only one dimension, auto-assign the dimension to all the flavors.
                String dimensionName = flavorDimensionList.get(0);
                for (ProductFlavorData<CoreProductFlavor> flavorData : productFlavors.values()) {
                    CoreProductFlavor flavor = flavorData.getProductFlavor();
                    if (flavor.getDimension() == null && flavor instanceof DefaultProductFlavor) {
                        ((DefaultProductFlavor) flavor).setDimension(dimensionName);
                    }
                }
            }

            // can only call this after we ensure all flavors have a dimension.
            configureDependencies();

            // Create iterable to get GradleProductFlavor from ProductFlavorData.
            Iterable<CoreProductFlavor> flavorDsl =
                    Iterables.transform(
                            productFlavors.values(),
                            ProductFlavorData::getProductFlavor);

            // Get a list of all combinations of product flavors.
            List<ProductFlavorCombo<CoreProductFlavor>> flavorComboList =
                    ProductFlavorCombo.createCombinations(
                            flavorDimensionList,
                            flavorDsl);

            for (ProductFlavorCombo<CoreProductFlavor>  flavorCombo : flavorComboList) {
                //noinspection unchecked
                createVariantDataForProductFlavors(
                        (List<ProductFlavor>) (List) flavorCombo.getFlavorList());
            }
        }
    }
```

首先，通过extension变量获得变体的风味维度（即flavor dimension），然后，判断productFlavors是否为空，若为空，则调用configureDependencies和createVariantDataForProductFlavors方法。若不为空，则判断flavorDimensionList是否为null，若为null，则报告错误。如果你之前升级过gradle3.0，那么对这个报错信息肯定很熟悉。它提示你，所有的flavors都必须有一个维度，即dimension。dimension是Gradle3.0以上新加的特性。具体的内容可以见**升级Gradle3.x**文章。

分析完populateVariantDataList方法后，我们继续看VariantManager#createAndroidTasks 方法的执行过程。主要先后调用三个方法，createTopLevelTestTasks，createTasksForVariantData以及createReportTasks方法。createTopLevelTestTasks主要是用来创建一些测试任务，在这里就不具体分析了，有兴趣的可以自行查看源码。下面我们主要分析一下createTasksForVariantData方法。





VariantManager#createTasksForVariantData

```java
/** Create tasks for the specified variant. */
    public void createTasksForVariantData(final VariantScope variantScope) {
        final BaseVariantData variantData = variantScope.getVariantData();
        final VariantType variantType = variantData.getType();

        final GradleVariantConfiguration variantConfig = variantScope.getVariantConfiguration();

        final BuildTypeData buildTypeData = buildTypes.get(variantConfig.getBuildType().getName());
        if (buildTypeData.getAssembleTask() == null) {
            buildTypeData.setAssembleTask(taskManager.createAssembleTask(buildTypeData));
        }

        // Add dependency of assemble task on assemble build type task.
        taskManager
                .getTaskFactory()
                .configure(
                        "assemble",
                        task -> {
                            assert buildTypeData.getAssembleTask() != null;
                            task.dependsOn(buildTypeData.getAssembleTask().getName());
                        });

        createAssembleTaskForVariantData(variantData);
        if (variantType.isForTesting()) {
    		// 省略代码
        } else {
            taskManager.createTasksForVariantScope(variantScope);
        }
    }
```
VariantManager类主要用于创建和管理变体。这个方法主要作用是创建变体的task。主要是将variantScope参数传给TaskManager的createTasksForVariantScope方法。createTasksForVariantScope方法逻辑很简单，在非测试环境下，直接调用TaskManager的createTasksForVariantScope方法。上面我们说过，AppPlugin的TaskManger实现类是ApplicationTaskManager。我们将代码定位到ApplicationTaskManager类的createTasksForVariantScope方法。



ApplicationTaskManager#createTasksForVariantScope

```java
@Override
    public void createTasksForVariantScope(@NonNull final VariantScope variantScope) {
        BaseVariantData variantData = variantScope.getVariantData();
        assert variantData instanceof ApplicationVariantData;
		// 这个方法会创建preBuild任务
        createAnchorTasks(variantScope);
        createCheckManifestTask(variantScope);
		// 这个主要是针对可穿戴设备
        // Configure variantData to generate embedded wear application.
        handleMicroApp(variantScope);

        // Create all current streams (dependencies mostly at this point)
        createDependencyStreams(variantScope);

        // Add a task to publish the applicationId.
        createApplicationIdWriterTask(variantScope);

        taskFactory.create(new MainApkListPersistence.ConfigAction(variantScope));
        taskFactory.create(new BuildArtifactReportTask.ConfigAction(variantScope));

        // Add a task to process the manifest(s)
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_MERGE_MANIFEST_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createMergeApkManifestsTask(variantScope));

        // Add a task to create the res values
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_GENERATE_RES_VALUES_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createGenerateResValuesTask(variantScope));

        // Add a task to compile renderscript files.
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_CREATE_RENDERSCRIPT_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createRenderscriptTask(variantScope));

        // Add a task to merge the resource folders
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_MERGE_RESOURCES_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                (Recorder.VoidBlock) () -> createMergeResourcesTask(variantScope, true));

        // Add tasks to compile shader
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_SHADER_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createShaderTask(variantScope));


        // Add a task to merge the asset folders
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_MERGE_ASSETS_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> {
                    createMergeAssetsTask(variantScope);
                });

        // Add a task to create the BuildConfig class
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_BUILD_CONFIG_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createBuildConfigTask(variantScope));

        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_PROCESS_RES_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> {
                    // Add a task to process the Android Resources and generate source files
                    createApkProcessResTask(variantScope);

                    // Add a task to process the java resources
                    createProcessJavaResTask(variantScope);
                });

        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_AIDL_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createAidlTask(variantScope));

        // Add NDK tasks
        if (!isComponentModelPlugin()) {
            recorder.record(
                    ExecutionType.APP_TASK_MANAGER_CREATE_NDK_TASK,
                    project.getPath(),
                    variantScope.getFullVariantName(),
                    () -> createNdkTasks(variantScope));
        } else {
            if (variantData.compileTask != null) {
                variantData.compileTask.dependsOn(getNdkBuildable(variantData));
            } else {
                variantScope.getCompileTask().dependsOn(getNdkBuildable(variantData));
            }
        }
        variantScope.setNdkBuildable(getNdkBuildable(variantData));

        // Add external native build tasks

        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_EXTERNAL_NATIVE_BUILD_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> {
                    createExternalNativeBuildJsonGenerators(variantScope);
                    createExternalNativeBuildTasks(variantScope);
                });

        // Add a task to merge the jni libs folders
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_MERGE_JNILIBS_FOLDERS_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createMergeJniLibFoldersTasks(variantScope));

        // Add data binding tasks if enabled
        createDataBindingTasksIfNecessary(variantScope, MergeType.MERGE);

        // Add a compile task
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_COMPILE_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> addCompileTask(variantScope));

        createStripNativeLibraryTask(taskFactory, variantScope);

        if (variantScope.getVariantData().getMultiOutputPolicy().equals(MultiOutputPolicy.SPLITS)) {
            if (extension.getBuildToolsRevision().getMajor() < 21) {
                throw new RuntimeException(
                        "Pure splits can only be used with buildtools 21 and later");
            }

            recorder.record(
                    ExecutionType.APP_TASK_MANAGER_CREATE_SPLIT_TASK,
                    project.getPath(),
                    variantScope.getFullVariantName(),
                    () -> createSplitTasks(variantScope));
        }

        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_PACKAGING_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> {
                    BuildInfoWriterTask buildInfoWriterTask =
                            createInstantRunPackagingTasks(variantScope);
                    createPackagingTask(variantScope, buildInfoWriterTask);
                });

        // create the lint tasks.
        recorder.record(
                ExecutionType.APP_TASK_MANAGER_CREATE_LINT_TASK,
                project.getPath(),
                variantScope.getFullVariantName(),
                () -> createLintTasks(variantScope));
    }

```

在上述代码中看到，大部分的任务都是在createTasksForVariantScope方法里使用taskFactory创建的。taskFactory是TaskFactory接口的对象。TaskFactory的实现类是TaskFactoryImp。

我们运行一个Android工程的assembleRelease任务，就可以看到任务的执行顺序，如下所示：

```
:app:preBuild UP-TO-DATE
:app:preReleaseBuild
:app:compileReleaseAidl
:app:compileReleaseRenderscript
:app:checkReleaseManifest
:app:generateReleaseBuildConfig
:app:prepareLintJar UP-TO-DATE
:app:mainApkListPersistenceRelease
:app:generateReleaseResValues
:app:generateReleaseResources
:app:mergeReleaseResources
:app:createReleaseCompatibleScreenManifests
:app:processReleaseManifest
:app:splitsDiscoveryTaskRelease
:app:processReleaseResources
:app:generateReleaseSources
:app:javaPreCompileRelease
:app:compileReleaseJavaWithJavac
:app:compileReleaseNdk NO-SOURCE
:app:compileReleaseSources
:app:lintVitalRelease
:app:mergeReleaseShaders
:app:compileReleaseShaders
:app:generateReleaseAssets
:app:mergeReleaseAssets
:app:transformClassesWithDexBuilderForRelease
:app:transformDexArchiveWithExternalLibsDexMergerForRelease
:app:transformDexArchiveWithDexMergerForRelease
:app:mergeReleaseJniLibFolders
:app:transformNativeLibsWithMergeJniLibsForRelease
:app:transformNativeLibsWithStripDebugSymbolForRelease
:app:processReleaseJavaRes NO-SOURCE
:app:transformResourcesWithMergeJavaResForRelease
:app:packageRelease
:app:assembleRelease
```

由于上述task太多，我们在此只选择其中一个task进行分析。我们选择compileReleaseAidl任务进行分析。

![](E:\笔记\images\compile${variant}Aidl Task.png)

TaskManager#createAidlTask



```java
public AidlCompile createAidlTask(@NonNull VariantScope scope) {
        AidlCompile aidlCompileTask = taskFactory.create(new AidlCompile.ConfigAction(scope));
        scope.setAidlCompileTask(aidlCompileTask);
        scope.getSourceGenTask().dependsOn(aidlCompileTask);
        aidlCompileTask.dependsOn(scope.getPreBuildTask());

        return aidlCompileTask;
    }
```

代码逻辑很简单，通过taskFactory创建AidlCompile类型的Task。然后，设置这个task的执行顺序。它是在preBuildTask之后执行，在sourceGenTask之前执行。

最后，我们定位到AidlCompile类中，分析这个Task如何执行的。

```java
@CacheableTask
public class AidlCompile extends IncrementalTask {
    // 省略代码
}
```

由上述代码，AidlCompile继承了IncrementalTask类。AidlCompile类型task是支持增量更新的。我们查看代码时，并没有发现@TaskAction注解的方法，因此，我们去父类IncrementTask中查找。**关于增量构建参考这里**。

IncrementTask#taskAction

```java
    @TaskAction
    void taskAction(IncrementalTaskInputs inputs) throws Exception {
        if (!isIncremental() || !inputs.isIncremental()) {
            getProject().getLogger().info("Unable do incremental execution: full task run");
            doFullTaskAction();
            return;
        }

        doIncrementalTaskAction(getChangedInputs(inputs));
    }
```

首先判断是否需要增量更新，如果不需要，则执行IncrementTask#doFullTaskAction方法。这个方法时抽象方法。我们定位到AidlCompile#doFullTaskAction的方法。



AidlCompile#doFullTaskAction

```java
 @Override
    protected void doFullTaskAction() throws IOException {
        // this is full run, clean the previous output
        File destinationDir = getSourceOutputDir();
        File parcelableDir = getPackagedDir();
        FileUtils.cleanOutputDir(destinationDir);
        if (parcelableDir != null) {
            FileUtils.cleanOutputDir(parcelableDir);
        }

        DepFileProcessor processor = new DepFileProcessor();

        try {
            compileAllFiles(processor);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        List<DependencyData> dataList = processor.getDependencyDataList();

        DependencyDataStore store = new DependencyDataStore();
        store.addData(dataList);

        try {
            store.saveTo(new File(getIncrementalFolder(), DEPENDENCY_STORE));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

```

doFullTaskAction方法主要是调用compileAllFiles方法执行具体的编译工作。

AidlCompile#compileAllFiles

```
 /**
     * Action methods to compile all the files.
     *
     * <p>The method receives a {@link DependencyFileProcessor} to be used by the {@link
     * com.android.builder.internal.compiler.SourceSearcher.SourceFileProcessor} during the
     * compilation.
     *
     * @param dependencyFileProcessor a DependencyFileProcessor
     */
    private void compileAllFiles(DependencyFileProcessor dependencyFileProcessor)
            throws InterruptedException, ProcessException, IOException {
        getBuilder().compileAllAidlFiles(
                sourceDirs.get(),
                getSourceOutputDir(),
                getPackagedDir(),
                getPackageWhitelist(),
                getImportDirs().getFiles(),
                dependencyFileProcessor,
                new LoggedProcessOutputHandler(getILogger()));
    }

```

AidlCompile#compileAllFiles方法其实只有一行，调用AndroidBuilder的compileAllAidlFiles方法。



```java
/**
     * Compiles all the aidl files found in the given source folders.
     *
     * @param sourceFolders all the source folders to find files to compile
     * @param sourceOutputDir the output dir in which to generate the source code
     * @param packagedOutputDir the output dir for the AIDL files that will be packaged in an aar
     * @param packageWhiteList a list of AIDL FQCNs that are not parcelable that should also be
     *     packaged in an aar
     * @param importFolders import folders
     * @param dependencyFileProcessor the dependencyFileProcessor to record the dependencies of the
     *     compilation.
     * @throws IOException failed
     * @throws InterruptedException failed
     */
    public void compileAllAidlFiles(
            @NonNull Collection<File> sourceFolders,
            @NonNull File sourceOutputDir,
            @Nullable File packagedOutputDir,
            @Nullable Collection<String> packageWhiteList,
            @NonNull Collection<File> importFolders,
            @Nullable DependencyFileProcessor dependencyFileProcessor,
            @NonNull ProcessOutputHandler processOutputHandler)
            throws IOException, InterruptedException, ProcessException {
        checkNotNull(sourceFolders, "sourceFolders cannot be null.");
        checkNotNull(sourceOutputDir, "sourceOutputDir cannot be null.");
        checkNotNull(importFolders, "importFolders cannot be null.");
        checkState(mTargetInfo != null,
                "Cannot call compileAllAidlFiles() before setTargetInfo() is called.");

        IAndroidTarget target = mTargetInfo.getTarget();
        BuildToolInfo buildToolInfo = mTargetInfo.getBuildTools();

        String aidl = buildToolInfo.getPath(BuildToolInfo.PathId.AIDL);
        if (aidl == null || !new File(aidl).isFile()) {
            throw new IllegalStateException("aidl is missing from '" + aidl + "'");
        }

        List<File> fullImportList = Lists.newArrayListWithCapacity(
                sourceFolders.size() + importFolders.size());
        fullImportList.addAll(sourceFolders);
        fullImportList.addAll(importFolders);

        AidlProcessor processor = new AidlProcessor(
                aidl,
                target.getPath(IAndroidTarget.ANDROID_AIDL),
                fullImportList,
                sourceOutputDir,
                packagedOutputDir,
                packageWhiteList,
                dependencyFileProcessor != null ?
                        dependencyFileProcessor : DependencyFileProcessor.NO_OP,
                mProcessExecutor,
                processOutputHandler);

        for (File dir : sourceFolders) {
            DirectoryWalker.builder()
                    .root(dir.toPath())
                    .extensions("aidl")
                    .action(processor)
                    .build()
                    .walk();
        }
    }
```

该方法主要功能是编译源文件中的aidl文件。

AndroidBuilder是主要的构建器类。 给出了处理构建的所有数据（例如DefaultProductFlavor，DefaultBuildType和依赖项），并在执行特定的构建步骤时使用它们。






核心类

配置相关

AndroidConfig

BaseExtension

AppExtension

LibraryExtension



变体

BaseVariant

BaseVariantImp

AndroidArtifactVariantImpl

InstallableVariantImpl



ApplicationVariantImp



Task相关类

AndroidVariantTask



```

```





