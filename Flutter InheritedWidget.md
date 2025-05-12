# Flutter InheritedWidget
- Inherited 继承的，InheritedWidget 是一个特殊的 Widget，Flutter 的 Theme 和 Locale 功能就是通过 InheritedWidget 实现主题和语言环境的动态切换
- Flutter 中的 Widget 树是单向数据流（父 Widget 向子 Widget 传递数据），如果需要数据跨越多层 Widget 传递时，那么就需要逐层手动传递参数，这样就会导致代码繁琐且难以维护，而 InheritedWidget 允许子 Widget 直接从祖先 Widget 获取数据，无需逐层手动传递（解决多层 Widget 间的数据传递问题，避免逐层手动传递的繁琐，通过使用 InheritedWidget 可以高效地共享数据，同时又保持代码的整洁）
- InheritedWidget 将数据存储在自身属性中（自定义 InheritedWidget 提供共享数据），并提供 of 静态方法让子孙 Widget 调用以获取数据，通常需要将 InheritedWidget 放置在 Widget 树的较高位置（比如放置在 Widget 树的根部，实现全应用访问），以确保子孙 Widget 可以访问到它（如果访问不到的话 of 方法会抛出异常）
- 子 Widget 通过调用 of 方法执行 dependOnInheritedWidgetOfExactType 方法获取最近的 InheritedWidget 实例并建立依赖（Flutter 会自动记录依赖关系，会将子 Widget 自身注册为依赖者）
- InheritedWidget 通过 updateShouldNotify 方法控制是否通知依赖此 InheritedWidget 的子 Widget 进行重建（若返回 true，则触发子 Widget 的 didChangeDependencies 方法）
- 性能优化：仅影响依赖此 InheritedWidget 数据的子 Widget，而非整个 Widget 树，避免不必要的全局重建

## InheritedWidget 使用流程
1 创建自定义 InheritedWidget 提供状态管理
```dart
class AppSharedDataInherited extends InheritedWidget {
  //提供共享数据
  final Color primaryColor;
  final double fontSize;

  const AppSharedDataInherited({
    super.key,
    required this.primaryColor,
    required this.fontSize,
    required Widget child,
  }) : super(child: child);

  @override
  bool updateShouldNotify(AppSharedDataInherited oldWidget) {
    //返回 true 时，所有依赖此 InheritedWidget 的子 Widget 会触发 didChangeDependencies 并重建
    return oldWidget.primaryColor != primaryColor || oldWidget.fontSize != fontSize; //数据变化决定是否通知依赖的子 Widget 重建
  }

  //提供 of 静态方法，方便子 Widget 访问数据
  static AppSharedDataInherited? of(BuildContext context) {
    //通过 dependOnInheritedWidgetOfExactType 获取最近的 InheritedWidget 实例，并建立依赖关系（子 Widget 会被注册为依赖此 InheritedWidget）
    return context.dependOnInheritedWidgetOfExactType<AppSharedDataInherited>();
  }
}
```

2 在 Widget 树中使用 InheritedWidget
```dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  Color _primaryColor = Colors.green;
  double _fontSize = 12;

  void changeParam(Color newColor) {
    setState(() {
      _primaryColor = _fontSize % 2 == 0 ? newColor : Colors.red;
      _fontSize = _fontSize + 3;
    });
  }

  @override
  Widget build(BuildContext context) {
    //用自定义 InheritedWidget 包裹子 Widget
    return AppSharedDataInherited(
        primaryColor: _primaryColor,
        fontSize: _fontSize,
        child: MaterialApp(
          home: Scaffold(
            appBar: AppBar(title: const Text('InheritedWidget 示例')),
            body: const MyBody(), 
            floatingActionButton: FloatingActionButton(
              onPressed: () => changeParam(Colors.blue), //改变值
              child: const Icon(Icons.add),
            ),
          ),
        ));
  }
}

```

3 子 Widget 获取状态数据
```dart
class MyBody extends StatelessWidget {
  const MyBody({super.key});

  @override
  Widget build(BuildContext context) {
    //通过 of 方法获取父 Widget 共享的状态数据
    final appSharedDataInherited = AppSharedDataInherited.of(context);
    final primaryColor = appSharedDataInherited?.primaryColor;
    final fontSize = appSharedDataInherited?.fontSize;
    return Center(
      child: Text(
        '子 Text 的颜色和文字大小来自 AppSharedDataInherited',
        style: TextStyle(color: primaryColor, fontSize: fontSize),
      ),
    );
  }
}
```

## 总结
- InheritedWidget 是 Flutter 中用于在 Widget 树中进行高效共享和传递数据的核心组件，适合需要跨层级传递数据的场景（比如用户登录状态、应用状态、全局配置等），Flutter 中内置的 Theme 主题和 Locale 国际化功能均基于 InheritedWidget 实现
- 同时 InheritedWidget 也是作为 Provider、Riverpod（增强改进版的 Provider） 等高级状态管理工具的基础
- InheritedWidget 适合简单的数据共享场景，对于复杂场景，推荐使用 Provider、Riverpod 和 Bloc 等状态管理工具
