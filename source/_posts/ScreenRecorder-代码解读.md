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
* [libRTMP使用说明](http://blog.csdn.net/leixiaohua1020/article/details/14229543)
* [带你吃透RTMP](http://mingyangshang.github.io/2016/03/06/RTMP%E5%8D%8F%E8%AE%AE/ "http://mingyangshang.github.io/2016/03/06/RTMP%E5%8D%8F%E8%AE%AE/")
* [直播推流实现RTMP协议的一些注意事项](https://www.jianshu.com/p/00aceabce944 "https://www.jianshu.com/p/00aceabce944")
* [RTMPdump（libRTMP） 源代码分析 8： 发送消息（Message）](http://blog.csdn.net/leixiaohua1020/article/details/12958747 "http://blog.csdn.net/leixiaohua1020/article/details/12958747")
* [简单的iOS直播推流——flv 编码与音视频时间戳同步](https://juejin.im/post/5a57272f6fb9a01cb0493ca0 "https://juejin.im/post/5a57272f6fb9a01cb0493ca0")

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
            Log.d(TAG, "danmaku msg: " + danmakuBeanList.get(0).getMessage());
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

## **AvcPackager**

{% codeblock lang:java %}
public static class AvcPackager {
    public static byte[] generateAvcDecoderConfigurationRecord(MediaFormat mediaFormat) {
        // 获取 csd-0 缓冲区的值，该值对应 SPS；csd-1 对应 PPS
        ByteBuffer spsByteBuff = mediaFormat.getByteBuffer("csd-0");
        ByteBuffer ppsByteBuff = mediaFormat.getByteBuffer("csd-1");
        // 跳过 4 个字节
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
        ByteArrayTools.intToTwoByteArray(result, 6, spsLength);
        int pos = 8 + spsLength;
        result[pos] = (byte) 0x01;// numOfPictureParameterSets，实际测试时发现总为0x01
        // result 数组第 8+spsLength+1 位放置 ppsLength 的高 8 位
        // result 数组第 8+spsLength+2 位放置 ppsLength 的低 8 位
        ByteArrayTools.intToTwoByteArray(result, pos + 1, ppsLength);
        return result;
    }
}
{% endcodeblock %}

### **generateAvcDecoderConfigurationRecord(MediaFormat mediaFormat)**

按`FLV`要求的`AVC`的`AVCDecoderConfigurationRecord`格式编码。

![](http://otkw6sse5.bkt.clouddn.com/ScreenRecorder-%E4%BB%A3%E7%A0%81%E8%A7%A3%E8%AF%BB1519666280252_1.png "Android 中 codec-specific data（编解码特征数据）")

其中可以看到`AVC`格式的`SPS`、`PPS`分别对应`csd-0`和`csd-1`。

## **FlvPackager**

{% codeblock lang:java %}
public static class FlvPackager {
    public static final int FLV_TAG_HEADER_LENGTH = 11;
    public static final int FLV_VIDEO_TAG_HEADER_LENGTH = 5;
    public static final int FLV_AUDIO_TAG_HEADER_LENGTH = 2;
    public static final int FLV_TAG_FOOTER_LENGTH = 4;
    public static final int NALU_HEADER_LENGTH = 4;

    public static void setAvcTag(byte[] dst, int pos, boolean isAvcSequenceHeader, boolean isIdr, int readDataLength) {
        // 高 4 位表示 FrameType：1 为关键帧，2 为非关键帧
        // 低 4 位表示 CodecID：7 为 AVC
        dst[pos] = isIdr ? (byte) 0x17 : (byte) 0x27;// FrameType | CodecID
        // 0 为 AVCDecoderConfigurationRecord，1 为 AVC NALU
        dst[pos + 1] = isAvcSequenceHeader ? (byte) 0x00 : (byte) 0x01;// AVCPacketType
        dst[pos + 2] = (byte) 0x00;// CompositionTime
        dst[pos + 3] = (byte) 0x00;// CompositionTime
        dst[pos + 4] = (byte) 0x00;// CompositionTime
        if (!isAvcSequenceHeader) {
            // 4 个字节的 readDataLength
            ByteArrayTools.intToFourByteArray(dst, pos + 5, readDataLength);
        }
    }

    public static void setAacTag(byte[] dst, int pos, boolean isAACSequenceHeader) {
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

### **setAvcTag(byte[] dst, int pos, boolean isAvcSequenceHeader, boolean isIdr, int readDataLength)**

按`FLV`要求的`AVC`的`TAG body`格式编码。

### **setAacTag(byte[] dst, int pos, boolean isAACSequenceHeader)**

按`FLV`要求的`AAC`的`TAG body`格式编码。

---

# **ScreenRecorder**

获取数据，并按照`FLV`封装格式编码。

{% codeblock lang:java %}
public class ScreenRecorder extends Thread {
    private static final String TAG = "ScreenRecorder";

    private static final String MIME_TYPE = "video/avc"; // H.264 Advanced Video Coding
    private static final int FRAME_RATE = 30; // 30fps
    private static final int IFRAME_INTERVAL = 2; // 2s between I-frames
    private static final int TIMEOUT_US = 10000;

    private ResFlvDataCollecter mDataCollecter;
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

    public ScreenRecorder(ResFlvDataCollecter dataCollecter, int width, int height, int bitRate, int dpi, MediaProjection mediaProjection) {
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
                            realData.position(bufferInfo.offset + 4);// 跳过用于表示上个 TAG 大小的 4 个字节
                            realData.limit(bufferInfo.offset + bufferInfo.size);
                            sendAvcData((bufferInfo.presentationTimeUs / 1000) - startTime, realData);
                        }
                    }
                    mediaCodec.releaseOutputBuffer(eobIndex, false);
                    break;
            }
        }
    }

    private void sendAvcDecoderConfigurationRecord(long timeMs, MediaFormat format) {
        byte[] avcDecoderConfigurationRecord = Packager.AvcPackager.generateAvcDecoderConfigurationRecord(format);
        int packetLen = Packager.FlvPackager.FLV_VIDEO_TAG_HEADER_LENGTH + avcDecoderConfigurationRecord.length;
        byte[] finalBuff = new byte[packetLen];
        // 在 finalBuff 数组最前面插入 5 个字节的 FLV video TAG header
        Packager.FlvPackager.setAvcTag(finalBuff, 0, true, true, avcDecoderConfigurationRecord.length);
        // 将 avcDecoderConfigurationRecord 数组的数据拼接到 finalBuff 数组后面
        System.arraycopy(avcDecoderConfigurationRecord, 0, finalBuff, Packager.FlvPackager.FLV_VIDEO_TAG_HEADER_LENGTH, avcDecoderConfigurationRecord.length);
        ResFlvData resFlvData = new ResFlvData();
        resFlvData.droppable = false;
        resFlvData.byteBuffer = finalBuff;
        resFlvData.size = finalBuff.length;
        resFlvData.dts = (int) timeMs;
        resFlvData.flvTagType = FLV_TAGTYPE_VIDEO;
        resFlvData.videoFrameType = ResFlvData.AVC_NALU_TYPE_IDR;
        mDataCollecter.collect(resFlvData, FLV_TAGTYPE_VIDEO);
    }

    private void sendAvcData(long timeMs, ByteBuffer data) {
        int realDataLength = data.remaining();
        int packetLen = Packager.FlvPackager.FLV_VIDEO_TAG_HEADER_LENGTH + Packager.FlvPackager.NALU_HEADER_LENGTH + realDataLength;
        byte[] finalBuff = new byte[packetLen];
        // 将 data 数组的数据放到 finalBuff 数组的第 Packager.FlvPackager.FLV_VIDEO_TAG_HEADER_LENGTH+Packager.FlvPackager.NALU_HEADER_LENGTH+1 个位置到结束
        data.get(finalBuff, Packager.FlvPackager.FLV_VIDEO_TAG_HEADER_LENGTH + Packager.FlvPackager.NALU_HEADER_LENGTH, realDataLength);
        // 获取 NALU 类型，计算后得 5 表示 NALU 类型是 AVC_NALU_TYPE_IDR
        int frameType = finalBuff[Packager.FlvPackager.FLV_VIDEO_TAG_HEADER_LENGTH + Packager.FlvPackager.NALU_HEADER_LENGTH] & 0x1F;
        // 在 finalBuff 数组最前面插入 9 个字节的 FLV video TAG（含 5 个字节的 header 和 4 个字节的 length）
        Packager.FlvPackager.setAvcTag(finalBuff, 0, false, frameType == 5, realDataLength);
        ResFlvData resFlvData = new ResFlvData();
        resFlvData.droppable = true;
        resFlvData.byteBuffer = finalBuff;
        resFlvData.size = finalBuff.length;
        resFlvData.dts = (int) timeMs;
        resFlvData.flvTagType = FLV_TAGTYPE_VIDEO;
        resFlvData.videoFrameType = frameType;
        mDataCollecter.collect(resFlvData, FLV_TAGTYPE_VIDEO);
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

获取已解码的输出缓冲的索引，或者一个状态值。当是状态值`MediaCodec.INFO_OUTPUT_FORMAT_CHANGED`时，生成并发送`AVC`的`AVCDecoderConfigurationRecord`；否则，生成并发送`AVC`数据。

## **sendAvcDecoderConfigurationRecord(long timeMs, MediaFormat format)**

生成并发送`AVC`的`AVCDecoderConfigurationRecord`。

## **sendAvcData(long timeMs, ByteBuffer data)**

生成并发送`AVC`数据。

---

# **FlvMetaData**

按`FLV`要求的`onMetaData`格式编码。

{% codeblock lang:Java %}
    public class FlvMetaData {
        private static final String NAME = "onMetaData";
        private static final int CODEC_ID_AAC = 7;
        private static final int CODEC_ID_AVC = 10;
        private static final int TYPE_NUMBER = 0;
        private static final int TYPE_STRING = 2;
        private static final int TYPE_ECMA_ARRAY = 8;
        private static final int EMPTY_SIZE = 21;
        private static final byte[] END_MARKER = {0x00, 0x00, 0x09};
    
        private int dataSize;
        private byte[] metaData;
        private ArrayList<byte[]> metaDataList;
        private int pointer;
    
        public FlvMetaData() {
            metaDataList = new ArrayList<>();
            dataSize = 0;
        }
    
        public FlvMetaData(ResCoreParameters parameters) {
            this();
            // audio
            setProperty("audiocodecid", CODEC_ID_AAC);
            switch (parameters.mediacodecAACBitRate) {
                case 32 * 1024:
                    setProperty("audiodatarate", 32);
                    break;
                case 48 * 1024:
                    setProperty("audiodatarate", 48);
                    break;
                case 64 * 1024:
                    setProperty("audiodatarate", 64);
                    break;
                default:
                    LogTools.e("不支持的 audiodatarate: " + parameters.mediacodecAACBitRate);
                    break;
            }
            switch (parameters.mediacodecAACSampleRate) {
                case 44100:
                    setProperty("audiosamplerate", 44100);
                    break;
                default:
                    LogTools.e("不支持的 audiosamplerate: " + parameters.mediacodecAACSampleRate);
                    break;
            }
            // video
            setProperty("videocodecid", CODEC_ID_AVC);
            setProperty("framerate", parameters.mediacodecAVCFrameRate);
            setProperty("width", parameters.videoWidth);
            setProperty("height", parameters.videoHeight);
        }
    
        private void setProperty(String key, int value) {
            addProperty(toFlvBytes(key), (byte) TYPE_NUMBER, toFlvBytes(value));
        }
    
        private void setProperty(String key, String value) {
            addProperty(toFlvBytes(key), (byte) TYPE_STRING, toFlvBytes(value));
        }
    
        private void addProperty(byte[] key, byte dataType, byte[] value) {
            int propertySize = key.length + 1 + value.length;
            byte[] property = new byte[propertySize];
    
            // property 数组的前 key.length 个元素是 key 数组
            System.arraycopy(key, 0, property, 0, key.length);
            // property 数组的第 key.length+1 个元素是 dataType
            property[key.length] = dataType;
            // property 数组的后 value.length 个元素是 value 数组
            System.arraycopy(value, 0, property, key.length + 1, value.length);
    
            metaDataList.add(property);
            dataSize += propertySize;
        }
    
        /**
         * 将 double 类型的值转为 8 字节的数组
         *
         * @param value
         * @return
         */
        private byte[] toFlvBytes(double value) {
            long l = Double.doubleToLongBits(value);
            return toUI(l, 8);
        }
    
        /**
         * 将 String 类型的值转为 值长度+2 字节的数组，其中前 2 个字节表示数组的长度，后面的是数据
         *
         * @param value
         * @return
         */
        private byte[] toFlvBytes(String value) {
            byte[] bytes = new byte[value.length() + 2];
            // bytes 数组的前 2 个元素表示 value 的长度
            System.arraycopy(toUI(value.length(), 2), 0, bytes, 0, 2);
            // bytes 数组第 3 个元素到最后一个元素是数据
            System.arraycopy(value.getBytes(), 0, bytes, 2, value.length());
            return bytes;
        }
    
        /**
         * 将 value 转为指定字节数的数组
         *
         * @param value
         * @param length
         * @return
         */
        private byte[] toUI(long value, int length) {
            byte[] bytes = new byte[length];
            for (int i = 0; i < length; i++) {
                bytes[length - 1 - i] = (byte) (value >> (8 * i) & 0xff);
            }
            return bytes;
        }
    
        /**
         * 构建 SCRIPT TAG
         *
         * @return
         */
        public byte[] getMetaData() {
            metaData = new byte[dataSize + EMPTY_SIZE];
            pointer = 0;
            // 设置下一个内容的类型
            addByte(TYPE_STRING);// 1 个字节
            addByteArray(toFlvBytes(NAME));// 12 个字节
            // 设置下一个内容的类型
            addByte(TYPE_ECMA_ARRAY);// 1 个字节
            addByteArray(toUI(metaDataList.size(), 4));// 4 个字节
            for (byte[] property : metaDataList) {
                addByteArray(property);// property长度 个字节
            }
            addByteArray(END_MARKER);// 3 个字节
            return metaData;
        }
    
        private void addByte(int value) {
            metaData[pointer] = (byte) value;
            pointer++;
        }
    
        private void addByteArray(byte[] value) {
            System.arraycopy(value, 0, metaData, pointer, value.length);
            pointer += value.length;
        }
    }
{% endcodeblock %}

---

# **screenrecorderrtmp**

通过和`librtmp`C库交互，实现数据流读/写。

{% codeblock lang:C %}
#include <jni.h>
#include <screenrecorderrtmp.h>
#include <malloc.h>
#include "rtmp.h"

/*
 * Class:     net_yrom_screenrecorder_rtmp_RtmpClient
 * Method:    open
 * Signature: (Ljava/lang/String;Z)J
 */
JNIEXPORT jlong JNICALL Java_net_yrom_screenrecorder_rtmp_RtmpClient_open(JNIEnv * env, jobject thiz, jstring url_, jboolean isPublishMode) {
 	LOGD("Java_net_yrom_screenrecorder_rtmp_RtmpClient_open(%s, %b)", url_, isPublishMode);
   	const char *url = (*env)->GetStringUTFChars(env, url_, 0);

    // 创建一个 RTMP 会话的句柄
   	RTMP* rtmp = RTMP_Alloc();
   	if (rtmp == NULL) {
   		LOGD("rtmp == NULL");
   		return NULL;
   	}
   	// 初始化句柄
   	RTMP_Init(rtmp);

    // 设置参数
   	int ret = RTMP_SetupURL(rtmp, url);
   	if (!ret) {
   	    // 清理会话
   		RTMP_Free(rtmp);
   		rtmp = NULL;
   		LOGD("RTMP_SetupURL: %d", ret);
   		return NULL;
   	}

   	if (isPublishMode) {
   	    // 设置可写
   		RTMP_EnableWrite(rtmp);
   	}

    // 建立 RTMP 链接中的网络连接
   	ret = RTMP_Connect(rtmp, NULL);
   	if (!ret) {
   	    // 清理会话
   		RTMP_Free(rtmp);
   		rtmp = NULL;
   		LOGD("RTMP_Connect: %d", ret);
   		return NULL;
   	}

    // 建立 RTMP 链接中的网络流
   	ret = RTMP_ConnectStream(rtmp, 0);
   	if (!ret) {
   	    // 关闭 RTMP 链接
   		RTMP_Close(rtmp);
   	    // 清理会话
   		RTMP_Free(rtmp);
   		rtmp = NULL;
   		LOGD("RTMP_ConnectStream: %d", ret);
   		return NULL;
   	}

   	(*env)->ReleaseStringUTFChars(env, url_, url);
   	LOGD("RTMP_OPENED");
   	return rtmp;
}

/*
 * Class:     net_yrom_screenrecorder_rtmp_RtmpClient
 * Method:    read
 * Signature: (J[BII)I
 */
JNIEXPORT jint JNICALL Java_net_yrom_screenrecorder_rtmp_RtmpClient_read(JNIEnv * env, jobject thiz, jlong rtmp, jbyteArray data_, jint offset, jint size) {
 	LOGD("Java_net_yrom_screenrecorder_rtmp_RtmpClient_read(%d, %d, %d)", rtmp, offset, size);
 	char* data = malloc(size*sizeof(char));
 	int readCount = RTMP_Read((RTMP*)rtmp, data, size);
 	if (readCount > 0) {
        (*env)->SetByteArrayRegion(env, data_, offset, readCount, data);
    }
    free(data);
    return readCount;
}

/*
 * Class:     net_yrom_screenrecorder_rtmp_RtmpClient
 * Method:    write
 * Signature: (J[BIII)I
 */
JNIEXPORT jint JNICALL Java_net_yrom_screenrecorder_rtmp_RtmpClient_write(JNIEnv * env, jobject thiz, jlong rtmp, jbyteArray data, jint size, jint type, jint ts) {
 	LOGD("Java_net_yrom_screenrecorder_rtmp_RtmpClient_write(%d, %d, %d, %d)", rtmp, size, type, ts);
 	RTMPPacket *packet = (RTMPPacket*)malloc(sizeof(RTMPPacket));
 	RTMPPacket_Alloc(packet, size);
 	RTMPPacket_Reset(packet);
 	// 设置 Chunk Stream ID
    if (type == RTMP_PACKET_TYPE_INFO) {// metadata
    	packet->m_nChannel = 0x03;// 自定义 Chunk Stream ID
    } else if (type == RTMP_PACKET_TYPE_VIDEO) {// video
    	packet->m_nChannel = 0x04;// 自定义 Chunk Stream ID
    } else if (type == RTMP_PACKET_TYPE_AUDIO) {// audio
    	packet->m_nChannel = 0x05;// 自定义 Chunk Stream ID
    } else {
    	packet->m_nChannel = -1;
    }

    // 设置 Stream ID
    packet->m_nInfoField2  =  ((RTMP*)rtmp)->m_stream_id;
 	LOGD("((RTMP*)rtmp)->m_stream_id: %d", ((RTMP*)rtmp)->m_stream_id);

 	jbyte *buffer = (*env)->GetByteArrayElements(env, data, NULL);
    memcpy(packet->m_body, buffer, size);
    // 设置 Message Header 的格式和长度：0
    packet->m_headerType = RTMP_PACKET_SIZE_LARGE;
    // 设置是否使用绝对时间戳：不使用
    packet->m_hasAbsTimestamp = FALSE;
    // 设置时间戳
    packet->m_nTimeStamp = ts;
    // 设置包类型
    packet->m_packetType = type;
    // 设置消息长度
    packet->m_nBodySize  = size;

    int ret = RTMP_SendPacket((RTMP*)rtmp, packet, 0);
    if (!ret) {
    	LOGD("end write error: %d", ret);
		return ret;
    }else{
    	LOGD("end write success");
		return 0;
    }

    RTMPPacket_Free(packet);
    free(packet);
    (*env)->ReleaseByteArrayElements(env, data, buffer, 0);
}

/*
 * Class:     net_yrom_screenrecorder_rtmp_RtmpClient
 * Method:    close
 * Signature: (J)I
 */
JNIEXPORT jint JNICALL Java_net_yrom_screenrecorder_rtmp_RtmpClient_close(JNIEnv * env, jlong rtmp, jobject thiz) {
 	LOGD("Java_net_yrom_screenrecorder_rtmp_RtmpClient_close(%d)", rtmp);
 	RTMP_Close((RTMP*)rtmp);
 	RTMP_Free((RTMP*)rtmp);
 	return 0;
}

/*
 * Class:     net_yrom_screenrecorder_rtmp_RtmpClient
 * Method:    getIpAddr
 * Signature: (J)Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_net_yrom_screenrecorder_rtmp_RtmpClient_getIpAddr(JNIEnv * env, jobject thiz, jlong rtmp) {
 	LOGD("Java_net_yrom_screenrecorder_rtmp_RtmpClient_getIpAddr(%d)", rtmp);
	if(rtmp!=0){
		RTMP* r= (RTMP*)rtmp;
		return (*env)->NewStringUTF(env, r->ipaddr);
	}else {
		return (*env)->NewStringUTF(env, "");
	}
}
{% endcodeblock %}

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
    private LinkedBlockingDeque<ResFlvData> frameQueue = new LinkedBlockingDeque<>(MAX_QUEUE_CAPACITY);
    private AtomicBoolean quit = new AtomicBoolean(false);
    private FlvMetaData flvMetaData;
    private ResCoreParameters coreParameters;

    private long jniRtmpPointer = 0;
    private int writeMsgNum = 0;

    public RtmpStreamingSender() {
        coreParameters = new ResCoreParameters();
        coreParameters.mediacodecAACBitRate = ResFlvData.AAC_BITRATE;
        coreParameters.mediacodecAACSampleRate = ResFlvData.AAC_SAMPLE_RATE;
        coreParameters.mediacodecAVCFrameRate = ResFlvData.FPS;
        coreParameters.videoWidth = ResFlvData.VIDEO_WIDTH;
        coreParameters.videoHeight = ResFlvData.VIDEO_HEIGHT;

        flvMetaData = new FlvMetaData(coreParameters);
    }

    @Override
    public void run() {
        while (!quit.get()) {
            if (frameQueue.size() > 0) {
                switch (state) {
                    case STATE.START:
                        if (TextUtils.isEmpty(mRtmpAddr)) {
                            LogTools.e("rtmp address is null!");
                            break;
                        }

                        // 建立 RTMP 链接
                        jniRtmpPointer = RtmpClient.open(mRtmpAddr, true);
                        if (jniRtmpPointer != 0) {
                            // 获取地址
                            String serverIpAddr = RtmpClient.getIpAddr(jniRtmpPointer);
                            LogTools.d("server ip address: " + serverIpAddr);

                            // 发送 onMetaData TAG
                            byte[] metaData = flvMetaData.getMetaData();
                            RtmpClient.write(jniRtmpPointer, metaData, metaData.length, ResFlvData.FLV_TAGTYPE_SCRIPT_DATA, 0);

                            state = STATE.RUNNING;
                        }
                        break;
                    case STATE.RUNNING:
                        synchronized (lock) {
                            --writeMsgNum;
                        }
                        ResFlvData flvData = frameQueue.pop();

                        // 如果发送队列中的数据过多，在条件满足的情况下，忽略这个数据
                        if (writeMsgNum >= (MAX_QUEUE_CAPACITY * 2 / 3) && flvData.flvTagType == ResFlvData.FLV_TAGTYPE_VIDEO && flvData.droppable) {
                            LogTools.d("senderQueue is crowded, abort a frame");
                            break;
                        }

                        // 发送 AVC TAG
                        int result = RtmpClient.write(jniRtmpPointer, flvData.byteBuffer, flvData.byteBuffer.length, flvData.flvTagType, flvData.dts);
                        if (result == 0) {
                            switch (flvData.flvTagType) {
                                case ResFlvData.FLV_TAGTYPE_VIDEO:
                                    LogTools.d("video frame sent: " + flvData.size);
                                    break;
                                case ResFlvData.FLV_TAGTYPE_AUDIO:
                                    LogTools.d("audio frame sent: " + flvData.size);
                                    break;
                            }
                        } else {
                            LogTools.e("write error: " + result);
                        }
                        break;
                    case STATE.STOPPED:
                        // 关闭 RTMP 链接
                        result = RtmpClient.close(jniRtmpPointer);
                        LogTools.d("close result: " + result);
                        break;
                }
            }
        }
        int result = RtmpClient.close(jniRtmpPointer);
        LogTools.e("close result: " + result);
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

    public void sendFood(ResFlvData flvData, int type) {
        synchronized (lock) {
            if (writeMsgNum <= MAX_QUEUE_CAPACITY) {
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
    private ResAudioClient audioClient;
    private ResCoreParameters coreParameters;
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

        coreParameters = new ResCoreParameters();
        audioClient = new ResAudioClient(coreParameters);
        if (!audioClient.prepare()) {
            LogTools.d("!!!!!audioClient.prepare() failed");
            return;
        }

        streamingSender = new RtmpStreamingSender();
        streamingSender.sendStart(rtmpAddr);
        ResFlvDataCollecter collecter = new ResFlvDataCollecter() {
            @Override
            public void collect(ResFlvData flvData, int type) {
                streamingSender.sendFood(flvData, type);
            }
        };
        screenRecorder = new ScreenRecorder(collecter, ResFlvData.VIDEO_WIDTH, ResFlvData.VIDEO_HEIGHT, ResFlvData.VIDEO_BITRATE, 1, mediaProjection);
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

向输入框中的地址进行`RTMP`推流。

## **startScreenCapture()**

开始屏幕捕捉。

## **stopScreenRecord()**

停止屏幕捕捉。

## **startScreenRecordService()**

开始监听推流进度。

## **stopScreenRecordService()**

停止监听推流进度。

---