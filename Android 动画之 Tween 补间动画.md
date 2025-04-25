# Android 动画之 Tween 补间动画
- 也被称为 View 视图动画，补间可以理解成在两个关键帧之间自动插入额外的帧以创建平滑过渡效果的过程，就是指两个关键帧定义了动画的起始和结束状态，由系统自动生成中间帧，从而形成连续流畅的动画效果的过程
- 补间动画不会改变视图的实际属性，只是改变了视图在视觉上呈现动画效果，所以不适合用在有事件交互的场景
- 补间动画可以通过 Java 代码或者 xml 文件来实现，定义动画的 xml 文件通常存放在 res/anim 目录下，通过 AnimationUtils#loadAnimation 加载文件

## TranslateAnimation 平移动画
- 改变视图的位置，实现视图平移的功能

```java
//其中 Delta 代表差值  
//Type 参数可以取值：Animation.ABSOLUTE 绝对值（默认）、Animation.RELATIVE_TO_SELF 相对于控件自身和 Animation.RELATIVE_TO_PARENT 相对于父控件
//(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)
TranslateAnimation translateAnimation = new TranslateAnimation(0.0F, 100.0F, 0.0F, 0.0F); //view 向右移动 100 个像素
//(int fromXType, float fromXValue, int toXType, float toXValue, int fromYType, float fromYValue, int toYType, float toYValue)
TranslateAnimation translateAnimation = new TranslateAnimation(Animation.RELATIVE_TO_SELF, -1.0F, Animation.RELATIVE_TO_SELF, 1.0F, 
    Animation.RELATIVE_TO_SELF, -1.0F, Animation.RELATIVE_TO_SELF, 1.0F); //假设从左上到右下 3 个 view 区域（区域 2 是原始位置），view 就是从区域 1 移动到区域 3
TranslateAnimation translateAnimation = new TranslateAnimation(Animation.RELATIVE_TO_PARENT, -0.5F, Animation.RELATIVE_TO_PARENT, 0.5F, 
    Animation.RELATIVE_TO_PARENT, 0.0F, Animation.RELATIVE_TO_PARENT, 0.0F); //-0.5F 代表从原始位置左上角往左方向离父控件的一半距离，0.5F 代表从原始位置左上角往右方向离父控件的一半距离

translateAnimation.setDuration(1000); //动画持续 1000 毫秒的时间
translateAnimation.setFillAfter(true); //动画结束后是否保持结束状态

translateAnimation.setStartOffset(0); //动画延迟开始的时间
translateAnimation.setRepeatCount(0); //动画重复次数（默认 0 不重复，Animation#INFINITE 代表无限重复）
translateAnimation.setRepeatMode(Animation.RESTART); //动画重复播放模式，Animation#RESTART 代表正序重放，Animation#REVERSE 代表倒序回放
translateAnimation.setInterpolator(AccelerateDecelerateInterpolator()); //先加速后减速（默认插值器）
//
view.startAnimation(translateAnimation);
```
动画定义文件 res/anim/view_translate.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:fromXDelta="0"
    android:toXDelta="100"
    android:fromYDelta="0"
    android:toYDelta="100"
    android:duration="1000" />
```

```java
Animation animation = AnimationUtils.loadAnimation(context, R.anim.view_translate);
view.startAnimation(animation);
```

## ScaleAnimation 缩放动画
- 改变视图的尺寸大小，可以指定缩放中心点以及设置 X 和 Y 方向上的缩放比例

```java
//(float fromX, float toX, float fromY, float toY)
ScaleAnimation scaleAnimation = new ScaleAnimation(1.0F, 0.5F, 1.0F, 0.5F); //view 缩小到原始位置左上角的四分之一（其实此时 pivotX 和 pivotY 两个都是 0）
//(float fromX, float toX, float fromY, float toY, float pivotX, float pivotY)
ScaleAnimation scaleAnimation = new ScaleAnimation(1.0F, 0.5F, 1.0F, 0.5F, 40.0F, 40.0F); //pivotX 和 pivotY 就是缩放中心点的坐标
//(float fromX, float toX, float fromY, float toY, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
ScaleAnimation scaleAnimation = new ScaleAnimation(1.0F, 0.5F, 1.0F, 0.5F, 
    Animation.RELATIVE_TO_SELF, 1.0F, Animation.RELATIVE_TO_SELF, 1.0F); //view 缩小到原始位置右下角的四分之一

scaleAnimation.setDuration(1000);
scaleAnimation.setFillAfter(true);
```

```xml
<!-- 50 具体数值代表像素，50% 百分比代表相对于控件自身的比例，50%p 百分比加 p 代表相对于父控件的比例 -->
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXScale="1.0"
    android:toXScale="0.5"
    android:fromYScale="1.0"
    android:toYScale="0.5"
    android:pivotX="50%" 
    android:pivotY="50%"
    android:duration="1000" />
```

## RotateAnimation 旋转动画
- 旋转视图，可以指定旋转中心点以及旋转的角度

```java
//(float fromDegrees, float toDegrees) 
RotateAnimation rotateAnimation = new RotateAnimation(0.0F, 360.0F); //view 绕着左上角顶点顺时针转圈（其实此时 pivotX 和 pivotY 两个都是 0）
//(float fromDegrees, float toDegrees, float pivotX, float pivotY)
RotateAnimation rotateAnimation = new RotateAnimation(0.0F, 360.0F, 40.0F, 0.0F); //view 绕着左上角顶点偏移 40 个像素的点顺时针转圈
//(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
RotateAnimation rotateAnimation = new RotateAnimation(0.0F, 360.0F, Animation.RELATIVE_TO_SELF, 0.5F, Animation.RELATIVE_TO_SELF, 0.5F); //view 绕着视图旋转中心点顺时针转圈

rotateAnimation.setDuration(1000);
rotateAnimation.setFillAfter(true);
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromDegrees="0"
    android:toDegrees="360"
    android:pivotX="50%"
    android:pivotY="50%"
    android:duration="1000" />
```

## AlphaAnimation 透明度动画
- 改变视图的透明度，比如实现淡入淡出的效果

```java
AlphaAnimation alphaAnimation = new AlphaAnimation(1.0F, 0.0F); //从完全不透明到完全透明

alphaAnimation.setDuration(1000);
alphaAnimation.setFillAfter(true);
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromAlpha="1.0"
    android:toAlpha="0.0"
    android:duration="1000" />
```

## AnimationSet
- 将多个动画组合在一起播放
```java
AnimationSet animationSet = new AnimationSet(true); //参数 shareInterpolator 表示是否统一使用 AnimationSet 共享的插值器
//
animationSet.addAnimation(translateAnim);
animationSet.addAnimation(scaleAnim);
animationSet.addAnimation(rotateAnim);
animationSet.addAnimation(alphaAnim);
//
view.startAnimation(animationSet);
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:duration="3000"
    android:fillAfter="true">
    <alpha
        android:fromAlpha="1.0"
        android:toAlpha="0.1" />
    <translate
        android:fromXDelta="0.0"
        android:toXDelta="100"
        android:fromYDelta="0.0"
        android:toYDelta="0.0" />
    <scale
        android:fromXScale="1.0"
        android:toXScale="0.5"
        android:fromYScale="1.0"
        android:toYScale="0.5" />
    <rotate
        android:fromDegrees="0.0"
        android:toDegrees="360"
        android:pivotX="50%"
        android:pivotY="50%" />
</set>
```

## Interpolator 插值器
- 用于控制动画的播放速度
- AccelerateDecelerateInterpolator 先加速后减速（默认插值器）
- LinearInterpolator 线性插值器，动画匀速播放
- AccelerateInterpolator 加速插值器，动画开始时速度较慢，逐渐加快
- DecelerateInterpolator 减速插值器，动画开始时速度较快，逐渐减慢

```java
animation.setInterpolator(new AccelerateInterpolator());
```

## AnimationListener 动画监听器
```java
animation.setAnimationListener(new Animation.AnimationListener() {
            @Override
            public void onAnimationStart(Animation animation) {
                //动画开始时执行的逻辑
            }

            @Override
            public void onAnimationEnd(Animation animation) {
                //动画结束时执行的逻辑
            }

            @Override
            public void onAnimationRepeat(Animation animation) {
                //动画重复时执行的逻辑
            }
        });
```