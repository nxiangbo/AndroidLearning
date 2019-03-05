# Android 序列化

## 

在Android中有两种序列化方式：Serializable和Parcelable
- Serializable：是为了保存对象的属性到本地文件、数据库、网络流、rmi以方便数据传输，当然这种传输可以是程序内的也可以是两个程序间的。Serializable序列化不保存静态变量，可以使用Transient关键字对部分字段不进行序列化，也可以覆盖writeObject、readObject方法以实现序列化过程自定义
- Parcelable：为了在程序内不同组件间以及不同Android程序间(AIDL)高效的传输数据而设计。

**Parcelable性能要高于Serializable**



## Serializable



## Parcelable

在Android Studio中安装**android parcelable code generator**插件，可以快速生成Parcelable类。

## 应用场景
- Serializable：在需要保存或网络传输数据时选择Serializable，因为android不同版本Parcelable可能不同，所以不推荐使用Parcelable进行数据持久化
- Parcelable：在内存间数据传输时推荐使用Parcelable，如activity间传输数据。

