# Android JNI 编程入门



[Android JNI编程—JNI基础](https://www.jianshu.com/p/aba734d5b5cd)

jni 是Java native interface的缩写，通过它可以实现Java代码和C/C++代码的相互调用。在Android中，提供了NDK可以方便实现jni。在创建Android项目时，勾选support C++即可。



- 静态注册native方法

JNI函数名为：Java_PkgName_ClassName_NativeMethodName格式的。

- 动态注册native方法

使用JNI_OnLoad函数。

```c
static JNINativeMethod gMethods[] = {
        {"method1",            "(Ljava/lang/String;Ljava/lang/String;)[B", (void *) method1},
        {"method2",            "([BLjava/lang/String;)Ljava/lang/String;", (void *) method2},
        {"hook_native",        "(Ljava/lang/reflect/Method;Ljava/lang/reflect/Method;)J", (void*)methodHook},
        {"restore_native",     "(Ljava/lang/reflect/Method;J)Ljava/lang/reflect/Method;",(void*)methodRestore}
};


static int registerNativeMethods(JNIEnv *env, const char *className,
                                 JNINativeMethod *gMethods, int numMethods) {
    jclass clazz;
    clazz = env->FindClass(className);
    if (clazz == NULL) {
        return JNI_FALSE;
    }
    if (env->RegisterNatives(clazz, gMethods, numMethods) < 0) {
        return JNI_FALSE;
    }
    return JNI_TRUE;
}

static int registerNatives(JNIEnv *env) {
    if (!registerNativeMethods(env, JNIREG_CLASS, gMethods,
                               sizeof(gMethods) / sizeof(gMethods[0]))) {
        return JNI_FALSE;
    }
    return JNI_TRUE;
}

JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
    LOGD("JNI_Onload");
    JNIEnv *env = NULL;
    jint result = -1;
    if (vm->GetEnv((void **) &env, JNI_VERSION_1_4) != JNI_OK) {
        return -1;
    }
    assert(env != NULL);
    jclass classEvaluateUtil = env->FindClass(kClassMethodHookChar);
    if (!registerNatives(env)) {
        return -1;
    }
    methodHookClassInfo.m1 = env -> GetStaticMethodID(classEvaluateUtil, "m1", "()V");
    methodHookClassInfo.m2 = env -> GetStaticMethodID(classEvaluateUtil, "m2", "()V");
    methodHookClassInfo.methodSize = reinterpret_cast<size_t>(methodHookClassInfo.m2) - reinterpret_cast<size_t>(methodHookClassInfo.m1);
    result = JNI_VERSION_1_4;
    return result;
}
```





## NDK 与JNI关系

JNI是实现Java与C/C++代码相互调用的手段，而NDK是在Android中实现JNI的工具开发包

在Android开发环境中，通过NDK可以实现JNI的功能。NDK主要用于快速开发C、C++动态库，并自动将so和应用打包成apk文件。

![ç¤ºæå¾](http://upload-images.jianshu.io/upload_images/944365-6607e9321d3cbddd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





## ndk-build

- [`Android.mk`](https://developer.android.com/ndk/guides/android_mk.html?hl=zh-cn)：必须在 `jni` 文件夹内创建 [`Android.mk`](https://developer.android.com/ndk/guides/android_mk.html?hl=zh-cn) 配置文件。 `ndk-build` 脚本将查看此文件，其中定义了模块及其名称、要编译的源文件、版本标志以及要链接的库。

- `Application.mk`

  ：此文件枚举并描述您的应用需要的模块。 这些信息包括：

  - 用于针对特定平台进行编译的 ABI。
  - 工具链。
  - 要包含的标准库（静态和动态 STLport 或默认系统）。