# Android资源管理框架学习

最近在学习Android资源的编译、打包、以及创建和使用的过程。主要是参考老罗的[Android资源管理框架（Asset Manager）简要介绍和学习计划](https://blog.csdn.net/luoshengyang/article/details/8738877)系列文章和谷歌官方文档，即[提供资源](https://developer.android.com/guide/topics/resources/providing-resources)和[支持多种屏幕](https://developer.android.com/guide/practices/screens_support)等文章。



Android应用主要是由代码和资源组成。资源主要是和UI相关的，Android中使用xml文件设计UI界面。这种做法很好的将代码逻辑和UI分开。

Android资源分为系统资源和应用资源两种。资源分为有资源Id的文件和没有资源Id的文件。asset文件夹下的文件是原生文件，没有资源id。res文件夹下的资源是有id的。

谷歌提供了资源编译和打包工具，即aapt（aapt目前已被弃用）和aapt2。

资源打包工具将资源文件打包成二进制文件以及resource.arsc文件。





