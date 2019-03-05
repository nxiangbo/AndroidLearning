

## 何谓Binder？

直观上说，Binder是一个类，实现了IBinder接口

从IPC角度来说，BInder是android中的一种跨进程通信方式； 

从Android的Framework层来说，Binder是ServiceManager连接各种Manager和相应ManagerService的桥梁； 

从Android应用层来说，Binder是**客户端**和**服务器端**通信的媒介，当bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端可以获取服务端提供的**服务**或者数据。服务包括普通服务和基于**AIDL服务**。 

Binder类中有两个重要的方法，linkToDeath和unlinkToDeath。**Binder**运行在服务器端进程，如果服务器端进程因为某些原因**异常终止**，此时，我们连接到服务器端的Binder连接断裂（Binder死亡），会导致远程调用失败。如果我们不知道Binder连接断裂，那么**客户端功能**会受到影响。 

为了解决此问题，Binder提供了两个配对方法，**linkToDeath**和**unlinkToDeath**。通过linkToDeath，我们可以给Binder设置一个**死亡代理**，当Binder死亡时，我们就会**收到通知**。此时我们就可以重新发起连接请求从而恢复连接。 

- DESCRIPTOR  Binder的唯一标识，一般用当前BInder的类名表示； 
- onTransact()此方法运行在Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法处理。 



**Android中IPC方式**



**使用Bundle**

四大组件中的三大组价（Activity，Service，Receiver）都是支持Intent中传递Bundle数据。由于Bundle实现了Parcelable接口，所以可以很方便的在不同进程间传输。缺点是传输的类型受限。 

举例：假如要在A进程中进行计算，计算完成后，它要启动B进程的一个组件并把结果传给B进程，但是这个计算结果**不支持**放入**Bundle**中，因此无法通过Intent传输。这个时候，**该如何处理？？** 

​          可以通过Intent启动进程B的**Service**组件，让Service进行后台计算，计算完毕后再启动目标组件。由于Service也在B进程中，可以用Intent传输。 



**使用文件共享**

两个进程通过读/写同一个文件来交换数据。通过共享文件的方式，对文件格式是没有要求的，只要读、写双方约定好格式即可。 



**使用Messenger（信使）**

通过它可以在不同进程中传递Message对象，在Message中放入我们需要传递的数据，就可以实现数据的进程间传递。它是基于AIDL的。 

Messenger方式**一次**只能处理**一个请求**，因此在服务器端不用考虑线程同步的问题。 



实现Messenger的步骤 

- 服务器端 

​     首先我们需要在服务端创建一个**Service**对象来处理客户端的请求，同时创建一个**Handler**，并通过它来创建一个**Messenger**对象，然后在Service的**onBind**中返回这个Messenger对象底层的Binder即可。 

- 客户端进程 

​     首先要**绑定**服务端的**Service**，绑定成功后，用服务端返回的**IBinder**对象创建一个**Messenger**，通过Messenger就可以向服务端发送消息了，发消息**类型**为**Message**对象。

​     如果需要服务端能够**回应**客户端的话，客户端还需要像服务端一样，创建一个**Handler**，并通过它创建一个Messenger对象，然后把这个对象通过**Message**的参数**replyTo**参数传递给服务器端，服务器端就可以根据这个参数进行回复。

------



**使用AIDL**



如果有大量的并发请求时，使用Messenger明显是不合适的；或者如果要跨进程调用服务端的方法时，也能用Messenger。 

AIDL进程间通信的流程为： 

- 服务端 

​     首先创建一个Service用来监听客户端的请求，然后创建一个AIDL文件，将暴露给客户端的接口在这个AIDL的文件中声明，，最后在Service中实现这个AIDL接口。 

- 客户端 

​     首先要**绑定**服务端的**Service**，绑定成功后,将服务端返回的Binder对象转成AIDL接口所属的类型，接着就可以调用AIDL的方法了。 

- AIDL接口的创建 



​     AIDL支持的数据类型 

1. 基本数据类型 
2. String和CharSequence 
3. List：只支持ArrayList 
4. Map：只支持HashMap 
5. Parcelable：所有实现此接口的对象 
6. AIDL 



Binder意外死亡时如何处理？？ 

有两种方式： 

1. 给Binder设置DeathRecipient监听，当Binder死亡时，我们会收到binderDied方法的回调。此时，就用到前面讲述的linkToDeath和unlinkToDeath方法。此方法运行在Binder线程池中，不能访问UI. 
2. 在onServiceDisconnected 中重新连接远程服务。此方法可以访问UI 



如何进行权限验证？？ 

默认情况下，我们的远程服务任何人都可以连接，为了安全，需要进行权限验证。 

1. 在onBind()方法中进行验证，如果不能通过，直接返回null 
2. 在服务端的onTransact方法中进行验证，如果验证失败，直接返回false。 



**AIDL工作流程**

首先创建一个**Service**和一个**AIDL**接口，接着创建**一个类****继承**自AIDL 接口中的**Stub类**，并**实现**Stub中的抽象方法，在Service的**onBind**方法中**返回**这个类的对象，然后客户端可以**绑定**服务端的**Service**，建立连接后就可以**访问**远程服务端的方法了。



**Binder连接池**

Binder连接池的主要作用就是将每个业务模块的Binder请求统一转发到远程Service中去执行，从而避免了Service的重复创建过程。