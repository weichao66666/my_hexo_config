---
layout: article
title: ffmpeg 命令常用功能
date: 2018-05-12 20:47:25
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **ffmpeg 命令参数组成**

## **信息查询部分**

### **查看支持的视频文件格式列表**

1、输入命令：

```shell
ffmpeg -formats
```

2、输出结果（只截取了部分）：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD1.png)

3、输出内容包括：

列数|说明
--|--
1|包含 2 个字段：<br>（1）是否支持 Demuxing 多媒体文件封装格式<br>（2）是否支持 Muxing 多媒体文件封装格式
2|多媒体文件格式
3|文件格式的详细说明

### **查看支持的编码列表**

1、输入命令：

```shell
ffmpeg -encoders
```

2、输出结果（只截取了部分）：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD2.png)

3、输出内容包括：

列数|说明
--|--
1|包含 6 个字段：<br>（1）编码类型是视频（V）、音频（A）、字幕（S）<br>（2）是否支持帧级别的多线程<br>（3）是否支持分片级别的多线程<br>（4）是否为实验版本<br>（5）是否支持 draw horiz band 模式<br>（6）是否支持直接渲染
2|编码格式
3|编码格式的说明

### **查看支持的解码列表**

1、输入命令：

```shell
ffmpeg -decoders
```

2、输出结果（只截取了部分）：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD3.png)

3、输出内容包括：

列数|说明
--|--
1|包含 6 个字段：<br>（1）解码类型是视频（V）、音频（A）、字幕（S）<br>（2）是否支持帧级别的多线程<br>（3）是否支持分片级别的多线程<br>（4）是否为实验版本<br>（5）是否支持 draw horiz band 模式<br>（6）是否支持直接渲染
2|解码格式
3|解码格式的说明

### **查看支持的滤镜列表**

1、输入命令：

```shell
ffmpeg -filters
```

2、输出结果（只截取了部分）：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD4.png)

3、输出内容包括：

列数|说明
--|--
1|包含 3 个字段：<br>（1）是否支持时间轴<br>（2）是否支持分片级别的多线程<br>（3）是否支持命令
2|滤镜名
3|转换方式：音频转音频（A->A）、视频转视频（V->V）等
4|滤镜作用说明

### **查看 FLV 封装支持的参数**

1、输入命令：

```shell
ffmpeg -h muxer=flv
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD5.png)

3、输出内容包括：

部分|说明
--|--
1|包括 4 个内容：<br>（1）扩展名<br>（2）MIME 类型<br>（3）默认的视频编码格式<br>（4）默认的音频编码格式
2|封装时支持的配置参数和说明

### **查看 FLV 解封装支持的参数**

1、输入命令：

```shell
ffmpeg -h demuxer=flv
```

2、输出结果（只截取了部分）：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD6.png)

3、输出内容包括：

部分|说明
--|--
1|扩展名
2|解封装时支持的配置参数和说明

### **查看 AVC 编码支持的参数**

1、输入命令：

```shell
ffmpeg -h encoder=h264
```

2、输出结果（只截取了部分）：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD7.png)

3、输出内容包括：

部分|说明
--|--
1|包括 3 个内容：<br>（1）支持的基本编码方式<br>（2）支持的多线程编码方式<br>（3）支持的像素的色彩格式
2|编码时支持的配置参数和说明

### **查看 AVC 解码支持的参数**

1、输入命令：

```shell
ffmpeg -h decoder=h264
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD8.png)

3、输出内容包括：

部分|说明
--|--
1|包括 2 个内容：<br>（1）支持的基本解码方式<br>（2）支持的多线程解码方式
2|解码时支持的配置参数和说明

### **查看 colorkey 滤镜支持的参数**

1、输入命令：

```shell
ffmpeg -h filter=colorkey
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD9.png)

3、输出内容包括：

部分|说明
--|--
1|包括 2 个内容：<br>（1）支持的色彩格式<br>（2）支持的多线程处理方式<br>（3）支持的输入和输出
2|支持的参数和说明

## **公共操作部分**

## **文件操作部分**

### **拆分出视频**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -vcodec copy -vbsf h264_mp4toannexb -an output.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD21.png)

### **拆分出音频**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -acodec copy -vn output.aac
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD22.png)

### **视频和音频合成**

1、输入命令：

```shell
ffmpeg -i output.mp4 -i output.aac -vcodec copy -acodec copy -absf aac_adtstoasc test.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD23.png)

### **切片**

segment 参数：

参数|类型|说明
--|--
reference_stream|字符串|切片参考用的 stream
segment_format|字符串|切片文件格式
segment_format_options|字符串|切片格式的私有操作参数
segment_list|字符串|切片列表主文件名
segment_list_flags|标签|m3u8 切片的存在形式：<br>live<br>cache
segment_list_size|整数|列表文件的长度
segment_list_type|标签|列表类型：<br>flat<br>csv<br>ext<br>ffconcat<br>m3u8<br>hls
segment_atclocktime|布尔|时钟频率生效参数，启动定时切片间隔用
segment_clocktime_offset|时间值|切片时钟偏移
segment_clocktime_wrap_duration|时间值|切片时钟回滚 duration
segment_time|字符串|切片的 duration
segment_time_delta|时间值|用于设置切片变化时间值
segment_times|字符串|设置切片的时间点
segment_frames|字符串|设置切片的帧位置
segment_wrap|整数|列表回滚阈值
segment_list_entry_prefix|字符串|写文件列表时写入每个切片路径的前置路径
segment_start_number|整数|列表中切片的起始基数
strftime|布尔|设置切片名为生成切片的时间点
break_non_keyframes|布尔|忽略关键帧，按照时间切片
individual_header_trailer|布尔|默认在每个切片中都写入文件头和文件结束容器
write_header_trailer|布尔|只在第一个文件写入文件头以及在最后一个文件写入文件结束容器
reset_timestamps|布尔|每个切片都重新初始化时间戳
initial_offset|时间值|设置初始化时间戳偏移

#### **segment_format**

HLS 切片格式是 MPEGTS，此处可以指定为其他格式，比如 MP4、FLV 等。
切割出来的 MP4 切片文件的时间戳与上一个 MP4 的结束时间戳是连续的。

1、输入命令：

```shell
ffmpeg -re -i guibu.mp4 -c copy -f segment -segment_format mp4 test_output-%d.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD31.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD32.png)

3、生成的文件：

test_output-0.mp4～test_output-14.mp4

#### **segment_list && segment_list_type**

##### **ffconcat**

生成 ffconcat 格式的文件索引列表。

1、输入命令：

```shell
ffmpeg -re -i guibu.mp4 -c copy -f segment -segment_format mp4 -segment_list_type ffconcat -segment_list output.lst test-output-%d.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD33.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD34.png)

3、生成的文件：

test_output-0.mp4～test_output-14.mp4
output.lst

4、output.lst 的内容：

```xml
ffconcat version 1.0
file test-output-0.mp4
file test-output-1.mp4
file test-output-2.mp4
file test-output-3.mp4
file test-output-4.mp4
file test-output-5.mp4
file test-output-6.mp4
file test-output-7.mp4
file test-output-8.mp4
file test-output-9.mp4
file test-output-10.mp4
file test-output-11.mp4
file test-output-12.mp4
file test-output-13.mp4
file test-output-14.mp4
```

##### **flat**

生成 FLAT 格式的文件索引列表。

1、输入命令：

```shell
ffmpeg -re -i guibu.mp4 -c copy -f segment -segment_format mp4 -segment_list_type flat -segment_list output.txt test-output-%d.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD35.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD36.png)

3、生成的文件：

test_output-0.mp4～test_output-14.mp4
output.txt

4、output.txt 的内容：

```xml
test-output-0.mp4
test-output-1.mp4
test-output-2.mp4
test-output-3.mp4
test-output-4.mp4
test-output-5.mp4
test-output-6.mp4
test-output-7.mp4
test-output-8.mp4
test-output-9.mp4
test-output-10.mp4
test-output-11.mp4
test-output-12.mp4
test-output-13.mp4
test-output-14.mp4
```

##### **csv**

生成 CSV 格式的文件索引列表。

1、输入命令：

```shell
ffmpeg -re -i guibu.mp4 -c copy -f segment -segment_format mp4 -segment_list_type csv -segment_list output.csv test-output-%d.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD37.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD38.png)

3、生成的文件：

test_output-0.mp4～test_output-14.mp4
output.csv

4、output.csv 的内容：

```xml
test-output-0.mp4,0.000000,10.080000
test-output-1.mp4,10.080000,16.680000
test-output-2.mp4,16.680000,26.680000
test-output-3.mp4,26.680000,36.680000
test-output-4.mp4,36.680000,46.680000
test-output-5.mp4,46.680000,56.680000
test-output-6.mp4,56.680000,66.680000
test-output-7.mp4,66.680000,71.200000
test-output-8.mp4,71.200000,79.440000
test-output-9.mp4,79.440000,89.440000
test-output-10.mp4,89.440000,99.240000
test-output-11.mp4,99.240000,109.240000
test-output-12.mp4,109.240000,114.520000
test-output-13.mp4,114.520000,124.520000
test-output-14.mp4,124.520000,126.520000
```

##### **m3u8**

生成 M3U8 格式的文件索引列表。

1、输入命令：

```shell
ffmpeg -re -i guibu.mp4 -c copy -f segment -segment_format mp4 -segment_list_type m3u8 -segment_list output.m3u8 test-output-%d.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD39.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD40.png)

3、生成的文件：

test_output-0.mp4～test_output-14.mp4
output.m3u8

4、output.m3u8 的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-ALLOW-CACHE:YES
#EXT-X-TARGETDURATION:11
#EXTINF:10.080000,
test-output-0.mp4
#EXTINF:6.600000,
test-output-1.mp4
#EXTINF:10.000000,
test-output-2.mp4
#EXTINF:10.000000,
test-output-3.mp4
#EXTINF:10.000000,
test-output-4.mp4
#EXTINF:10.000000,
test-output-5.mp4
#EXTINF:10.000000,
test-output-6.mp4
#EXTINF:4.520000,
test-output-7.mp4
#EXTINF:8.240000,
test-output-8.mp4
#EXTINF:10.000000,
test-output-9.mp4
#EXTINF:9.800000,
test-output-10.mp4
#EXTINF:10.000000,
test-output-11.mp4
#EXTINF:5.280000,
test-output-12.mp4
#EXTINF:10.000000,
test-output-13.mp4
#EXTINF:2.000000,
test-output-14.mp4
#EXT-X-ENDLIST
```

#### **reset_timestamps**

使每一片切片的时间戳归 0。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f segment -segment_format mp4 -reset_timestamps 1 test_output-%d.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD41.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD42.png)

3、生成的文件：

test_output-0.mp4～test_output-22.mp4

#### **segment_times**

按照指定的时间点切片，比如第 1 分钟、第 2 分钟、第 3 分钟。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f segment -segment_format mp4 -segment_times 60,120,180 test_output-%d.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD43.png)

3、生成的文件：

test_output-0.mp4～test_output-2.mp4

#### **ss**

指定剪切开头部分。
比如从一个视频文件的第 10 秒钟开始截取内容。

1、输入命令：

```shell
ffmpeg -ss 10 -i guibu.flv -c copy output.ts
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD44.png)

#### **t**

指定视频总长度。
比如只截取一个视频文件的前 10 秒。

1、输入命令：

```shell
ffmpeg -i guibu.flv -c copy -t 10 -copyts output.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD45.png)

#### **output_ts_offset**

指定输出文件的 start_time。
比如指定从第 120 秒开始，截取 10 秒的视频。

1、输入命令：

```shell
ffmpeg -i guibu.flv -c copy -t 10 -output_ts_offset 120 output.mp4
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD46.png)

## **视频操作部分**

## **音频操作部分**

## **字幕操作部分**

---

# **ffmpeg 封装转换和编码转换**

## **封装转换**

AVFormatContext 部分参数：

参数|类型|说明
--|--
avioflags|标记<br>direct|format 的缓冲设置，默认为 0（有缓冲）<br>无缓冲
probesize|整数|在进行媒体数据处理前获得文件内容的大小
fflags|标记<br>flush_packets<br>genpts<br>nofillin<br>igndts<br>discadcorrupt<br>sortdts<br>keepside<br>fastseek<br>latm<br>nobuffer<br>bitexact|<br>立即将 packets 数据写入文件中<br>输出时按照正常规则产生 pts<br>不填写可以精确计算缺失的值<br>忽略 dts<br>丢弃损坏的帧<br>尝试以 dts 的顺序为准输出<br>不合并数据<br>快速 seek 操作，但是不够精确<br>设置 RTP MP4_LATM 生效<br>直接读取或写出，不存入 buffer，用于在直播采集时降低延迟<br>不写入随机或者不稳定的数据
seek2any|整数|支持随意位置 seek，这个 seek 不以 keyframe 为参考
analyzeduration|整数|制定解析媒体所需要花销的时间，值越大解析越准确，直播时减小该值可降低延迟
codec_whitelist|列表|设置可解析的 codec 的白名单
format_whitelist|列表|设置可解析的 format 的白名单
output_ts_offset|整数|设置输出文件的起始时间

## **编码转换**

AVCodecContext 部分参数：

参数|类型|说明
--|--
b|整数|设置音频与视频码率，默认是 200 kb/s，可单独使用 b:v 设置视频码率，使用 b:a 设置音频码率
ab|整数|设置音频的码率，默认是 128 kb/s
g|整数|设置视频 GOP 大小，默认是 12 帧一个 GOP
ar|整数|设置音频采样率，默认为 0
ac|整数|设置音频通道数，默认为 0
bf|整数|设置连续编码为 B 帧的个数，默认为 0
maxrate|整数|最大码率设置，与 bufsize 一同使用，默认为 0
minrate|整数|最小码率设置，配合 maxrate 和 bufsize 可以设置为 CBR 模式，默认为 0
bufsize|整数|设置控制码率的 buffer 大小，默认为 0
keyint_min|整数|设置关键帧最小间隔，默认为 25
sc_threshold|整数|设置场景切换支持，默认为 0
me_threshold|整数|设置运动估计阈值，默认为 0
mb_threshold|整数|设置宏块阈值，默认为 0
profile|整数|设置音视频的 profile，默认为 -99
level|整数|设置音视频的 level，默认为 -99
timecode_frame_start|整数|设置 GOP 的开始时间，需要在 non-drop-frame 默认情况下使用
channel_layout|整数|设置音频通道的布局格式
threads|整数|设置编解码工作的线程数

## **一个封装转换+编码转换的例子**

1、输入命令：

```shell
ffmpeg -i ~/sample.mkv -vcodec mpeg4 -b:v 200k -r 15 -an output.mp4
```

2、输出的信息：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD10.png)

---