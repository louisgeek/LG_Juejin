# Flutter Channel 通信机制
- 在 Flutter 中，Channel 通道是实现 Flutter 与原生平台（比如 Android 和 IOS 等）之间通信的核心机制，通过三种不同类型的通道来实现数据的传递和方法的调用
- Channel 分为 MethodChannel、EventChannel 和 BasicMessageChannel

## MethodChannel 方法通道
- MethodChannel 是最常用的通道，用于 Flutter 端和原生端之间进行方法调用和返回结果，适合一次性调用（比如获取设备信息）
- 双向同步，请求-响应模式
```dart
//定义通道名称（需与原生端保持一致）
final MethodChannel _channel = MethodChannel('com.example.method_channel');
//调用原生端方法
Future<void> _callNativeMethod() async {
  try {
    final String result = await _channel.invokeMethod('getAndroidInfo');
    print('Android 返回信息: $result');
  } on PlatformException catch (e) {
    print('调用失败: ${e.message}');
  }
}
//处理原生端调用
_channel.setMethodCallHandler(_handleMethodCall);
Future<dynamic> _handleMethodCall(MethodCall call) async {
  if (call.method == 'callFlutterFun') {
    print('Android 调用时传入的参数: ${call.arguments}');
    return 'Flutter received';
    return result;
  }
  return null;
}
```

```kotlin
class MainActivity : FlutterActivity() {
    private val METHOD_CHANNEL = "com.example.method_channel"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        //
        val channel = MethodChannel(flutterEngine.dartExecutor.binaryMessenger, METHOD_CHANNEL)
        channel.setMethodCallHandler { call, result ->
            //处理 Flutter 端调用
            if (call.method == "getAndroidInfo") {
                result.success("Hello from Android")
            } else {
                result.notImplemented()
            }
        }
    }
    fun callFlutterMethod() {
        val channel = MethodChannel(flutterEngine.dartExecutor.binaryMessenger, METHOD_CHANNEL)
        //调用 Flutter 端方法
        channel.invokeMethod("callFlutterFun", "Hello from Android")
    }
}
```

## EventChannel 事件通道
- EventChannel 用于 Flutter 端持续接收原生端的事件流（比如传感器数据监听、网络状态监听和电池状态监听等）
- 单向异步
- EventChannel 底层也是 MethodChannel 实现的
```dart
//定义通道名称（需与原生端保持一致）
final EventChannel _channel = EventChannel('com.example.event_channel');
//监听原生端事件
void _listenToEvent() {
  _channel.receiveBroadcastStream().listen(
    (event) => print("接收到 Android 的事件数据: $event"),
    onError: (error) => print("错误: $error"),
  );
}
```

```kotlin
class MainActivity : FlutterActivity() {
    private val EVENT_CHANNEL = "com.example.event_channel"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        //
        val channel = EventChannel(flutterEngine.dartExecutor.binaryMessenger, EVENT_CHANNEL)
        channel.setStreamHandler(object : EventChannel.StreamHandler {
            //
            private var eventSink: EventChannel.EventSink? = null

            override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
                //
                eventSink = events
                //模拟发送事件
                for (i in 1..5) {
                    eventSink?.success("Event $i from Android")
                    Thread.sleep(1000) //模拟延迟
                }
            }

            override fun onCancel(arguments: Any?) {
                //释放资源
                eventSink = null
            }
        })
    }
}
```

## BasicMessageChannel 基础消息通道
- BasicMessageChannel 用于 Flutter 端和原生端之间进行字符串或二进制等信息的传递，适合双向通信、简单数据交换和大文件传输
- 双向异步
- 编码器：StandardMessageCodec（默认）、StringCodec、JSONMessageCodec 和 BinaryCodec
```dart
final BasicMessageChannel<String> _channel = BasicMessageChannel('com.example.message_channel', StringCodec());

void _sendMessage() {
//处理来自原生端的消息
_channel.setMessageHandler((message) async {
      print('Received message: $message');
      //业务处理
      return 'Reply from Flutter';
    });
}
//给原生端发消息
void _sendMessage() {
  _channel.send('Hello from Flutter')
    .then((reply) => print('Received reply: $reply'))
    .catchError((error) => print('发送失败: ${error.message}'));
}
void _sendMessage2() async {
  try {
    String reply = await _channel.send('Hello from Flutter');
    print('Received reply: $reply');
  } on PlatformException catch (e) {
    print('发送失败: ${e.message}');
  }
}
```

```kotlin
class MainActivity : FlutterActivity() {
    private val MESSAGE_CHANNEL = "com.example.message_channel"
    
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        //
        val channel = BasicMessageChannel(flutterEngine.dartExecutor.binaryMessenger, MESSAGE_CHANNEL, StringCodec.INSTANCE)
        channel.setMessageHandler { message, reply ->
            //处理来自 Flutter 的消息
            println("Received message: $message")
            //通过 reply 给 Flutter 端作回应
            reply.reply("Reply from Android")
        }
    }
    //
    fun sendMessage() {
      val channel = BasicMessageChannel(flutterEngine.dartExecutor.binaryMessenger, MESSAGE_CHANNEL, StringCodec.INSTANCE)
      //给 Flutter 端发消息
      channel.send("Hello from Android") { reply ->
          println("Received reply: $reply")
      }
    }
}
```