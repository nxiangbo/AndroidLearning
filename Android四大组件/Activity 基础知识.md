# Activity基础知识

## Activity 生命周期

![808004-20160104111412137-1733773288.gif](images/Activity-lifecycle.png)



- 打开应用 onCreate()->onStart()->onResume
- 按BACK键 onPause()->onStop()->onDestory()
- 按HOME键 onPause()->onStop()
- 再次启动 onRestart()->onStart()->onResume()

## Fragment 生命周期

![808004-20160104111456621-1865584274](images/fragment-lifecycle.png)

![808004-20160104111511106-1584734283](images/Activity-fragment.png)

- 切换到该Fragment onAttach() onCreate() onCreateView() onActivityCreated() onStart() onResume()
- 屏幕灭掉 onPause() onSaveInstanceState() onStop()
- 屏幕解锁 onStart() onResume()
- 切换到其他Fragment onPause() onStop() onDestroyView()
- 切换回本身的Fragment onCreateView() onActivityCreated() onStart() onResume()
- 回到桌面 onPause() onSaveInstanceState() onStop()
- 回到应用 onStart() onResume()
- 退出应用 onPause() onStop() onDestroyView() onDestroy() onDetach()

 ## Activity启动模式

有四种启动模式：
1. **standard 标准模式（默认的模式）**：每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否存在。一个任务栈可以有多个实例，每个实例可以属于不同的任务栈。
     举例：栈内有ABCD ==> 启动D == > ABCDD

2. **singleTop 栈顶复用模式**：如果新Activity位于任务栈的栈顶，那么此Activity不会被重新创建。系统会调用其onNewIntent；如果新Activity不是位于任务栈栈顶，那么会重新创建该Activity。
     举例：栈内 ABCD ==> 启动D ==> ABCD onCreate()和onStart()不执行
     **应用场景：**适合接收通知启动的内容显示页面。

3. **singleTask 栈内复用模式**：这是一种单例模式，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例。
     举例：栈内 ADBC ==> 启动D ==> AD 
               A为standard B和C为singleTask 启动A 创建一个新的任务栈x,创建A实例==>在A中启动B,B为singleTask 创建任务栈y，入栈==>启动C ,入栈y==>再启动A，入栈y，此时，A(x),BCA(y). 如果启动B,出栈CA,此时 A(x),B(y)
     **应用场景：**适合作为程序入口点。例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走`onNewIntent`，并且会清空主界面上面的其他页面。

4. **singleInstance 单实例模式**：这是一种加强的singleTask，除了具有singleTask的特性外，此种模式只能单独的位于一个栈中。
     举例：如果A是singleInstance，启动A后，系统会为它创建一个新的任务栈。
     **应用场景：**适合需要与程序分离开的页面。如闹铃提醒等。



### 如何给Activity指定启动模式？？

* 在AndroidMenifest中的Activity标签中  android：launchMode=“singleTask”
* 用Intent设置标志位  intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)

两者之间的区别为
* 第二种的优先级高于第一种。
* 第一种不能为Activity设定FLAG_ACTIVITY_CLEAR_TOP标签 ；第二种无法为Activity指定singleInstance



常用Activity的Flags

- FLAG_ACTIVITY_NEW_TASK ==singleTop
- FLAG_ACTIVITY_CLEAR_TOP  ==singleTask
- FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS 有此标记的Activity不会出现在历史Activity列表中

## 任务栈

然而什么是Activity所需的任务栈呢？？
默认 情况下，所有Activity所需的任务栈的名字为应用的包名。
参数：TaskAffinity(任务相关性) ：标识了一个Activity所需要的任务栈的名字。 此属性主要和singleTask配对使用。

任务栈有两种：
* 前台任务栈
* 后台任务栈 处于暂停状态

### 查看Activity 任务栈

adb shell dumpsys activity  检测Activity的任务栈

