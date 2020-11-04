# Android aidl 详解

## 什么是aidl

AIDL是一个缩写，全称是Android Interface Definition Language。主要用于进程间通信。

## aidl语法

AIDL 使用简单语法，使您能通过可带参数和返回值的一个或多个方法来声明接口。 参数和返回值可以是任意类型，甚至可以是其他 AIDL 生成的接口。

必须使用 Java 编程语言构建 `.aidl` 文件。每个 `.aidl` 文件都必须定义单个接口，并且只需包含接口声明和方法签名。

AIDL支持的数据类型 

1. 基本数据类型 
2. String和CharSequence 
3. List：只支持ArrayList 
4. Map：只支持HashMap 
5. Parcelable：所有实现此接口的对象 
6. AIDL 



## 自定义aidl接口

```java
// IBook.aidl
package com.example.nxiangbo.binderexample.aidl;

// Declare any non-default types here with import statements

parcelable Book;
```



```java
// IBookManager.aidl
package com.example.nxiangbo.binderexample.aidl;

// Declare any non-default types here with import statements
import com.example.nxiangbo.binderexample.aidl.Book;
import com.example.nxiangbo.binderexample.aidl.IOnNewArrivedListener;
import java.util.List;

interface IBookManager {
    List<Book> getBookList();
    void addBook(in Book book);
    void registerListener(IOnNewArrivedListener listener);
    void unregisterListener(IOnNewArrivedListener listener);
}
```



```java
// IOnNewArrivedListener.aidl
package com.example.nxiangbo.binderexample.aidl;

// Declare any non-default types here with import statements
import com.example.nxiangbo.binderexample.aidl.Book;
interface IOnNewArrivedListener {
    void onNewBookArrived(in Book newBook);
}
```



## 如何使用

### 服务端

```java
package com.example.nxiangbo.binderexample;

public class BookService extends Service {
    private AtomicBoolean mIsServiceDestroy = new AtomicBoolean(false);
    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();
    private RemoteCallbackList<IOnNewArrivedListener> mListenerList = new RemoteCallbackList<>();
    private Binder mBinder = new IBookManager.Stub() {
        @Override
        public List<Book> getBookList() throws RemoteException {
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            mBookList.add(book);
        }

        @Override
        public void registerListener(com.example.nxiangbo.binderexample.aidl.IOnNewArrivedListener listener) throws RemoteException {
            mListenerList.register(listener);
        }

        @Override
        public void unregisterListener(com.example.nxiangbo.binderexample.aidl.IOnNewArrivedListener listener) throws RemoteException {
            mListenerList.unregister(listener);
        }
    };

    public BookService() {

    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        mBookList.add(new Book(1, "算法"));
        mBookList.add(new Book(2, "数据结构"));

        new Thread(new ServiceWorker()).start();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        mIsServiceDestroy.set(true);
    }

    private class ServiceWorker implements Runnable {

        @Override
        public void run() {
            while (!mIsServiceDestroy.get()) {
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                int bookId = mBookList.size() + 1;
                Book newBook = new Book(bookId, "bookId#" + bookId);
                try {
                    onNewArrived(newBook);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void onNewArrived(Book newBook) throws RemoteException {
        mBookList.add(newBook);
        int size = mListenerList.beginBroadcast();
        for (int i = 0; i < size; i++) {
            IOnNewArrivedListener listener = mListenerList.getBroadcastItem(i);
            if (listener != null) {
                listener.onNewBookArrived(newBook);
            }

        }
        mListenerList.finishBroadcast();
    }
}
```



### 客户端

```java
package com.example.nxiangbo.binderexample;

import android.annotation.SuppressLint;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Handler;
import android.os.IBinder;
import android.os.Message;
import android.os.RemoteException;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import com.example.nxiangbo.binderexample.aidl.Book;
import com.example.nxiangbo.binderexample.aidl.IBookManager;
import com.example.nxiangbo.binderexample.aidl.IOnNewArrivedListener;

import java.util.List;

public class MainActivity extends AppCompatActivity {

    private static final int MESSAGE_NEW_BOOK_ARRIVED = 1;
    private IBookManager mBookManager;

    @SuppressLint("HandlerLeak")
    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what) {
                case MESSAGE_NEW_BOOK_ARRIVED:
                    Log.d("nxb", "receive new book"+ msg.obj.toString());
                    break;
            }
        }
    };

    private IOnNewArrivedListener mOnNewArrivedListener = new IOnNewArrivedListener.Stub() {
        @Override
        public void onNewBookArrived(Book newBook) throws RemoteException {
            mHandler.obtainMessage(MESSAGE_NEW_BOOK_ARRIVED, newBook).sendToTarget();
        }
    };

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            try {
                mBookManager = bookManager;
                List<Book> books = bookManager.getBookList();
                Log.d("nxb", books.toString());
                bookManager.registerListener(mOnNewArrivedListener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Intent intent = new Intent(this, BookService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mBookManager!= null && mBookManager.asBinder().isBinderAlive()) {
            try {
                mBookManager.unregisterListener(mOnNewArrivedListener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        unbindService(mConnection);
    }
}
```







