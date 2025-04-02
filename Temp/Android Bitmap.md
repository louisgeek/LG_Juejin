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

对得到的 Bitmap 实例调用 compress() 方法，并指定一个图片的质量，通常是在 0~100 之间

## 大图加载
使用 BitmapRegionDecoder 来部分加载图片  在控件的 onDraw() 方法中进行绘制，然后使用 GestureDetector 来检查手势，当移动图片的时候调用 invalid() 方法进行重绘即可