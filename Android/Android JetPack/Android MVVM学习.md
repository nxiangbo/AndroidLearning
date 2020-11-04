# Android MVVM学习

[如何构建Android MVVM应用框架](https://tech.meituan.com/android_mvvm.html)

[MVVM维基百科](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)

[todo-mvvm-live](https://github.com/googlesamples/android-architecture/tree/dev-todo-mvvm-live)

[浅谈MVC、MVP和MVVM架构模式](https://draveness.me/mvx)



MVVM是由微软在2005年发明的，它可以看作是PM（Presentation Model ）的变体。MVVM是由View、ViewModel和Model组成。

**View:** 对应于Activity和XML，负责View的绘制以及与用户交互。
**Model:** 实体模型。
**ViewModel:** 负责完成View与Model间的交互，负责业务逻辑。

![Model-View-ViewModel](https://raw.githubusercontent.com/Draveness/analyze/master/contents/architecture/images/mvx/Model-View-ViewModel.jpg)



### View

View层做的就是和UI相关的工作，我们只在XML、Activity和Fragment写View层的代码，View层不做和业务相关的事，也就是我们在Activity不写业务逻辑和业务数据相关的代码，更新UI通过数据绑定实现，尽量在ViewModel里面做（更新绑定的数据源即可），Activity要做的事就是初始化一些控件（如控件的颜色，添加RecyclerView的分割线），View层可以提供更新UI的接口（但是我们更倾向所有的UI元素都是通过数据来驱动更改UI），View层可以处理事件（但是我们更希望UI事件通过Command来绑定）。**简单地说：View层不做任何业务逻辑、不涉及操作数据、不处理数据，UI和数据严格的分开。**

### ViewModel

ViewModel层做的事情刚好和View层相反，ViewModel只做和业务逻辑和业务数据相关的事，不做任何和UI相关的事情，ViewModel 层不会持有任何控件的引用，更不会在ViewModel中通过UI控件的引用去做更新UI的事情。ViewModel就是专注于业务的逻辑处理，做的事情也都只是对数据的操作（这些数据绑定在相应的控件上会自动去更改UI）。同时DataBinding框架已经支持双向绑定，让我们可以通过双向绑定获取View层反馈给ViewModel层的数据，并对这些数据上进行操作。关于对UI控件事件的处理，我们也希望能把这些事件处理绑定到控件上，并把这些事件的处理统一化，为此我们通过BindingAdapter对一些常用的事件做了封装，把一个个事件封装成一个个Command，对于每个事件我们用一个ReplyCommand去处理就行了，ReplyCommand会把你可能需要的数据带给你，这使得我们在ViewModel层处理事件的时候只需要关心处理数据就行了，具体见**MVVM Light Toolkit 使用指南**的Command部分。再强调一遍：ViewModel 不做和UI相关的事。

### Model

Model层最大的特点是被赋予了数据获取的职责，与我们平常Model层只定义实体对象的行为截然不同。实例中，数据的获取、存储、数据状态变化都是Model层的任务。Model包括实体模型（Bean）、Retrofit的Service ，获取网络数据接口，本地存储（增删改查）接口，数据变化监听等。Model提供数据获取接口供ViewModel调用，经数据转换和操作并最终映射绑定到View层某个UI元素的属性上。