# Gradle 知识点

## gradle zip 下载镜像
https://mirrors.cloud.tencent.com/gradle/

```java
//bin all 等 zip 下载存放路径
D:\DevToolsCache\.gradle\wrapper\dists
//特别的 src 下载存放路径（哈希算法就是文件的 sha1 值）
D:\DevToolsCache\.gradle\caches\modules-2\files-2.1\gradle\gradle\8.11.1\9c644d15409b381dbb7955662d16d55acf90e909\gradle-8.11.1-src.zip
D:\DevToolsCache\.gradle\caches\modules-2\files-2.1\gradle\gradle\8.6\992d01a52586a897215f4a816bf303e30367606b\gradle-8.6-src.zip
```

## 哈希算法
```java
//获取 zip 下载缓存的目录名
//https://github.com/gradle/gradle/blob/master/platforms/core-runtime/wrapper-shared/src/main/java/org/gradle/wrapper/PathAssembler.java

 private static String getHash(String string) {
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            byte[] bytes = string.getBytes("UTF-8");
            messageDigest.update(bytes);
            return new BigInteger(1, messageDigest.digest()).toString(36);
        } catch (Exception e) {
            throw new RuntimeException("Could not hash input string.", e);
        }
 }
 public static void main(String[] args) {
    String distributionUrl = "https://services.gradle.org/distributions/gradle-8.12-bin.zip";
    System.out.println(getHash(distributionUrl)); //cetblhg4pflnnks72fxwobvgv
 }
```

```groovy
import java.security.MessageDigest

def distributionUrl = 'https://services.gradle.org/distributions/gradle-8.12-bin.zip'

task getGradleWrapperDiskCacheName doLast {
    MessageDigest messageDigest = MessageDigest.getInstance('MD5')
    messageDigest.update(distributionUrl.bytes)
    def name = new BigInteger(1, messageDigest.digest()).toString(36)
    println("source: $distributionUrl")
    println("wrapper disk cache name : $name") //cetblhg4pflnnks72fxwobvgv

}
```

## AGP 版本和 Gradle 版本
- 对应关系：https://developer.android.google.cn/studio/releases/gradle-plugin?hl=zh-cn#updating-plugin，比如 AGP 版本 4.2 最低对应 Gradle 版本 6.7.1，AGP 版本 8.0 最低对应 Gradle 版本 8.0，AGP 版本 8.11 最低对应 Gradle 版本 8.13
- AGP 版本：plugins 中配置 id 'com.android.application' version '8.0.0' apply false（旧版：buildscript.dependencies 中配置 classpath 'com.android.tools.build:gradle:4.2.0'）
- Gradle 版本：\gradle\wrapper\gradle-wrapper.properties
- AS 查看当前项目版本：File -> Project Structure -> Project 中

组合示例
```shell
com.android.application 
agp = "7.3.0"
kotlin = "1.8.20" #1.8.10 有点问题
coreKtx = "1.6.0"
distributionUrl=https\://services.gradle.org/distributions/gradle-7.4-bin.zip
```

```shell
com.android.application 
agp = "8.12.0"
kotlin = "2.0.21"
coreKtx = "1.10.1"
distributionUrl=https\://services.gradle.org/distributions/gradle-8.13-bin.zip
```

## maven
```groovy
//            url layout.buildDirectory.dir("maven-repo")
//            url layout.projectDirectory.dir("maven-repo")
//            url  uri("$project.projectDir/reposss")
            url  uri("${project.rootDir}/maven-repo")
//            url  uri('/../local-4repo')
//            url "file:///${project.buildDir}/repos"
//            url uri("file://${project.projectDir}/repomaven") // 本地仓库路径


maven { url 'https://maven.aliyun.com/repository/google' }
maven { url 'https://maven.aliyun.com/repository/central' }
//maven { url 'https://maven.aliyun.com/repository/public' }
maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
maven { url 'https://www.jitpack.io' }
```


## 依赖库版本问题
- 如果 lib 库有引入高版本的依赖库 ，那主项目里的该依赖库会全部替换成高版本的，即使代码里显式引入低版本也不生效（默认情况下），所以推荐 lib 里尽量引入低版本的依赖库，这样主项目里依赖高低版本都可以，比较灵活（比如 `implementation 'androidx.constraintlayout:constraintlayout:1.1.3'`）





## dependencyResolutionManagement
Gradle 7.0 及以上 settings.gradle 文件中的 dependencyResolutionManagement.repositories 对应 Gradle 7.0 以下 build.gradle 文件中的 allprojects.repositories
类似 maven { url 'https://jitpack.io' } 这种需要写在 dependencyResolutionManagement.repositories 下或者 allprojects.repositories 下



gradlew -q  mobile_petv:assemblePetVGoogleplayRelease

```shell
//打印包依赖关系  app 代表模块名称
//CMD
gradlew -q app:dependencies
//PowerShell
.\gradlew -q app:dependencies
```


gradlew assembleRelease 可以简写为 ./gradlew aR