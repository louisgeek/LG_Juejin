# Android 音视频开发之 MediaCodec 编码摄像头画面到 MP4 文件
 
## 同步方式
```java
public class MediaCodecManager {
    private static final String MIME_TYPE = MediaFormat.MIMETYPE_VIDEO_AVC; //"video/avc" 即 H.264
    private static final int BIT_RATE = 4000000; 
    private static final int FRAME_RATE = 30; 
    private static final int I_FRAME_INTERVAL = 1;
    //
    private MediaMuxer mMediaMuxer;
    private MediaCodec mMediaCodec;
    private int mTrackIndex;

    public void encoder_CameraPreviewFrameToVideo_Sync_Init(String videoPath, int width, int height) throws Exception {
        //MediaMuxer.OutputFormat.MUXER_OUTPUT_WEBM
        mMediaMuxer = new MediaMuxer(videoPath, MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4);
        //
        MediaFormat mediaFormat = MediaFormat.createVideoFormat(MIME_TYPE, width, height);
//        mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420Planar); //YUV 420P：I420（YU12），YV12
//        mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420PackedPlanar);
        mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420SemiPlanar); //YUV 420SP：NV12，NV21
//        mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatSurface);
//        mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420Flexible);
        mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, BIT_RATE);
        mediaFormat.setInteger(MediaFormat.KEY_FRAME_RATE, FRAME_RATE);
        mediaFormat.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, I_FRAME_INTERVAL);

        //创建编码器
        mMediaCodec = MediaCodec.createEncoderByType(MIME_TYPE);
        //配置编码器
        mMediaCodec.configure(mediaFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);
        //启动编码器
        mMediaCodec.start();
        //
        //        presentationTimeUs = 0;
    }

    //处理每一帧数据
    public void encoder_CameraPreviewFrameToVideo_Sync_EncodeFrame(byte[] frameData) {
        boolean outputBufferEnd = false;
        //
        MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
        int timeoutUs = 10_000; //microseconds
        //
        //==================== 将数据传入 MediaCodec 编码的过程 ====================
        //通过 dequeueInputBuffer 获取可用的输入缓冲区索引，如果返回 -1 表示暂无可用的（出队列）
        int inputBufferIndex = mMediaCodec.dequeueInputBuffer(timeoutUs);
        if (inputBufferIndex >= 0) {
            //通过 getInputBuffer 拿到 inputBufferIndex 索引对应的 ByteBuffer
            ByteBuffer inputBuffer = mMediaCodec.getInputBuffer(inputBufferIndex);
            if (inputBuffer != null) {
                //从文件或流中读取待解码数据填充到 inputBuffer 中（往 inputBuffer 里写数据，返回的 sampleSize 代表实际写入数据的长度）
                inputBuffer.clear();
                //将待编码的数据填充到输入缓冲区
                inputBuffer.put(frameData);
                //提交输入缓冲区到编码器进行处理
                //此buffer的PTS(以微秒为单位)
                long presentationTimeUs = System.nanoTime() / 1000;
                mMediaCodec.queueInputBuffer(inputBufferIndex, 0, frameData.length, presentationTimeUs, 0);
//                presentationTimeUs += 1000000 / FRAME_RATE;
            }
        }
        //==================== 从 MediaCodec 获取编码后的数据的过程 ====================
        //通过 dequeueOutputBuffer 获取可用的输出缓冲区索引，如果返回 -1 表示暂无可用的（出队列）
        while (!outputBufferEnd) {
            int outputBufferIndex = mMediaCodec.dequeueOutputBuffer(bufferInfo, timeoutUs);
            if (outputBufferIndex >= 0) {
                //判断是否解码完成
                if ((bufferInfo.flags & MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0) {
                    outputBufferEnd = true;  //让 releaseOutputBuffer 有机会执行
//                    break;
                }
                if ((bufferInfo.flags & MediaCodec.BUFFER_FLAG_CODEC_CONFIG) != 0) {
                    //目的是忽略 codec config 数据
                    bufferInfo.size = 0;
                }
                //通过 getOutputBuffer 拿到 outputBufferIndex 索引对应的 ByteBuffer
                ByteBuffer outputBuffer = mMediaCodec.getOutputBuffer(outputBufferIndex);
                if (outputBuffer != null) {
                //处理输出缓冲区内容（编码后的数据）
//                byte[] outputData = new byte[bufferInfo.size];
//                outputBuffer.get(outputData);
//                writeToFile(outputData);
                    mMediaMuxer.writeSampleData(mTrackIndex, outputBuffer, bufferInfo);
                }
                //
                //releaseOutputBuffer 释放输出缓冲区，以便 MediaCodec 可以继续填充存放新的编码数据
                //如果 render 设置为 true 意味着此输出缓冲区内容（编码后的数据）会自动渲染到与 MediaCodec 关联的 Surface 上，用完会自动释放
                //如果 render 设置为 false 意味着已经自行手动处理完数据了（比如提取一帧视频画面数据保存为图片），不需要将内容自动渲染到 Surface 上
                mMediaCodec.releaseOutputBuffer(outputBufferIndex, false);
            } else if (outputBufferIndex == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) {
                //输出格式发生变化
                //编码器输出缓存区格式改变，通常在存储数据之前且只会改变一次
                MediaFormat newMediaFormat = mMediaCodec.getOutputFormat();
//                newMediaFormat.setByteBuffer("csd-0",outputBuffer);
                mTrackIndex = mMediaMuxer.addTrack(newMediaFormat);
                if (mTrackIndex >= 0) {
                    mMediaMuxer.start();
                }
            } else if (outputBufferIndex == MediaCodec.INFO_TRY_AGAIN_LATER) {
                //注意这个分支！！！
                //意味着当前没有数据可处理，或者处理数据需要更多时间
//                try {
//                    Thread.sleep(10);
//                } catch (InterruptedException e) {
//                    e.printStackTrace();
//                }
                outputBufferEnd = true;
//                break;
            }
        }
    }

    public void encoder_CameraPreviewFrameToVideo_Sync_Release() {
        mMediaMuxer.stop();
        mMediaMuxer.release();
        //停止和释放 Codec
        mMediaCodec.stop();
        mMediaCodec.release();
    } 

}  
```


