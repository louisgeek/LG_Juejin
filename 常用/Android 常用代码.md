# Android 常用代码


## isMainThread
```java
//androidx.arch.core.executor.DefaultTaskExecutor#isMainThread
 public boolean isMainThread() {
    return Looper.getMainLooper().getThread() == Thread.currentThread();
 }
```

## IP、Email 等正则表达式
```java
//androidx.core.util.PatternsCompat
androidx.core.util.PatternsCompat#IP_ADDRESS
androidx.core.util.PatternsCompat#EMAIL_ADDRESS
```

```java
ContextCompat.getMainExecutor
ContextCompat.checkSelfPermission
ContextCompat.getString
ContextCompat.getDrawable
```

## SQLite
SQLite 查询前 10 条最新记录
```sql
SELECT * FROM records 
ORDER BY created_at DESC --按时间戳降序排列 updated_at
LIMIT 10; --限制返回前 10 条
```

SQLite 查询第 11-20 条最新记录
```sql
SELECT * FROM records 
ORDER BY created_at DESC
LIMIT 10 OFFSET 10; -- OFFSET 跳过前 10 条
```


## retrofit2.AndroidMainExecutor
```java
final class AndroidMainExecutor implements Executor {
    private final Handler handler = new Handler(Looper.getMainLooper());

    public void execute(Runnable r) {
        this.handler.post(r);
    }
}
```



------------------------------------------

1.隐藏标题栏（要在setContentView()之前）

    this.requestWindowFeature(Window.FEATURE_NO_TITLE);
2.隐藏状态栏（全屏 要在setContentView()之前）

    this.getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
3.XML隐藏标题栏、隐藏状态栏（全屏 应用单个Activity）
在AndroidManifest.xml的配置文件里面的<activity>标签添加属性：

    android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
4.XML隐藏标题栏、隐藏状态栏（全屏 应用全局） 

```xml
    <!-- Application theme. -->
    <style name="AppTheme" parent="AppBaseTheme">
    <!-- All customizations that are NOT specific to a particular API-level can go here. -->
    <!-- 隐藏状态栏 -->
    <item name="android:windowFullscreen">true</item>
    <!-- 隐藏标题栏 -->
    <item name="android:windowNoTitle">true</item>
    </style>
```










```shell
//Resizer.bat
start javaw -jar Resizer-1.4.2.jar
exit
```


Android View Binding in Thread error NullPointerException

https://stackoverflow.com/questions/70526892/android-view-binding-in-thread-error-nullpointerexception



## logcat

^(?!(ANDROID_LOG_FROM_APP|ANDROID_LOG_FROM_MT|DSP|BSP|chatty|sssss))


注意运算符的优先级，并用括号明确表达式的操作顺序，避免使用默认优先级



 #禁用增量编译 show duplicated file
kapt.incremental.apt=false




## gitbook

npm config set cache D:\DevTools\node-v14.17.6-win-x64\node_cache
npm config set prefix D:\DevTools\node-v14.17.6-win-x64\node-global
npm config set registry http://registry.npm.taobao.org
npm config set registry https://registry.npmjs.org
npm install -g gitbook-cli

gitbook init






#!/bin/bash
wd=`pwd`
sed -i 's/[ ]*//g' BatchClone.txt
for path in `cat BatchClone.txt`
do
#${wd} /d/LouisGitlab/hikthermalComponent
cd "${wd}"
dir=${path#*//}
#$dir /github.com/louisgeek/GitBatchClone.git
if [ ! -x "$dir" ]; then
echo "= Creating: project in ${wd} ="
#mkdir -p "$dir"
#cd "$dir"
#deletepath=`pwd`
#rm -rf "${deletepath}"
#cd ..
#${path} https://github.com/louisgeek/GitBatchClone.git
echo "- Running: git clone ${path} -"
git clone "${path}"
echo "= Successfully cloned ${path} ="
fi
done












https://curl.se/ca/cacert.pem

https://curl.se/docs/sslcerts.html

SSL certificate problem: unable to get local issuer certificate
//其实安装的时候可以设置
git config --global http.sslbackend schannel

git config --global http.sslcainfo /mingw64/ssl/certs/ca-bundle.crt





allprojects {

    repositories {
        maven {
            url 'http://10.14.80.31:8085/repository/android-maven-public/'
            allowInsecureProtocol true
        }
        all { ArtifactRepository repo ->
            if (repo instanceof MavenArtifactRepository) {
                def url = repo.url.toString()

                if (!url.startsWith('http://10.14.80.31:')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by My Maven."
                    remove repo
                }
            }
        }
    }

    buildscript {
        repositories {
            maven {
                url 'http://10.14.80.31:8085/repository/android-maven-public/'
                allowInsecureProtocol true
            }
            all { ArtifactRepository repo ->
                if (repo instanceof MavenArtifactRepository) {
                    def url = repo.url.toString()
                    if (!url.startsWith('http://10.14.80.31:')) {
                        project.logger.lifecycle "buildscript Repository ${repo.url} replaced by My Maven."
                        remove repo
                    }
                }
            }
        }
    }

}



    String ByteArrayToHexString(byte[] inarray) {
        int i, j, in;
        String[] hex = { "0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A",
            "B", "C", "D", "E", "F" };
        String out = "";

        for (j = 0; j < inarray.length; ++j) {
            in = (int) inarray[j] & 0xff;
            i = (in >> 4) & 0x0f;
            out += hex[i];
            i = in & 0x0f;
            out += hex[i];
        }
        return out;
    } 
	


```C
extern "C"
JNIEXPORT jstring JNICALL
Java_com_hikvision_common_EncryptConst_getHeNanAESKey(JNIEnv *env, jclass clazz) {
    std::string data = "567502e0e087c22b";
    return env->NewStringUTF(data.c_str());
}
```



  externalNativeBuild {
            cmake {
                cppFlags ''
                abiFilters 'armeabi-v7a'
//                abiFilters 'arm64-v8a'
            }
        }

   externalNativeBuild {
        cmake {
            path file('CMakeLists.txt')
//            path file('src/main/cpp/CMakeLists.txt')
            version '3.10.2'
        }
    }





