# Android Init、Zygote 和 SystemServer 进程

## Init 进程
- 由 Linux 系统内核启动，是第一个用户空间进程，其 PID 为 1
- 启动和管理 adbd（adb 调试守护进程），logd（日志系统守护进程）等守护进程
- 启动 ServiceManager 进程（Binder 服务管家，管理 Service 的注册和查询）
- 解析 init.rc 文件后 fork 生成 Zygote 进程

## Zygote 进程
- 由 Init 进程启动，是所有 App 应用程序进程的父进程
- 初始化 Java 虚拟机并注册 JNI 方法，预加载 Android 框架的核心类库
- 通过 fork 复制自身来启动 SystemServer 进程
- 创建 Server Socket 用于接收创建新进程的请求消息
- 负责通过 fork 机制创建 App 应用程序进程，新创建的进程继承了 Zygote 进程已经初始化好的资源，从而加快 App 应用的启动速度
 
## SystemServer 进程
- 由 Zygote 进程启动 ，是 Zygote 孵化的第一个进程
- 通过 createSystemContext 来创建一个 ContextImpl 系统上下文，为系统服务提供了运行所需的上下文环境和各种功能支持
- 创建 SystemServiceManager，用于管理服务的生命周期，处理服务的启动、停止和重启
- 负责启动各种系统核心服务，包括 ActivityManagerService、PackageManagerService 和 WindowManagerService 等