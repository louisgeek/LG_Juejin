# Dart Stream 的 2 种类型

## Single-Subscription Stream 单订阅流
- 是默认的 Stream 类型，只能被一个监听器订阅（多次订阅会抛出异常，即使取消后也无法重复订阅）
- 适用于一对一的异步事件处理，事件需要按顺序提供并且不能丢失
- 适用于网络请求、文件读取等场景

```dart
Stream<int> myStream = Stream.value(42);
//
Stream<int> iterableStream = Stream.fromIterable([1, 2, 3, 4, 5]);
//
StreamController<int> streamController = StreamController();
Stream<int> dataStream = streamController.stream;
```

```dart
dataStream.listen(
  (data) => print("Received: $data"), //数据回调
  onError: (error) => print("Error: $error"), //错误回调（可选）
  onDone: () => print("Stream completed"), //完成回调（可选）
);
```

## Broadcast Stream 广播流
- 允许多个监听器同时订阅，数据会广播给所有订阅者
- 适用于一对多的事件广播场景，即使没有监听者，事件也仍会被发送，不再使用后需要手动关闭
- 适用于用户登录状态变化（状态广播）、传感器数据监听等场景
- 局限性：不能保留已发送的事件（新的订阅者无法收到旧的已发送的时间）

利用 asBroadcastStream 将单订阅 Stream 转换为广播 Stream
```dart
Stream<int> singleStream = Stream.periodic(const Duration(seconds: 1), (count) => count);
Stream<int> broadcastStream = singleStream.asBroadcastStream();
```

使用 StreamController.broadcast 直接创建广播 Stream
```dart
StreamController<int> controller = StreamController.broadcast();
Stream<int> broadcastStream = controller.stream;
```

```dart
broadcastStream.listen(
  (data) => print("Listener 1: $data"),
);

broadcastStream.listen(
  (data) => print("Listener 2: $data"),
);
```

## 总结
- 单订阅流是默认的 Stream 类型，只能被一个监听器订阅（多次订阅会抛出异常，即使取消后也无法重复订阅）
- 单订阅流适用于一对一的异步事件处理，事件需要按顺序提供并且不能丢失（比如网络请求、文件读取等）
- 广播流允许多个监听器同时订阅，数据会广播给所有订阅者
- 广播流适用于一对多的事件广播场景，即使没有监听者，事件也仍会被发送，不再使用后需要手动关闭（比如用户登录状态变化（状态广播）、传感器数据监听等）
- 广播流的局限性：不能保留已发送的事件（新的订阅者无法收到旧的已发送的时间）