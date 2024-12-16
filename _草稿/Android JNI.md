JNI Java 本地接口

- Java Native Interface 

- 是 Java 和其他语言沟通的桥梁
- Android 中提供一套 NDK 工具
- 

## extern "C" JNIEXPORT
- 函数标识
- extern "C" 让编译器以C的方式去编译它，如果不加的话，对于静态注册的函数而言，函数名发生了变更

JNIEnv *env 一定永远是每个jni函数的第一个参数  通过env去创建的对象，会由GC来管理它的内存与释放


```java
public class JNIUtils {
    // 加载native-jni
    static {
        System.loadLibrary("native-jni");
    }
    //java调C中的方法都需要用native声明且方法名必须和c的方法名一样
    public native String stringFromJNI();
}
```


```kotlin
companion object {
        // Used to load the 'myjni' library on application startup.
        init {
            System.loadLibrary("myjni")
        }
    }
```


##   LLDB
一种调试程序，Android Studio 使用它来调试原生代码。（目前已知 Android4.0.4 版本及之后的版本都已将LLDB调试工具内置到了NDK中）



- JNI 是属于 Java 的，与 Android 无直接关系。
，允许Java代码调用本地（C/C++）代码。JNI提供了Java和本地代码之间的桥梁 。


## NDK
而 NDK 的全称是 Native Development Kit，和 SDK 的全称是 Software Development Kit 一样，都是开发工具包。NDK 是 Android 开发的工具包，主要用作 C/C++ 开发，提供了相关的动态库。
Android NDK 是一套允许您使用 C 和 C++ 等语言，以原生代码实现部分应用的工具集。在开发某些类型的应用时，这有助于您重复使用以这些语言编写的代码库。
Android NDK 是一组允许您将 C 或 C++（“原生代码”）嵌入到 Android 应用中的工具，NDK描述的是工具集。
NDK 是 Native Development Kit 的缩写，是 Android 的工具开发包。作用是更方便和快速开发 C/C++ 的动态库，并自动将动态库与应用一起打包到 apk。NDK 是属于 Android 的，与 Java 无直接关系。
这套工具集允许您为 Android 使用 C 和 C++ 代码，并提供众多平台库，让您可以管理原生 Activity 和访问物理设备组件，例如传感器和触摸输入。



## ndk-build
ndk-build 是Android NDK提供的一个构建工具，用于编译、链接和构建本地代码。通过编写Android.mk文件，你可以配置项目的构建过程，包括静态/动态库的编译和链接，以及JNI本地方法的静态注册。


## CMake
CMake 就是一个跨平台构建工具，支持包括C/C++/Java等多种语言的工程构建
一款外部构建工具，可与 Gradle 搭配使用来构建原生库。如果您只计划使用 ndk-build，则不需要此组件。
make 是一种跨平台的构建工具，可以用于配置、编译和构建项目。在Android开发中，可以使用CMake来代替ndk-build，并且它提供更灵活和现代的构建配置。你可以编写CMakeLists.txt文件来描述项目的结构和依赖关系


## CMakeLists

CMakeLists.txt是CMake构建脚本。结合CMake的命令和内部变量，我们可以做很多事情



```
MyJni
```

## JNI_OnLoad




 
Java类型  JNI类型  C/C++类型   描述
boolean（布尔类型）jbooleanunsigned short无符号8位byte（字节类型）jbytesigned char有符号8位char（字符型）jcharunsigned short无符号16位shor（短整型）jshortshort有符号16位int（整型）jint/jsizeint有符号32位long（长整型）jlonglong/long long(_int64)有符号64位float（浮点型）jfloatfloat32位double（双精度浮点型）jdoubledouble64位
 
，在 Java 与 C++ 的函数进行匹配时，编译器会根据函数名、参数和返回值信息一起去匹配

，所以Java调用JNI函数时，编译器是根据JNI的函数名去匹配的。但是C++支持重载，所以Java调用JNI函数时，编译器会根据JNI的函数名+参数一起去匹配，所以出现找不到对应JNI函数的异常。



