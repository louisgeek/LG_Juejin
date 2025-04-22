# Flutter 动画之 Explicit 显式动画
- Explicit 显式动画指的是通过手动配置和控制动画过程来实现动画效果的方式，提供对动画状态（比如正向播放、反向播放和重复播放等）的精细化管理，与隐式动画（由 Flutter 自动管理动画过程）相对应
- 显式动画的核心在于使用 AnimationController 和 Animation
- 显式动画可以精细控制，灵活性较高

## AnimationController
- AnimationController 继承自 Animation，动画控制器，控制着动画的播放、暂停等状态，管理动画的 duration 持续时间和 vsync（TickerProvider）参数
- 可以监听动画的状态变化（比如如是否完成、是否前进或后退等）

## Animation
- Animation 是一个抽象类，代表动画的值随时间的变化，比如通过 Tween#animate 方法绑定到 AnimationController 生成 Animation
- Tween 子类支持颜色、尺寸和区域等多种类型（比如 ColorTween、SizeTween 和 RectTween）
- Tween 定义了动画的 begin 起始值和 end 结束值（比如 0.0 -> 1.0），Tween 补间由系统自动生成中间值

## 使用 addListener 和 setState 的方式
- 1 初始化动画控制器：在 State 对象的 initState 方法中创建 AnimationController 实例，并指定动画的 duration 持续时间和 vsync（TickerProvider）参数，记得在 dispose 方法中进行释放
- 2 定义动画值的变化范围：使用 Tween 等类定义动画值的起始值和结束值（比如使用 Tween<double> 定义一个从 0.0 到 1.0 的变化范围）
- 3 监听动画值的变化：通过 Animation#addListener 方法添加监听器，在监听器中调用 setState 方法来触发 UI 界面重绘，从而更新界面以反映动画元素的位置、大小等属性的改变
- 4 启动和控制动画：调用 AnimationController 的 forward、reverse 或 repeat 等方法来启动正向播放、反向播放或循环播放
```dart
class ExplicitAnimationWidget extends StatefulWidget {
  const ExplicitAnimationWidget({super.key});

  @override
  State<ExplicitAnimationWidget> createState() =>
      _ExplicitAnimationWidgetState();
}

//SingleTickerProviderStateMixin 实现了 TickerProvider 抽象类，通过混入让 State 成为一个 TickerProvider
class _ExplicitAnimationWidgetState extends State<ExplicitAnimationWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    //1 初始化动画控制器
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this, //TickerProvider，将 State 传入
    );
    //2 定义动画值的变化范围
    //Animation<Color?> _colorAnimation = ColorTween(begin: Colors.red, end: Colors.green).animate(_controller);
    _animation = Tween<double>(begin: 50.0, end: 200.0).animate(_controller);

    //3 监听动画值的变化
    _animation.addListener(() {
      setState(() {}); //触发 UI 更新
    });
    _animation.addStatusListener((status) {
      //监听状态变化
      //if (_controller.isCompleted) {
      if (status == AnimationStatus.completed) {
        //动画完成时触发
        _controller.reverse(); //反向播放
      }
    });
    
    _startAnimation();
  }

  //4 启动和控制动画
  void _startAnimation() {
    _controller.forward(); //正向播放
    //_controller.repeat(); //循环播放
  }

  @override
  void dispose() {
    //释放资源
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('显式动画示例'),
      ),
      body: Center(
        child: SizedBox(
          width: _animation.value, //动画元素的属性改变
          height: _animation.value, //动画元素的属性改变
          child: const FlutterLogo(),
        ),
      ),
    );
  }
}
```

## 使用 AnimatedWidget 组件
- 其实 AnimatedWidget 内部就是封装 addListener 和 setState 的方式，相当于使用 AnimatedWidget 可以省略监听的逻辑（无需手动调用 addListener 和 setState），将动画逻辑与 UI 组件解耦，方便复用动画
- 1 初始化动画控制器：在 State 对象的 initState 方法中创建 AnimationController 实例，并指定动画的 duration 持续时间和 vsync（TickerProvider）参数，记得在 dispose 方法中进行释放
- 2 定义动画值的变化范围：使用 Tween 等类定义动画值的起始值和结束值（比如使用 Tween<double> 定义一个从 0.0 到 1.0 的变化范围）
- 3 启动和控制动画：调用 AnimationController 的 forward、reverse 或 repeat 等方法来启动正向播放、反向播放或循环播放
```dart
class MyAnimatedWidget extends StatefulWidget {
  const MyAnimatedWidget({super.key});

  @override
  State<MyAnimatedWidget> createState() => _MyAnimatedWidgetState();
}

//SingleTickerProviderStateMixin 实现了 TickerProvider 抽象类，通过混入让 State 成为一个 TickerProvider
class _MyAnimatedWidgetState extends State<MyAnimatedWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    //1 初始化动画控制器
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this, //TickerProvider，将 State 传入
    );
    //2 定义动画值的变化范围
    //Animation<Color?> _colorAnimation = ColorTween(begin: Colors.red, end: Colors.green).animate(_controller);
    _animation = Tween<double>(begin: 50.0, end: 200.0).animate(_controller);

    _animation.addStatusListener((status) {
      //监听状态变化
      //if (_controller.isCompleted) {
      if (status == AnimationStatus.completed) {
        //动画完成时触发
        _controller.reverse(); //反向播放
      }
    });

    _startAnimation();
  }

  //3 启动和控制动画
  void _startAnimation() {
    _controller.forward(); //正向播放
    //_controller.repeat(); //循环播放
  }

  @override
  void dispose() {
    //释放资源
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('AnimatedWidget 示例'),
      ),
      body: Center(
        child: CustomAnimatedWidget(animation: _animation),
      ),
    );
  }
}

//自定义 AnimatedWidget
class CustomAnimatedWidget extends AnimatedWidget {
  const CustomAnimatedWidget({super.key, required Animation<double> animation})
      : super(listenable: animation);

  @override
  Widget build(BuildContext context) {
    final animation = listenable as Animation<double>;
    return SizedBox(
      width: animation.value, //动画元素的属性改变
      height: animation.value, //动画元素的属性改变
      child: const FlutterLogo(),
    );
  }
}
```

## 使用以 Transition 结尾的组件
- 系统内置的显式动画
- 1 初始化动画控制器：在 State 对象的 initState 方法中创建 AnimationController 实例，并指定动画的 duration 持续时间和 vsync（TickerProvider）参数，记得在 dispose 方法中进行释放
- 2 定义动画值的变化范围：使用 Tween 等类定义动画值的起始值和结束值（比如使用 Tween<double> 定义一个从 0.0 到 1.0 的变化范围）
- 3 启动和控制动画：调用 AnimationController 的 forward、reverse 或 repeat 等方法来启动正向播放、反向播放或循环播放
```dart
//SlideTransition 继承自 AnimatedWidget
//ScaleTransition 继承自 MatrixTransition，而 MatrixTransition 又继承自 AnimatedWidget
//RotationTransition 继承自 MatrixTransition，而 MatrixTransition 又继承自 AnimatedWidget
//FadeTransition 继承自 SingleChildRenderObjectWidget，而 SingleChildRenderObjectWidget 又继承自 RenderObjectWidget
//
//平移动画，通过 position 控制位置
SlideTransition
//缩放动画，通过 scale 控制缩放
ScaleTransition
//旋转动画，通过 turns 控制旋转次数
RotationTransition
//透明度动画（比如实现淡入淡出效果），通过 opacity 控制不透明度
FadeTransition
//...
```

```dart
//SingleTickerProviderStateMixin 实现了 TickerProvider 抽象类，通过混入让 State 成为一个 TickerProvider
class _ScaleTransitionWidgetState extends State<ScaleTransitionWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    //1 初始化动画控制器
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this, //TickerProvider，将 State 传入
    );
    //2 定义动画值的变化范围
    _animation = Tween<double>(begin: 0.4, end: 1.0).animate(_controller);

    _animation.addStatusListener((status) {
      //监听状态变化
      //if (_controller.isCompleted) {
      if (status == AnimationStatus.completed) {
        //动画完成时触发
        _controller.reverse(); //反向播放
      }
    });

    _startAnimation();
  }

  //3 启动和控制动画
  void _startAnimation() {
    _controller.forward(); //正向播放
    //_controller.repeat(); //循环播放
  }

  @override
  void dispose() {
    //释放资源
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ScaleTransition 示例'),
      ),
      body: Center(
        child: ScaleTransition(
          scale: _animation,
          child: const SizedBox(
            width: 100,
            height: 100,
            child: FlutterLogo(),
          ),
        ),
      ),
    );
  }
}
```
