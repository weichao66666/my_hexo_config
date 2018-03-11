---
layout: article
title: 使用 libstreaming 实现同屏时的代码执行流程
date: 2018-03-11 18:39:10
tags:
categories: 
copyright: true
---

# **Reference**

* [libstreaming](https://github.com/fyhertz/libstreaming "https://github.com/fyhertz/libstreaming")
* [MediaMirror](https://github.com/weichao66666/MediaMirror "https://github.com/weichao66666/MediaMirror")

---

# **概述**

实现同屏需要一个服务端和至少一个客户端。服务端是发送音视频数据的，数据来源可以是相机、屏幕捕捉等。客户端是接收音视频数据的，接收到的数据会实时在屏幕上显示。

`MediaMirror`是`libstreaming`的 demo，实现了服务端通过相机采集数据，并发送数据给客户端，客户端实时在屏幕上显示的功能。

实现同屏的操作是：
1、服务端和客户端连接同一个 AP。
2、服务端安装在 Android 6.0+ 系统上需要手动授权。
3、客户端需要修改 IP 地址指向服务端。
4、先启动服务端，再启动客户端。

`MediaMirror`基于`libstreaming`，对代码有修改，以下分析基于`MediaMirror`。

---

# **启动服务端**

## **入口 Activity**

代码很少，全部贴出来：

{% codeblock lang:java %}
public class MainActivity extends AppCompatActivity implements SurfaceHolder.Callback {
    private SurfaceView surfaceView;

    private Session session;
    private RtspServer rtspServer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        surfaceView = findViewById(R.id.surface);

        surfaceView.getHolder().addCallback(this);
        startRtspServer();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopRtspServer();
    }

    private void startRtspServer() {
        rtspServer = new RtspServer();
        rtspServer.start();
    }

    private void stopRtspServer() {
        rtspServer.stop();
    }

    private void buildSession() {
        session = SessionBuilder.getInstance()
                .setSurfaceView(surfaceView)
                .setPreviewOrientation(90)
                .setContext(this)
                .setAudioEncoder(SessionBuilder.AUDIO_AAC)
                .setAudioQuality(new AudioQuality(16000, 32000))
                .setVideoEncoder(SessionBuilder.VIDEO_H264)
                .setVideoQuality(new VideoQuality(320, 240, 20, 500000))
                .build();
    }

    /*SurfaceHolder.Callback start*/

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        buildSession();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        session.release();
    }

    /*SurfaceHolder.Callback end*/
}
{% endcodeblock %}

布局文件中只有一个普通的`SurfaceView`，在`SurfaceView`回调`surfaceCreated(SurfaceHolder holder)`方法时会调用`buildSession()`方法，`buildSession()`设置了`Session`的参数并最终调用`SessionBuilder#build()`方法进行初始化。

然后，创建了`RtspServer`实例并启动。

### **SessionBuilder#build()**

创建 H.264 视频的对应流并设置给`Session`：

{% codeblock lang:java %}
H264Stream stream = new H264Stream(mCameraFacing);
if (mContext != null) {
    stream.setSharedPreferences(PreferenceManager.getDefaultSharedPreferences(mContext));
}
session.addVideoTrack(stream);
{% endcodeblock %}

创建 AAC 音频的对应流并设置给`Session`：

{% codeblock lang:java %}
AacStream stream = new AacStream();
session.addAudioTrack(stream);
if (mContext != null) {
    stream.setPreferences(PreferenceManager.getDefaultSharedPreferences(mContext));
}
{% endcodeblock %}

#### **H264Stream 构造函数**

`H264Stream`是`VideoStream`的子类，这里调用了`VideoStream`的有参构造函数：

{% codeblock lang:java %}
super(cameraId);
{% endcodeblock %}

创建了`H264Packetizer`实例：

{% codeblock lang:java %}
packetizer = new H264Packetizer();
{% endcodeblock %}

##### **VideoStream 有参构造函数**

`VideoStream`是`MediaStream`的子类，这里调用了`MediaStream`的无参构造函数。

##### **H264Packetizer 构造函数**

`H264Packetizer`是`AbstractPacketizer`的子类，这里调用了`AbstractPacketizer`的无参构造函数。

###### **AbstractPacketizer 构造函数**

创建了`RtpSocket`实例：

{% codeblock lang:java %}
rtpSocket = new RtpSocket();
{% endcodeblock %}

####### **RtpSocket 构造函数**

创建 UDP 的数据包数组：

{% codeblock lang:java %}
datagramPackets = new DatagramPacket[bufferCount];
{% endcodeblock %}

创建`MulticastSocket`实例：

{% codeblock lang:java %}
multicastSocket = new MulticastSocket();
{% endcodeblock %}

#### **AacStream 构造函数**

`AacStream`是`AudioStream`的子类，这里调用了`AudioStream`的无参构造函数。

##### **AudioStream 构造函数**

`AudioStream`是`MediaStream`的子类，这里调用了`MediaStream`的无参构造函数。

### **RtspServer#start()**

创建`RequestListener`实例：

{% codeblock lang:java %}
requestListener = new RequestListener();
{% endcodeblock %}

#### **RequestListener 构造函数**

`RequestListener`是一个线程，在构造函数中会创建`ServerSocket`实例，并开启这个线程：

{% codeblock lang:java %}
serverSocket = new ServerSocket(port);
start();
{% endcodeblock %}

##### **RequestListener#run()**

创建`WorkerThread`实例，`WorkerThread`同样是个线程，创建后立刻开启线程：

{% codeblock lang:java %}
new WorkerThread(serverSocket.accept()).start();
{% endcodeblock %}

###### **WorkerThread 有参构造函数**

创建客户端输入流的缓冲流和输出流：

{% codeblock lang:java %}
mInput = new BufferedReader(new InputStreamReader(client.getInputStream()));
mOutput = client.getOutputStream();
{% endcodeblock %}

创建`Session`实例：

{% codeblock lang:java %}
session = new Session();
{% endcodeblock %}

####### **WorkerThread#run()**

等待客户端连接，如果接收到客户端请求，解析该请求，并做出对应的响应，最后发送响应给客户端。

当客户端断开连接后，停止流传输并释放资源：

{% codeblock lang:java %}
boolean streaming = isStreaming();
session.syncStop();
if (streaming && !isStreaming()) {
    postMessage(MESSAGE_STREAMING_STOPPED);
}
session.release();

try {
    mClient.close();
} catch (IOException e) {
    e.printStackTrace();
}
{% endcodeblock %}

---

# **启动客户端**

## **入口 Activity**

代码很少，全部贴出来：

{% codeblock lang:java %}
public class MainActivity extends AppCompatActivity implements SurfaceHolder.Callback, MediaPlayer.OnPreparedListener {
    private static final String VIDEO_URL_HOST = "192.168.0.114";
    private static final String VIDEO_URL_PORT = "8086";// RtspServer.DEFAULT_RTSP_PORT

    private SurfaceView surfaceView;

    private MediaPlayer mediaPlayer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        surfaceView = findViewById(R.id.surface);

        surfaceView.getHolder().addCallback(this);
    }

    private void configureMediaPlayer(Uri videoUri) {
        if (mediaPlayer == null) {
            mediaPlayer = new MediaPlayer();
        }
        mediaPlayer.setDisplay(surfaceView.getHolder());
        mediaPlayer.setOnPreparedListener(this);
        try {
            mediaPlayer.setDataSource(this, videoUri);
            mediaPlayer.prepareAsync();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void releaseMediaPlayer() {
        surfaceView.getHolder().removeCallback(this);
        mediaPlayer.release();
    }

    /*SurfaceHolder.Callback start*/

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        StringBuilder videoUrlBuilder = new StringBuilder();
        videoUrlBuilder.append("rtsp://" + VIDEO_URL_HOST + ":" + VIDEO_URL_PORT + "/");
        Uri videoUri = Uri.parse(videoUrlBuilder.toString());
        configureMediaPlayer(videoUri);
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        releaseMediaPlayer();
    }

    /*SurfaceHolder.Callback end*/

    /*MediaPlayer.OnPreparedListener start*/

    @Override
    public void onPrepared(MediaPlayer mp) {
        mp.start();
    }

    /*MediaPlayer.OnPreparedListener end*/
}
{% endcodeblock %}

布局文件中只有一个 Android 原生的`VideoView`，`VideoView`是`SurfaceView`的子类，在`SurfaceView`回调`surfaceCreated(SurfaceHolder holder)`方法时，调用`configureMediaPlayer()`方法，该方法中会创建`MediaPlayer`实例，并在设置`MediaPlayer`的参数后开始准备，最后在`onPrepared(MediaPlayer mp)`回调中开始播放。

`VideoView`支持`RTSP`，`RTSP`请求都是自发的。

### **服务端接收到请求**

解析该请求：

{% codeblock lang:java %}
request = Request.parseRequest(mInput);
{% endcodeblock %}

并做出对应的响应：

{% codeblock lang:java %}
response = processRequest(request);
{% endcodeblock %}

最后发送响应给客户端。

{% codeblock lang:java %}
response.send(mOutput);
{% endcodeblock %}

#### **RtspServer#processRequest(Request request)**

该方法会根据不同的客户端请求，执行相应处理，并拼接响应报文，最后返回给客户端。

1、当是`DESCRIBE`请求时，调用了`handleRequest()`方法（该方法创建了`Session`，所以该请求是建立同步的入口）：

{% codeblock lang:java %}
session = handleRequest(request.uri, mClient);
{% endcodeblock %}

拼接会话描述信息：

{% codeblock lang:java %}
StringBuilder builder = new StringBuilder();
builder.append("Content-Base: " + mClient.getLocalAddress().getHostAddress() + ":" + mClient.getLocalPort() + "/")// Content-Base: 10.4.70.229:8086/
        .append("\r\n")
        .append("Content-Type: application/sdp")// Content-Type: application/sdp
        .append("\r\n");
response.attributes = builder.toString();
response.content = session.getSessionDescription();
response.status = Response.STATUS_OK;
{% endcodeblock %}

2、当是`SETUP`请求时，会将音视频通道分别映射到客户端指定的端口，如果客户端没有指定端口，则使用默认端口：

{% codeblock lang:java %}
Pattern pattern;
Matcher matcher;
int port1, port2, ssrc, trackId, src[];
String destination;

pattern = Pattern.compile("trackID=(\\w+)", Pattern.CASE_INSENSITIVE);
matcher = pattern.matcher(request.uri);

if (!matcher.find()) {
    response.status = Response.STATUS_BAD_REQUEST;
    return response;
}

trackId = Integer.parseInt(matcher.group(1));

if (!session.isTrackExist(trackId)) {
    response.status = Response.STATUS_NOT_FOUND;
    return response;
}

pattern = Pattern.compile("client_port=(\\d+)-(\\d+)", Pattern.CASE_INSENSITIVE);
matcher = pattern.matcher(request.headers.get("transport"));
IStream track = session.getTrack(trackId);

if (!matcher.find()) {
    int[] ports = track.getDestinationPorts();
    port1 = ports[0];
    port2 = ports[1];
} else {
    port1 = Integer.parseInt(matcher.group(1));
    port2 = Integer.parseInt(matcher.group(2));
}

ssrc = track.getSsrc();
src = track.getLocalPorts();
destination = session.getDestination();

track.setDestinationPorts(port1, port2);
{% endcodeblock %}

调用了`Session#syncStart()`方法，该方法用于开始同步：

{% codeblock lang:java %}
session.syncStart(trackId);
{% endcodeblock %}

拼接会话描述信息：

{% codeblock lang:java %}
StringBuilder builder = new StringBuilder();
builder.append("Transport: RTP/AVP/UDP;" + (InetAddress.getByName(destination).isMulticastAddress() ? "multicast" : "unicast")
        + ";destination=" + session.getDestination()
        + ";client_port=" + port1 + "-" + port2
        + ";server_port=" + src[0] + "-" + src[1]
        + ";ssrc=" + Integer.toHexString(ssrc)
        + ";mode=play"
)// Transport: RTP/AVP/UDP;unicast;destination=10.4.70.110;client_port=57148-57149;server_port=39848-33032;ssrc=4b5eeef9;mode=play
        .append("\r\n")
        .append("Session: " + "1185d20035702ca")// Session: 1185d20035702ca
        .append("\r\n")
        .append("Cache-Control: no-cache")// Cache-Control: no-cache
        .append("\r\n");
response.attributes = builder.toString();
response.status = Response.STATUS_OK;
{% endcodeblock %}

##### **RtspServer#handleRequest(String uri, Socket client)**

调用了`UriParser#parse()`方法，该方法用于解析客户端传递的参数：

{% codeblock lang:java %}
Session session = UriParser.parse(uri);
{% endcodeblock %}

设置会话的源地址和目的地址：

{% codeblock lang:java %}
session.setOrigin(client.getLocalAddress().getHostAddress());
if (session.getDestination() == null) {
    session.setDestination(client.getInetAddress().getHostAddress());
}
{% endcodeblock %}

##### **Session#syncStart(int id)**

开始同步流：

{% codeblock lang:java %}
stream.start();
{% endcodeblock %}

对于`H264`视频，依次调用了`H264Stream#start()`->`VideoStream#start()`->`MediaStream#start()`->`MediaStream#encodeWithMediaCodec()`->`VideoStream#encodeWithMediaCodec()`->`VideoStream#encodeWithMediaCodecMethod1()`。

###### **VideoStream#encodeWithMediaCodecMethod1()**

开始相机预览：

{% codeblock lang:java %}
camera.startPreview();
{% endcodeblock %}

调试编码参数后运行编码器：

{% codeblock lang:java %}
EncoderDebugger debugger = EncoderDebugger.debug(mSharedPreferences, mQuality.resX, mQuality.resY);
final Nv21Convertor convertor = debugger.getNv21Convertor();
mediaCodec = MediaCodec.createByCodecName(debugger.getEncoderName());
MediaFormat mediaFormat = MediaFormat.createVideoFormat("video/avc", mQuality.resX, mQuality.resY);
mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, mQuality.bitrate);
mediaFormat.setInteger(MediaFormat.KEY_FRAME_RATE, mQuality.framerate);
mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, debugger.getEncoderColorFormat());
mediaFormat.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, 1);
mediaCodec.configure(mediaFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);
mediaCodec.start();
{% endcodeblock %}

将预览回调的帧数据进行转换，再由编码器编码：

{% codeblock lang:java %}
for (int i = 0; i < 10; i++) {
    camera.addCallbackBuffer(new byte[convertor.getBufferSize()]);
}
camera.setPreviewCallbackWithBuffer(new Camera.PreviewCallback() {
    ByteBuffer[] inputBuffers = mediaCodec.getInputBuffers();

    @Override
    public void onPreviewFrame(byte[] data, Camera camera) {
        try {
            int bufferIndex = mediaCodec.dequeueInputBuffer(500000);
            if (bufferIndex >= 0) {
                inputBuffers[bufferIndex].clear();
                if (data == null) {
                    Log.e(TAG, "Symptom of the \"Callback buffer was to small\" problem...");
                } else {
                    convertor.convert(data, inputBuffers[bufferIndex]);
                }
                long now = System.nanoTime() / 1000;
                mediaCodec.queueInputBuffer(bufferIndex, 0, inputBuffers[bufferIndex].position(), now, 0);
            } else {
                Log.e(TAG, "No buffer available !");
            }
        } finally {
            VideoStream.this.camera.addCallbackBuffer(data);
        }
    }
});
{% endcodeblock %}

将数据写入打包器，启动打包器线程：

{% codeblock lang:java %}
packetizer.setInputStream(new MediaCodecInputStream(mediaCodec));
packetizer.start();
{% endcodeblock %}

####### **H264Packetizer#run()**

调用了`send()`方法发送数据。

######## **H264Packetizer#send()**

生成`NAL`单元`LENGTH`：

{% codeblock lang:java %}
fill(header, 0, NALU_LENGTH_LENGTH);
{% endcodeblock %}

生成时间戳和`NAL`单元长度：

{% codeblock lang:java %}
timestamp = ((MediaCodecInputStream) inputStream).getLastBufferInfo().presentationTimeUs * 1000L;
naluLength = inputStream.available() + 1;
{% endcodeblock %}

如果出现`NAL`单元类型为`IDR`类型，需要调用`AbstractPacketizer#send()`方法发送该数据：

{% codeblock lang:java %}
buffer = rtpSocket.requestBuffer();
rtpSocket.markNextPacket();
rtpSocket.updateTimestamp(timestamp);
System.arraycopy(mStapA, 0, buffer, RtpSocket.RTP_HEADER_LENGTH, mStapA.length);
super.send(RtpSocket.RTP_HEADER_LENGTH + mStapA.length);
{% endcodeblock %}

如果`NAL`长度低于设置的最大包大小，会调用`AbstractPacketizer#send()`方法发送该数据，否则，拆分`NAL`单元为`FU-A`单元再调用`AbstractPacketizer#send()`方法发送：

{% codeblock lang:java %}
// Small NAL unit => Single NAL unit
if (naluLength <= MAX_PACKET_SIZE - RtpSocket.RTP_HEADER_LENGTH - 2) {
    buffer = rtpSocket.requestBuffer();
    // RTP 头后面的一个字节用于表示前一个 NALU 的长度
    buffer[RtpSocket.RTP_HEADER_LENGTH] = header[4];
    fill(buffer, RtpSocket.RTP_HEADER_LENGTH + 1, naluLength - 1);
    rtpSocket.updateTimestamp(timestamp);
    rtpSocket.markNextPacket();
    super.send(naluLength + RtpSocket.RTP_HEADER_LENGTH);
    // Log.d(TAG,"----- Single NAL unit - len:"+len+" delay: "+delay);
}
// Large NAL unit => Split nal unit
else {
    // Set FU-A header
    header[1] = (byte) (header[4] & 0x1F);  // FU header type
    header[1] += 0x80; // Start bit
    // Set FU-A indicator
    header[0] = (byte) ((header[4] & 0x60) & 0xFF); // FU indicator NRI
    header[0] += 28;

    while (sum < naluLength) {
        buffer = rtpSocket.requestBuffer();
        buffer[RtpSocket.RTP_HEADER_LENGTH] = header[0];
        buffer[RtpSocket.RTP_HEADER_LENGTH + 1] = header[1];
        rtpSocket.updateTimestamp(timestamp);
        if ((len = fill(buffer, RtpSocket.RTP_HEADER_LENGTH + 2, Math.min(naluLength - sum, MAX_PACKET_SIZE - RtpSocket.RTP_HEADER_LENGTH - 2))) < 0) {
            return;
        }

        sum += len;
        // Last packet before next NAL
        // 下一个 NAL 前的最后一个包
        if (sum >= naluLength) {
            // End bit on
            buffer[RtpSocket.RTP_HEADER_LENGTH + 1] += 0x40;
            rtpSocket.markNextPacket();
        }
        super.send(len + RtpSocket.RTP_HEADER_LENGTH + 2);
        // Switch start bit
        header[1] = (byte) (header[1] & 0x7F);
        // Log.d(TAG,"----- FU-A unit, sum:"+sum);
    }
}
{% endcodeblock %}

######### **AbstractPacketizer#send(int length)**

调用了`RtpSocket#commitBuffer()`方法：

{% codeblock lang:java %}
rtpSocket.commitBuffer(length);
{% endcodeblock %}

########## **RtpSocket#commitBuffer(int length)**

调用`updateSequence()`方法，`updateSequence()`方法调用了`setLong()`方法，`setLong()`方法用于扩充数组索引长度。原来是 1 个字节表示一个索引，则最多可以表示 256 个索引，现在用连续的 2 个字节表示一个索引。

设置`UDP`包当前索引对应数据的长度：

{% codeblock lang:java %}
datagramPackets[bufferInputIndex].setLength(length);
{% endcodeblock %}

如果`UDP`包下一个索引已超过设置的缓冲大小时，从头开始替换：

{% codeblock lang:java %}
if (++bufferInputIndex >= bufferCount) {
    bufferInputIndex = 0;
}
{% endcodeblock %}

如果`RtpSocket`线程未启动，则启动该线程：

{% codeblock lang:java %}
if (mThread == null) {
    mThread = new Thread(this);
    mThread.start();
}
{% endcodeblock %}

########### **RtpSocket#run()**

调用`SenderReport#update()`方法更新数据：

{% codeblock lang:java %}
senderReport.update(datagramPackets[bufferOutputIndex].getLength(), (timestamps[bufferOutputIndex] / 100L) * (mClock / 1000L) / 10000L);
{% endcodeblock %}

忽略前 30 帧，之后发送缓冲区中的数据：

{% codeblock lang:java %}
if (count++ > 30) {
    if (transport == TRANSPORT_UDP) {
        multicastSocket.send(datagramPackets[bufferOutputIndex]);// TODO
    } else {
        sendTcp();
    }
}
{% endcodeblock %}

如果`UDP`包下一个索引已超过设置的缓冲大小时，从头开始发送：

{% codeblock lang:java %}
if (++bufferOutputIndex >= BUFFER_COUNT) {
    bufferOutputIndex = 0;
}
{% endcodeblock %}

############ **SenderReport#update(int length, long rtpTimestamp)**

更新数据，最后调用了`send()`方法：

{% codeblock lang:java %}
send(System.nanoTime(), rtpTimestamp);
{% endcodeblock %}

############ **SenderReport#send(long ntpTimestamp, long rtpTimestamp)**

发送`RTCP`数据：

{% codeblock lang:java %}
multicastSocket.send(datagramPacket);
{% endcodeblock %}

---