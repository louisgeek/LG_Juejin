# Android 音视频开发之 MediaCodec 解码 MP4 文件进行播放
- 1 先从 MediaCodec 拿到一个空闲的 buffer
- 2 然后将待解码的数据填充到这个 buffer 里面
- 3 紧接着通知 MediaCodec 数据已经填充完毕
- 4 MediaCodec 内部会对数据进行解码操作
- 5 然后将 MediaCodec 解码后的数据输出给 Surface 作显示

## 同步方式
```java
public static void decoder_VideoToSurface_Sync(String videoPath, Surface surface) throws Exception {
    boolean inputBufferEnd = false;
    boolean outputBufferEnd = false;
    //媒体提取器，用于从数据源（如媒体文件或网络流）中提取音频和视频等轨道数据（解封装操作）供 MediaCodec 进行后续的解码和播放等操作
    MediaExtractor mediaExtractor = new MediaExtractor();
    mediaExtractor.setDataSource(videoPath);
    //
    int trackIndex = -1;
    MediaFormat mediaFormat = null;
    String mimeType = "";
    //获取轨道数量
    int trackCount = mediaExtractor.getTrackCount();
    for (int i = 0; i < trackCount; i++) {
        mediaFormat = mediaExtractor.getTrackFormat(i);
        mimeType = mediaFormat.getString(MediaFormat.KEY_MIME);
        if (mimeType != null && mimeType.startsWith("video/")) {
            //mimeType 比如 "video/avc" 即 H.264，"video/hevc" 即 H.265，"audio/mp4a-latm" 即 AAC
            trackIndex = i;
            break;
        }
    }
    if (trackIndex == -1) {
        Log.e(TAG, "trackIndex == -1");
        return;
    }
    //选择轨道，后续操作将针对选中轨道的数据
    mediaExtractor.selectTrack(trackIndex);
    //
    //创建 Codec
    MediaCodec mediaCodec = MediaCodec.createDecoderByType(mimeType);
    //配置 Codec，支持传入 Surface 用于显示视频（如果是解码视频的话），否则传入 null 即可
    mediaCodec.configure(mediaFormat, surface, null, 0);
    //启动 Codec
    mediaCodec.start();
    //
    MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
    int timeoutUs = 10_000; //microseconds
    //
    while (!outputBufferEnd) {
        //==================== 将数据传入 MediaCodec 解码的过程 ====================
        //通过 dequeueInputBuffer 获取可用的输入缓冲区索引，如果返回 -1 表示暂无可用的（出队列）
        int inputBufferIndex = mediaCodec.dequeueInputBuffer(timeoutUs);
        if (inputBufferIndex >= 0) {
            //通过 getInputBuffer 拿到 inputBufferIndex 索引对应的 ByteBuffer
            ByteBuffer inputBuffer = mediaCodec.getInputBuffer(inputBufferIndex);
            if (inputBuffer != null) {
                //从文件或流中读取待解码数据填充到 inputBuffer 中（往 inputBuffer 里写数据，返回的 sampleSize 代表实际写入数据的长度）
                int sampleSize = mediaExtractor.readSampleData(inputBuffer, 0); //样本数据
                if (sampleSize >= 0) {
                    long sampleTime = mediaExtractor.getSampleTime(); //microseconds
                    int sampleFlags = mediaExtractor.getSampleFlags();
                    //通过 queueInputBuffer 将填充完数据的 inputBuffer 加入到输入缓冲区中等待解码处理（入队列）
                    mediaCodec.queueInputBuffer(inputBufferIndex, 0, sampleSize, sampleTime, sampleFlags);
                    //跳到下一个 sample 方便再次读取数据（读取下次采样视频帧）
                    mediaExtractor.advance();
                } else {
                    //如果没有可用数据的话就传入 BUFFER_FLAG_END_OF_STREAM 标记代表结束流
                    mediaCodec.queueInputBuffer(inputBufferIndex, 0, 0, 0, MediaCodec.BUFFER_FLAG_END_OF_STREAM);
                    inputBufferEnd = true;
                }
            }
        }
        //==================== 从 MediaCodec 获取解码后的数据的过程 ====================
        //通过 dequeueOutputBuffer 获取可用的输出缓冲区索引，如果返回 -1 表示暂无可用的（出队列）
        int outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, timeoutUs);
        if (outputBufferIndex >= 0) {
            //判断 Codec 操作是否完成
            if ((bufferInfo.flags & MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0) {
                    outputBufferEnd = true; //让 releaseOutputBuffer 有机会执行
                //break;
            }
            //通过 getOutputBuffer 拿到 outputBufferIndex 索引对应的 ByteBuffer
            ByteBuffer outputBuffer = mediaCodec.getOutputBuffer(outputBufferIndex);
            if (outputBuffer != null) {
                //处理输出缓冲区内容（解码后的数据），比如可以将内容渲染到 Surface 上
            }
            //
            //releaseOutputBuffer 释放输出缓冲区，以便 MediaCodec 可以继续填充存放新的编码数据
            //如果 render 设置为 true 意味着此输出缓冲区内容（解码后的数据）会自动渲染到与 MediaCodec 关联的 Surface 上，用完会自动释放
            //如果 render 设置为 false 意味着已经自行手动处理完数据了（比如提取一帧视频画面数据保存为图片），不需要将内容自动渲染到 Surface 上
            mediaCodec.releaseOutputBuffer(outputBufferIndex, true);
        } else if (outputBufferIndex == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) {
            //输出格式发生变化
            MediaFormat newMediaFormat = mediaCodec.getOutputFormat();
        }
    }
    //
    mediaExtractor.release();
    //停止和释放 Codec
    mediaCodec.stop();
    mediaCodec.release();
}
```


## 异步方式
```java
public static void decoder_VideoToSurface_Async(Surface surface, String videoPath) throws Exception {
    AtomicBoolean inputBufferEnd = new AtomicBoolean(false);
    AtomicBoolean outputBufferEnd = new AtomicBoolean(false);
    //媒体提取器，用于从数据源（如媒体文件或网络流）中提取音频和视频等轨道数据（解封装操作）供 MediaCodec 进行后续的解码和播放等操作
    MediaExtractor mediaExtractor = new MediaExtractor();
    mediaExtractor.setDataSource(videoPath);
    //
    int trackIndex = -1;
    MediaFormat mediaFormat = null;
    String mimeType = "";
    //获取轨道数量
    int trackCount = mediaExtractor.getTrackCount();
    for (int i = 0; i < trackCount; i++) {
        mediaFormat = mediaExtractor.getTrackFormat(i);
        mimeType = mediaFormat.getString(MediaFormat.KEY_MIME);
        if (mimeType != null && mimeType.startsWith("video/")) {
            //mimeType 比如 "video/avc" 即 H.264，"video/hevc" 即 H.265，"audio/mp4a-latm" 即 AAC
            trackIndex = i;
            break;
        }
    }
    if (trackIndex == -1) {
        Log.e(TAG, "trackIndex == -1");
        return;
    }
    //选择轨道，后续操作将针对选中轨道的数据
    mediaExtractor.selectTrack(trackIndex);
    //
    //创建 Codec
    MediaCodec mediaCodec = MediaCodec.createDecoderByType(mimeType);
    //配置 Codec，支持传入 Surface 用于显示视频（如果是解码视频的话），否则传入 null 即可
    mediaCodec.configure(mediaFormat, surface, null, 0);
    //
    mediaCodec.setCallback(new MediaCodec.Callback() {
        @Override
        public void onInputBufferAvailable(@NonNull MediaCodec mediaCodec, int inputBufferIndex) {
            //存在可用的输入缓冲区
            //==================== 将数据传入 MediaCodec 解码的过程 ====================
            if (inputBufferIndex >= 0) {
                //通过 getInputBuffer 拿到 inputBufferIndex 索引对应的 ByteBuffer
                ByteBuffer inputBuffer = mediaCodec.getInputBuffer(inputBufferIndex);
                if (inputBuffer != null) {
                    //从文件或流中读取待解码数据填充到 inputBuffer 中（往 inputBuffer 里写数据，返回的 sampleSize 代表实际写入数据的长度）
                    int sampleSize = mediaExtractor.readSampleData(inputBuffer, 0); //样本数据
                    if (sampleSize >= 0) {
                        long sampleTime = mediaExtractor.getSampleTime(); //microseconds
                        int sampleFlags = mediaExtractor.getSampleFlags();
                        //通过 queueInputBuffer 将填充完数据的 inputBuffer 加入到输入缓冲区中等待解码处理（入队列）
                        mediaCodec.queueInputBuffer(inputBufferIndex, 0, sampleSize, sampleTime, sampleFlags);
                        //跳到下一个 sample 方便再次读取数据（读取下次采样视频帧）
                        mediaExtractor.advance();
                    } else {
                        //如果没有可用数据的话就传入 BUFFER_FLAG_END_OF_STREAM 标记代表结束流
                        mediaCodec.queueInputBuffer(inputBufferIndex, 0, 0, 0, MediaCodec.BUFFER_FLAG_END_OF_STREAM);
                        inputBufferEnd.set(true);
                    }
                }
            }
        }
        @Override
        public void onOutputBufferAvailable(@NonNull MediaCodec mediaCodec, int outputBufferIndex, @NonNull MediaCodec.BufferInfo bufferInfo) {
            //存在可用的输出缓冲区
            //==================== 从 MediaCodec 获取解码后数据的过程 ====================
            if (outputBufferIndex >= 0) {
                //判断 Codec 操作是否完成
                if ((bufferInfo.flags & MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0) {
                    //
                    outputBufferEnd.set(true);
                }
                //通过 getOutputBuffer 拿到 outputBufferIndex 索引对应的 ByteBuffer
                ByteBuffer outputBuffer = mediaCodec.getOutputBuffer(outputBufferIndex);
                if (outputBuffer != null) {
                    //处理输出缓冲区内容（解码后的数据），比如可以将内容渲染到 Surface 上
                }
                //
                //releaseOutputBuffer 释放输出缓冲区，以便 MediaCodec 可以继续填充存放新的编码数据
                //如果 render 设置为 true 意味着此输出缓冲区内容（解码后的数据）会自动渲染到与 MediaCodec 关联的 Surface 上，用完会自动释放
                //如果 render 设置为 false 意味着已经自行手动处理完数据了（比如提取一帧视频画面数据保存为图片），不需要将内容自动渲染到 Surface 上
                mediaCodec.releaseOutputBuffer(outputBufferIndex, true);
            }
        }
        @Override
        public void onError(@NonNull MediaCodec mediaCodec, @NonNull MediaCodec.CodecException e) {
            Log.e(TAG, "onError: ", e);
        }
        @Override
        public void onOutputFormatChanged(@NonNull MediaCodec mediaCodec, @NonNull MediaFormat mediaFormat) {
            //输出格式发生变化
            MediaFormat newMediaFormat = mediaCodec.getOutputFormat();
        }
    });
    //启动 Codec
    mediaCodec.start();
    //
    //注意调用的线程！！！
    while (!outputBufferEnd.get()) {
        Thread.sleep(10);
    }
    //
    mediaExtractor.release();
    //停止和释放 Codec
    mediaCodec.stop();
    mediaCodec.release();
}
```


