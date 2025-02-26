# Android MultiDex
- 在 Andorid 低版本中，应用程序的单个 dex 文件（Dalvik Executable）可包含的最大方法数有 65536 限制（2^16，也叫 64K 方法数限制），当应用包含大量的代码和三方库时，就很容易超过这个限制，会导致编译错误
- MultiDex 就是用来解决这个限制的技术，不过 Android 5.0（API 21）及以后的版本，系统会默认启用 MultiDex，且不需要依赖 MultiDex 库，默认就支持从 apk 文件加载多个 dex 文件
- MultiDex 的原理就是将应用的代码拆分成多个 dex 文件（主 dex 文件和多个次 dex 文件），主 dex 文件中包含应用启动所必需的类，系统会在应用启动时加载主 dex 文件，然后在需要时动态加载次 dex 文件，从而允许应用程序包含更多的方法
- 在构建应用时通过 Gradle 插件利用 MultiDex 工具对应用代码进行切分多个 dex 文件（通常命名 classes.dex、classes2.dex、classes3.dex）
- 在运行时通过特殊的 ClassLoader 类加载器在多个 dex 文件中搜索加载对应的方法进行使用

## 1 启用 MultiDex
- 需要在 Gradle 中配置 multiDexEnabled 属性启用 MultiDex 支持

```groovy
android {
    defaultConfig {
        ...
        multiDexEnabled true
    }
        ...
}
dependencies {
    ...
    //Dalvik 虚拟机需要依赖，而 ART 虚拟机不需要
    implementation 'androidx.multidex:multidex:2.0.1'
}
```

## 2 修改 Application 类
- 如果没有自定义 Application 类，可以直接在 AndroidManifest.xml 文件中将 android:name 属性设置成 androidx.multidex.MultiDexApplication
```xml
<application
    android:name="androidx.multidex.MultiDexApplication">
    ...
</application>
```

- 如果有自定义 Application 类，可以修改现有自定义 Application 类去继承自 MultiDexApplication 类
```java
public class MyApplication extends MultiDexApplication {
    @Override
    public void onCreate() {
        super.onCreate();
    }
}
```

- 或者在自定义 Application 类中重写 attachBaseContext 方法以调用 MultiDex 的 install 方法
```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
    }
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        //
        MultiDex.install(this);
    }
}
```