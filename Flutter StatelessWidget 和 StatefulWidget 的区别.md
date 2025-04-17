# Flutter StatelessWidget 和 StatefulWidget 的区别
- StatelessWidget 和 StatefulWidget 是构建 UI 用户界面的两种基础组件，主要区别在于状态管理（数据变化）、 ​​UI 更新机制​​和生命周期等方面
- StatelessWidget 适用于静态 UI，而 StatefulWidget 适用于需要动态更新的 UI

## StatelessWidget 无状态组件
- 是一种无状态的、不可变的组件，一旦构建后属性无法修改，不会随时间变化（在下次构建之前都不会改变，需要更新展示内容就得需要重新构建，直接丢弃原有的 Widget 树，重新构建一棵新的  Widget 树）
- 组件的 UI 外观和行为仅由初始的配置参数（构造函数中的参数）决定
- 仅包含 build 方法（UI 由 build 方法描述，根据配置参数构建 UI），每次参数变化时重新构建 UI （build 只会被调用一次，除非父组件触发更新）  
- 无状态管理，性能较好，适合静态内容（比如 Text 文本、Icon 图标和 Container 容器等）

```dart
void main() {
  runApp(const MyText(title: "StatelessWidget 标题文本"));
}
class MyText extends StatelessWidget {
  //配置参数（构造函数中的参数）
  final String title;

  const MyText({super.key, required this.title});

  @override
  Widget build(BuildContext context) {
    //配置参数
    //return Text(title, textDirection: TextDirection.ltr);
    return Center(
      child: Text(title, textDirection: TextDirection.ltr),
    );
  }
}
```

## StatefulWidget 有状态组件
- 是一种有状态的组件（包含一个 State 对象），State 状态可以随时间变化，可以动态修改其状态并触发 UI 重新渲染
- 包括如 createState、initState、build（UI 由 build 方法描述，根据 State 状态构建 UI）、didUpdateWidget 和 dispose 等方法
- 适合需要动态更新的场景（如表单输入、列表刷新、计数器和动画等）

```dart
void main() => runApp(const MyCounter());
class MyCounter extends StatefulWidget {
  const MyCounter({super.key});

  @override
  State<MyCounter> createState() => _MyCounterState();
}
class _MyCounterState extends State<MyCounter> {
  int _counter = 0;

  void _incrementCounter() {
    //调用 setState 标记 UI 需要重新渲染（触发 UI 更新）
    setState(() {
      _counter++;
    });
    //setState(() => _counter++);
  }

  @override
  Widget build(BuildContext context) {
    //
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text("StatefulWidget 标题文本")),
        body: Center(
          child: Column(
            children: [
              Text("MyCounter: $_counter"),
              ElevatedButton(
                onPressed: _incrementCounter, //调用 _incrementCounter 方法
                child: const Text("Increment"),
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void initState() {
    super.initState();
    //组件初始化时调用
    //进行初始化操作（比如加载数据）
  }

  @override
  void dispose() {
    //组件销毁时调用
    //进行释放资源操作（比如取消订阅）
    super.dispose();
  }
}
```

## 总结
- StatelessWidget 只依赖于构造函数中的参数来构建 UI，不维护任何内部状态，在需要更新 UI 时，必须重新构建该 Widget 的实例
- StatefulWidget 维护一个可变的 State 状态，可以响应用户输入或其他事件并更新 UI，当状态发生变化时，通过调用 setState 方法来通知触发 UI 更新