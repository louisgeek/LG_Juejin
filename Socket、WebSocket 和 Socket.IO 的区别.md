# Socket、WebSocket 和 Socket.IO 的区别

## Socket
- Socket 套接字是为了方便使用 TCP、UDP 而抽象出来的一层 API 接口（几乎所有编程语言都支持 Socket 编程，Java 里也提供了遵循 Socket 编程理念的 API 供开发者使用，比如 Socket 和 DatagramSocket 类）
- Socket 算是传输层的接口
- Socket 自定义性强，灵活性高，需要手动处理连接、断连、心跳检测、断线重连和解析数据格式等逻辑，需要手动管理连接状态和数据传输

## WebSocket
- WebSocket 是一种基于 TCP 协议的双向通信（全双工）协议，属于应用层的协议，作为 HTML5 标准的一部分，以 ws:// 或 wss:// 开头
- WebSocket 通过 HTTP 的握手过程升级切换为持久化连接
- WebSocket 需要手动处理心跳检测、断线重连等逻辑

## Socket.IO
- Socket.IO 是基于 Node.js 的框架，不过也有 Java 等客户端的实现，跨平台兼容性强
- Socket.IO 可以是基于 WebSocket 协议上进行的封装（封装主要是为了形成统一通用的实时通信接口，具体可以通过 WebSocket、HTTP 长轮询或其他通信方式实现），而且支持在 WebSocket 不可用的情况下，能够提供 HTTP 长轮询作为备选方式（自动降级）处理数据（比如当前环境不支持 WebSocket 时，能够自动选择最佳的方式来实现网络实时通信）
- Socket.IO 默认封装了心跳检测、断线重连等逻辑
- Socket.IO 自带房间和命名空间特性，支持分组广播

## 总结
- Socket 是底层网络接口，WebSocket 是应用层协议，Socket.IO 是应用层框架
- Socket 适合对性能要求较高的场景，WebSocket 适合实时性要求较高的场景，Socket.IO 适合快速开发多人在线应用的场景