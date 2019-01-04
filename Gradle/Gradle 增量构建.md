# Gradle 增量构建

## 任务的输入、输出以及依赖

内置任务（如JavaCompile）声明了输入集合（Java源文件）和输出集合（class文件）。Gradle使用此信息来确定任务是否是最新的并且是否需要执行任何工作。如果没有任何输入或输出发生更改，Gradle可以跳过该任务。总之，我们将这种行为成为Gradle增量构建支持。

为了利用增量构建支持，您需要向Gradle提供有关任务的输入和输出的信息。他可以将任务配置为只有输出。在执行这个任务之前，Gradle会检查输出，如果输出结果没有改变，则跳过这个任务。在实际的构建过程中，任务通常也有输入 - 包括源文件，资源和属性。Gradle在执行任务之前，会检查任务的输入和输出有没有改变。

通常一个任务的输出是作为另一个任务的输入。获得任务之间的执行顺序是至关重要的。Gradle不依赖于在构建脚本中定义任务的顺序。您可以通过声明一个任务之间的依赖关系，例如consumer.dependsOn producer，明确告诉Gradle两个任务之间的顺序。

## 声明任务依赖

在这一节，我们看一个简单的例子。在这个项目中，我们需要创建一个zip文件。generator任务创建文件的方式并不重要 - 它生成包含递增数字的文件。

build.gradle

```groovy
apply plugin: 'base'

task generator() {
    doLast {
        def generatedFileDir = file("$buildDir/generated")
        generatedFileDir.mkdirs()
        for (int i=0; i<10; i++) {
            new File(generatedFileDir, "${i}.txt").text = i
        }
    }
}

task zip(type: Zip) {
    dependsOn generator
    from "$buildDir/generated"
}
```

这个脚本是可以工作的，但是它还有一些问题。zip任务中重复`generator`任务的输出目录，并使用dependsOn显式设置zip任务的依赖关系。Gradle似乎每次都执行`generator`任务，但不执行zip任务。现在是时候指出Gradle的最新检查与其他工具不同，例如Make。即Gradle比较输入和输出的校验和，而不仅仅是文件的时间戳。虽然每次运行`generator`任务并覆盖其所有输出文件，但内容不会更改，并且zip任务不需要再次运行。zip任务输入的校验和没有改变。跳过最新任务可让Gradle避免不必要的工作并加快开发反馈循环。



## 声明任务的输入和输出

现在，让我们看一下为什么`generator`任务每次都会执行。我们可以通过添加`--info`参数查看原因。

```groovy
It is not up-to-date because:Task has not declared any outputs.
```

我们从打印的信息可以看到，Gradle不知道任务产生任何输出。默认情况下，如果一个任务没有任何输出，它会被认为是out-of-date。任务的输出可以使用TaskOutputs声明。任务的输出可以是文件也可以是输出。`outputs`使用如下所示：

build.gradle

```groovy
task generator() {
    def generatedFileDir = file("$buildDir/generated")
    outputs.dir generatedFileDir
    doLast {
        generatedFileDir.mkdirs()
        for (int i=0; i<10; i++) {
            new File(generatedFileDir, "${i}.txt").text = i
        }
    }
}
```

如果我们连续运行两次修改后的任务，我们可以看到，在第二次运行后的信息是：`generator`任务是up-to-date（最新的）。我们在运行时添加`--info`参数，可以看到如下信息。

```
Skipping task ':generator' as it is up-to-date (took 0.001 secs).

```

当任务的输入和输出都没有改变时，Gradle会跳过这个任务。在大型项目中，可以节省很多的项目构建时间。