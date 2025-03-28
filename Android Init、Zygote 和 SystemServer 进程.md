# Android Init、Zygote 和 SystemServer 进程

## Init 进程
- 由 Linux 系统内核启动，是第一个用户空间进程，其 PID 为 1，负责初始化系统属性、挂载文件系统和启动守护进程等
- 启动和管理 adbd（adb 调试守护进程），logd（日志系统守护进程）和 ServiceManager（Binder 服务管家，管理 Service 的注册和查询）等守护进程
- 解析 init.rc 文件后 fork 生成启动 Zygote 进程

## Zygote 进程
- 由 Init 进程启动，是受精卵、孵化器（Zygote 是所有 App 应用程序进程的父进程），负责启动虚拟机、预加载资源、启动 SystemServer 进程和孵化应用进程
- 初始化 Java 虚拟机并注册 JNI 方法，反射调用 ZygoteInit#main 方法预加载 Android 框架的核心类库和资源
- 创建 Server Socket 用于接收创建新进程的请求消息，使得 Zygote 能够响应 AMS（SystemServer 进程里）的请求，以创建新的应用程序进程
- 负责通过 fork 机制创建 App 应用程序进程，新创建的进程继承了 Zygote 进程已经初始化好的资源，从而加快 App 应用的启动速度
- 通过 fork 复制自身来启动 SystemServer 进程
 
## SystemServer 进程
- 由 Zygote 进程启动，是 Zygote 孵化的第一个进程
- 通过 createSystemContext 来创建一个 ContextImpl 系统上下文，初始化 ActivityThread，为系统服务提供了运行所需的上下文环境和各种功能支持
- 创建 SystemServiceManager，用于管理服务的生命周期，处理服务的启动、停止和重启
- 调用 SystemServer#main 启动各种系统核心服务，包括 AMS、PMS 和 WMS 等