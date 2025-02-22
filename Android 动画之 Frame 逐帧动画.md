# Android 动画之 Frame 逐帧动画
- 简称帧动画，是一种通过按顺序快速播放一系列图像（通常称为帧）来形成动画效果的技术，类似于传统动画制作或者电影放映的原理
- 如果包含大量高分辨率的图像可能会导致内存占用过高，可能会导致应用性能下降，所以可以根据实际情况合理压缩图像的大小和控制帧数
- 帧动画可以通过 Java 代码或者 xml 文件来实现，定义动画的 xml 文件通常存放在 res/drawable 目录下
- 另外需要注意 AnimationDrawable#start 和 AnimationDrawable#stop 方法必须在 UI 主线程中调用
##

## Java 代码方式
```xml
<ImageView
    android:id="@+id/imageView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

```java
ImageView imageView = findViewById(R.id.imageView);
//
AnimationDrawable animationDrawable = new AnimationDrawable();
//设为 true 表示动画播放完后停止，设为 false 代表会一直循环播放
animationDrawable.setOneShot(false)
//添加每一帧的图像资源，duration 指定该帧显示的时长
animationDrawable.addFrame(ResourcesCompat.getDrawable(resources,R.drawable.frame1), 100);
animationDrawable.addFrame(ResourcesCompat.getDrawable(resources,R.drawable.frame2), 100);
animationDrawable.addFrame(ResourcesCompat.getDrawable(resources,R.drawable.frame3), 100);
animationDrawable.addFrame(ResourcesCompat.getDrawable(resources,R.drawable.frame3), 100);
//...
imageView.setImageDrawable(animationDrawable);
//或者 imageView.setBackground(animationDrawable);
//
animationDrawable.start(); //开始播放
//animationDrawable.stop(); //停止播放
```

## xml 文件方式
```xml
<!-- frame_animation.xml -->
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item android:drawable="@drawable/frame1" android:duration="100" />
    <item android:drawable="@drawable/frame2" android:duration="100" />
    <item android:drawable="@drawable/frame3" android:duration="100" />
    <item android:drawable="@drawable/frame4" android:duration="100" />
</animation-list>
```

src
```xml
<ImageView
    android:id="@+id/imageView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/frame_animation" />
```

```java
ImageView imageView = findViewById(R.id.imageView);
//
AnimationDrawable animationDrawable = (AnimationDrawable) imageView.getDrawable();
//
animationDrawable.start(); //开始播放
//animationDrawable.stop(); //停止播放
```

background
```xml
<ImageView
    android:id="@+id/imageView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="@drawable/frame_animation" />
```

```java
ImageView imageView = findViewById(R.id.imageView);
//
AnimationDrawable animationDrawable = (AnimationDrawable) imageView.getBackground();
//
animationDrawable.start(); //开始播放
//animationDrawable.stop(); //停止播放
```

