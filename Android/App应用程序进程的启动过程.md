# App应用程序进程的启动过程

Android的应用程序进程都是通过Zygote进程fork的子进程，包括system_server和app的进程。

在启动app时，若当前app的进程还没有启动，ActivityManagerService会请求Zygote进程fork该app的进程。app进程创建后，会获得一个虚拟机的实例（就可以使用java进行开发组件），同时，还可以获得一个binder线程池和一个消息循环。



## App进程的创建过程

