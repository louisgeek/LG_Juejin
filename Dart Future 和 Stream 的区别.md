# Dart Future 和 Stream 的区别

## Future 单次异步结果
- 用于表示一个尚未完成的异步操作的结果，承诺最终会返回一个值（或错误），适用于如网络请求、数据库查询、文件读取和延迟任务等
- Future 无需手动关闭
- Future 的所有 API 的返回值仍然是一个 Future 对象，所以可以很方便的进行链式调用

使用 async 创建 Future
```dart
Future<String> fetchData() async {
  var response = await http.get(Uri.parse('https://api.example.com/data'));
  print(response.body);
  return "Data fetched successfully!";
}
```

通过 then 来使用 Future
```dart
fetchData()
  .then((value) => print("Result: $value")) //成功回调
  .catchError((error) => print("Error: $error")); //失败错误处理
  .whenComplete(() => print("Task done")); //完成回调，不管成功或失败都会执行
```

通过 async 和 await 来使用 Future
```dart
void gainData() async {
  try {
    String data = await fetchData();
    print(data);
  } catch (e) {
    print("Error: $e");
  }
}
```

## Stream 连续异步事件流
- 用于表示一系列异步事件的序列（数据流），可以持续发送多个数据（或错误），适用于如传感器数据、实时推送、键盘输入、轮询请求和定时器等
- Stream 需要手动关闭

使用 StreamController 创建 Stream
```dart
//StreamController 用于管理流的创建、发送数据和关闭
StreamController<String> streamController = StreamController<String>();
Stream<String> dataStream = streamController.stream;
//
streamController.sink.add("Data 1");
streamController.sink.add("Data 2");
```

使用 async* 创建 Stream
```dart
Stream<String> fetchDataStream() async* {
  for (int i = 0; i < 3; i++) {
    await Future.delayed(const Duration(seconds: 1)); //模拟异步耗时
    yield "Data $i"; //生成发送数据
  }
}
```

通过 listen 监听 Stream 的事件
```dart
streamController.stream.listen(
  (data) => print("Received: $data"), //数据回调
  onError: (error) => print("Error: $error"), //错误回调（可选）
  onDone: () => print("Stream completed"), //完成回调（可选）
);
```

```dart
//不再需要发送数据后要手动关闭
streamController.close();
```

## Future 转换为 Stream
```dart
//使用 Stream.fromFuture 将单个 Future 转为流（发送一次数据后完成）
Future<String> futureData = fetchData();
Stream<String> streamData = Stream.fromFuture(futureData);
//转换多个 Future
Stream<String> streamData2 = Stream.fromFutures([futureData,futureData2]);
```

```dart
Stream<String> streamFromFutures(Iterable<Future<String>> futures) async* {
  for (final future in futures) {
    var result = await future;
    yield result; //发送 Future 结果
  }
}
//
Stream<String> streamFromFuture(Future<String> future) async* {
  final result = await future;
  yield result; //发送 Future 结果
  yield "Additional data"; //可追加其他数据
}
```