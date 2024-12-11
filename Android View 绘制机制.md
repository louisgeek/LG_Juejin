# Android View 绘制机制
- View 的绘制机制包括 Measure 测量、Layout 布局和 Draw 绘制三个阶段
- ViewRootImpl 来统一协调管理整个视图树的绘制流程

```java
//视图绘制的起点
ViewRootImpl#scheduleTraversals
-ViewRootImpl#doTraversal
--ViewRootImpl#performTraversals
---ViewRootImpl#performMeasure
---ViewRootImpl#performLayout
---ViewRootImpl#performDraw
```

## ViewRootImpl
- ViewRootImpl 是 Android 负责管理整个 View 层级结构的核心枢纽，是视图树和系统底层服务（比如 WindowManagerService）的桥梁，每个 Activity 的 Window 都对应一个 ViewRootImpl 实例（将 ViewRootImpl 和 DecorView 建立关联），负责处理窗口相关的各种事务（比如窗口的添加、移除和布局变更等）以及管理视图树的绘制和交互事件的分发
    - 视图树的绘制：负责驱动视图树完成 Measure 测量、Layout 布局和 Draw 绘制三大流程，确保界面能够正确呈现在屏幕上，绘制流程是从 DecorView 根视图开始，然后递归地遍历整个 View 树，在遍历过程中依次对每个 View 执行测量、布局和绘制三大操作
    - 事件的分发：处理来自系统的输入事件分发和生命周期变化


## Measure 测量
- 这个阶段的目的就是确定 View 和其子 View 各自所期望的合适的大小尺寸
- 在这个阶段中系统会遍历整个视图树，从根视图开始依次逐个测量每个 View 的尺寸，去调用 View 的 measure 测量方法
- DecorView 根视图的测量由 ViewRootImpl 发起，根据 Window 的大小等因素构建出初始的 MeasureSpec 传递给根视图
- 对于后续的 ViewGroup 来说，需要遍历自身的子 View，依据自身的布局规则（比如 LinearLayout 和 RelativeLayout 等的布局规则）为每个子 View 构建相应的 MeasureSpec 并传递下去，并要求子 View 进行自我测量，最后根据所有子 View 的大小来最终确定 ViewGroup 自身的大小
- 而对于 View 来说，会根据自身的内容（比如 TextView 根据文本内容长度以及结合父容器约束等情况）计算出自己期望的尺寸，并将结果通过 setMeasuredDimension 方法保存起来，完成自身的测量
- ViewGroup 提供 measureChildren 和 measureChild 方法用于测量子 View，measureChildren 就是 for 循环遍历所有的子 View 传入 measureChild 方法，内部再调用 View 的 measure 方法，除非有比较特殊的需求，大部分的场景都是可以直接使用这两个方法去测量

performMeasure -> View#measure -> View#onMeasure
```java
//MeasureSpec 测量规格，两个 MeasureSpec 参数包含了父 View 对子 View 测量的尺寸要求，用于指导 View 实际该如何测量自己
ViewRootImpl#performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec)
-View#measure(int widthMeasureSpec, int heightMeasureSpec)
--View#onMeasure(int widthMeasureSpec, int heightMeasureSpec)
//View#onMeasure
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    //子 View 会根据父容器的规格要求和自身的布局参数（如 LayoutParams）来计算出自己的宽高，并通过 View#setMeasuredDimension 方法保存计算结果
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

MeasureSpec 
- View 静态内部类 MeasureSpec 对应一个 32 位 int 值，高 2 位代表 specMode 测量模式，低 30 位代表 specSize 测量大小

MeasureSpec specMode 测量模式
- MeasureSpec.AT_MOST：父 View 不允许子 View 的大小超过自身的最大值，通常对应 layout_width 或 layout_height 设置成 wrap_content 的情况
- MeasureSpec.EXACTLY：父 View 明确指定了子 View 的精确大小，对应 layout_width 或 layout_height 设置为具体的数值或者是 match_parent 的情况
- MeasureSpec.UNSPECIFIED：父 View 对子 View 不作任何限制，子 View 可以是任意大小，一般用于系统内部的测量过程


## Layout 布局
- 这个阶段的目的就是根据测量阶段得到的大小尺寸来精确地确定每个 View 在父容器中应该显示（摆放）的位置，也就是坐标位置
- 同样从根视图开始，ViewGroup 会遍历子 View，根据子 View 的测量宽高结合自身的布局规则逻辑去调用子 View 的 layout 方法，传入相应的参数来最终确定子 View 在父容器中的具体摆放位置，对于 ViewGroup 来说必须重写 onLayout 方法来实现布局子 View 的逻辑
- 对于 View 而言，普通的 View 通常不需要重写 onLayout 方法


performLayout -> View#layout -> View#onLayout
```java
//期望的尺寸
ViewRootImpl#performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth, int desiredWindowHeight)
//View 的左上角和右下角的坐标
-View#layout(int left, int top, int right, int bottom)
--View#onLayout(boolean changed, int left, int top, int right, int bottom)
```


## Draw 绘制
- 这个阶段的目的就是在布局阶段流程完成后，根据位置、大小和自身属性等将 View 的内容绘制到屏幕上，绘制过程包括绘制背景、绘制内容、绘制子 View 和绘制装饰（如指示器和滚动条等）
- 同样从根视图开始，ViewGroup 会遍历子 View，去调用 View 的 draw 方法来将 View 的内容绘制到屏幕上，而 draw 方法会调用 onDraw 方法，通常需要重写完成自定义 View 的绘制逻辑，通常 ViewGroup 不需要重写 onDraw 方法，因为作为容器没有自己的内容需要进行显示
- 对于 ViewGroup 来说，还需要调用 dispatchDraw 方法来发起其各个子 View 的绘制

performDraw -> View#draw -> View#onDraw
```java
ViewRootImpl#performDraw()
-View#draw(@NonNull Canvas canvas)
--View#drawBackground(@NonNull Canvas canvas) //绘制背景
--View#onDraw(@NonNull Canvas canvas) //绘制视图本身，必须重写来实现自身的绘制逻辑
--View#dispatchDraw(@NonNull Canvas canvas) //绘制子视图
--View#onDrawForeground(@NonNull Canvas canvas) //绘制前景
```