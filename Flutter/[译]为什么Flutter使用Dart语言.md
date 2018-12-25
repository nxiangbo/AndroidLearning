# [译]为什么Flutter使用Dart语言

许多语言学家认为，一个人的自然语言的表达方式会影响他们的思考方式([affects how they think](https://en.wikipedia.org/wiki/Linguistic_relativity))。计算机语言是否适用于这种理念呢？使用不同计算机语言的程序员经常对同一问题提出完全不同的解决方案。有一个比较极端的例子，计算机科学家取消了goto语句，以便于鼓励程序员编写更结构化的程序。

这与Flutter和Dart有什么关系呢？确实有点关系。Flutter团队早期评估了实际中计算机语言，选择Dart是因为它与他们正在构建的用户接口相匹配。

Dart是开发人员喜欢Flutter的一个重要原因。正如一条tweet所说：

下面列举了Dart的一些特征，这对Flutter是不可或缺的：

- Dart 使用AOT（Ahead Of Time）编译成快速，可预测的本地代码。它允许几乎所有的Flutter都用Dart编写。这不仅使得Flutter允许速度更快，而且所有东西（包括所有的部件）都可以定制。
- Dart也可以使用JIT（Just in time）编译，用于极快的开发周期和改变游戏规则的工作流程（包括Flutter流行的秒级状态热重载）。
- Dart可以更容易的创建60fps之内的平滑动画和过渡。Dart可以在没有锁的情况下分配对象和垃圾收集。并且，与JavaScript类似，Dart没有使用抢占式调度和共享内存（从而加锁）。因为Flutter 应用被编译成本地代码，所以他们不需要**在执行过程中**建立一个缓慢的桥梁（例如，JavaScript到本地代码）。Flutter应用启动速度也会更快。
- 由于Dart的可声明，可编程布局具有比较容易阅读和良好的可视化，因此，Flutter不需要单独的声明性的布局语言，如JSX或者XML，也不需要单独的可视化界面构建器。
- 开发者发现Dart非常容易学习，因为它有静态语言用户和动态语言用户所熟悉的特征。

并非所有的特征都是Dart独有的，然而，这些特征的组合使得在实现Flutter方面，Dart具有独一无二的优势。可以这么说，如果没有Dart，Flutter很难变得如此强大。

![flutter-loves-dart](images/flutter-loves-dart.png)

本文的剩余部分将会更深入的介绍Dart的特性（包括它的标准库），这些特性使Dart成为实现Flutter的最佳语言。

## 编译和执行

*如果你已经比较熟悉静态和动态语言，AOT和JIT编译，以及虚拟机等主题，那么，你可以跳过这一节*

从历史上看，计算机语言可以分为两类：[静态语言](https://en.wikipedia.org/wiki/Compiled_language)，Fortran和C，它们的变量类型是在编译时输入）和[动态态语言](https://en.wikipedia.org/wiki/Interpreted_language)（Smalltalk和JavaScript，它们的变量类型可以在运行时更改）。静态语言可以被编译成可被机器直接执行的本地机器码（或者汇编代码）。动态语言由解释器直接执行，而不会生成机器代码。

当然，事情最终变的更加复杂。虚拟机的概念变的流行。它实际上是模拟硬件机器的高级解释器。虚拟机可以更轻松的将代码移植到新的硬件平台上。在此情况下，虚拟机的输入语言往往是中间语言。例如，程序语言(例如，Java)被编译成中间语言（字节码），然后在虚拟机（Java虚拟机）上执行。

此外，目前还有即时编译器（JIT）。JIT编译器在程序执行期间运行，即时编译。在创建程序期间（在运行时之前）执行的原始编译器现在称为AOT编译器。

一般来说，只有静态语言适合使用AOT编译成本地机器码，因为机器语言需要知道数据类型，而在动态语言中，数据类型不能够提前确定的。因此，动态语言通常被解释或者JIT编译。

当在开发期间完成AOT编译时，它总是导致更慢的开发周期（对程序进行更改和能够执行程序以查看更改结果之间的时间）。但是AOT编译导致程序可以可预测的执行，而不会在运行时分析和编译。AOT编译的程序执行速度更快（因为它们已经被编译）。

反过来说，JIT提供了更快的开发周期，但是执行速度较慢。尤其是，JIT编译器的启动时间较慢，因为当程序运行时，JIT编译器必须在执行代码之前进行分析和编译。研究表明，如果APP的启动时间超过几秒钟，许多人会放弃这个APP。

到这里，背景知识就结束了。将AOT和JIT的优点结合起来，不是很棒吗？请继续阅读。

## Dart的编译和执行

在开发Dart之前，Dart团队成员已经在高级编译器和虚拟机上做过开创性的工作，包括动态语言（如JavaScript的V8引擎和Smalltalk的Strongtalk）和静态语言（如Java的Hotspot编译器）。它们利用这些经验使得Dart在编译和执行方面异常灵活。

Dart是适合使用AOT和JIT编译的极少数的语言之一(也许是唯一的“主流”语言)。支持这两种编译方式为Dart和（特别是）Flutter提供了显着的优势。

在开发期间使用JIT编译，编译速度是非常快的。然后，当APP准备发布时，使用AOT编译。因此，借助先进的工具和编译器，Dart可以提供两全其美的优势：极快的开发周期，快速的执行和启动时间。

Dart在编译和执行方面的灵活性并不止于此。例如，Dart可以编译成JavaScript，因此可以由浏览器执行。这使得移动端的代码和web端的代码可以复用。开发者有报告指出，移动端和web端的代码复用率可以达到70%。Dart也可以通过编译为本机代码，或者通过编译为JavaScript并与node.js一起使用来在服务器上使用。

最后，Dart还提供了一个独立的虚拟机。它使用Dart语言本身作为中间语言（本质上类似一个解释器）。

Dart可以有效地编译AOT或JIT，解释或转换成其他语言。 Dart编译和执行不仅非常灵活，而且速度特别快。

下一节将举例说明

## 有状态热加载

速度极快的热加载是Flutter最受欢迎的功能之一。在开发过程中，Flutter使用JIT编译器，可以在一秒钟内重新加载并继续执行代码。应用程序状态会尽可能在重新加载时保留，因此应用程序可以从停止的位置继续。

![Flutter-sub-second-show](images/Flutter-sub-second-show.gif)

除非亲身体验，否则很难想象快速的热加载在开发过程中是多么重要。开发人员指出，它改变了他们创建应用程序的方式，将其描述为将应用程序绘制成生命。

下面是移动开发者对Flutter热加载的评价。

> 我测试了一下热重载，因此，我改变了一个颜色，保存我的修改，然后，我爱上了它:heart:
>
> 这个功能让人着迷。我认为Visual Studio中的编辑和继续（Edit & Continue）功能很好用，但热加载简直令人震惊。 只是这一个功能，我认为移动开发人员的工作效率可以提高两倍。
>
> 这对我来说真的是一个改变游戏规则的人。 当我部署代码并且需要很长时间时，我会失去焦点，我会做其他事情，当我回到模拟器/设备时，我已经忘记了我想要测试的内容。 没有什么比用5分钟去控制2px更令人沮丧的了。 随着Flutter出现，这一切都不会存在。

Flutter的热加载让人更容易尝试新的idea或者替换方案，这大大提高了创造力。

到目前为止，我们谈论了Dart对开发者更加友好。下一节将介绍Dart是如何轻易的创建让用户满意的流畅APP。

## 避免卡顿

一个快速的APP很好，但是一个流畅的APP更好。一个快速动画是生涩的，仍是比较糟糕的。然而，避免卡顿是非常困难的，因为引起卡顿的原因有很多。Dart有很多功能可以避免很多事情卡顿。

当然，使用Flutter也可能写出卡顿的APP。Dart有助于提高可预测性，让开发人员更好地控制应用程序的流畅性，从而更轻松地提供最佳的用户体验。

结果是？

> 以60 fps为运行标准，使用Flutter创建的用户界面比使用其他跨平台开发框架创建的用户界面执行得更好。

并且不仅仅比跨平台的APP表现好，而且还和native APP表现一样好。

> UI 是非常流畅...我从没见过如此流畅的APP

### AOT编译和“桥”

我们已经讨论了Dart具有保持APP流畅的特点，那就是Dart可以被AOT编译成本地代码。预先编译的AOT代码比JIT编译的代码更可以预测，因为在运行时，不会停下来执行JIT分析和编译。

然而，AOT编译的代码具有更大的优势，即避免了“JavaScript 桥”。当动态类型语言（如JavaScript）需要与本地代码在平台上互操作时，他们不得不通过“桥”进行通信。这导致上下文切换时必须保存大量的状态变量（可能需要二次存储）。这些上下文不仅会降低程序运行速度，而且还会造成卡顿。

![](E:\AndroidLearning\Flutter\images\bridge.png)

注意：即便编译后的代码需要一个接口与平台代码进行交互，且这也可以称之为“桥”，它也比动态类型语言所需要的“桥”的运行速度快的多。此外，由于Dart允许将小部件等内容移动到应用程序中，因此减少了“桥”的需求。

### 抢占式调度，时间片和共享资源

大部分支持多个并发执行线程（包括Java，Kotlin，Objective-C和Swift）的计算机语言都使用[抢占式](https://en.wikipedia.org/wiki/Preemption_%28computing%29)调度线程。每个线程在执行时，会为线程分配一个时间片，如果这个线程在分配的时间片内没有执行完，则这个线程会被其他线程抢占。但是，如果在更新线程（如内存）之间共享的资源时发生抢占，则会导致[竞争条件](https://en.wikipedia.org/wiki/Race_condition)。

因为竞争条件会引起非常严重的bug，包括app crash，数据丢失，所以这是非常糟糕的。同时，由于他们依赖于[独立线程之间的相对时间](https://en.wikipedia.org/wiki/Race_condition#Software)，因此这些bug很难发现和修改。当您在调试器中运行应用程序时，竞争条件停止显示自己是很常见的。

处理竞争条件的典型方法是使用锁机制组织其他线程访问共享资源，但是锁本身可能会引起卡顿或者[更严重的问题](https://en.wikipedia.org/wiki/Dining_philosophers_problem)（包括[死锁](https://en.wikipedia.org/wiki/Deadlock)和[饥饿](https://en.wikipedia.org/wiki/Starvation_%28computer_science%29)）。

Dart对这个问题采取了不同的方法。在Dart中，线程是隔离的，不共享资源。因此，也避免了使用锁机制。线程通过通道进行通信。类似于[Erlang](https://www.erlang.org/)的*actors*和JavaScript的*web workers*

与JavaScript一样，Dart是单线程的，这意味着它根本不允许抢占。相反，线程明确地产生（使用[async / await](https://www.dartlang.org/tutorials/language/futures)，[Futures](https://www.dartlang.org/tutorials/language/futures)或[Streams](https://www.dartlang.org/tutorials/language/streams)）。这使开发人员可以更好地控制执行。单线程帮助开发人员确保关键功能（包括动画和转换）执行完成，而不会被抢占。这不仅对于UI而且对于其他客户端 - 服务器代码而言是一个很大的优势。

当然，如果开发人员忘记控制权，这可能会延迟执行其他代码。然而，我们发现这个问题比使用锁机制（因为竞争条件很难定位）更容易定位并修复



### 分配和垃圾收集

另一个引起卡顿的是垃圾收集。在使用锁机制时，访问共享资源确实是一个特殊的例子。但是锁可能会在收集可用内存时阻止整个应用程序运行。但是，Dart几乎可以在没有锁的情况下执行垃圾收集。

Dart使用[分代垃圾收集机制](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Generational_GC_.28ephemeral_GC.29)，这对于分配许多短期对象来说特别快（非常适合像Flutter这样为每一帧重建不可变视图树的响应式用户界面）。Dart可以使用单个指针凹凸分配对象（无需锁定），这可以使滑动和动画更流畅，不会卡顿。



## 统一布局

Dart的另一个有点是，Flutter不会在您的程序和其他模板或布局语言（如JSX或XML）之间拆分布局，也不需要单独的可视化布局工具。这里是一个简单的Flutter View，使用Dart语言写的。

```dart
new Center(child:
  new Column(children: [
    new Text('Hello, World!'),
    new Icon(Icons.star, color: Colors.green),
  ])
)
```

![A view in Dart and what it produces](E:\AndroidLearning\Flutter\images\star.png)

请注意，可视化此代码生成的输出是多么容易（即使您没有使用Dart的经验）。

注意，Flutter现在使用Dart 2，布局会变得更加简单清楚，因为new关键字可以忽略。所以静态布局看起来更像是用声明性布局语言编写的，如下所示。

```dart
Center(child:
  Column(children: [
    Text('Hello, World!'),
    Icon(Icons.star, color: Colors.green),
  ])
)
```

然而，我知道你可能在思考--为何将缺少特定的布局语言称为优点？但它实际上是一个改变游戏规则的人。这里是一名开发者写的文章，标题是“[Why native app developers should take a serious look at Flutter](https://hackernoon.com/why-native-app-developers-should-take-a-serious-look-at-flutter-e97361a1c073)”。

>在Flutter中，布局只使用Dart代码。没有XML和其他模板语言。也没有视觉设计工具/storyboarding 工具。
>
>我的预感是，听到这个，你们中的许多人甚至可能会有点畏缩。首先，这也是我的反应。使用可视化工具进行布局并不会更容易。 不在代码中编写各种约束逻辑会使事情变得过于复杂吗？
>
>结果证明我的答案是否定的。还有男孩！这是多么令人大开眼界。

答案的第一部分是上面提到的热重载。

> 我不能强调这是如何比Android的Instant Run或任何类似的解决方案提前几年。它只是工作，即使是在大型的应用程序上，也非常非常快。这就是Dart对你的影响力。
>
> 实际上，可视化编辑器是多余的。我也使用过XCode提供的不错的auto layout。

Dart创建了简洁容易理解的布局，同时，非常快的热加载可以让你更快看到执行结果。这包括布局的非静态部分。

> 结果，我在Flutter（Dart）中编写布局比使用Android / XCode更有效率。一旦掌握了它（对我而言是几个星期），由于很少进行上下文的切换，可以节省很多开销。开发者不需要切换到设计界面，然后选择鼠标并开始点击。然后思考是否必须以编程方式完成某些事情，如何实现这一点等。一切都是可编程的。并且Flutter的API设计的非常好。它是非常直观的，并且比auto layout 和XML布局更强大。

例如，这里有为一个list添加分割线，可以写成如下方式：

```dart
return new ListView.builder(itemBuilder: (context, i) {
  if (i.isOdd) return new Divider(); 
  // rest of function
});
```

在Flutter中，不管是静态布局还是动态布局，他们都在一个地方。新Dart工具（包括Flutter Inspector 和大纲视图（ the outline view））使复杂，漂亮的布局更加容易。



## Dart是专有语言吗

不，Dart（和Flutter一样）是完全开源的语言，有干净的许可证，也遵循[ECMA标准](https://www.ecma-international.org/publications/standards/Ecma-408.htm)。Dart在谷歌内外非常受欢迎。在Google内部，它是增长最快的语言之一，被Adwords，Flutter，Fuchsia和其他人使用。在外部，Dart相关项目有100多个人贡献代码。

Dart开放性的一个更好的指标是Google以外的社区的增长。例如，我们看到源源不断的来自第三方的关于Dart（包括Flutter和AngularDart）的文章和视，我在本文中引用了其中一些。

除了Dart本身的外部提交者之外，公共Dart包存储库中有超过3000个package，包括用于Firebase，Redux，RxDart，国际化，加密，数据库，路由，集合等的库。

## Dart 程序员很难找到吗

