# Flutter Riverpod 
Riverpod 是 Flutter 的状态管理/依赖注入方案（Riverpod 非常适合依赖注入，例如可以轻松提供 ApiService、Repository等）
Riverpod是 Flutter/Dart 的响应式状态管理框架，由 Provider 的作者开发，解决了传统 Provider 的诸多痛点。
无 Context 依赖	可在任意位置读取状态，无需 BuildContext
自动 dispose	不再手动管理监听和资源释放

Provider：声明“一个值如何被创建”（状态/依赖） 
ref核心工具：读状态、改状态、监听状态
ChangeNotifierProvider：兼容旧代码，通常不推荐在新项目中使用
FutureProvider 异步数据获取 / StreamProvider监听数据流：处理异步数据。
使用 ConsumerWidget(整个 Widget 都依赖状态)
使用 Consumer(只有部分 Widget 依赖状态)
ConsumerStatefulWidget 
// 重置计数器为初始值 ref.invalidate(counterProvider);？？？？？


```dart
void main() {
  //runApp(const MyApp());
  runApp(
    //设置 ProviderScope 为根 Widget，所有 Provider 状态都存储在这里（用 ProviderScope 包裹整个应用，作为状态容器）
    ProviderScope(  //ProviderScope 继承自 StatefulWidget
      child: MyApp(),
    ),
  );
}
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MyHomePage(),
    );
  }
}
```

```dart
//定义 StateProvider
final counterStateProvider = StateProvider<int>((ref) {
  return 0; //初始值为 0
});
//class CounterWidget extends StatelessWidget {
class CounterWidget extends ConsumerWidget { //使用 ConsumerWidget 代替 StatelessWidget（让这个 Widget 具备"读取和监听 Provider"的能力）
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    //watch 监听订阅状态，当值发生改变时，会触发重建依赖它的 Widget（这里就是 CounterWidget），而重建就会重新走 build 方法，而本次 build 执行时，ref.watch 就拿到了最新的数据，最终用新的数据渲染出新的 UI
    final count = ref.watch(counterStateProvider);
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          //点击修改数据：用 read 一次性读取值（不订阅监听），必须通过 notifier 获取控制器，再通过控制器修改状态，state 就是存的值
          onPressed: () => ref.read(counterStateProvider.notifier).state++,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

## 读写
- ref.watch(xxxProvider)：订阅监听状态变化（在 build 中或在其他 Provider 内使用，实现 UI 根据自动刷新），简单的说就是：1 取消上次的监听（如果不是第一次），2 设置监听，3 监听到值发生改变时，通知系统重建依赖它的 Widget
- ref.read(xxxProvider)：一次性读取值或调用其方法，不订阅监听（通常在事件回调中使用，比如按钮点击等事件）
- ref.read(xxxProvider.notifier)：获取控制器（通常是一个 StateController），用于修改状态，修改后自动通知所有 watch 了它的 Widget
- ref.listen(xxxProvider, (prev, next) {})：监听状态变化，通常用于执行副作用（比如导航、显示 SnackBar）

## Provider 提供只读值（不可变）
- 适合：配置、Repository、计算属性（由其他 Provider 推导）  固定 / 计算后的只读数据 todo
```dart
//创建一个只读 Provider，它提供数字 0
final numProvider = Provider<int>((ref) => 0);
//返回固定字符串
final helloProvider = Provider<String>((ref) => "Hello");
final baseUrlProvider = Provider<String>((ref) => "https://api.example.com");
//返回对象
final authRemoteDataSourceProvider = Provider<AuthRemoteDataSource>((ref) {
  final dioClient = ref.read(dioClientProvider); //依赖另一个 Provider
  return AuthRemoteDataSource(dioClient.dio);
});
final authRepositoryProvider = Provider(
  (ref) => AuthRepository(ref.read(authRemoteDataSourceProvider)),
);
```

## StateProvider 提供简单可变状态
- 适合：简单计数器、开关状态、当前 tab、筛选条件等（适合简单状态） todo？
```dart
final counterStateProvider = StateProvider<int>((ref) => 0);
//使用
ref.read(counterStateProvider.notifier).state++;
```

## NotifierProvider 提供复杂可变状态（复杂逻辑）
- 将状态和修改状态的逻辑封装在一个 Notifier 类中
- NotifierProvider 是 StateNotifierProvider 的替代品
```dart
//把计数器逻辑改造到 Notifier 中
//定义 Notifier
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;  //build 里返回初始值
  void increment() => state++;
  void decrement() => state--;
}
//定义 NotifierProvider
final counterNotifierProvider = NotifierProvider<CounterNotifier, int>(CounterNotifier.new);
//使用
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterNotifierProvider);
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () => ref.read(counterNotifierProvider.notifier).increment(),
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

```dart
//旧的 StateNotifierProvider 的写法
class CounterStateNotifier extends StateNotifier<int> {
  final Ref ref; //把 ref 传进来才能读其他 provider
  CounterStateNotifier(this.ref) : super(0); //初始值在构造函数
  void increment() => state++;
  void decrement() => state--;
}
final counterStateNotifierProvider = StateNotifierProvider<CounterStateNotifier, int>((ref) => CounterStateNotifier(ref)); //需要传 ref
```

## FutureProvider 异步状态（网络请求、本地读取） 
- 网络请求、数据库读取等异步场景，自带加载中、成功和错误的状态管理（通常 Riverpod 会自动生成和管理），通常被用来替代系统的 FutureBuilder
- 请求刚发出去时，状态会被设置为 AsyncLoading，请求成功返回时，状态会被设置为 AsyncData(data)，请求抛出异常时，状态会被设置为 AsyncError(error, stackTrace)
- 通过 switch 或 when 处理 3 个状态，通过 maybeWhen 处理部分状态
```dart
final asyncData = ref.watch(dataProvider);
return asyncData.when(
  data: (data) => Text(data.name),
  loading: () => const CircularProgressIndicator(),
  error: (e, st) => Text('error: $e'),
);
//return switch (asyncData) {
//  AsyncLoading() => CircularProgressIndicator(),
//  AsyncError(:final error) => Text('出错了: $error'),
//  AsyncData(:final data) => Text('名称: ${data.name}'),
//};

//return asyncData.maybeWhen(
//  data: (data) => Text('名称: ${data.name}'),
//  // 其他情况统一走 orElse
//  orElse: () => CircularProgressIndicator(),
//);
```

```dart
//定义 FutureProvider
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
});
//使用
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);
    print('userAsync.isLoading: ${userAsync.isLoading}');
    print('userAsync.hasValue: ${userAsync.hasValue}');
    print('userAsync.hasError: ${userAsync.hasError}');
    print('userAsync.value: ${userAsync.value}');
    print('userAsync.error: ${userAsync.error}');
    //直接取值方式
    final  user = userAsync.value;
    //分状态处理方式
    return userAsync.when(
      data: (user) => Text('Hello, ${user.name}'),
      loading: () => CircularProgressIndicator(),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}
```





## StreamProvider
// 定义
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return FirebaseFirestore.instance
      .collection('messages')
      .snapshots()
      .map((snap) => snap.docs.map(Message.fromDoc).toList());
});



## 注解

函数式
```dart
//自动生成 helloProvider = Provider<String>
@riverpod
String hello(Ref ref) {
  return 'Hello';
}
```

```dart
//自动生成 userProvider = FutureProvider<User>
@riverpod
Future<User> user(UserRef ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
}
```

类式 Notifier
- 当业务逻辑比较复杂时，使用类来组织代码
```dart
//自动生成 counterProvider, 自动生成 _$Counter（继承自 $Notifier）
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0; //build 里返回初始值
  void increment() => state++;
  void decrement() => state--;
}
//使用
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

需要配置时才用大写
```dart
//@riverpod 等价于 @Riverpod()，内部 keepAlive 参数默认为 false
@Riverpod(keepAlive: true)
int counter(Ref ref) => 0;
```