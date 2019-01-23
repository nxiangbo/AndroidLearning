# Service详解

## Service生命周期

![img](E:\AndroidLearning\Android四大组件\images\service_lifecycle.png)





## startService vs bindService

You usually use `bindService()` if your calling component(`Activity`) will need to communicate with the `Service` that you are starting, through the `ServiceConnection`. If you do not want to communicate with the `Service` you can use just `startService()`. You Can see below diffrence between service and bind service.

> 启动
>
> 当应用组件（如 Activity）通过调用 `startService()` 启动服务时，服务即处于“启动”状态。一旦启动，服务即可在后台无限期运行，即使启动服务的组件已被销毁也不受影响。 已启动的服务通常是执行单一操作，而且不会将结果返回给调用方。例如，它可能通过网络下载或上传文件。 操作完成后，服务会自行停止运行。
>
> 绑定
>
> 当应用组件通过调用 `bindService()` 绑定到服务时，服务即处于“绑定”状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。 仅当与另一个应用组件绑定时，绑定服务才会运行。 多个组件可以同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。

