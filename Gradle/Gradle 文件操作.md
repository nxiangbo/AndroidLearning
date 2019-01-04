# Gradle 文件操作

几乎每个构建都涉及到文件的操作。因此，为了方便文件操作，Gradle专门设计了操作文件的API。

这些API包含两部分：

- 指定要处理的文件和目录
- 指定要如何处理文件和目录

## 拷贝文件（Copy）

将特定文件拷贝到指定目录。

```groovy
task copyCache(type:Copy){
	from file("${buildDir}/cache/hello.txt")
	into file("${buildDir}/log")
}
```

Project.file()方法是用于创建当前项目下的一个文件或者目录。

我们也可以不使用file()方法，直接使用路径。

```groovy
task copyCache2(type:Copy){
	from "${buildDir}/cache/hello.txt"
	into "${buildDir}/log2"
}
```

当然，我们也可以同时拷贝多个文件到指定目录。

```groovy
task copyFiles(type:Copy){
	from "${buildDir}/cache/hello.txt", "${buildDir}/generated/1.txt"
	into "${buildDir}/log3"
}
```

现在考虑另外一种情况，我们需要拷贝指定目录下的pdf格式的文件。

```groovy
task copyPdfReportsForArchiving(type: Copy) {
    from "${buildDir}/reports"
    include "*.pdf"
    into "${buildDir}/toArchive"
}
```

![copy-with-flat-filter-example](E:\笔记\images\copy-with-flat-filter-example.png)



我们也可以拷贝reports目录以及它的子目录下的pdf文件。

```groovy
task copyAllPdfReportsForArchiving(type: Copy) {
    from "${buildDir}/reports"
    include "**/*.pdf"
    into "${buildDir}/toArchive"
}
```

![copy-with-deep-filter-example](E:\笔记\images\copy-with-deep-filter-example.png)



## 创建归档文件（zip，tar等）

Gradle 提供了Zip，Tar，Jar等任务，可以将文件打包成.zip，.tar或者.jar格式的文件。

例如，我们可以将\${buildDir}/toArchive目录下的文件，打包成\${buildDir}/dist目录的my-distribution.zip文件

```groovy
task packageDistribution(type: Zip) {
    archiveName = "my-distribution.zip"
    destinationDir = file("${buildDir}/dist")

    from "${buildDir}/toArchive"
}
```

## 打开档案文件

Project.zipTree和Project.tarTree方法可以从相应的归档文件生成FileTree。FileTree可以用在from方法中。如下所示。

```groovy
task unpackFiles(type: Copy) {
    from zipTree("${buildDir}/dist/my-distribution.zip")
    into "${buildDir}/resources"
}
```



## 创建目录

我们可以使用Project.mkdir方法创建目录。

```groovy
task ensureDirectory {
    doLast {
        mkdir "images"
    }
}
```



## 移动文件或目录

Gradle并没有提供移动文件或者目录的方法。不过我们可以使用ant的方法。

```groovy
task moveReports {
    doLast {
        ant.move file: "${buildDir}/reports",
                 todir: "${buildDir}/toArchive"
    }
}
```



## 删除文件和目录

在Gradle中，可以使用Delete任务或者Project.delete方法删除文件或者目录。

删除目录

```groovy
task myClean(type: Delete) {
    delete buildDir
}
```

如果想要控制删除哪些文件，我们可以使用FileCollection和FileTree。

删除源文件夹下的临时文件

```groovy
task cleanTempFiles(type: Delete) {
    delete fileTree("src").matching {
        include "**/*.tmp"
    }
}
```

D:\Code2\roundImage>aapt2 link -o output.apk -I C:\Users\niuxiangbo\AppData\Loca
l\Android\Sdk\platforms\android-28\android.jar D:\Code2\roundImage\drawable_prod
uct_default.png.flat --manifest D:\Code2\roundImage\app\src\main\AndroidManifest
.xml -v