# Android Studio 打印中文乱码
- 首先不管 Java 源代码用的是什么编码格式, 编译成的 class 文件的编码格式都是一样的
- Android Studio 中打印中文乱码的问题主要分为 Build Output 打印乱码（Build 视图中 Build Output Tab 中）和 Run Console 打印乱码（Run 视图中 main 方法类名命名的 Tab 中 Console 中）两种
- Java 虚拟机在运行时有一个 JVM 参数 file.encoding，代码中的 FileReader、String.getBytes、网络、序列化等操作均会受其影响
- 另外 Android 项目中的 gradle.properties 文件中指定的 JVM 参数 org.gradle.jvmargs=-Dfile.encoding=UTF-8 仅影响 Gradle 构建过程本身（比如编译、打包等），不直接影响应用程序运行时 JVM 参数
- 通过在 \android-studio\bin\studio64.exe.vmoptions 文件中或者通过 Help -> Edit Custom VM Options 操作中添加 -Dfile.encoding=UTF-8 配置可以覆盖 file.encoding 的值（同时 vmoptions 文件中配置也会作为命令行参数传递给 Gradle，优先级高于 Gradle 构建配置）
- Android Studio -> Settings -> Editor -> File Encodings -> Project Encoding 的设置可以覆盖 file.encoding 的值（优先级比 vmoptions 文件中配置要高，但奇怪的是却影响不了 Build Output 乱码，而 vmoptions 文件中的配置却可以影响）


## 乱码解决
- JDK 17 及以前，file.encoding 的值默认是从操作系统获取的（比如中国大陆 Windows 的编码默认是 GBK），而从 JDK 19 及以后（JDK 18 比较特殊不考虑） file.encoding 的值统一默认为 UTF-8 了（不从操作系统获取了，除非显式用 -Dfile.encoding=COMPAT 回退到 JDK 17 及以前的行为），而且也不推荐通过 -Dfile.encoding=GBK 之类的配置去覆盖
- JDK 17 及以前，标准输出流（比如 System.out 等）受 file.encoding 影响，Run Console 乱码通常不会出现（如果遇到 Build Output 乱码问题，就在 vmoptions 文件中添加 -Dfile.encoding=UTF-8 配置去解决）
- JDK 19 及以后，标准输出流不再受 file.encoding 影响了，而是新增了 -Dstdout.encoding 和 -Dstderr.encoding 两个参数用来专门指定 System.out 和 System.err 的字符编码，如果不指定就从操作系统获取字符编码（所以标准输出流默认采用的是 GBK，而 file.encoding 决定控制台为 UTF-8），可以在 Run -> Edit Configurations 中在 Application 类别下 main 方法类名命名的配置下的 Modify options 选项勾选 Add VM options 后在新出现的 VM Options 输入框中增加 -Dstdout.encoding=UTF-8 -Dstderr.encoding=UTF-8 配置（或者通过 vmoptions 文件里添加 -Dfile.encoding=GBK 配置进行强制覆盖）来解决 Run Console 乱码
