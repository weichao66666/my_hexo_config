---
layout: article
title: ScreenRecorder 代码解读
date: 2018-02-27 01:33:52
tags:
categories: 
copyright: true
---

# **Reference**

* [ScreenRecorder](https://github.com/eterrao/ScreenRecorder "https://github.com/eterrao/ScreenRecorder")
* [Android实现录屏直播（三）MediaProjection + VirtualDisplay + librtmp + MediaCodec实现视频编码并推流到rtmp服务器](http://blog.csdn.net/zxccxzzxz/article/details/55230272 "http://blog.csdn.net/zxccxzzxz/article/details/55230272")
* [Android获取实时屏幕画面](https://github.com/ele828/blog/blob/master/source/_posts/Android%E5%B1%8F%E5%B9%95%E7%9B%B4%E6%92%AD%E6%96%B9%E6%A1%88.md "https://github.com/ele828/blog/blob/master/source/_posts/Android%E5%B1%8F%E5%B9%95%E7%9B%B4%E6%92%AD%E6%96%B9%E6%A1%88.md")
* [MediaCodec](https://developer.android.com/reference/android/media/MediaCodec.html "https://developer.android.com/reference/android/media/MediaCodec.html")
* [MediaFormat](https://developer.android.com/reference/android/media/MediaFormat.html "https://developer.android.com/reference/android/media/MediaFormat.html")
* [MediaProjection](https://developer.android.com/reference/android/media/projection/MediaProjection.html "https://developer.android.com/reference/android/media/projection/MediaProjection.html")
* [Buffer](https://developer.android.com/reference/java/nio/Buffer.html "https://developer.android.com/reference/java/nio/Buffer.html")
* [ByteBuffer](https://developer.android.com/reference/java/nio/ByteBuffer.html "https://developer.android.com/reference/java/nio/ByteBuffer.html")
* [H264码流中SPS PPS详解](https://zhuanlan.zhihu.com/p/27896239 "https://zhuanlan.zhihu.com/p/27896239")
* [H.264 NALU语法结构](http://blog.csdn.net/newthinker_wei/article/details/8748442 "http://blog.csdn.net/newthinker_wei/article/details/8748442")
* [H264 获取SPS与PPS（附源码）](http://blog.csdn.net/zgyulongfei/article/details/7538523 "http://blog.csdn.net/zgyulongfei/article/details/7538523")
* [通过RTMP play分析FLV格式详解](http://blog.csdn.net/sweibd/article/details/52189907 "http://blog.csdn.net/sweibd/article/details/52189907")
* [RTMP中FLV流到标准h264、aac的转换](http://www.cnblogs.com/chef/archive/2012/07/18/2597279.html)

---

# **MyApplication**

加载`screenrecorderrtmp`JNI库；保存 Context。

{% codeblock lang:java %}
public class MyApplication extends Application {
    static {
        System.loadLibrary("screenrecorderrtmp");
    }

    private static Context context;

    @Override
    public void onCreate() {
        super.onCreate();
        context = getApplicationContext();
    }

    public static Context getContext() {
        return context;
    }
}
{% endcodeblock %}

---

# **LaunchActivity**

获取权限；设置按钮点击后跳转到屏幕捕捉界面或相机捕捉界面。

{% codeblock lang:java %}
public class LaunchActivity extends AppCompatActivity {
    private static final int REQUEST_CODE_STREAM = 1;
    private static String[] PERMISSIONS_STREAM = {
            Manifest.permission.CAMERA,
            Manifest.permission.RECORD_AUDIO,
            Manifest.permission.WRITE_EXTERNAL_STORAGE
    };

    @BindView(R.id.btn_screen_record)
    Button btnScreenRecord;
    @BindView(R.id.btn_camera_record)
    Button btnCameraRecord;

    boolean authorized = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_launch);
        ButterKnife.bind(this);
        verifyPermissions();
    }

    @OnClick({R.id.btn_screen_record, R.id.btn_camera_record})
    public void onViewClicked(View view) {
        switch (view.getId()) {
            case R.id.btn_screen_record:
                ScreenRecordActivity.launchActivity(this);
                break;
            case R.id.btn_camera_record:
                CameraActivity.launchActivity(this);
                break;
        }
    }

    public void verifyPermissions() {
        int CAMERA_permission = ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA);
        int RECORD_AUDIO_permission = ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO);
        int WRITE_EXTERNAL_STORAGE_permission = ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
        if (CAMERA_permission != PackageManager.PERMISSION_GRANTED
                || RECORD_AUDIO_permission != PackageManager.PERMISSION_GRANTED
                || WRITE_EXTERNAL_STORAGE_permission != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, PERMISSIONS_STREAM, REQUEST_CODE_STREAM);
            authorized = false;
        } else {
            authorized = true;
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_CODE_STREAM) {
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED
                    && grantResults[1] == PackageManager.PERMISSION_GRANTED
                    && grantResults[2] == PackageManager.PERMISSION_GRANTED) {
                authorized = true;
            }
        }
    }
}
{% endcodeblock %}

---

# **ScreenRecordListenerService**

通知开始屏幕捕捉；监听推流进度。

{% codeblock lang:java %}
@SuppressLint("LongLogTag")
public class ScreenRecordListenerService extends Service {
    private static final String TAG = "ScreenRecordListenerService";

    public static final int REQUEST_CODE_PENDING = 0x01;
    private static final int NOTIFICATION_ID = 3;

    private NotificationManager notificationManager;
    private NotificationCompat.Builder builder;
    private IScreenRecorderAidlInterface.Stub binder = new IScreenRecorderAidlInterface.Stub() {
        @Override
        public void startScreenRecord(Intent bundleData) throws RemoteException {
        }

        @Override
        public void sendDanmaku(List<DanmakuBean> danmakuBeanList) throws RemoteException {
            Log.e(TAG, "danmaku msg = " + danmakuBeanList.get(0).getMessage());
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
        initNotification();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        if (notificationManager != null) {
            notificationManager.cancel(NOTIFICATION_ID);
        }
    }

    private void initNotification() {
        builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.drawable.ic_launcher)
                .setContentTitle(getResources().getString(R.string.app_name))
                .setContentText("您正在录制视频内容哦")
                .setOngoing(true)
                .setDefaults(Notification.DEFAULT_VIBRATE);
        Intent intent = new Intent(this, ScreenRecordActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, REQUEST_CODE_PENDING, intent, PendingIntent.FLAG_UPDATE_CURRENT);
        builder.setContentIntent(pendingIntent);
        notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        notificationManager.notify(NOTIFICATION_ID, builder.build());
    }
}
{% endcodeblock %}

---

# **Packager**

## **H264Packager**

{% codeblock lang:java %}
public static class H264Packager {
    public static byte[] generateAvcDecoderConfigurationRecord(MediaFormat mediaFormat) {
        // 获取 csd-0 缓冲区的值，该值对应 SPS；csd-1 对应 PPS
        ByteBuffer spsByteBuff = mediaFormat.getByteBuffer("csd-0");
        ByteBuffer ppsByteBuff = mediaFormat.getByteBuffer("csd-1");
        // 跳过 4 个字节的 configurationVersion，固定使用了 0x01
        spsByteBuff.position(4);
        ppsByteBuff.position(4);
        // 获取缓冲区剩余字节数
        int spsLength = spsByteBuff.remaining();
        int ppsLength = ppsByteBuff.remaining();
        // 11 字节包括：
        // configurationVersion（1 字节）、
        // AVCProfileIndication（1 字节）、
        // profile_compatibility（1 字节）、
        // AVCLevelIndication（1 字节）、
        // 6bit的reserved + 2bit的lengthSizeMinusOne（1 字节）、
        // 3bit的reserved + 5bit的numOfSequenceParameterSets（1 字节）、
        // SPS 的长度（2 字节）、
        // numOfPictureParameterSets（1 字节）、
        // PPS 的长度（2 字节）
        int length = 11 + spsLength + ppsLength;
        byte[] result = new byte[length];
        // 从 result 数组第 9 位开始放置 spsByteBuff，一直放置完
        spsByteBuff.get(result, 8, spsLength);
        // 从 result 数组第 9+spsLength+3 位开始放置 ppsByteBuff，一直放置完
        ppsByteBuff.get(result, 8 + spsLength + 3, ppsLength);
        result[0] = (byte) 0x01;// configurationVersion，实际测试时发现总为0x01
        result[1] = result[9];// AVCProfileIndication
        result[2] = result[10];// profile_compatibility
        result[3] = result[11];// AVCLevelIndication
        result[4] = (byte) 0xFF;// 6bit的reserved + 2bit的lengthSizeMinusOne，实际测试时发现总为0xff
        result[5] = (byte) 0xE1;// 3bit的reserved + 5bit的numOfSequenceParameterSets，实际测试时发现总为0xe1
        // result 数组第 7 位放置 spsLength 的高 8 位
        // result 数组第 8 位放置 spsLength 的低 8 位
        ByteArrayTools.intToByteArrayTwoByte(result, 6, spsLength);
        int pos = 8 + spsLength;
        result[pos] = (byte) 0x01;// numOfPictureParameterSets，实际测试时发现总为0x01
        // result 数组第 8+spsLength+1 位放置 ppsLength 的高 8 位
        // result 数组第 8+spsLength+2 位放置 ppsLength 的低 8 位
        ByteArrayTools.intToByteArrayTwoByte(result, pos + 1, ppsLength);
        return result;
    }
}
{% endcodeblock %}

### **generateAvcDecoderConfigurationRecord(MediaFormat mediaFormat)**

将`AVC`格式重新拼接。

1、SPS & PPS

>**SPS**：Sequence Parameter Sets，针对一个连续编码视频序列的参数。

>**PPS**：Picture Paramater Set，针对一个序列中某一幅图像或者某几幅图像的参数。

![](http://otkw6sse5.bkt.clouddn.com/ScreenRecorder-%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB1519666280252_1.png "Android 中 codec-specific data（编解码特征数据）")

其中可以看到`AVC`格式的`SPS`、`PPS`分别对应`csd-0`和`csd-1`。

2、从 AVC 报文看报头数据结构

![](http://otkw6sse5.bkt.clouddn.com/ScreenRecorder-%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB1519666280252_2.png)

图中阴影部分对应了报头全部数据

数据|数据位置的说明
-|-
0x61 0x76 0x63 0x43|字符`avcC`
0x01|configurationVersion
0x42|AVCProfileIndication
0x00|profile_compatibility
0x1F|AVCLevelIndication
0xFF|6bit的reserved + 2bit的lengthSizeMinusOne
0xE1|3bit的reserved + 5bit的numOfSequenceParameterSets
0x00 0x09|`SPS`的长度
0x67 0x42 0x00 0x1f 0xe9 0x02 0xc1 0x2c 0x80|`SPS`的内容（和长度对应）
0x01|numOfPictureParameterSets
0x00 0x04|`PPS`的长度
0x68 0xce 0x06 0xf2|`PPS`的内容（和长度对应）

## **FlvPackager**

{% codeblock lang:java %}
public static class FlvPackager {
    public static final int FLV_TAG_LENGTH = 11;
    public static final int FLV_VIDEO_TAG_LENGTH = 5;
    public static final int FLV_AUDIO_TAG_LENGTH = 2;
    public static final int FLV_TAG_FOOTER_LENGTH = 4;
    public static final int NALU_HEADER_LENGTH = 4;

    public static void fillFlvVideoTag(byte[] dst, int pos, boolean isAvcSequenceHeader, boolean isIdr, int readDataLength) {
        // 高 4 位表示 FrameType：1 为 I 帧，2 为 P 帧
        // 低 4 位表示 CodecID：7 为 AVC
        dst[pos] = isIdr ? (byte) 0x17 : (byte) 0x27;// FrameType | CodecID
        // 当数据为 AVC 头时没有 video/audio 存在，否则，有 video 存在
        dst[pos + 1] = isAvcSequenceHeader ? (byte) 0x00 : (byte) 0x01;// AVCPacketType
        dst[pos + 2] = (byte) 0x00;// CompositionTime
        dst[pos + 3] = (byte) 0x00;// CompositionTime
        dst[pos + 4] = (byte) 0x00;// CompositionTime
        if (!isAvcSequenceHeader) {
            // 将 readDataLength 分为 4 字节
            ByteArrayTools.intToByteArrayFull(dst, pos + 5, readDataLength);
        }
    }

    public static void fillFlvAudioTag(byte[] dst, int pos, boolean isAACSequenceHeader) {
        /**
         * UB[4] 10=AAC
         * UB[2] 3=44kHz
         * UB[1] 1=16-bit
         * UB[1] 0=MonoSound
         */
        dst[pos] = (byte) 0xAE;
        dst[pos + 1] = isAACSequenceHeader ? (byte) 0x00 : (byte) 0x01;
    }
}
{% endcodeblock %}

### **fillFlvVideoTag(byte[] dst, int pos, boolean isAVCSequenceHeader, boolean isIDR, int readDataLength)**

插入`FLV`video 类型的`TAG`。

### **fillFlvAudioTag(byte[] dst, int pos, boolean isAACSequenceHeader)**

插入`FLV`audio 类型的`TAG`。

1、AVC & I帧 & P帧 & B帧 & IDR 图像

>**AVC**：对于一段变化不大图像画面，我们可以先编码出一个完整的图像A1帧，随后的A2帧就不编码全部图像，只写入与A1帧的差别，这样A2帧的大小就只有A1帧的1/10或更小！A2帧之后的A3帧如果变化不大，我们可以继续以参考A2的方式编码A3帧，这样循环下去。这段图像我们称为一个序列（序列就是有相同特点的一段数据）。
当某个图像与之前的图像变化很大，无法参考前面的帧来生成，那我们就结束上一个序列，开始下一个序列，也就是对这个图像生成一个完整帧B1，随后的图像就参考B1生成，只写入与B1的差别内容。

>**I帧**：完整编码的帧。

>**P帧**：参考之前的`I帧`或`P帧`生成的只包含差异部分编码的帧叫`P帧`。

>**B帧**：参考前后的帧编码的帧叫`B帧`。

>**IDR 图像**：一个序列的第一个图像叫做`IDR图像`（立即刷新图像），`IDR图像`都是`I帧`图像。`IDR图像`用于解码的重同步，当解码器解码到`IDR 图像`时，立即将参考帧队列清空，将已解码的数据全部输出或抛弃，重新查找参数集，开始一个新的序列。这样，如果前一个序列出现重大错误，在这里可以重新同步。`IDR图像`之后的图像永远不会使用`IDR图像`之前的数据来解码。

2、从 FLV 报文看报头数据结构

![](http://otkw6sse5.bkt.clouddn.com/ScreenRecorder-%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB1519666280252_3.png)

数据|数据位置的说明
-|-
0x46 0x4c 0x56|字符`FLV`
0x01|版本号
0x05|右起第 0 位和第 2 位分别表示 video 与 audio 存在的情况（1 表示存在，0 表示不存在）
0x00 0x00 0x00 0x09|报头数据长度

3、NALU key-value

key|value
-|-
NALU_TYPE_SLICE|1
NALU_TYPE_DPA|2
NALU_TYPE_DPB|3
NALU_TYPE_DPC|4
NALU_TYPE_IDR|5
NALU_TYPE_SEI|6
NALU_TYPE_SPS|7
NALU_TYPE_PPS|8
NALU_TYPE_AUD|9
NALU_TYPE_EOSEQ|10
NALU_TYPE_EOSTREAM|11
NALU_TYPE_FILL|12

---

# **ScreenRecorder**

按照服务端接收格式整理数据。

{% codeblock lang:java %}
public class ScreenRecorder extends Thread {
    private static final String TAG = "ScreenRecorder";

    private static final String MIME_TYPE = "video/avc"; // H.264 Advanced Video Coding
    private static final int FRAME_RATE = 30; // 30fps
    private static final int IFRAME_INTERVAL = 2; // 2s between I-frames
    private static final int TIMEOUT_US = 10000;

    private RESFlvDataCollecter mDataCollecter;
    private int mWidth;
    private int mHeight;
    private int mBitRate;
    private int mDpi;
    private MediaProjection mMediaProjection;

    private long startTime = 0;
    private MediaCodec mediaCodec;
    private Surface surface;
    private AtomicBoolean quit = new AtomicBoolean(false);
    private MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
    private VirtualDisplay virtualDisplay;

    public ScreenRecorder(RESFlvDataCollecter dataCollecter, int width, int height, int bitRate, int dpi, MediaProjection mediaProjection) {
        super(TAG);
        mDataCollecter = dataCollecter;
        mWidth = width;
        mHeight = height;
        mBitRate = bitRate;
        mDpi = dpi;
        mMediaProjection = mediaProjection;
        startTime = 0;
    }

    private void release() {
        if (mediaCodec != null) {
            mediaCodec.stop();
            mediaCodec.release();
            mediaCodec = null;
        }
        if (virtualDisplay != null) {
            virtualDisplay.release();
        }
        if (mMediaProjection != null) {
            mMediaProjection.stop();
        }
    }

    @Override
    public void run() {
        try {
            prepareEncoder();
            /*将画面投影到 VirtualDisplay 中*/
            virtualDisplay = mMediaProjection.createVirtualDisplay(TAG, mWidth, mHeight, mDpi,
                    DisplayManager.VIRTUAL_DISPLAY_FLAG_PUBLIC/*设置可以将其他显示器的内容反射到该 VirtualDisplay 上（TODO 默认只能是自身应用？）*/,
                    surface/*VirtualDisplay 将图像渲染到 Surface 中*/,
                    null, null);
            Log.d(TAG, "created virtual display: " + virtualDisplay);
            recordVirtualDisplay();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            release();
        }
    }

    private void prepareEncoder() {
        try {
            // 创建 video/avc 类型的编码器，在这个场景下，MediaCodec 只允许使用 video/avc 编码类型，使用其他的编码会 crash
            mediaCodec = MediaCodec.createEncoderByType(MIME_TYPE);
            // 设置视频格式：video/avc
            MediaFormat format = MediaFormat.createVideoFormat(MIME_TYPE, mWidth, mHeight);
            // 设置颜色格式：Surface
            format.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatSurface);
            // 设置比特率
            format.setInteger(MediaFormat.KEY_BIT_RATE, mBitRate);
            // 设置帧率：30fps
            format.setInteger(MediaFormat.KEY_FRAME_RATE, FRAME_RATE);
            // 设置关键帧间隔时间：2s
            format.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, IFRAME_INTERVAL);
            // 编码器应用格式
            mediaCodec.configure(format, null, null,
                    MediaCodec.CONFIGURE_FLAG_ENCODE/*设置该 MediaCodec 作为编码器使用*/);
            // 创建 Surface
            surface = mediaCodec.createInputSurface();
            mediaCodec.start();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private void recordVirtualDisplay() {
        while (!quit.get()) {
            // 最多阻塞 10s，用于获取已成功解码的输出缓冲区的索引（缓冲的元数据会放到 bufferInfo 中）或者一个状态值
            int eobIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, TIMEOUT_US);
            switch (eobIndex) {
                case MediaCodec.INFO_OUTPUT_FORMAT_CHANGED:
                    LogTools.d("MediaCodec.INFO_OUTPUT_FORMAT_CHANGED:" + mediaCodec.getOutputFormat().toString());
                    sendAvcDecoderConfigurationRecord(0, mediaCodec.getOutputFormat());
                    break;
                case MediaCodec.INFO_TRY_AGAIN_LATER:
                    LogTools.d("MediaCodec.INFO_TRY_AGAIN_LATER");
                    break;
                default:
                    LogTools.d("mediaCodec eobIndex=" + eobIndex);
                    /**
                     * we send sps pps already in INFO_OUTPUT_FORMAT_CHANGED
                     * so we ignore MediaCodec.BUFFER_FLAG_CODEC_CONFIG
                     */
                    if (bufferInfo.flags != MediaCodec.BUFFER_FLAG_CODEC_CONFIG && bufferInfo.size != 0) {
                        if (startTime == 0) {
                            startTime = bufferInfo.presentationTimeUs / 1000;
                        }
                        //
                        ByteBuffer realData = mediaCodec.getOutputBuffer(eobIndex);
                        if (realData != null) {
                            realData.position(bufferInfo.offset + 4);// TODO 为什么 + 4？？
                            realData.limit(bufferInfo.offset + bufferInfo.size);
                            sendRealData((bufferInfo.presentationTimeUs / 1000) - startTime, realData);
                        }
                    }
                    mediaCodec.releaseOutputBuffer(eobIndex, false);
                    break;
            }
        }
    }

    private void sendAvcDecoderConfigurationRecord(long timeMs, MediaFormat format) {
        byte[] avcDecoderConfigurationRecord = Packager.H264Packager.generateAvcDecoderConfigurationRecord(format);
        int packetLen = Packager.FlvPackager.FLV_VIDEO_TAG_LENGTH + avcDecoderConfigurationRecord.length;
        byte[] finalBuff = new byte[packetLen];
        // 在 finalBuff 数组最前面插入 5 个字节的 FLV video TAG
        Packager.FlvPackager.fillFlvVideoTag(finalBuff, 0, true, true, avcDecoderConfigurationRecord.length);
        // 将 avcDecoderConfigurationRecord 数组的数据拼接到 finalBuff 数组后面
        System.arraycopy(avcDecoderConfigurationRecord, 0, finalBuff, Packager.FlvPackager.FLV_VIDEO_TAG_LENGTH, avcDecoderConfigurationRecord.length);
        RESFlvData resFlvData = new RESFlvData();
        resFlvData.droppable = false;
        resFlvData.byteBuffer = finalBuff;
        resFlvData.size = finalBuff.length;
        resFlvData.dts = (int) timeMs;
        resFlvData.flvTagType = FLV_RTMP_PACKET_TYPE_VIDEO;
        resFlvData.videoFrameType = RESFlvData.NALU_TYPE_IDR;
        mDataCollecter.collect(resFlvData, FLV_RTMP_PACKET_TYPE_VIDEO);
    }

    private void sendRealData(long timeMs, ByteBuffer realData) {
        int realDataLength = realData.remaining();
        int packetLen = Packager.FlvPackager.FLV_VIDEO_TAG_LENGTH + Packager.FlvPackager.NALU_HEADER_LENGTH + realDataLength;
        byte[] finalBuff = new byte[packetLen];
        // 将 realData 数组的数据放到 finalBuff 数组的第 Packager.FlvPackager.FLV_VIDEO_TAG_LENGTH+Packager.FlvPackager.NALU_HEADER_LENGTH+1 个位置到结束
        realData.get(finalBuff, Packager.FlvPackager.FLV_VIDEO_TAG_LENGTH + Packager.FlvPackager.NALU_HEADER_LENGTH, realDataLength);
        // 获取 NALU 类型，计算后得 5 表示 NALU 类型是 NALU_TYPE_IDR
        int frameType = finalBuff[Packager.FlvPackager.FLV_VIDEO_TAG_LENGTH + Packager.FlvPackager.NALU_HEADER_LENGTH] & 0x1F;
        // 在 finalBuff 数组最前面插入 9 个字节的 FLV video TAG
        Packager.FlvPackager.fillFlvVideoTag(finalBuff, 0, false, frameType == 5, realDataLength);
        RESFlvData resFlvData = new RESFlvData();
        resFlvData.droppable = true;
        resFlvData.byteBuffer = finalBuff;
        resFlvData.size = finalBuff.length;
        resFlvData.dts = (int) timeMs;
        resFlvData.flvTagType = FLV_RTMP_PACKET_TYPE_VIDEO;
        resFlvData.videoFrameType = frameType;
        mDataCollecter.collect(resFlvData, FLV_RTMP_PACKET_TYPE_VIDEO);
    }

    public boolean getStatus() {
        return !quit.get();
    }

    public void quit() {
        quit.set(true);
    }
}
{% endcodeblock %}

## **prepareEncoder()**

准备编码器。

## **recordVirtualDisplay()**

获取已解码的输出缓冲的索引，或者一个状态值。当是状态值`MediaCodec.INFO_OUTPUT_FORMAT_CHANGED`时，生成`AVC`序列头部，插入`FLV`video 类型的`TAG`，并发送。否则，直接将数据插入`FLV`video 类型的`TAG`并发送。

## **sendAVCDecoderConfigurationRecord(long tms, MediaFormat format)**

生成`AVC`序列头部，插入`FLV`video 类型的`TAG`，并发送。

## **sendRealData(long tms, ByteBuffer realData)**

将数据插入`FLV`video 类型的`TAG`并发送。

---

# **RtmpStreamingSender**

维护一个发送队列，在连接服务端成功后，不断发送数据到服务端，如果发送速率低于添加速率，会跳过部分帧。当屏幕捕捉后，尝试添加到队列中，如果队列已满，则放弃添加到队列中。

{% codeblock lang:java %}
public class RtmpStreamingSender implements Runnable {
    private static final String TAG = "RtmpStreamingSender";

    private static class STATE {
        private static final int START = 0;
        private static final int RUNNING = 1;
        private static final int STOPPED = 2;
    }

    private volatile int state;

    private String mRtmpAddr = null;

    private final Object lock = new Object();
    private static final int MAX_QUEUE_CAPACITY = 50;
    private LinkedBlockingDeque<RESFlvData> frameQueue = new LinkedBlockingDeque<>(MAX_QUEUE_CAPACITY);
    private AtomicBoolean quit = new AtomicBoolean(false);
    private FLvMetaData fLvMetaData;
    private RESCoreParameters coreParameters;

    private long jniRtmpPointer = 0;
    private int maxQueueLength = 50;
    private int writeMsgNum = 0;

    public RtmpStreamingSender() {
        coreParameters = new RESCoreParameters();
        coreParameters.mediacodecAACBitRate = RESFlvData.AAC_BITRATE;
        coreParameters.mediacodecAACSampleRate = RESFlvData.AAC_SAMPLE_RATE;
        coreParameters.mediacodecAVCFrameRate = RESFlvData.FPS;
        coreParameters.videoWidth = RESFlvData.VIDEO_WIDTH;
        coreParameters.videoHeight = RESFlvData.VIDEO_HEIGHT;

        fLvMetaData = new FLvMetaData(coreParameters);
    }

    @Override
    public void run() {
        while (!quit.get()) {
            if (frameQueue.size() > 0) {
                switch (state) {
                    case STATE.START:
                        LogTools.d("RESRtmpSender,WorkHandler,tid=" + Thread.currentThread().getId());
                        if (TextUtils.isEmpty(mRtmpAddr)) {
                            LogTools.e("rtmp address is null!");
                            break;
                        }
                        jniRtmpPointer = RtmpClient.open(mRtmpAddr, true);
                        if (jniRtmpPointer != 0) {
                            String serverIpAddr = RtmpClient.getIpAddr(jniRtmpPointer);
                            LogTools.d("server ip address = " + serverIpAddr);

                            byte[] MetaData = fLvMetaData.getMetaData();
                            RtmpClient.write(jniRtmpPointer, MetaData, MetaData.length, RESFlvData.FLV_RTMP_PACKET_TYPE_INFO, 0);

                            state = STATE.RUNNING;
                        }
                        break;
                    case STATE.RUNNING:
                        synchronized (lock) {
                            --writeMsgNum;
                        }
                        RESFlvData flvData = frameQueue.pop();
                        if (writeMsgNum >= (maxQueueLength * 2 / 3) && flvData.flvTagType == RESFlvData.FLV_RTMP_PACKET_TYPE_VIDEO && flvData.droppable) {
                            LogTools.d("senderQueue is crowded, abort a frame");
                            break;
                        }

                        int result = RtmpClient.write(jniRtmpPointer, flvData.byteBuffer, flvData.byteBuffer.length, flvData.flvTagType, flvData.dts);
                        if (result == 0) {
                            if (flvData.flvTagType == RESFlvData.FLV_RTMP_PACKET_TYPE_VIDEO) {
                                LogTools.d("video frame sent = " + flvData.size);
                            } else {
                                LogTools.d("audio frame sent = " + flvData.size);
                            }
                        } else {
                            LogTools.e("writeError = " + result);
                        }

                        break;
                    case STATE.STOPPED:
                        result = RtmpClient.close(jniRtmpPointer);
                        LogTools.e("close result = " + result);
                        break;
                }
            }
        }
        int result = RtmpClient.close(jniRtmpPointer);
        LogTools.e("close result = " + result);
    }

    public void sendStart(String rtmpAddr) {
        synchronized (lock) {
            writeMsgNum = 0;
        }
        mRtmpAddr = rtmpAddr;
        state = STATE.START;
    }

    public void sendStop() {
        synchronized (lock) {
            writeMsgNum = 0;
        }
        state = STATE.STOPPED;
    }

    public void sendFood(RESFlvData flvData, int type) {
        synchronized (lock) {
            if (writeMsgNum <= maxQueueLength) {
                frameQueue.add(flvData);
                ++writeMsgNum;
            } else {
                LogTools.d("senderQueue is full, abort a frame");
            }
        }
    }

    public final void quit() {
        quit.set(true);
    }
}
{% endcodeblock %}

---

# **ScreenRecordActivity**

控制屏幕捕捉和推流。

{% codeblock lang:java %}
public class ScreenRecordActivity extends Activity implements View.OnClickListener {
    private static final String TAG = "ScreenRecordActivity";

    private static final int REQUEST_CODE = 1;

    private Button button;
    private EditText rtmpAddrET;

    private String rtmpAddr;
    private boolean isRecording;
    private MediaProjectionManager mediaProjectionManager;
    private ScreenRecorder screenRecorder;
    private RtmpStreamingSender streamingSender;
    private ExecutorService executorService;
    private RESAudioClient audioClient;
    private RESCoreParameters coreParameters;
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
        }
    };

    public static void launchActivity(Context context) {
        Intent intent = new Intent(context, ScreenRecordActivity.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = findViewById(R.id.button);
        button.setOnClickListener(this);
        rtmpAddrET = findViewById(R.id.et_rtmp_address);

        mediaProjectionManager = (MediaProjectionManager) getSystemService(MEDIA_PROJECTION_SERVICE);
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (isRecording) {
            stopScreenRecordService();// 在录屏状态，切换回应用，则停止录屏
        }
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (isRecording) {
            startScreenRecordService();// 最小化应用后，开始录屏
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (screenRecorder != null) {
            stopScreenRecord();
        }
    }

    @Override
    public void onClick(View v) {
        if (screenRecorder != null) {
            stopScreenRecord();
        } else {
            startScreenCapture();
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        Log.d(TAG, "onActivityResult(" + requestCode + ", " + resultCode + ")");
        MediaProjection mediaProjection = mediaProjectionManager.getMediaProjection(resultCode, data);// 获取屏幕捕捉
        if (mediaProjection == null) {
            Log.e(TAG, "media projection is null");
            return;
        }

        rtmpAddr = rtmpAddrET.getText().toString().trim();
        if (TextUtils.isEmpty(rtmpAddr)) {
            Log.e(TAG, "rtmp address cannot be null");
            Toast.makeText(this, "rtmp address cannot be null", Toast.LENGTH_SHORT).show();
            return;
        }

        coreParameters = new RESCoreParameters();
        audioClient = new RESAudioClient(coreParameters);
        if (!audioClient.prepare()) {
            LogTools.d("!!!!!audioClient.prepare() failed");
            return;
        }

        streamingSender = new RtmpStreamingSender();
        streamingSender.sendStart(rtmpAddr);
        RESFlvDataCollecter collecter = new RESFlvDataCollecter() {
            @Override
            public void collect(RESFlvData flvData, int type) {
                streamingSender.sendFood(flvData, type);
            }
        };
        screenRecorder = new ScreenRecorder(collecter, RESFlvData.VIDEO_WIDTH, RESFlvData.VIDEO_HEIGHT, RESFlvData.VIDEO_BITRATE, 1, mediaProjection);
        screenRecorder.start();
        audioClient.start(collecter);

        executorService = Executors.newCachedThreadPool();
        executorService.execute(streamingSender);

        button.setText("Stop Recorder");
        Toast.makeText(this, "Screen recorder is running...", Toast.LENGTH_LONG).show();
        moveTaskToBack(true);// 最小化应用
    }

    private void startScreenCapture() {
        Log.d(TAG, "startScreenCapture()");
        isRecording = true;
        Intent captureIntent = mediaProjectionManager.createScreenCaptureIntent();// 获取封装好的用于开始屏幕捕捉的 intent
        startActivityForResult(captureIntent, REQUEST_CODE);
    }

    private void stopScreenRecord() {
        Log.d(TAG, "stopScreenRecord()");
        screenRecorder.quit();
        screenRecorder = null;
        if (streamingSender != null) {
            streamingSender.sendStop();
            streamingSender.quit();
            streamingSender = null;
        }
        if (executorService != null) {
            executorService.shutdown();
            executorService = null;
        }
        button.setText("Restart recorder");
    }

    private void startScreenRecordService() {
        Log.d(TAG, "startScreenRecordService()");
        if (screenRecorder != null && screenRecorder.getStatus()) {
            Intent intent = new Intent(this, ScreenRecordListenerService.class);
            bindService(intent, connection, BIND_AUTO_CREATE);
            startService(intent);
        }
    }

    private void stopScreenRecordService() {
        Log.d(TAG, "stopScreenRecordService()");
        Intent intent = new Intent(this, ScreenRecordListenerService.class);
        stopService(intent);
        if (screenRecorder != null && screenRecorder.getStatus()) {
            Toast.makeText(this, "现在正在进行录屏直播哦", Toast.LENGTH_SHORT).show();
        }
    }
}
{% endcodeblock %}

## **onActivityResult(int requestCode, int resultCode, Intent data)**




## **startScreenCapture()**

开始屏幕捕捉。

## **stopScreenRecord()**

停止屏幕捕捉。

## **startScreenRecordService()**

开始监听推流进度。

## **stopScreenRecordService()**

停止监听推流进度。

---