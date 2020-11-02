# handler机制

## 处理消息

Looper.prepare 创建Looper和消息队列

1. 创建Looper对象，创建Looper时，创建消息队列
2. 将Looper对象set到ThreadLocal

Looper.loop 启动Looper，处理消息
1. myLooper方法（从ThreadLocal中get Looper对象）中获取当前线程的Looper
2. 拿到该Looper的MessageQueue队列
3. 启动消息循环（死循环），不断的从消息队列中取出消息（queue.next）
	a. queue.next -> nativePollOnce -> pollOnce -> pollInner -> epoll_wait
4. 将msg 分发给相应的Handler进行处理（msg.target.dispatchMessage(msg)）
5. 在Handler#handleMessage中处理消息任务

## 发送消息

Handler#sendMessage 主要是将msg添加到消息队列相应位置

Handler#sendMessage -> Handler#enqueueMessage -> MessageQueue#enqueueMessage

循环队列中，如果没有消息，线程会处于休眠状态，不会占用CPU。等有消息时，会唤醒线程。

Handler底层使用epoll机制，当队列中无消息时，Looper#pollOnce会调用epoll_wait方法，等待新的事件发生

## epoll机制

epoll机制：linux内核I/O事件通知机制

int epoll_create(int size);
创建epoll对象，并返回一个epoll文件描述符

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
向 epfd 对应的内核epoll 实例添加、修改或删除对 fd 上事件 event 的监听。op 可以为 EPOLL_CTL_ADD, EPOLL_CTL_MOD, EPOLL_CTL_DEL 分别对应的是添加新的事件，修改文件描述符上监听的事件类型，从实例上删除一个事件。

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
当 timeout 为 0 时，epoll_wait 永远会立即返回。而 timeout 为 -1 时，epoll_wait 会一直阻塞直到任一已注册的事件变为就绪。当 timeout 为一正整数时，epoll 会阻塞直到计时 timeout 毫秒终了或已注册的事件变为就绪。