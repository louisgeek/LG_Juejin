# Flutter ChangeNotifier 和 ValueNotifier 的区别
- ChangeNotifier 和 ValueNotifier 是两种用于状态管理工具，两者均基于发布-订阅模式设计（采用观察者模式）
- ChangeNotifier 需要自定义状态变量，而 ValueNotifier 可以直接使用内置 value 属性存储状态

## ChangeNotifier
- ChangeNotifier 实现了 Listenable 接口，用于管理复杂状态，需要手动调用 notifyListeners 函数触发通知（通过 addListener、removeListener 添加移除监听器）
- 当值实际发生变化时，可以通过调用 notifyListeners 函数通知所有监听器
- 适用于需要管理多个组件间的多个属性状态或是较为复杂的状态变化逻辑的场景，比如表单验证、多组件联动（用户登录状态发生变化，需要更新多个页面的显示）
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

class CounterNotifierStatefulWidget extends StatefulWidget {
  const CounterNotifierStatefulWidget({super.key});

  @override
  State<CounterNotifierStatefulWidget> createState() => _CounterNotifierStatefulWidgetState();
}

class _CounterNotifierStatefulWidgetState extends State<CounterNotifierStatefulWidget> {
  //
  final CounterNotifier _counterNotifier = CounterNotifier();

  @override
  void initState() {
    super.initState();
    _counterNotifier.addListener(() {
      print('addListener Callback');
      setState(() {}); //触发 UI 更新
    });
  }

  @override
  Widget build(BuildContext context) {
    print('build');
    return Scaffold(
      appBar: AppBar(
        title: const Text('ChangeNotifier 示例'),
      ),
      body: Center(
          child: Text(
        'count: ${_counterNotifier.count}',
      )),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _counterNotifier.increment(),
        child: const Icon(Icons.add),
      ),
    );
  }

  @override
  void dispose() {
    _counterNotifier.dispose();
    super.dispose();
  }
}

```

## ValueNotifier
- ValueNotifier 继承自 ChangeNotifier 并实现了 ValueListenable 接口，专门用于管理单个值的状态，在设置新值时，如果新值与旧值不同，ValueNotifier 内部会自动调用 ChangeNotifier#notifyListeners 函数触发通知
- ValueNotifier 内部维护一个 value 值，可以通过这个 value 直接操作值，然后结合 ValueListenableBuilder 实现 UI 的自动更新
- 适用于多个组件间的单个值的简单状态管理，比如主题切换、计数器
- 局部刷新：利用 ValueListenableBuilder 仅重建依赖该值的组件，避免直接 setState 重建整个页面（ValueListenableBuilder 内部通过 setState 重建自身）
```dart
class ValueNotifierStatefulWidget extends StatefulWidget {
  const ValueNotifierStatefulWidget({Key? key}) : super(key: key);

  @override
  State<ValueNotifierStatefulWidget> createState() => _ValueNotifierStatefulWidgetState();
}

class _ValueNotifierStatefulWidgetState extends State<ValueNotifierStatefulWidget> {
  //
  final ValueNotifier<int> _myValueNotifier = ValueNotifier<int>(0); //初始值为 0
  
  @override
  Widget build(BuildContext context) {
    print('build');
    return Scaffold(
      appBar: AppBar(
        title: const Text('ValueNotifier 示例'),
      ),
      body: Center(
          //ValueListenableBuilder 继承自 StatefulWidget
          child: ValueListenableBuilder<int>(
              valueListenable: _myValueNotifier, //valueListenable 表示一个可监听的数据源
              builder: (BuildContext context, int value, Widget? child) {
                print('builder');
                //builder 只会在 value 值变化时被调用，此处的 value 就是 _myValueNotifier 管理的值发生变化后的新值
                return Text('count: ${_myValueNotifier.value}');
              })),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          _myValueNotifier.value++; //数据变化时触发 ValueListenableBuilder 重新构建
        },
        child: const Icon(Icons.add),
      ),
    );
  }

  @override
  void dispose() {
    _myValueNotifier.dispose();
    super.dispose();
  }
}
```

## 总结
- 单值简单状态优先采用 ValueNotifier，简单直观，多值复杂状态用 ChangeNotifier，更灵活，支持复杂逻辑扩展自定义，更适合复杂状态管理
- 对于多个关联值，应该改用 ChangeNotifier，而非多个 ValueNotifier 实现

