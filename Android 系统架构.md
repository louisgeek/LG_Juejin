#  Android 系统架构
- Android 系统架构采用分层设计，从上到下分为五层：应用程序层、应用程序框架层、系统运行库层、硬件抽象层和 Linux 内核层

![Android 系统架构](https://gitee.com/louisgeek/LG_Notes/raw/master/images/android_xitongjiagou.png)

## Application Layer 应用程序层
- System Apps 用户直接交互的应用程序，比如 Email 邮件、Camera 相机、Brower 浏览器和三方应用等

## Application Framework Layer 应用程序框架层
- Java API Framework 提供核心 API 和 Framework 框架供开发者直接调用，包括 Window Manager 窗口管理、View System 视图系统、Activity Manager 活动管理、Content Providers 内容提供者、Resource Manager 资源管理和 Notification Manager 通知管理等

## System Runtime Layer 系统运行库层
- 包括 Native C/C++ Libraries 原生 C/C++ 程序库和 Android Runtime 运行时（ART）

### 原生 C/C++ 程序库
- 提供了一系列底层的系统功能，能被 Android 系统中的不同组件所使用，比如 Webkit 浏览器引擎、Media Framework 媒体框架、OpenGL ES 3D 图形渲染和 SQLite 数据库

### Android 运行时
- 负责执行 Android 应用程序的字节码（.dex 文件），ART 通过 AOT 安装时提前预编译（Dalvik 通过 JIT 运行时即时编译）技术来提高应用程序的性能（空间换时间，耗费更多存储空间，安装时间拉长）
- 管理内存分配和垃圾回收
- Core Libraries 核心库：提供了 Java SE API 的绝大数功能，也提供了 Android 的核心 API，允许开发者利用 Java 编写 Android 应用
        
管理内存分配和垃圾回收使得每个Android 程序拥有一个独立的进程中，都拥有自己的虚拟机实例
完成生命周期管理、堆栈管理、内存管理、垃圾回收

## Hareware Abstraction Layer 硬件抽象层
-  HAL 将硬件抽象化，隐藏各平台硬件细节差异，提供统一接口（比如 Audio 音频、Camera 摄像头和 Bluetooth 蓝牙），从而保证了 Android 系统的跨平台可移植性，同时也让软硬件并行测试成为了可能

## Linux Kernel Linux 内核层
- 基于 Linux 内核，负责硬件设备的驱动管理（比如 Audio 驱动、Camera 驱动、USB 驱动、Wifi 驱动和 Binder IPC 驱动等）、内存管理、进程调度和电源管理等功能
