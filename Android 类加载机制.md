# Android 类加载机制
- Android 设计基于 Java 的类加载机制，但针对 Android 特性进行了一些优化和扩展，负责将二进制数据文件（.dex 文件）加载到内存中
- Lazy Loading 懒加载：类只有在第一次被使用时才会被加载（按需加载），而不是一次性全部加载，这样可以减少内存占用，提高程序的启动速度
- Parent Delegation Model 双亲委派模型：通过双亲委派确保类的安全性和唯一性 

## ClassLoader 类加载器
- BootClassLoader 启动类加载器：负责加载系统核心类库（比如 java.lang、android.util 和 android.app 等），使用 C++ 实现的，在 Java 中无法直接获取，不可直接访问
- PathClassLoader：负责加载已安装 Apk 的 Dex 文件（比如 /data/app/xxx.apk 中的 classes.dex 文件）
- DexClassLoader：负责动态加载指定路径下未安装 Apk 的 Dex 文件（是实现插件化、热修复等的关键条件）
- Custom ClassLoader 自定义类加载器：可以继承 java.lang.ClassLoader 并重写 findClass 方法实现，通常用于加载非标准路径中的类（比如动态代理、加密文件中加载和网络下载后加载等）

## 双亲委派模型
- 类加载时，先委托父加载器（比如 PathClassLoader 的父加载器是 BootClassLoader）尝试加载，若父加载器无法加载，再由自身加载
- 不同类加载器负责不同范围的类
- 安全性：确保核心类（比如 android.app 包）由最顶层的 BootClassLoader 加载
- 唯一性：一个类只能由一个类加载器加载，同一个类不会被加载多次，避免重复加载，保证加载的类的唯一性

```java
//父子类加载器之间是 Composition 组合关系，而不是 Inheritance 继承关系
Custom ClassLoader -> DexClassLoader -> PathClassLoader -> BootClassLoader
```