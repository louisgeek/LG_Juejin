# Flutter 动画之 Implicit 隐式动画
- Implicit 隐式动画指的是通过修改组件的属性自动触发平滑过渡效果的动画实现方式（状态驱动），和显式动画（需要通过 AnimationController 手动控制）不同，隐式动画通过封装底层动画逻辑实现，无需手动控制管理动画
- 隐式动画需要指定动画的开始和结束状态，以及动画的持续时间等参数
- 隐式动画开发效率较高，但灵活性较低（依赖预设属性和默认动画）
- 动画一旦开始，只能从初始状态到目标状态，无法循环或反向播放
- 通过嵌套多个隐式动画组件可实现复合效果（比如同时调整位置和透明度）

```dart
//系统内置的隐式动画（以 Animated 开头的组件，都继承自 ImplicitlyAnimatedWidget）
//动画触发依赖于属性变化
//AnimatedContainer：动态改变容器属性（比如宽高、颜色和对齐方式等）
//AnimatedDefaultTextStyle：控制文本样式过渡
//AnimatedSlide​：控制滑动
//AnimatedScale​：控制缩放
//AnimatedRotation​：控制旋转
//AnimatedOpacity​​：控制透明度渐变
//AnimatedPositioned​​：在 Stack 中平滑移动子组件（必须配合 Stack 使用）
//AnimatedSwitcher：组件切换时添加过渡动画（比如淡入淡出）

//AnimatedIcon：图标动画
//AnimatedTheme：主题切换时的平滑过渡
//AnimatedPadding：动态控制边距
//AnimatedAlign：动态调整子组件对齐方式
```

## AnimatedContainer
```dart
class MyAnimatedContainer extends StatefulWidget {
  const MyAnimatedContainer({super.key});

  @override
  State<MyAnimatedContainer> createState() => _MyAnimatedContainerState();
}

class _MyAnimatedContainerState extends State<MyAnimatedContainer> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _flag = !_flag),
        child: const Icon(Icons.animation),
      ),
      body: Center(
        child: AnimatedContainer(
            duration: const Duration(milliseconds: 500),
            width: _flag ? 100 : 200,
            height: _flag ? 100 : 200,
            color: _flag ? Colors.blue : Colors.red,
            alignment: _flag ? Alignment.topLeft : Alignment.bottomRight,
            child: const Text('test AnimatedContainer', textDirection: TextDirection.rtl)),
      ),
    );
  }
}
```

## AnimatedDefaultTextStyle
```dart
class MyAnimatedDefaultTextStyle extends StatefulWidget {
  const MyAnimatedDefaultTextStyle({super.key});

  @override
  State<MyAnimatedDefaultTextStyle> createState() =>
      _MyAnimatedDefaultTextStyleState();
}

class _MyAnimatedDefaultTextStyleState
    extends State<MyAnimatedDefaultTextStyle> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag = !_flag),
          child: const Icon(Icons.animation),
        ),
        body: Center(
            child: AnimatedDefaultTextStyle(
          style: _flag
              ? const TextStyle(fontSize: 24)
              : const TextStyle(fontSize: 16),
          duration: const Duration(milliseconds: 500),
          child: const Text('test AnimatedDefaultTextStyle'),
        )));
  }
}
```

## AnimatedSlide
```dart
class MyAnimatedSlide extends StatefulWidget {
  const MyAnimatedSlide({super.key});

  @override
  State<MyAnimatedSlide> createState() => _MyAnimatedSlideState();
}

class _MyAnimatedSlideState extends State<MyAnimatedSlide> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag = !_flag),
          child: const Icon(Icons.animation),
        ),
        body: Center(
            child: AnimatedSlide(
                offset: _flag ? const Offset(1.0, 0.0) : const Offset(0.0, 0.0),
                duration: const Duration(milliseconds: 500),
                child: Container(
                  width: 100,
                  height: 100,
                  color: Colors.blue,
                  child: const Center(
                    child: Text('test AnimatedSlide'),
                  ),
                ))));
  }
}
```

## AnimatedScale
```dart
class MyAnimatedScale extends StatefulWidget {
  const MyAnimatedScale({super.key});

  @override
  State<MyAnimatedScale> createState() => _MyAnimatedScaleState();
}

class _MyAnimatedScaleState extends State<MyAnimatedScale> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag = !_flag),
          child: const Icon(Icons.animation),
        ),
        body: Center(
            child: AnimatedScale(
          scale: _flag ? 3.0 : 1.5,
          duration: const Duration(milliseconds: 500),
          child: Container(
            width: 100,
            height: 100,
            color: Colors.blue,
            child: const Center(
              child: Text('test AnimatedScale'),
            ),
          ),
        )));
  }
}
```

## AnimatedRotation
```dart
class MyAnimatedRotation extends StatefulWidget {
  const MyAnimatedRotation({super.key});

  @override
  State<MyAnimatedRotation> createState() => _MyAnimatedRotationState();
}

class _MyAnimatedRotationState extends State<MyAnimatedRotation> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag = !_flag),
          child: const Icon(Icons.animation),
        ),
        body: Center(
            child: AnimatedRotation(
          turns: _flag ? 1 : 0, //旋转的圈数
          duration: const Duration(seconds: 1),
          child: const Icon(
            Icons.refresh,
            size: 200,
            color: Colors.blue,
          ),
        )));
  }
}
```

## AnimatedOpacity​​
```dart
class MyAnimatedOpacity extends StatefulWidget {
  const MyAnimatedOpacity({super.key});

  @override
  State<MyAnimatedOpacity> createState() => _MyAnimatedOpacityState();
}

class _MyAnimatedOpacityState extends State<MyAnimatedOpacity> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag = !_flag),
          child: const Icon(Icons.animation),
        ),
        body: Center(
            child: AnimatedOpacity(
          opacity: _flag ? 1.0 : 0.2,
          duration: const Duration(milliseconds: 500),
          child: const Text('test AnimatedOpacity​​ Fade In/Out'),
        )));
  }
}
```

## AnimatedPositioned
```dart
class MyAnimatedPositioned extends StatefulWidget {
  const MyAnimatedPositioned({super.key});

  @override
  State<MyAnimatedPositioned> createState() => _MyAnimatedPositionedState();
}

class _MyAnimatedPositionedState extends State<MyAnimatedPositioned> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag = !_flag),
          child: const Icon(Icons.animation),
        ),
        body: Center(
          child: Stack(
            children: [
              const Positioned(
                top: 100,
                left: 100,
                child: Text('固定位置文本', style: TextStyle(fontSize: 20)),
              ),
              AnimatedPositioned(
                duration: const Duration(milliseconds: 500),
                top: _flag ? 100 : 300,
                left: _flag ? 100 : 300,
                child: Container(
                  width: 100,
                  height: 100,
                  color: Colors.blue,
                ),
              ),
              const Positioned(
                top: 300,
                left: 300,
                child: Text('固定位置文本222', style: TextStyle(fontSize: 20)),
              ),
            ],
          ),
        ));
  }
}
```

## AnimatedSwitcher
```dart
class MyAnimatedSwitcher extends StatefulWidget {
  const MyAnimatedSwitcher({super.key});

  @override
  State<MyAnimatedSwitcher> createState() => _MyAnimatedSwitcherState();
}

class _MyAnimatedSwitcherState extends State<MyAnimatedSwitcher> {
  int _flag = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag++),
          child: const Icon(Icons.animation),
        ),
        body: Center(
          child: AnimatedSwitcher(
            duration: const Duration(milliseconds: 500),
            transitionBuilder: (Widget child, Animation<double> animation) {
              return FadeTransition(
                //淡入淡出
                opacity: animation,
                child: ScaleTransition(
                  //缩放
                  scale: animation,
                  child: child,
                ),
              );
            },
            child: Text(
              'test AnimatedSwitcher $_flag',
              //key: UniqueKey(), //生成唯一 key
              key: ValueKey(_flag), //必须设置唯一 key
            ),
          ),
        ));
  }
}
```

## TweenAnimationBuilder 补间动画构建器
- TweenAnimationBuilder 继承自 ImplicitlyAnimatedWidget
- 侧重使用 Tween 来定义动画，​​允许自定义任意类型的补间动画​​，比系统内置的隐式动画要灵活
- 支持复杂动画逻辑（比如颜色渐变、路径动画和多属性同步动画等）
```dart
class MyTweenAnimationBuilder extends StatefulWidget {
  const MyTweenAnimationBuilder({super.key});

  @override
  State<MyTweenAnimationBuilder> createState() => _MyTweenAnimationBuilderState();
}

class _MyTweenAnimationBuilderState extends State<MyTweenAnimationBuilder> {
  bool _flag = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () => setState(() => _flag = !_flag),
          child: const Icon(Icons.animation),
        ),
        body: Center(
          child: TweenAnimationBuilder<double>(
            tween: Tween<double>(begin: 10, end: 20),
            duration: const Duration(seconds: 5),
            builder: (context, value, child) {
              return Text("test TweenAnimationBuilder ${value.toInt()}",
                  style: TextStyle(fontSize: value));
            },
          ),
        ));
  }
}
```

## 自定义隐式动画
- 直接继承 ImplicitlyAnimatedWidget
- ImplicitlyAnimatedWidget 内部对 AnimationController 进行了封装，无需手动控制动画控制器
```dart
class MyImplicitlyAnimatedWidget extends StatefulWidget {
  const MyImplicitlyAnimatedWidget({super.key});

  @override
  State<MyImplicitlyAnimatedWidget> createState() =>
      _MyImplicitlyAnimatedWidgetState();
}

class _MyImplicitlyAnimatedWidgetState
    extends State<MyImplicitlyAnimatedWidget> {
  double _width = 100;
  double _height = 100;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() {
          _width = _width > 40 ? 40 : 100;
          _height = _height > 40 ? 40 : 100;
        }),
        child: const Icon(Icons.animation),
      ),
      body: Center(
          child: CustomImplicitlyAnimatedWidget(
        width: _width,
        height: _height,
        duration: const Duration(seconds: 1),
      )),
    );
  }
}

//自定义 ImplicitlyAnimatedWidget
class CustomImplicitlyAnimatedWidget extends ImplicitlyAnimatedWidget {
  const CustomImplicitlyAnimatedWidget(
      {super.key,
      required super.duration,
      required this.width,
      required this.height});

  final double width;
  final double height;

  @override
  AnimatedWidgetBaseState<CustomImplicitlyAnimatedWidget> createState() =>
      _CustomAnimatedWidgetBaseState();
}

//AnimatedWidgetBaseState 继承自 ImplicitlyAnimatedWidgetState
class _CustomAnimatedWidgetBaseState
    extends AnimatedWidgetBaseState<CustomImplicitlyAnimatedWidget> {
  Tween<double>? _widthTween;
  Tween<double>? _heightTween;

  @override
  void forEachTween(TweenVisitor<dynamic> visitor) {
    //visitor 的 tween 代表当前的 tween，第一次调用为 null
    //初始化每一个可变化属性的 tween
    //更新每一个可变化属性的 tween
    //可同时管理多个属性，定义属性初始值及变化逻辑
    _widthTween = visitor(_widthTween, widget.width, (dynamic value) => Tween<double>(begin: value as double)) as Tween<double>?;

    _heightTween = visitor(_heightTween, widget.height, (dynamic value) => Tween<double>(begin: value as double)) as Tween<double>?;
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      width: _widthTween!.evaluate(animation),
      height: _heightTween!.evaluate(animation),
      color: Colors.blue,
    );
  }
}
```






