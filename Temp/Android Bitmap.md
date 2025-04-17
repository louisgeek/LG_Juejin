# Android Bitmap
Android Bitmap 内存大小，注意事项，如何优化
Android Bitmap 内存计算规则

## Bitmap 格式
Bitmap.Config.ALPHA_8: 只存储透明度，不存储色值，一个像素占 8 位（1 个字节）
Bitmap.Config.ARGB_4444: ARGB 各用 4 位存储，一个像素占 16 位（2个字节）
Bitmap.Config.ARGB_8888: ARGB 各用 8 位存储，一个像素占 32 位（4个字节）
Bitmap.Config.RGB_565: 只存储色值，不存储透明度，默认不透明，RGB 分别占 5 位，6 位，5 位，一个像素占 16 位（2个字节）

## Bitmap 内存计算
- 占用内存 = 像素总数量 * 每个像素占用的字节
- 像素总数量 = 宽度像素 * 高度像素就是 width * height
- 占用内存 = width * height * 每个像素占用的字节

### 图片大小计算
Bitmap.Config.ARGB_8888 每个像素的 ARGB 共占用 4 个字节：占用内存 = width * height * 4
1280 * 720 分辨率 ARGB_8888 格式的图片：1280 * 720 * 4 = 3686400 Byte（3.52 MB）

## 质量压缩和尺寸压缩
可以通过设置 decode() 方法的 Options 的 inJustDecodeBounds 字段为 false 来实现。真正加载的时候再设置其为 true. 这样进行采样的时候就进行了第一次压缩，叫做邻近采样
通过 BitmapFactory.Options 设置 inSampleSize 压缩图片，减少内存占用，从源头降低回收压力
对得到的 Bitmap 实例调用 compress() 方法，并指定一个图片的质量，通常是在 0~100 之间

## 大图加载
使用 BitmapRegionDecoder 来部分加载图片  在控件的 onDraw() 方法中进行绘制，然后使用 GestureDetector 来检查手势，当移动图片的时候调用 invalid() 方法进行重绘即可







## Bitmap 内存
- Bitmap 在内存中存储分为：Java Bitmap 对象、Native Bitmap 对象和对应的 pixels 像素数据，像素数据占用内存最大
- Android 3（API 11）之前，像素数据存在 Native 内存里，必须要手动调用 recycle 来释放像素数据
- Android 3 ~ 7 中像素数据存在 Java 堆内存里，GC 会自动回收未引用的 Bitmap，如果手动调用 recycle 方法，像素数据不会立即释放（时机会延后一点）
- Android 8（API 26）中像素数据重新存在 Native 内存里，通过引用机制（NativeAllocationRegistry，利用虚引用感知 Java 对象被回收的时机，来回收 Native 内存）辅助回收确保 Bitmap 对象被垃圾回收时，其底层 Native 内存也会被回收，如果手动调用 recycle 方法，像素数据会立即释放
- 官方推荐：从 Android 8.0（API 26）开始，平台会在 Bitmap 不再被引用时自动释放其内存，因此通常不需要调用 recycle
- 使用 Glide 等库处理图片加载，无需手动回收
- 不过处理大图等场景，还是可以考虑手动调用 recycle 以避免内存压力


