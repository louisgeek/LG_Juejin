# Flutter Provider
- Provider 是 Flutter 官方推荐的状态管理库，基于 InheritedWidget 和 ChangeNotifier 实现跨组件状态共享与响应式更新，是一个轻量级的状态管理解决方案，旨在简化复杂的状态管理流程，从而能够更高效地管理应用中的状态共享逻辑，同时保持代码的可读性和可维护性

## ChangeNotifierProvider
- 状态提供者，继承自 InheritedProvider（InheritProvider 本质上是 InheritWidget 实现的），用于状态数据管理
- 结合 ChangeNotifier 类，当状态变化时通知依赖的 Widget 更新

## Consumer 
- 状态消费者，用于监听状态数据变化并触发 builder 中的 UI 更新
- 支持局部刷新 UI，仅重新构建依赖状态的部分 Widget，避免不必要的重建，推荐缩小 Consumer 范围，仅在需要更新的 Widget 上使用 Consumer，减少开销
- 监听整个数据模型对象（完整状态），任何属性变化都会引起 Consumer 重建（即使属性未被使用）
```dart
//...
Consumer<CounterNotifier>(
  builder: (context, counterNotifier, child) {
    return Text('count: ${counterNotifier.count}');
  },
  child: const Icon(Icons.add), //通过 child 参数传递无需重建的子组件
);
//...
```

## Selector
- 用于对 Provider 提供的数据字段进行选择过滤，通过关注数据模型对象中的部分属性来优化性能（通过 selector 函数筛选指定字段，只有当所选择的部分属性数据改变时，才会触发重新构建）
- Selector 提供了比 Consumer 更精细的控制，支持通过 shouldRebuild 自定义比较逻辑（默认用 == 比较）
```dart
//...
Selector<CounterNotifier, int>(
  selector: (context, counterNotifier) => counterNotifier.count, //假设 CounterNotifier 维护了多个字段，只监听其中的 count 字段
  builder: (context, count, child) {
    return Text('count: $count');
  },
  child: const Icon(Icons.add), //通过 child 参数传递无需重建的子组件
);
//...
```

## Provider.of、context.read 和 context.watch
- 直接获取状态（不监听变化）
```dart
//不监听状态变化，仅获取状态，不会触发当前组件重建
Provider.of<CounterNotifier>(context, listen: false).increment();
//不能在 build 方法内调用
context.read<CounterNotifier>().increment(); //内部就是 Provider.of<T>(this, listen: false)
//主动监听状态变化，当状态变化时触发组件重建（只能在 build 方法内调用），watch 内部本质是 dependOnInheritedWidgetOfExactType 方法的封装
context.watch<CounterNotifier>().increment(); //内部就是 Provider.of<T>(this)
```

## Provider 使用流程
1 创建自定义 ChangeNotifier 提供状态管理
```dart
//Model 用于管理状态并触发通知
class CounterNotifier with ChangeNotifier { //mixin
//class CounterNotifier extends ChangeNotifier {
  int _count = 0; //私有字段

  //提供只读属性
  int get count => _count;

  void increment() {
    _count++; //修改状态
    notifyListeners(); //手动触发更新，通知所有监听者
  }
}
```

2 在 Widget 树中使用 Provider
```dart
void main() {
  runApp(
    //用 Provider 包裹子 Widget
    ChangeNotifierProvider(
      create: (context) => CounterNotifier(), //通过 ChangeNotifierProvider 提供状态
      child: const MyApp(),
    ),
  );
}
```

3 子 Widget 获取状态数据
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        home: Scaffold(
      appBar: AppBar(title: const Text('Provider 示例')),
      body: Center(
        //通过 Consumer 来访问状态数据
        child: Consumer<CounterNotifier>(
          builder: (context, counterNotifier, child) {
            return Text("count: ${counterNotifier.count}");
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          //通过 Provider.of 方法获取实例并获取状态数据
          final counterNotifier = Provider.of<CounterNotifier>(context, listen: false);
          counterNotifier.increment(); //直接调用方法
          print("count: ${counterNotifier.count}"); //直接访问属性
          //内部就是 Provider.of<T>(this, listen: false)
          final counterNotifier2 = context.read<CounterNotifier>();
          print("count2: ${counterNotifier2.count}");
        },
        child: const Icon(Icons.add),
      ),
    ));
  }
}
```