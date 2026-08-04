

## 基础设施 Provider
```dart
//keepAlive 为 true
@Riverpod(keepAlive: true)
SharedPreferences sharedPreferences(Ref ref) {
    // 定义时抛出 UnimplementedError 作为占位
  throw UnimplementedError('Must be overridden in main()');
}

//  sembast: ^3.7.2
final sembastDbProvider = Provider<Database>(
  (ref) => throw UnimplementedError('sembastDbProvider not initialized'),
);
```


```dart
//main.dart
//在 main 方法中初始化
final prefs = await SharedPreferences.getInstance();
final sembastDb = await openSembastDb();

//在 ProviderScope.overrides 中注入
runApp(
  ProviderScope(
    overrides: [
      sharedPreferencesProvider.overrideWithValue(prefs),
      sembastDbProvider.overrideWithValue(sembastDb),
      ...
    ],
    child: ...,
  ),
);
```



## 依赖注入链
- 全部用 ref.watch
```dart
// DataSource 依赖 Dio
@riverpod
AuthRemoteDataSource authRemoteDataSource(Ref ref) {
  // watch：Dio 变化时自动刷新
  final dioClient = ref.watch(dioClientProvider);
  return AuthRemoteDataSource(dioClient);
}

// Repository 依赖 DataSource
@riverpod
AuthRepository authRepository(Ref ref) {
  return AuthRepository(ref.watch(authRemoteDataSourceProvider));
}

// UseCase 依赖 Repository
@riverpod
LoginUseCase loginUseCase(Ref ref) {
  return LoginUseCase(ref.watch(authRepositoryProvider));
}

```


## Notifier
```dart
//watch 驱动 build 自动加载数据
@Riverpod(keepAlive: true)
class AutoLoadNotifier extends _$AutoLoadNotifier {
  @override
  Future<Data?> build() async {
     // 有 watch 了就自动触发
    return _fetch(CachePolicy.cacheFirst);
  }

  Future<void> refresh(CachePolicy policy) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => _fetch(policy));
  }

  Future<Data?> _fetch(CachePolicy policy) async {
    final result = await ref.read(someUseCaseProvider).call(id, policy);
    switch (result) {
      case Success(:final data):
        return data;
      case Failure(:final code, :final msg):
        LogUtil.w(_tag, '_fetch failed: $code $msg');
        return null;
    }
  }
}
```


```dart
//外部事件驱动，build 只给初始值
@Riverpod(keepAlive: true)
class ManualLoadNotifier extends _$ManualLoadNotifier {
  @override
  Future<Data?> build(String id) async {
    return null; //初始值
  }

  Future<void> refresh(CachePolicy policy) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => _fetch(policy));
  }

  Future<Data?> _fetch(CachePolicy policy) async {
    final result = await ref.read(someUseCaseProvider).call(id, policy);
    switch (result) {
      case Success(:final data):
        return data;
      case Failure(:final code, :final msg):
        LogUtil.w(_tag, '_fetch failed: $code $msg');
        return null;
    }
  }

}

```



## Effect Stream 监听
- 一次性 Effect 监听也可以塞进 State 字段 + ref.listen 消费后 clearEffect 来处理（但 clearEffect 很容易遗忘），更推荐下面这种方式
```dart
//绑定 State 生命周期，且在 dispose 里统一 cancel（而不是 Provider 生命周期）
//主要是解决了 autoDispose 导致可能会丢事件和在 build 中调用会重复订阅这两个问题
mixin EffectListenMixin<T extends ConsumerStatefulWidget> on ConsumerState<T> {
  final List<StreamSubscription> _effectSubscriptionList = [];
  final Set<Stream<dynamic>> _registeredStreams = {};

  /// 在 build() 里调用，可安全地被多次重建触发，反复调用而不重复订阅
  /// - 同一个 Stream 只会订阅一次（按 Stream 实例去重），避免重复注册导致 effect 被处理多次
  /// - 若底层 Notifier 被重建产生新的 Stream（新实例），会自动补充订阅新的 Stream
  void listenEffectInBuild<E>(Stream<E> stream, void Function(E) action) {
    if (_registeredStreams.contains(stream)) return;
    _registeredStreams.add(stream);
    //
    _effectSubscriptionList.add(
      stream.listen((event) {
        if (!mounted) return;
        action(event);
      }),
    );
  }

  @override
  void dispose() {
    for (final effectSubscription in _effectSubscriptionList) {
      effectSubscription.cancel();
    }
    super.dispose();
  }
}


//firmware_update_page.dart 
class _FirmwareUpdatePageState extends ConsumerState<FirmwareUpdatePage> with EffectListenMixin {

  @override
  Widget build(BuildContext context) {
    //...
    // 监听一次性副作用
    listenEffectInBuild(ref.read(firmwareUpgradeProvider(widget.deviceId).notifier).effectStream, _handleEffect);
    //...
    }
 }

//firmware_upgrade_provider.dart
@Riverpod(keepAlive: true)
class FirmwareUpgradeNotifier extends _$FirmwareUpgradeNotifier {
  //...
  final _effectController = StreamController<FirmwareUpgradeEffect>.broadcast();

  Stream<FirmwareUpgradeEffect> get effectStream => _effectController.stream;
  //...   
}
```


## keepWatchProvider 页面级保活
- 自定义扩展 ref.keepWatchProvider 来强化语义

```dart
// core/extensions/widget_ref_extension.dart
extension WidgetRefExtension on WidgetRef {
  /// watch 持有 Provider 不被 autoDispose 释放
  void keepWatchProvider(ProviderListenable provider) {
    watch(provider);
  }
}

//语义上比直接写 ref.watch(xxx) 更清楚——这里不用返回值，只是为了保活
ref.keepWatchProvider(videoPlayerSessionProvider(_playbackSessionId));

```