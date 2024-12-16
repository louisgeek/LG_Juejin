# Java JNI
- JNI 是 Java Native Interface 的缩写，即 Java 本地接口，是 Java 提供的一种技术方案，作用就是能让 Java 与本地其他语言（如 C/C++）进行交互，JNI 是连接 Java 与 Native 之间的通讯桥梁，可以看作是个转换器
- Java 层调用 Native 层是通过 JNI 来实现的，而 Native 层调用 Java 层则是通过反射来调用的
- Java 层的反射 API 其实本质上是调用 C++ 的函数去实现的，所以 C++ 中的反射可以去直接调用 C++ 的函数去实现，会比调用 Java 中的反射要更加直接，减少了一层调用链
- JNI 是 C 语法实现的，所以在 cpp 文件中需要添加 extern "C" 来声明当前代码是 C 的语法，另外 extern "C" 可以修饰 #include，声明导入的头文件是 C 的语法

## Java 调用 Native
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
        //
        boolean booleanValue = doSometing(9);
        //Java 打印 doSometing 方法返回 booleanValue 值是：true
        Log.e("TAG", "Java 打印 doSometing 方法返回 booleanValue 值是：" + booleanValue);
    }
    /**
     * A native method that is implemented by the 'myjni' native library,
     * which is packaged with this application.
     */
    public native String stringFromJNI(); //java native 本地方法
    public native boolean doSometing(int intValue);
    //
    public String onSometing(float floatValue) {
        //Java 打印 onSometing 方法参数 floatValue 值是：11.6
        Log.e("TAG", "Java 打印 onSometing 方法参数 floatValue 值是：" + floatValue);
        return "floatValue=" + floatValue;
    }
}
```

```cpp
#include <jni.h>
#include <string>
//extern "C" 代表需要让编译器以 C 的语法来编译，JNIEXPORT 是宏，这里代表默认可见，JNICALL 是宏，是空实现，可以省略
extern "C" JNIEXPORT jstring JNICALL
Java_com_louis_myjni_MainActivity_stringFromJNI(
        JNIEnv* env,           //JNIEnv 指的是当前 java 环境，利用 JNIEnv 可以操作 java 层代码
        jobject /* this */) {  //jobject 指的是 JNI 函数对应的 java native 本地方法所在类的实例对象，如果本地方法是 static 的话则类型就需要改成 jclass，代表的是类对象
    std::string hello = "Hello from C++";
    //利用 UTF-8 char 字符数组构造出 jstring 并返回，通过 env 去创建的对象会由 GC 来管理内存释放
    return env->NewStringUTF(hello.c_str());
}
//
extern "C"
JNIEXPORT jboolean JNICALL
Java_com_louis_myjni_MainActivity_doSometing(JNIEnv *env, jobject thiz, jint int_value) {
    jfloat float_value = (jfloat) int_value;
    jfloat float_value_new = 2.6F + float_value;

    //通过 Java 实例对象 thiz 来获取 Java 类对象
    jclass clazz = env->GetObjectClass(thiz);
    //通过 jclass 获取对应的函数 ID
    jmethodID methodId = env->GetMethodID(clazz, "onSometing", "(F)Ljava/lang/String;");
    //
    //通过 thiz 对象和函数 ID 来调用函数，无返回值用 CallVoidMethod，返回值 int 用 CallIntMethod
    jobject backObj = env->CallObjectMethod(thiz, methodId, float_value_new);
    jstring backStr = (jstring) (backObj);
    //
    const char *newStr = env->GetStringUTFChars(backStr, 0);
    LOGE("C 打印 onSometing 方法返回值是：%s", newStr);//C 打印 onSometing 方法返回值是：floatValue=11.6
    return true;
}
```

## extern "C" JNIEXPORT jstring JNICALL
```cpp
//jni.h 文件
//可见性，default 默认可见
#define JNIEXPORT  __attribute__ ((visibility ("default")))
//空实现 可以省略
#define JNICALL
```
 
## JNI 函数命名格式
```cpp
//Java 开头，xxx.yyy.zzz 包名的点换成下划线，然后跟上类名和对应的 java native 方法的方法名，四者用下划线隔开
Java_<包名>_<类名>_<方法名>
Java_xxx_yyy_zzz_MainActivity_stringFromJNI
//比如
Java_com_louis_myjni_MainActivity_stringFromJNI
```

## 函数签名
- 函数的签名：函数的名称、参数类型和返回类型
- Java 和 C++ 支持函数重载（方法重载），而 C 不支持函数重载






## 特点
- 可以提高性能，比如 Native 层去实现视频渲染等会更加高效
- 方便进行平台之间的移植


## 静态库和动态库
- Static Library 静态库：编译时静态链接到目标程序，形成一个可执行文件，其中 .a（Unix/Linux，archive 归档文件的缩写）或 .lib（Windows）是静态库常见的扩展名
- Shared Library 动态库（共享库）：在运行时动态链接到目标程序，其中 .so（Unix/Linux，shared object 共享对象的缩写）或 .dll（Windows）是动态库常见的扩展名
 





