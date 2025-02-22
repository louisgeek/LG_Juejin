# Android 动画之 Property 属性动画
- 属性动画能对对象的任意属性（比如位置、大小、颜色、透明度等）进行动画处理，对象的属性会从初始状态逐渐变为结束状态，这个过程中会创建多个连续的中间状态，逐帧播放后就会形成属性渐变效果，从而产生平滑的过渡效果
- 属性动画是通过赋值改变对象的属性值来实现动画效果的，所以可以进行事件交互，相对于补间动画来说属性动画的功能更加强大和灵活，属性动画的核心思想就是通过反射技术操作属性的 getter 和 setter 方法，如果需要对自定义属性实现动画效果的话，就需要保证自定义属性对应的 getter 和 setter 方法是可被访问的
- 属性动画可以通过 Java 代码或者 xml 文件来实现，定义动画的 xml 文件通常存放在 res/animator 目录下，通过 AnimatorInflater#loadAnimator 加载文件


## ValueAnimator 
- ValueAnimator 可以用于生成一系列的动画数值变化，但不会直接应用到 view 对象的属性上，而是需要通过监听 ValueAnimator 的更新事件监听器然后在数据回调里进行手动应用

```java
ValueAnimator valueAnimator = ValueAnimator.ofFloat(0f, 100f);
//
valueAnimator.setDuration(1000); //设置动画时长
valueAnimator.setStartDelay(0); //动画延迟开始的时间
valueAnimator.setRepeatCount(0); //动画重复次数（默认 0 不重复，ValueAnimator#INFINITE 代表无限重复）
valueAnimator.setRepeatMode(ValueAnimator.RESTART); //动画重复播放模式，ValueAnimator#RESTART 代表正序重放，ValueAnimator#REVERSE 代表倒序回放
valueAnimator.setInterpolator(AccelerateDecelerateInterpolator()) //先加速后减速（默认插值器）
//更新事件监听器
valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        //value 从 0 逐步递增到 100
        float value = (float) animation.getAnimatedValue();
        //手动应用变更属性
        view.setTranslationX(value);
    }
});
//
valueAnimator.start();
```


## ObjectAnimator
- ObjectAnimator 继承自 ValueAnimator，可以将计算出来的动画数值直接应用到目标对象的指定属性上，专门用于对 view 对象的特定属性进行动画处理

```java
//translationX 和 translationY 是在 X 和 Y 轴上的平移属性，rotation 是旋转属性，scaleX 和 scaleY 是在 X 和 Y 轴上的缩放属性，alpha 是透明度属性
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(view, "translationX", 0.0F, 100.0F);
//
objectAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
objectAnimator.setDuration(1000);
//
objectAnimator.start();
```

```xml
<!-- valueType 可以取值 intType、floatType -->
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:propertyName="translationX"
    android:duration="1000"
    android:valueFrom="0"
    android:valueTo="100"
    android:valueType="floatType"/>
```

```java
Animator animator = AnimatorInflater.loadAnimator(context, R.animator.view_translate);
animator.setTarget(view);
animator.start();
```


## AnimatorSet
- 将多个动画组合在一起播放

```java
ObjectAnimator animator1 = ObjectAnimator.ofFloat(view, "translationX", 0f, 100f);
ObjectAnimator animator2 = ObjectAnimator.ofFloat(view, "translationY", 0f, 100f);
//
AnimatorSet animatorSet = new AnimatorSet();
//
//animatorSet.play(animator1); //添加一个动画播放
animatorSet.playTogether(animator1, animator2); //添加多个动画，同时播放
//animatorSet.playSequentially(animator1, animator2); //添加多个动画，依次播放
//animatorSet.play(animator1).with(animator2); //同时播放两个动画
//animatorSet.play(animator1).before(animator2); //先播放 animator1 动画，再播放 animator2 动画
//animatorSet.play(animator1).after(animator2); //先播放 animator2 动画，再播放 animator1 动画
//
animatorSet.start();
```

```xml
<!-- ordering 设置动画的播放顺序，together 表示同时播放，sequentially 表示依次播放 -->
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:ordering="together">
    <objectAnimator
        android:propertyName="scaleX"
        android:duration="1000"
        android:valueFrom="1.0"
        android:valueTo="2.0"
        android:valueType="floatType" />
    <objectAnimator
        android:propertyName="scaleY"
        android:duration="1000"
        android:valueFrom="1.0"
        android:valueTo="2.0"
        android:valueType="floatType" />
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


## AnimatorListener 动画监听器
```java
animator.addListener(new Animator.AnimatorListener() {
    @Override
    public void onAnimationStart(Animator animation) {
        //动画开始时执行的逻辑
    }

    @Override
    public void onAnimationEnd(Animator animation) {
        //动画结束时执行的逻辑
    }

    @Override
    public void onAnimationCancel(Animator animation) {
        //动画取消时执行的逻辑
    }

    @Override
    public void onAnimationRepeat(Animator animation) {
        //动画重复时执行的逻辑
    }
});
```




