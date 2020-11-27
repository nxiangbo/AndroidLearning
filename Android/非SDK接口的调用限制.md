# 非sdk接口的调用限制

为了规范系统ＡＰＩ的使用，Android P 之后，系统对API的调用做了限制。对限制API，Google不保证API的稳定性。API主要分为黑名单、灰名单、白名单等。对于黑名单的ＡＰＩ，系统会抛出错误。灰名单的会做出警告。



## API限制实现原理

在走反射调用逻辑时，系统做了校验。对于灰名单中的API，给予warn提示。对于黑名单的API，直接拒绝访问。

以Class#getDeclaredMethod方法为例

```java
 public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
        throws NoSuchMethodException, SecurityException {
        return getMethod(name, parameterTypes, false);
    }

private Method getMethod(String name, Class<?>[] parameterTypes, boolean recursivePublicMethods)
            throws NoSuchMethodException {
        if (name == null) {
            throw new NullPointerException("name == null");
        }
        if (parameterTypes == null) {
            parameterTypes = EmptyArray.CLASS;
        }
        for (Class<?> c : parameterTypes) {
            if (c == null) {
                throw new NoSuchMethodException("parameter type is null");
            }
        }
        Method result = recursivePublicMethods ? getPublicMethodRecursive(name, parameterTypes)
                                               : getDeclaredMethodInternal(name, parameterTypes);
        // Fail if we didn't find the method or it was expected to be public.
        if (result == null ||
            (recursivePublicMethods && !Modifier.isPublic(result.getAccessFlags()))) {
            throw new NoSuchMethodException(getName() + "." + name + " "
                    + Arrays.toString(parameterTypes));
        }
        return result;
    }
	
```



```java
private native Method getDeclaredMethodInternal(String name, Class<?>[] args);
```



getDeclaredMethodInternal这是一个native方法，经过JNI调用，进入如下方法：

[-> java_lang_Class.cc]

```CPP
static jobject Class_getDeclaredMethodInternal(JNIEnv* env, jobject javaThis,
                                               jstring name, jobjectArray args) {
  ScopedFastNativeObjectAccess soa(env);
  StackHandleScope<1> hs(soa.Self());
  DCHECK_EQ(Runtime::Current()->GetClassLinker()->GetImagePointerSize(), kRuntimePointerSize);
  DCHECK(!Runtime::Current()->IsActiveTransaction());
  Handle<mirror::Method> result = hs.NewHandle(
      mirror::Class::GetDeclaredMethodInternal<kRuntimePointerSize, false>(
          soa.Self(),
          DecodeClass(soa, javaThis),
          soa.Decode<mirror::String>(name),
          soa.Decode<mirror::ObjectArray<mirror::Class>>(args)));
  //检测该方法是否允许访问【小节2.4】
  if (result == nullptr || ShouldBlockAccessToMember(result->GetArtMethod(), soa.Self())) {
    return nullptr;
  }
  return soa.AddLocalReference<jobject>(result.Get());
}
```

### ShouldBlockAccessToMember

[-> java_lang_Class.cc]

```CPP
template<typename T>
ALWAYS_INLINE static bool ShouldBlockAccessToMember(T* member, Thread* self)
    REQUIRES_SHARED(Locks::mutator_lock_) {
  //【小节2.5】
  hiddenapi::Action action = hiddenapi::GetMemberAction(
      member, self, IsCallerTrusted, hiddenapi::kReflection);
  //对于不是允许级别的接口，则通知相应监听器
  if (action != hiddenapi::kAllow) {
    hiddenapi::NotifyHiddenApiListener(member);
  }
  return action == hiddenapi::kDeny;
}
```

这里是限制反射访问，此处的access_method等于hiddenapi::kReflection，除此之外还有其他几种模式，如下

#### 2.4.1 AccessMethod

```CPP
enum AccessMethod {
  kNone,        // 测试模式，不会出现在实际场景访问权限
  kReflection,  // Java反射调用
  kJNI,         // JNI调用过程
  kLinking,     // 动态链接过程
};
```

#### 2.4.2 限制场景

ShouldBlockAccessToMember是限制内部API访问的核心路径

- kReflection反射过程：
  - Class_newInstance：对象实例化
  - Class_getDeclaredConstructorInternal：构造方法
  - Class_getDeclaredMethodInternal：获取方法
  - Class_getDeclaredField：获取字段
  - Class_getPublicFieldRecursive：获取字段
- kJNI的JNI调用过程：
  - FindMethodID：查找方法
  - FindFieldID：查找字段
- kLinking动态链接：
  - UnstartedClassNewInstance
  - UnstartedClassGetDeclaredConstructor
  - UnstartedClassGetDeclaredMethod
  - UnstartedClassGetDeclaredField









参考稳定

[另一种绕过 Android P以上非公开API限制的办法](http://weishu.me/2019/03/16/another-free-reflection-above-android-p/)

[理解Android P内部ＡＰＩ的限制调用机制](http://gityuan.com/2019/01/26/hidden_api/)