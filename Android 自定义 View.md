# Android 自定义 View
- 三种类型：组合现有控件、继承现有控件和直接继承 View 或 ViewGroup

## 基本流程
- 1 继承 View 或其子类
- 2 构造方法：至少实现 3 个构造方法，可选通过 TypedArray 获取 attrs.xml 中的自定义属性
- 3 重写 onMeasure 方法：测量 View 的大小，通常必须处理 wrap_content 模式的尺寸计算（否则默认填充父布局）
- 4 重写 onSizeChanged（可选），确定 View 的大小
- 5 重写 onLayout（仅 ViewGroup）：布局子 View，为每个子 View 调用 View#layout 方法
- 6 重写 onDraw 方法：绘制 View 的内容
- 7 处理触摸事件：重写 onTouchEvent 方法并按需返回 true 消费事件

```java
public class MyView extends View {
    public MyView(Context context) {
        super(context); //代码中 View 实例化调用
    }

    public MyView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs); //xml 中 View 实例化调用 
    }

    public MyView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public MyView(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }
}
```

## MeasureSpec 测量规格
- View.MeasureSpec 封装了 Mode 和 Size，Mode 分为 EXACTLY、AT_MOST 和 UNSPECIFIED
- EXACTLY 父控件精确指定子控件的尺寸大小，match_parent 或精确值
- AT_MOST 父控件指定子控件最大尺寸的上限，wrap_content
- UNSPECIFIED 父控件不限制子控件尺寸大小，通常在 ScrollView 中使用
```java
//MeasureSpec 包装 Mode 和 Size  
//int 类型 32 位，2 位存 mode，30 位存 size
int measureSpec = MeasureSpec.makeMeasureSpec(int size, int mode)
```

## Measure 测量流程
- View 测量：ViewRootImpl#performMeasure -> View#measure -> View#onMeasure -> View#setMeasuredDimension
```java
//View#onMeasure
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    //onMeasure 默认的实现逻辑
    //最终需要调用 setMeasuredDimension 设置测量后的尺寸
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
//View#getDefaultSize 计算控件的尺寸大小
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec); //得到测量的模式
    int specSize = MeasureSpec.getSize(measureSpec); //得到具体的数值
    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST: //最大尺寸是父控件的尺寸
    case MeasureSpec.EXACTLY: //精确尺寸（父控件确切指定）
        result = specSize;
        break;
    }
    return result;
}
```

- ViewGroup 测量：ViewRootImpl#performMeasure -> View#measure（比如 AbsoluteLayout）-> View#onMeasure（AbsoluteLayout 重写） -> ViewGroup#measureChildren 遍历测量子控件 -> ViewGroup#measureChild -> View#measure（子控件）-> View#onMeasure -> View#setMeasuredDimension
- 先测量子控件的尺寸大小，最后测量自身的尺寸大小
```java
//ViewGroup#measureChildren
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}
//ViewGroup#measureChild
protected void measureChild(View child, int parentWidthMeasureSpec,
        int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();
    //子控件的大小由父控件的 MeasureSpec 和子控件自身的 LayoutParams 共同决定
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);
    //View#measure
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

## Layout 布局流程
- View 布局：ViewRootImpl#performLayout -> View#layout -> View#onMeasure（可能会调用）-> View#onLayout
- 确认控件的坐标位置
- 先确定自身的位置，然后确定子控件位置
```java
//View#onLayout
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    //空实现
}
```

- ViewGroup 布局：ViewRootImpl#performLayout -> ViewGroup#layout -> View#layout（super.layout）-> View#onMeasure（可能会调用）-> View#onLayout
```java
//ViewGroup#layout
@Override
public final void layout(int l, int t, int r, int b) {
    if (!mSuppressLayout && (mTransition == null || !mTransition.isChangingLayout())) {
        if (mTransition != null) {
            mTransition.layoutChange(this);
        }
        //View#layout
        super.layout(l, t, r, b);
    } else {
        // record the fact that we noop'd it; request layout when transition finishes
        mLayoutCalledWhileSuppressed = true;
    }
}
```

## Draw 绘制流程
- ViewRootImpl#performDraw -> View#draw -> View#drawBackground -> View#onDraw -> View#dispatchDraw -> onDrawForeground
```java
//View#onDraw
protected void onDraw(@NonNull Canvas canvas) {
    //空实现
}
```


## onSizeChanged
- 确定 View 的大小，在 View 的大小发生改变后才会执行（屏幕旋转导致 View 所在的布局空间尺寸变化等场景），可以根据新的宽高重新调整绘制的一些布局、重新初始化与尺寸相关的资源等操作
- 既然 onMeasure 已经测量过 View 的大小了，那为啥还要确定一次？：因为 measure 测量是 View 自身的尺寸大小，而 View 能展现的大小还受父控件的影响