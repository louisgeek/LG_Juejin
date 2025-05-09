# Dart 单线程模型
- 单线程模型是一种在单个线程中处理所有任务的编程模型，这种设计避免了多线程复杂的锁竞争和同步问题，降低开发复杂度
- Dart 代码（包括 UI 渲染、事件处理、异步任务）默认运行在单一线程（UI 主线程）中，所有代码按顺序执行，尽管 Dart 是单线程的，但它通过引入 Isolate 隔离区（隔离单元）的机制来实现类似多线程的功能
- Isolate 是 Dart 中的独立执行单元，每个 Isolate 都有自己的内存空间，Isolate 之间不能直接共享内存，而是需要通过消息传递（SendPort 和 ReceivePort）来进行通信
- 每个 Isolate 都会维护一个 Event Loop 事件循环来处理异步任务（Isolate 承担了事件循环调度职责），包含两个任务队列，分别是 Microtask Queue 微任务队列和 Event Queue 事件队列

## Isolate 隔离区
- 主 Isolate（UI 主线程）：是应用启动时自动创建的第一个 Isolate，也是默认的执行环境，主 Isolate 的入口是 main 函数，从这里开始执行同步代码，之后进入事件循环处理异步任务（在 Flutter 中，主 Isolate 还负责 UI 渲染和交互处理，所以也可以称为 UI Isolate）
- 子 Isolate：处理耗时操作或独立任务，通过消息传递机制与主 Isolate 解耦，适用于 CPU 计算密集型的耗时任务（比如复杂数学运算、加解密操作、图片编解码处理、图片缩放裁剪旋转、文件读写和 JSON 解析）

## Isolate#spawn 手动创建 Isolate
- 启动新的子 Isolate 并执行一个指定的入口函数（初始函数），需要手动管理生命周期和消息传递，适合长期任务
- 子 Isolate 内异常不会自动传播到主 Isolate，需要手动 try-catch 捕获并通过 SendPort 发送返回异常
```dart
void main() {
  //创建主 Isolate 的 ReceivePort
  final ReceivePort mainReceivePort = ReceivePort();
  //启动子 Isolate，第一个参数指定一个入口函数（必须是一个顶层或静态函数），第二个参数是入口函数的初始参数（通常传入主 Isolate 的 sendPort 方便进行通信）
  Isolate.spawn(isolateFun, mainReceivePort.sendPort);
  //主 Isolate 监听消息
  mainReceivePort.listen((message) {
    if (message is SendPort) {
      print("接收子 Isolate 发送的 SendPort");
      final SendPort childSendPort = message;
      childSendPort.send("给子 Isolate 发送的消息");
    } else {
      print('主 Isolate 收到消息：$message');
    }
  });
  //处理完成后及时关闭 Port 端口
  //mainReceivePort.close();
}
//isolateFun 函数会在新 Isolate 中运行
void isolateFun(SendPort sendPort) {
  //创建子 Isolate 的 ReceivePort
  final childReceivePort = ReceivePort();
  //将子 Isolate 的 SendPort 发送给主 Isolate
  sendPort.send(childReceivePort.sendPort);
  //子 Isolate 监听消息
  childReceivePort.listen((message) {
    if (message is SendPort) {
      print("接收主 Isolate 发送的 SendPort");
      final SendPort mainReceivePort = message;
      mainReceivePort.send("给主 Isolate 发送的消息");
    } else {
      print('子 Isolate 收到消息：$message');
    }
  });
  //处理完成后及时关闭 Port 端口
  //childReceivePort.close();
}
```

## Isolate#run 简化 Isolate 创建
- 用于启动一个新的子 Isolate 执行一个入口函数，并等待其返回结果（立即返回 Future），适合一次性短期任务，会自动处理 Isolate 的创建、通信和销毁，无需手动管理 Port 端口
- Isolate#run 内部也是通过 Isolate#spawn 方式来创建新的 Isolate 的，而且对 Isolate#spawn 已经进行了 try-catch 处理
```dart
void main() async {
  const fileName = 'data.json';
  final jsonData = await Isolate.run(() => _readAndParseJson(fileName));
  print('JSON 数据长度：${jsonData.length}');
}

Map<String, dynamic> _readAndParseJson(String fileName) {
  final fileData = File(fileName).readAsStringSync();
  return jsonDecode(fileData) as Map<String, dynamic>;
}
```

## Flutter 的 compute 函数
- Flutter 提供 compute 函数简化 Isolate 使用，只能在 Flutter 中运行，在 Flutter 中推荐优先使用 compute
- 内部实现：在平台侧通过 Isolate.run 封装，在 Web 侧通过 await null 实现
```dart
int heavyComputation(int input) => input * 2;

Future<int> calculateFun() async {
  final result = await compute(heavyComputation, inputData);
  print('calculateFun result：$result');
}
```

## IsolatePool 池
- 通过使用 IsolatePool 复用 Isolate，避免频繁创建和销毁 Isolate 时的开销，适用于需要重复执行轻量任务的场景

## 双队列事件循环
- Microtask Queue 微任务队列：用于高优先级、轻量的异步后处理，通过 scheduleMicrotask 或 Future#microtask 提交的任务、Future#then 的回调和 await 后续的逻辑，具有最高优先级（比如微任务处理网络请求完成后的 UI 数据更新，确保快速响应）
- Event Queue 事件队列：用于外部事件或延迟任务，Future#delayed、IO 操作（网络请求、文件读写）回调、 Timer 定时器和用户交互（比如 Flutter 的 UI 事件）
- 事件循环每次都会优先清空 Microtask Queue 中的任务，再去处理 Event Queue 中的任务，微任务队列清空后，仅会从事件队列中取出一个任务执行，执行完毕后立即回到微任务队列，检查是否有新的微任务需要处理（比如主 Isolate 中，main 函数的同步代码执行完毕后，事件循环启动 -> 先清空微任务队列 -> 再处理事件队列的一个任务 -> 如此循环往复）
```dart
void main() {
  print('同步代码');
  //加入微任务队列
  scheduleMicrotask(() {
    print("微任务1");
    scheduleMicrotask(() => print("微任务2")); //微任务可链式添加
  });
  Future.microtask(() => print('微任务3')); //加入微任务队列
  Future(() => print('事件任务')).then(() => print('事件任务的微任务')); //回调是微任务
  Future.delayed(Duration.zero, () => print('延迟事件任务')); //加入事件队列
  //
  print('同步代码');
}
```

```dart
Future<void> fetchData() async {
  print('开始请求');
  var response  = await http.get(Uri.parse('https://api.com')); //await 之后的代码包装成微任务，加入微任务队列
  print('处理数据：$response '); //await 后续的逻辑，等价于 .then(() => print(...))
}
```

```dart
Timer(Duration(seconds: 2), () {
  print("延迟事件任务"); //加入事件队列
});
```

## 总结
- Dart 的单线程模型通过 Isolate 隔离区的双队列事件循环机制来实现高效并发处理，在简化并发编程的同时，实现了高效的异步处理和多任务并行，也保证了 UI 流畅性（事件循环确保界面渲染优先，避免卡顿）
- 内存隔离：每个 Isolate 拥有独立内存堆，不共享数据，通过 SendPort ReceivePort 消息传递通信
- Future 适合耗时不超过 16ms 的操作（轻量级异步任务），对于超过 16ms 以上的操作推荐使用 Isolate（复杂耗时操作，可以说 CPU 密集型任务、耗时操作必须要放到子 Isolate）
- PS：Flutter UI 相关逻辑（包括 setState）是在主 Isolate 中执行，无需额外的线程安全处理，不过需保证异步回调中不执行耗时同步操作
- PS：网络请求实际是由 Dart 运行时委托给操作系统内核处理的，网络请求完成后将数据告诉 Dart 层，通过事件循环回调处理结果，因此不会导致卡顿
