# Android NDK 之崩溃排查
- 解析崩溃的内存地址（比如 #00 pc 7a3b6c  /data/app/.../lib/arm64/libapp.so）

## ndk-stack 批量解析
- ndk-stack 是 NDK 开发用来解析崩溃日志，需要结合未剥离的 .so 文件（确保编译时保留调试符号，避免 strip 剥离符号）中的调试符号，还原堆栈信息（比如 Windows 上的 \AndroidSdk\ndk\27.0.12077973\ndk-stack.cmd 文件）
- 能够自动匹配 .so 调试符号，批量解析日志中的所有堆栈帧，直接输出文件名、行号和函数名（自动将崩溃 pc 地址映射到源代码的行号和方法名）


### 1 保存日志到文件后进行解析
- 最终文件内容必须包含 *** *** *** *** *** 开头那一行才能解析
```shell
# 将带崩溃日志的 logcat 内容捕获输出（Ctrl+C 停止）
adb logcat > D:/temp/arm64-v8a/crash.log
# 也可以按需捕获筛选出 NDK-StackTrace:E （只保留标签为 NDK-StackTrace 的 Error 级日志，对应代码中 LOGE 输出）的日志
adb logcat -s NDK-StackTrace:E > crash_log.txt

# 可以先确认设备 ABI，以保证选择正确的 .so 文件
adb shell cat /proc/cpuinfo
adb shell getprop ro.product.cpu.abi
adb shell getprop ro.product.cpu.abilist
```

```shell
# ndk-stack 存放在 \AndroidSdk\ndk\27.0.12077973 目录下
# -sym 参数（--symbols）指定包含符号表（调试信息）的 .so 文件（或 .debug 文件）的父目录（工具会自动递归查找目录下所有文件，比如 Debug 模式的 .so 或 Release 模式的 .so.debug）
# -dump 指定崩溃日志文件路径
ndk-stack -sym D:/temp/arm64-v8a -dump D:/temp/arm64-v8a/crash.log
```

```shell
# 也可以直接将结果输出到指定文件
ndk-stack -sym D:/temp/arm64-v8a -dump D:/temp/arm64-v8a/crash.log > D:/temp/arm64-v8a/crash_dump.log
```

### 2 通过 adb logcat 实时输出日志
```shell
adb logcat | ndk-stack -sym $PROJECT_PATH/app/build/intermediates/cmake/debug/obj/arm64-v8a
adb logcat | ndk-stack -sym $PROJECT_PATH/app/build/intermediates/cxx/debug/obj/arm64-v8a
adb logcat | ndk-stack -sym $PROJECT_PATH/app/build/intermediates/cxx/debug/<hash>/obj/arm64-v8a
```

## addr2line 单个地址解析
- addr2line 是 NDK 工具链中的底层工具，用于解析单个堆栈地址对应的文件名、行号和函数名，适合快速验证单个地址
- 新版本的 NDK 统一名为 llvm-addr2line，不再区分架构前缀，所有架构使用同一个命令（比如 Windows 上的 \AndroidSdk\ndk\27.0.12077973\toolchains\llvm\prebuilt\windows-x86_64\bin>llvm-addr2line.exe 文件，旧版本的 NDK 必须使用与 .so 架构一致的工具进行操作）

```shell
# 00000000011cc48c 是单个堆栈地址
# -e 参数（--exe）指定符号表 .so 文件
llvm-addr2line -e D:/temp/arm64-v8a/libpwrtcsdkandroid.so 00000000011cc48c
# 打印出对应的文件名、行号
/home/ubuntu/rtc-build/paho.mqtt.cpp/include/mqtt/message.h:251
```

```shell
# -f 参数（--functions）显示函数名
llvm-addr2line -e D:/temp/arm64-v8a/libpwrtcsdkandroid.so -f 00000000011cc48c
_ZNK4mqtt7message9get_topicEv
/home/ubuntu/rtc-build/paho.mqtt.cpp/include/mqtt/message.h:251
```

```shell
# -C 参数（--demangle）解码 C++ 符号（Demangle），就是还原 C++ 符号
llvm-addr2line -e D:/temp/arm64-v8a/libpwrtcsdkandroid.so -f -C 00000000011cc48c
mqtt::message::get_topic() const
/home/ubuntu/rtc-build/paho.mqtt.cpp/include/mqtt/message.h:251
```
