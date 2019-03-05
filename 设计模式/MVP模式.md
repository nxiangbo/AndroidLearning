# MVP模式
MVP模式的核心思想：
MVP把Activity中的UI逻辑抽象成View接口，把业务逻辑抽象成Presenter接口，Model类还是原来的Model。


![7B2AD8B8-492C-40EC-8956-E13D6E59A85B](/Users/nxiangbo/Documents/AndroidLearning/设计模式/images/mvp.png)

## 优点
- 分离了视图逻辑和业务逻辑，降低了耦合

- Activity只处理生命周期的任务，代码变得更加简洁

- 视图逻辑和业务逻辑分别抽象到了View和Presenter的接口中去，提高代码的可阅读性

- Presenter被抽象成接口，可以有多种具体的实现，所以方便进行单元测试

- 把业务逻辑抽到Presenter中去，避免后台线程引用着Activity导致Activity的资源无法被系统回收从而引起内存泄露和OOM

