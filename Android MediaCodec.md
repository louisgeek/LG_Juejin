# Android MediaCodec
- Android MediaCodec 媒体编解码器是用于实现编解码音视频数据的组件，为开发者提供了直接访问底层多媒体编解码器的能力，从而实现高效的音视频编码和解码操作，支持多种编解码器格式（比如 H.264、H.265、VP8、VP9、AAC、MP3 等）
- Encoder 编码器，可以将原始的音视频数据（比如原始未压缩的视频帧或者音频样本）转换成压缩格式（比如 H.264、AAC 等），以便存储或者传输
- Decoder 解码器，就是将压缩后的音视频数据还原成原始未压缩的可播放格式
- MediaCodec 支持同步和异步两种操作模式。同步模式下，开发者需要手动管理输入输出缓冲区的获取和释放；异步模式下，MediaCodec 会通过回调通知开发者缓冲区的可用状态


压缩和解压缩媒体数据
## MediaCodec 生命周期


- Stopped 状态
    - Uninitialized 子状态，当新创建一个 MediaCodec 时就进到了 Stopped 状态，即处于 Uninitialized 子状态
    - Configured 子状态，Uninitialized 子状态下调用 configure 方法后就进到了 Configured 子状态
    - Error 子状态，Error 子状态下可以调用 reset 方法返回到 Uninitialized 子状态或调用 release 方法进入到 Released 状态
- Executing 状态
    - Flushed 子状态，Configured 子状态下调用 start 方法后就进到了 Executing 状态，即处于 Flushed 子状态
    - Running 子状态
    - End-of-Stream 子状态
- Released 状态


- 处于 Executing 状态时，调用 stop 方法可以返回到 Uninitialized 子状态
- 处于 Executing 或 Stopped  状态时，调用 reset 方法可以返回到 Uninitialized 子状态


"video/avc" - H.264/AVC video           "video/hevc" - H.265/HEVC video
```
createByCodecName 根据组件名创建codec
createDecoderByType 根据特定MIME类型(如"video/avc")创建codec
createEncoderByType 根据特定MIME类型(如"video/avc")创建codec
```
MediaFormat mFormat = MediaFormat.createVideoFormat("video/avc", 640 ,480);     // 创建MediaFormat

MediaCodec mediaCodec =MediaCodec.createDecoderByType(MediaFormat.MIMETYPE_VIDEO_AVC);



```
createEncoderByType/createDecoderByType
configure
start
while(true) {
     dequeueInputBuffer  //从输入流队列中取数据进行编码操作 
     getInputBuffers     //获取需要编码数据的输入流队列，返回的是一个ByteBuffer数组 
     queueInputBuffer    //输入流入队列 
     dequeueOutputBuffer //从输出队列中取出编码操作之后的数据
     getOutPutBuffers    // 获取编解码之后的数据输出流队列，返回的是一个ByteBuffer数组
     releaseOutputBuffer //处理完成，释放ByteBuffer数据
}
stop
release
```


初始化 MediaMuxer：指定输出文件路径和格式。
创建 MediaCodec 编码器：分别创建视频和音频编码器，并进行配置。
启动编码器和 MediaMuxer：开始编码和封装过程。
编码数据：将原始音视频数据送入编码器进行编码。
写入 MediaMuxer：将编码后的音视频数据写入 MediaMuxer。
停止和释放资源：编码完成后，停止编码器和 MediaMuxer，并释放相关资源。


初始化 MediaCodec 编码器：配置视频格式、比特率、帧率等参数。
启动编码器。
接收并编码视频帧：从摄像头或其它源获取原始视频数据，送入编码器。
获取编码后的数据：通过 MediaCodec 的输出缓冲区获取编码后的 H.264 数据。
使用 MediaMuxer 封装：将编码后的数据封装到 MP4 容器中，添加必要的轨道和元数据。
释放资源：停止并释放 MediaCodec 和 MediaMuxer。




使用 output Surface 时，我们可以选择是否在 Surface 上渲染每个 output buffer，有三个选择：

不渲染缓冲区：调用 releaseOutputBuffer(bufferId, false)。
使用默认时间戳渲染缓冲区：调用 releaseOutputBuffer(bufferId, true)。
使用特定时间戳渲染缓冲区：调用 releaseOutputBuffer(bufferId, timestamp)。

从 Android 6.0(API 等级 23) 开始，默认时间戳是 buffer 的 PTS( presentation timestamp)，时间会转换为纳秒。此外，从 Android 6.0(API 等级 23) 开始，我们可以使用 setOutputSurface 方法动态更改 output Surface。

使用 input Surface 时，input buffers 将不可访问，因为 buffers 会自动从 input Surface 传递到 codec。此时调用 dequeueInputBuffer 将抛出 IllegalStateException 异常；调用 getInputBuffers() 会返回一个只读的虚假 ByteBuffer[] 数组。

使用 input Surface 时，我们需要调用 signalEndOfInputStream() 以发出 end-of-stream 信号。在 signalEndOfInputStream 方法调用之后，input Surface 会立即停止向 codec 提交数据。

//Requests a Surface to use as the input to an encoder, in place of input buffers. This may only be 
//called after configure(MediaFormat, Surface, MediaCrypto, int) and before start().
//调用此方法，官方有这么一段话，意思是必须在configure之后 start()之前调用。
mInputSurface =  mMediaCodec.createInputSurface();
// 创建输入Surface，需在configure之后、start之前调用
2public Surface createInputSurface ()

3// 设置输入Surface
4public void setInputSurface (Surface surface)
5// 发送流结束的信号
6public void signalEndOfInputStream ()


1// 设置输出Surface
2public void setOutputSurface (Surface surface)

3// false表示不渲染这个buffer，true表示使用默认的时间戳渲染这个buffer
4public void releaseOutputBuffer (int index, boolean render)
5// 使用指定的时间戳渲染这个buffer
6public void releaseOutputBuffer (int index, long renderTimestampNs)


Android机型数据必须以16位对齐：摄像头采集360x640的图像实际上获取到的是384x640，采集180x320的图像获取到的是192x320，这就会导致编码图像绿屏的情况。因此数据在传入编码器之前需要先进行裁剪。


部分机型编码画面出现大量马赛克：presentationTimeUs需设定为实际时间戳，时间戳用于指示音视频数据的播放或处理时间，错误的时间戳会导致输出码率与设定值不符的情况。可以使用System.nanoTime()获取纳秒级别的系统时间。

部分鸿蒙机型编码出现绿条：通常情况下Android机型数据都是以16位对齐，但鸿蒙海思芯片不支持，需要单独适配。可以创建指定编码器OMX.google.h264.encoder，或采用NV12编码。

// 如果API<=19，需要根据BufferInfo的offset偏移量调整ByteBuffer的位置
// 并且限定将要读取缓存区数据的长度，否则输出数据会混乱
if (mBufferInfo.size != 0) {
	if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.KITKAT) {
		outputBuffer.position(mBufferInfo.offset);
		outputBuffer.limit(mBufferInfo.offset + mBufferInfo.size);
	}
	// mMuxer.writeSampleData(mTrackIndex, encodedData, bufferInfo);
}



// extradata中是Annex-B格式的SPS、PPS NALU数据
//SPS设为"csd-0", PPS设为"csd-1"
mediaFormat.setByteBuffer("csd-0", extradata);



/** * 裁剪视频： * seek的部分我纠结过，想来想去只有这么seek，即便丢帧也不会出现绿屏的情况，因为第一帧不是关键帧就会绿屏 * 其次，因为视频的帧，不只有关键帧，还有前向、双向预测帧，所以不能先prev_sync再一个个advance读下一个帧 * close_sync的模式是不确定的，要么前要么后，或许直接使用next_sync更好点 * readEOS出现的原因，我debug的时候，发现进入到endTime那里的次数超出了一次，差不多会有三次进入，这是没必要的。 * 转码： * 难度在理解解码的outputBuffer与编码的inputBuffer的关系上，一开始我还傻傻的用队列去存，后来发现没这必要， * 直接塞到编码器无疑效率更高。 *
