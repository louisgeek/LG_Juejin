# Flutter autoDispose、keepAlive 和 ref.keepAlive 的区别
- 注解方式是新的写法，Xxx.autoDispose 是传统的写法
- @riverpod 默认就是传统的 autoDispose 自动销毁机制，无订阅就自动销毁，页面关闭就自动销毁（就是 @Riverpod()）
- @Riverpod(keepAlive: true) 永远不自动销毁，需手动 invalidate 进行销毁
- ref.keepAlive 临时阻止自动销毁，可通过 link#close 恢复自动销毁

## autoDispose 自动销毁
```dart
//显式加上 .autoDispose 修饰符, 当没有任何一个活跃的 watch 在订阅这个 Provider，就是引用计数（监听者数量）归零时，Provider 数据就被自动回收（简单的说就是 widget 销毁、页面关闭时自动销毁）
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

# keepAlive
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
- 是 Ref 类的方法，返回一个 KeepAliveLink，临时阻止（挂起） autoDispose 销毁数据（只对开启了 autoDispose 机制的 Provider 有用）
- 可通过 KeepAliveLink#close  撤销

列表场景：列表滚动时保留，关掉页面时自动销毁
```dart
@riverpod
Future<Item> listItem(Ref ref) async {
  ref.keepAlive(); //有监听时保留，无监听时自动销毁
}
```

条件缓存：异步成功后才缓存
```dart
@riverpod
Future<Data> fetchData(FetchDataRef ref) async {
  try {
    final data = await api.getData();
    ref.keepAlive(); //只有成功才缓存（阻止 autoDispose 销毁数据）
    return data;
  } catch (e) {
    //失败不缓存，下次重试重建
    rethrow;
  }
}
```

带过期时间的缓存：到时后恢复自动销毁
```dart
@riverpod
Future<Data> fetchData(FetchDataRef ref) async {
  final link = ref.keepAlive(); //阻止 autoDispose 销毁数据
  //5分钟后关闭，恢复 autoDispose 自动销毁机制
  Timer(const Duration(minutes: 5), () => link.close());

  final data = await api.getData();
  return data;
}
```

## keepAlive: true 和 ref.keepAlive 同时设置
- keepAlive: true 优先级最高，会覆盖 ref.keepAlive() 方法（因为 keepAlive: true 直接把 autoDispose 机制禁用了，而 ref.keepAlive 仅在 autoDispose 开启时有效）

## Family Provider
- 强烈建议 Family Provider 不用 keepAlive: true，而是应优先用 ref.keepAlive 动态控制
- 当使用 Family 传递参数时，Riverpod 会为每一个不同的参数（id） 创建一个独立的 Provider 实例
```dart
@riverpod
Future<Item> listItem(ListItemRef ref, int itemId) async {
  ref.keepAlive(); 
  return fetchItem(itemId);
}
```

## 总结
- 个人认为 keepAlive: true 如果叫 autoDispose: false 就更好理解和记忆了，或者 ref.keepAlive 改叫 ref.pauseAutoDispose 或 ref.suspendAutoDispose 更清晰合适
- @riverpod 无订阅监听就销毁
- 用户登录信息、应用全局配置： `@Riverpod(keepAlive: true)` 永远不销毁
- 列表页数据、Family Provider： `@riverpod + ref.keepAlive` 动态设置临时关闭 autoDispose 自动释放数据机制，等到页面关掉后自动释放
- 成功后才缓存 `@riverpod + 按条件执行 ref.keepAlive`
- 定时过期缓存 `@riverpod + Timer 按时间执行 link.close`