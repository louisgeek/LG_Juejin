# Android Studio 记录

## Android Studio 下载归档
- 切到英文
https://developer.android.google.cn/studio/archive

## Android Studio 运行 Java 的 main 方法
- 在项目的 .idea/gradle.xml 文件中的  <GradleProjectSettings> 标签里添加配置
```xml
//注意 Settings 里修改 Gradle JDK 行为会重置修改 .idea/gradle.xml 文件
<option name="delegatedBuild" value="false" />
```

## Android Gradle 插件和 Android Studio 兼容性
https://developer.android.google.cn/studio/releases?hl=zh-cn#android_gradle_plugin_and_android_studio_compatibility


## 路径
```java
//系统目录
C:\Users\<User>\AppData\Local\Google
//用户配置目录（比如 Edit Custom VM Options 操作生成的 studio64.exe.vmoptions 文件）
C:\Users\<User>\AppData\Roaming\Google
```

JetBrains IDE 
```java
//系统目录 idea.system.path
C:\Users\<User>\AppData\Local\JetBrains
//用户配置目录 idea.config.path 
C:\Users\<User>\AppData\Roaming\JetBrains
```

## 安全软件实时扫描
- Microsoft Defender
```java
//Windows 安全中心 -> 病毒和威胁防护 -> 管理设置 -> 排除项
D:\DevToolsCache\.gradle
D:\DevTools\AndroidSdk
D:\LouisWork\LouisDemo
C:\Users\admin\AppData\Local\Google\AndroidStudio2024.3.2
```

## 文件头注释自动生成
- 打开  Settings -> Editor -> Code Style -> File And Code Templates -> Include -> File Header
```java
/**
 * Created by ${USER} on ${DATE}.
 */
```

## 自动导包 
打开 Setting -> Editor -> General -> Auto Import 
勾选 Java 和 Kotlin 下的复选框
```java
Add unambiguous imports on the fly //自动添加无歧义的 import（这里的 fly 大概的意思就是在输入时或者文件获取焦点时）
Optimize imports on the fly //自动优化清除无用的 import
```














2 Logcat 打印输出颜色分明设置 
打开Setting->Editor->Colors & Fonts->Android Logcat 
选新Logcat风格修改Foregound颜色（我AS是2.0 p4要取消勾选 Use inherited attributes再设置） 
分别选一个分类的信息修改对应的颜色 建议颜色值

```xml
V BBBBBB
D 0070BB
I 48BB31
W BBBB23
E FF0006
A 8F0005
```


4 代码提示不区分大小写 
打开 Setting -> Editor -> General -> CodeCompletion 
 


8 cannot run program “git.exe”…

打开Settings -> Version Control -> Git 之后，在选项 “Path to Git Executable” 你可以看到 “git.exe” , 填入当前安装git的安装的目录的\bin\git.exe，后面可以用test按钮测试下是否可用



9 变量名 带m前缀 静态变量s前缀
在 Settings - Editor - Code Style - Java 的最后一个 Tab ：Code Generation 里有个 Naming 表格，其中 Field 的 Name prefix 列填入 m 就可以让 member 变量以 m 开头时自动提示了
Static Filed 前缀为:s


11 去掉第一次初始化功能
进入刚安装的Android Studio目录下的bin目录。找到idea.properties文件，用文本编辑器打开。 在idea.properties文件末尾添加一行： disable.android.first.run=true


## 安装问题
- Android Studio 4.2.2 
- Android Studio 2020.3.1

 install apk 夜神 7.1.2  模拟器报如下错误

```shell
Installation did not succeed.
The application could not be installed: INSTALL_PARSE_FAILED_NO_CERTIFICATES

List of apks:
[0] 'D:\LouisWork\MyApplication\app\build\outputs\apk\debug\app-debug.apk'
APK signature verification failed.
```

夜神 5.1.1  模拟器没有出现该问题

而同环境下

逍遥安卓 7.1.2  模拟器没有出现该问题


## 插件

https://plugins.zhile.io
IDE Eval Reset

https://repo.idechajian.com
BetterIntelliJ

激活地址
```
C:\Users\Public\.jetbrains
```



### adb 安装 apk 报 INSTALL_FAILED_TEST_ONLY

Android Studio 上直接运行是可以的，但是打成 debug apk 给测试使用 adb install -r xxx.apk 时，会报错提示：INSTALL_FAILED_TEST_ONLY

```java
方法一：在项目中的全局配置 gradle.properties 文件中设置 android.injected.testOnly = false
方法二：命令加 -t 属性：adb install -t -r xxx.apk 
```



