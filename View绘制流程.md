# View绘制流程

在Activity#onResume后，将DecorView添加到Window，遍历DecorView下的View树，依次measure，layout，draw View。

将DecorView添加到Window，具体实现在WindowManagerGlobal#addView中。addView 的逻辑：创建ViewRootImpl实例，然后调用ViewRootImpl#setView方法。

```mermaid
sequenceDiagram
	ActivityThread ->> ActivityThread: handleResumeActivity
	
	ActivityThread ->> ViewManager: addView
	ViewManager ->> WindowManagerImpl: addView
	WindowManagerImpl ->> WindowManagerGlobal:addView
	WindowManagerGlobal ->> ViewRootImpl:new ViewRootImp 调 setView
	ViewRootImpl ->> ViewRootImpl:requestLayout
	ViewRootImpl ->> ViewRootImpl:scheduleTraversals
	
	
	
```

```mermaid
sequenceDiagram
	ViewRootImpl ->> Choreographer:postCallBack(TraversalRunnable run)
	TraversalRunnable ->> ViewRootImpl:doTraversal
	ViewRootImpl ->> ViewRootImpl:performTraversals
	%% measure view大小
	ViewRootImpl ->> ViewRootImpl:performMeasure
	ViewRootImpl ->> View:measure
	View ->> View:onMeasure
	
	%% layout
	ViewRootImpl ->> ViewRootImpl:performLayout
	ViewRootImpl ->> View:layout
	View ->> View:onLayout
	
	%% draw
	ViewRootImpl ->> ViewRootImpl:performDraw
	ViewRootImpl ->> View:draw
	View ->> View:onDraw
	
```

