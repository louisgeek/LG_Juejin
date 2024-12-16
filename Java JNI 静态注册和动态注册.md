# Java JNI 静态注册和动态注册
- 静态注册是指 Java native 方法和对应的本地函数按照预先定义好的命名规则来建立关联，实现了 Java native 方法与本地函数实现的绑定，这些函数会在 Java 类加载时被静态注册到 JVM 中，以确保 JVM 能够正确地找到对应的函数
- 动态注册是通过提供一个 Java native 方法和本地函数一一对应的映射表，然后在 JNI_OnLoad 回调里将映射表注册给 JVM，这样一来 JVM 就可以用这个映射表来调用相应的函数了，而不必像静态注册那样通过函数名去查找需要调用的函数，所以本地函数名不需要遵循特定命名规则
- PS：在 JNI 中静态注册和静态库之间没有直接的关系，无论是静态注册还是动态注册都可以生成动态库（.so文件）或者静态库（.a 文件）


```java
public class MainActivity extends AppCompatActivity {
    // Used to load the 'myjni' library on application startup.
    static {
        System.loadLibrary("myjni");
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //
        String str = stringFromJNI();
        Log.e("TAG", "onCreate: str="+str);
    }
    /**
     * A native method that is implemented by the 'myjni' native library,
     * which is packaged with this application.
     */
    public native String stringFromJNI(); //java native 方法
}
```


## 静态注册
- 本地函数的函数名需要遵循 Java_类全限定名_方法名 的格式（类全限定名中的 . 需要用 _ 替换）命名规范

```cpp
extern "C" JNIEXPORT jstring JNICALL
Java_com_louis_myjni_MainActivity_stringFromJNI(
        JNIEnv* env,           //JNIEnv 指的是当前 JNI 环境，利用 JNIEnv 可以操作 java 层代码
        jobject /* this */) {  //jobject 指的是 JNI 函数对应的 java native 方法所在类的实例对象，如果 native 方法是 static 的话则类型就需要改成 jclass，代表的是类对象
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```



## 动态注册
- 当 Java 层调用 System.loadLibrary 方法时，JNI_OnLoad 函数会在库被加载到 Java 虚拟机时调用
- PS：比如 Android 系统的 JNI 函数和 FFmpeg 相关函数和都是使用的动态注册

```cpp
//定义本地函数，函数名任意
jstring stringFromJNI_JNI_OnLoad(JNIEnv *env, jobject/* this */) {
    std::string hello = "Hello from C++  stringFromJNI_JNI_OnLoad";
    return env->NewStringUTF(hello.c_str());
}

//定义一个 JNINativeMethod 结构体数组（映射表），结构体包含 Java 方法名、方法签名（方法描述符）以及对应关联的本地函数的指针
JNINativeMethod nativeMethods[] = {
    //stringFromJNI 对应 Java native 方法，stringFromJNI_JNI_OnLoad 对应本地函数
    {"stringFromJNI", "()Ljava/lang/String;", (jstring *) stringFromJNI_JNI_OnLoad}
};

//Java 调用 System.loadLibrary 时此 JNI_OnLoad 函数就会执行
//JavaVM *vm 参数是一个指向 Java 虚拟机的指针，通过这个指针可以访问 JVM 的功能
//void *reserved 是一个保留参数，通常不使用
JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env;
    //获取与当前线程关联的 JNI 环境
    if (vm->GetEnv((void **) &env, JNI_VERSION_1_6) != JNI_OK) {
        LOGE("获取 JNI 环境失败");
        return JNI_ERR;
    }
    //通过类的全限定名来查找并加载一个类，返回一个指向该类的 jclass 引用
    jclass clazz = env->FindClass("com/louis/myjni/MainActivity");
    if (clazz == NULL) {
        LOGE("获取 jclass 失败");
        return JNI_ERR;
    }
    //计算 arr 数组的长度 int length = sizeof(arr) / sizeof(arr[0]);
    int nMethods = sizeof(nativeMethods) / sizeof(nativeMethods[0]);
    //参数 nMethods 指定数组中 NativeMethod 的数量
    if (env->RegisterNatives(clazz, nativeMethods, nMethods) != JNI_OK) {
        LOGE("注册本地函数失败");
        return JNI_ERR;
    }
    LOGE("注册成功");
    return JNI_VERSION_1_6; //返回当前支持的 JNI 版本号
}
```

## 总结
- 静态注册简单易用，函数必须要有类似 JNIEXPORT jstring JNICALL 这样的相关声明，而且函数名必须要按照 JNI 的命名规则去命名，由于 JNI 函数名暴露，所以存在一定的安全隐患
- 动态注册灵活性高，不依赖于固定的命名规则，不过需要自行注册手动关联，可以做到在运行时根据条件选择不同的实现，更适合复杂项目