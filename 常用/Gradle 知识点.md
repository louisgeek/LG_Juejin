# Gradle 知识点


## gradle zip 下载镜像
https://mirrors.cloud.tencent.com/gradle/


## gradle src 下载存放路径
D:\DevToolsCache\.gradle\caches\modules-2\files-2.1\gradle\gradle\8.11.1\9c644d15409b381dbb7955662d16d55acf90e909\gradle-8.11.1-src.zip



```java
//https://github.com/gradle/gradle/blob/master/platforms/core-runtime/wrapper-shared/src/main/java/org/gradle/wrapper/PathAssembler.java

//distributionUrl=https://services.gradle.org/distributions/gradle-8.12-bin.zip
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


