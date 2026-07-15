# Flutter autoDispose、keepAlive 和 ref.keepAlive 的区别
- 注解方式是新的写法，Xxx.autoDispose 是传统的写法
- @riverpod 默认就是传统的 autoDispose 自动销毁机制，无订阅就自动销毁，页面关闭就自动销毁（@riverpod 就是 @Riverpod()）
- @Riverpod(keepAlive: true) 永远不自动销毁，需手动 invalidate 进行销毁
- ref.keepAlive 临时阻止自动销毁，可通过 link#close 恢复自动销毁（也可以用 invalidate 强制立即销毁）

## autoDispose 自动销毁
```dart
//显式加上 .autoDispose 修饰符, 当所有监听者（watch/listen）离开时，Provider 自动销毁，就是引用计数（监听者数量）归零时（简单的说就是 widget 销毁、页面关闭时自动销毁）
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  return fetchUser();
});
```
对应
```dart
//动态生成 userProvider
@riverpod //@riverpod 默认就是 autoDispose，而 @riverpod 就是 Riverpod(keepAlive = false, ...)
Future<User> user(UserRef ref) async {
  return fetchUser();
}
```

## keepAlive
```dart
//不带 autoDispose，全局常驻，这个 Provider 的数据一直保留在内存里（简单的说就是 widget 销毁、页面离开时都不会自动销毁，需要通过调用 invalidate 手动销毁）
final userProvider = FutureProvider<User>((ref) async {
  return fetchUser();
});
```
对应
```dart
//动态生成 userProvider
@Riverpod(keepAlive: true, ...) //keepAlive: true，永久关闭 autoDispose 机制，无论是否有被监听，数据永远不自动销毁
Future<User> user(UserRef ref) async {
  return fetchUser();
}
```
keepAlive: true 的典型场景（全局单例、用户信息、应用配置、主题等）
```dart
//用户登录信息，App 全局唯一且一直需要保留
@Riverpod(keepAlive: true)
class AuthNotifier extends _$AuthNotifier {
  @override
  User? build() => null;

  void login(User user) => state = user;
  void logout() => state = null;
}
```

## ref.keepAlive
- 是 Ref 类的方法，返回一个 KeepAliveLink，临时阻止（挂起） autoDispose 销毁数据（只对开启了 autoDispose 机制的 Provider 有用，本质上 ref.keepAlive 就是给 autoDispose 机制加一个手动控制的开关）
- 必须在 provider 的 build 方法中调用
- 一旦调用就永久保活，不过可通过 KeepAliveLink#close 撤销（就是恢复 provider 的 autoDispose 正常销毁逻辑）
- 即使没有监听者，只要 keepAlive 的 link 没有 close，provider 就不会被销毁（就和 keepAlive: true 完全等价）
- 常配合 ref.onCancel（无监听时触发）、ref.onResume（重新有监听时触发）一起使用

条件缓存：异步成功后才缓存
```dart
//官方推荐写法
@riverpod
Future<Data> fetchData(FetchDataRef ref) async {
  try {
    final data = await api.getData();
    ref.keepAlive(); //成功才阻止（阻止 autoDispose 销毁数据）
    return data;
  } catch (e) {
    //失败不阻止，下次重试重建
    rethrow;
  }
}
```

带过期时间的缓存：到时间后恢复自动销毁
```dart
@riverpod
Future<Data> fetchData(FetchDataRef ref) async {
  final link = ref.keepAlive(); //阻止 autoDispose 销毁数据
  //5分钟后关闭，恢复 autoDispose 自动销毁机制（无论成功失败都定时释放）
  Timer(const Duration(minutes: 5), link.close);
  return api.getData();
}
```

配合使用：成功才缓存 + 缓存5分钟后过期
```dart
@riverpod
Future<Data> fetchData(FetchDataRef ref) async {
  try {
    final data = await api.getData();
    final link = ref.keepAlive(); //成功才阻止（阻止 autoDispose 销毁数据）
    //5分钟后关闭，恢复 autoDispose 自动销毁机制
    Timer(const Duration(minutes: 5), link.close);
    return data;
  } catch (e) {
    //失败不阻止，下次重试重建
    rethrow;
  }
}
```

无人监听后定时再释放
```dart
@riverpod
Future<ItemDetail> itemDetail(Ref ref, String itemId) async {
  final link = ref.keepAlive();
  Timer? timer;
  ref.onDispose(() {
    //资源销毁
    timer?.cancel();
  });
  ref.onCancel(() {
    // 离开页面后 30 秒无人用，释放缓存
    timer = Timer(const Duration(seconds: 30), link.close);
  });
  ref.onResume(() {
    // 重新进入页面，取消释放计划
    timer?.cancel();
  });
  return api.fetchItemDetail(itemId);
}
```

## keepAlive: true 和 ref.keepAlive 同时设置
- keepAlive: true 优先级最高，使 ref.keepAlive() 变得无意义（因为 keepAlive: true 直接把 autoDispose 机制禁用了，而 ref.keepAlive 仅在 autoDispose 开启时有效）

## Family Provider
- 强烈建议 Family Provider 不用 keepAlive: true，而是应优先用 ref.keepAlive 动态控制
- 当使用 Family 传递参数时，Riverpod 会为每一个不同的参数（id） 创建一个独立的 Provider 实例
- 用 keepAlive: true 会让所有实例永久驻留内存
- 用 ref.keepAlive + onCancel 可以做到"用完缓存一段时间，超时自动释放"
```dart
// 搜索过的多个个实例全部永久驻留内存，这样就造成实例无限堆积的
@Riverpod(keepAlive: true)
Future<List<Item>> searchResult(SearchResultRef ref, String keyword) async {
  return api.search(keyword);
}

// ref.keepAlive + onCancel 的好处
// 离开搜索结果页 30 秒后自动释放，按需缓存、超时释放，而不是无限堆积实例
@riverpod
Future<List<Item>> searchResult(SearchResultRef ref, String keyword) async {
  final link = ref.keepAlive();
  ref.onCancel(() {
    Timer(const Duration(seconds: 30), link.close);
  });
  return api.search(keyword);
}
```

## 总结
- 个人认为 keepAlive: true 如果叫 autoDispose: false 就更好理解和记忆了，或者 ref.keepAlive 改叫 ref.pauseAutoDispose 或 ref.suspendAutoDispose 更清晰合适
- @riverpod 无订阅监听就销毁
- 用户登录信息、应用全局配置： `@Riverpod(keepAlive: true)` 永远不销毁
- 列表页数据、Family Provider： `@riverpod + ref.keepAlive` 动态设置临时关闭 autoDispose 自动释放数据机制，等到页面关掉后自动释放
- 成功后才缓存 `@riverpod + 按条件执行 ref.keepAlive`
- 定时过期缓存 `@riverpod + Timer 按时间执行 link.close`