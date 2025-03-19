# Android Glide
- Glide 可以进行图片缓存，还支持 Gif、WebP、缩略图，甚至是 Video
- 绑定生命周期：可以使加载图片的生命周期动态管理起来
- 高效的缓存策略：支持内存、Disk 缓存，Glide 缓存支持多种规格， Glide 可以根据 ImageView 的大小来缓存相应大小的图片尺寸
- 内存开销小：默认的 Bitmap 格式是 RGB_565 格式的

Glide 会自动根据 ImageView 的大小来决定加载图片的大小

## 缓存
内存缓存、磁盘缓存、网络

## 优化 Glide 加载性能
- 使用缓存（内存缓存 + 磁盘缓存）减少重复加载
- 控制图片尺寸，避免加载过大图片
- 及时取消未完成的任务，减少资源浪费


## 内存缓存
- skipMemoryCache(true) //true 关闭内存缓存
- 内存缓存由活动缓存（弱引用缓存）和 LruCache 内存缓存组成
- 弱引用缓存：弱引用的HashMap
- LruCache：使用 LinkedHashMap 实现 LruCache，根据 Lru 算法来管理图片。最近使用过的图片文件会被插入到链表的头部，而长时间未使用的图片则会被移除。


弱引用缓存、LruCache缓存、DiskLruCache缓存
其目的是减少流量消耗，加快响应速度，并减少 Bitmap 的创建销毁所导致的内存占用。以下是内存缓存原理概述：

弱引用：Glide使用弱引用来维护缓存，即key是缓存的key（由图片url、width、height等参数组成），value是图片资源对象的弱引用形式。


## 磁盘缓存
- diskCacheStrategy(DiskCacheStrategy.ALL)
- 使用 DiskLruCache 来实现磁盘缓存



内存缓存动态扩容
setMemoryCache(DynamicLruCache(context))  


采用BitmapPool复用内存


##
Glide 内存缓存如何控制大小
如果自己去实现图片库，怎么做？



##### 写个图片浏览器，说出你的思路？



```java
Glide.with(holder.id_iv.getContext())
                .load(dataItem.imageUrl)
                .override(150, 100)
                .into(new SimpleTarget<GlideDrawable>() {
                    @Override
                    public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
                        holder.id_iv.setImageDrawable(resource);
                    }
                });
```

 
 
线程切换

 
格式解码

 
图像大小

快速滑动



RequestManager

RequestManagerFragment

- SupportRequestManagerFragment



```
public class SupportRequestManagerFragment extends android.support.v4.app.Fragment
```

```
public class RequestManagerFragment extends android.app.Fragment
```



RequestManagerRetriever







1 Glide 如何关联生命周期的？

Glide#with 方法内会创建并添加一个 Fragment ，通过在其内部进行主动回调各个生命周期的方式实现监听，关联的 FragmentRequestManager 就能感知生命周期了，然后它里面由把事件传递给了 TargetTracker 

TargetTracker 统一管理所有 Target

Glide 加载图片的时候，最终调用的 into(ImageView view) 其实就是封装了一个 ImageViewTarget，ImageViewTarget 继承 ViewTarget，它又继承 BaseTarget ，然后 BaseTarget 实现了 Target 接口

