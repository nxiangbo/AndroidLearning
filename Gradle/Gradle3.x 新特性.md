# Gradle3.x 新特性

Gradle3.x要求如下

- Gradle4.4或者更高版本
- Build Tools 27.0.3或者更高

## 新的Dex编译工具，D8

默认情况下，Android Studio 使用D8作为Dex编译工具。DEX编译是将.class字节码转换为ART的.dex字节码的过程。与之前的dex工具相比，D8的编译速度更快，输出的.dex文件更小。如果要关闭D8的话，可以在gradle.properties文件中设置：

```groovy
android.enableD8=false
```

## 优化

- 通过细粒度的任务图更好地实现多模块项目的并行性 
- 当依赖项被更改时，Gradle并不会重新编译没有访问该依赖项的API的模块，从而使得构建速度更快。可以使用Gradle的新依赖项配置来限制哪些依赖项将其API泄漏到其他模块：implementation，api，compileOnly和runtimeOnly。
- 由于per-dexing而增加的构建速度更快。现在，每个类都被编译为单独的DEX文件，并且只有被修改的类才会被重新定义。
- 通过优化某些任务以使用chached输出来提高构建速度。要从此优化中受益，您需要首先启用Gradle构建缓存
- 使用AAPT2改进了增量资源处理，AAPT2现在默认启用。也可以在gradle.properties文件中设置`android.enableAapt2=false`,关掉该功能。

## 新功能

- [Variant-aware dependency management](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html#variant_aware). 
- 支持Java 8，弃用Jack
- 

