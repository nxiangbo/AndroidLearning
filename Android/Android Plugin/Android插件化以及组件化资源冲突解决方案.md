# Android插件化以及组件化资源冲突解决方案

在插件化中，如果不同插件的资源id一样，那么就会引起资源冲突。为了解决资源id冲突，通常的解决方案是进行资源分区。因为资源ID是int类型的值，由packageId，type，和偏移量组成，而每个App的默认packageId都是0x7f。而资源分区就是修改资源的packageId来解决资源冲突。

那么，如何修改资源的packageId呢？

通常有三种做法，第一种是修改aapt源码（如，atlas），第二种是修改aapt打包生成的二进制资源文件，即.arsc文件（如，small，virtualApk）。第三种是宿主和插件之前，插件和插件之间使用不同的AssetManager加载资源（如，DroidPlugin）。

此外，随着资源打包工具aapt2的更新，在buildToolsVersion 28.0版本及以上的aapt2中，引入了`--allow-reserved-package-id`和`--package-id`参数。利用这两个参数，可以指定App的资源packageId。



在组件化中，也会存在资源冲突的问题。当主module合并多个业务module时，不同module之间的命名一样的资源只会保留一份。至于保留哪一份，官方给定了相应的优先级。然而，不管保留哪一份，都不是我们想要看到的结果。我们需要的是所有的资源都可以被正确加载。

注意，组件化中的资源冲突，不再是资源id的冲突。而是因为不同module中资源命名一样，合并资源的过程中只会保留一份。这就必然造成部分资源丢失的问题。

通常的解决方案是在android DSL中添加resourcePrefix字段。

然而，除此之外，是否还有其他比较好的解决方案？

使用aapt2进行资源分区？

答案是否定的。因为aapt2资源分区是在aapt2的link阶段修改资源的packageId。而组件化资源冲突是由于module中的资源merge得到主module时，资源名称不一样造成的。



---

在网络上，有检测资源冲突插件（不完善），有讲解组件化方案，顺带说一下组件化中的资源冲突。

我的想法

能否在编译module过程中，自动为资源加上前缀。引用资源的地方也要修改

更改合并资源规则（在网上没有找到合并资源规则源码相关）





