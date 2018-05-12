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

{% codeblock lang:shell %}
ffmpeg -formats
{% endcodeblock %}

2、输出结果（只截取了部分）：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD1.png)

3、输出内容包括：

列数|说明
--|--
1|包含 2 个字段：<br>（1）是否支持 Demuxing 多媒体文件封装格式<br>（2）是否支持 Muxing 多媒体文件封装格式
2|多媒体文件格式
3|文件格式的详细说明

### **查看支持的编码列表**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -encoders
{% endcodeblock %}

2、输出结果（只截取了部分）：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD2.png)

3、输出内容包括：

列数|说明
--|--
1|包含 6 个字段：<br>（1）编码类型是视频（V）、音频（A）、字幕（S）<br>（2）是否支持帧级别的多线程<br>（3）是否支持分片级别的多线程<br>（4）是否为实验版本<br>（5）是否支持 draw horiz band 模式<br>（6）是否支持直接渲染
2|编码格式
3|编码格式的说明

### **查看支持的解码列表**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -decoders
{% endcodeblock %}

2、输出结果（只截取了部分）：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD3.png)

3、输出内容包括：

列数|说明
--|--
1|包含 6 个字段：<br>（1）解码类型是视频（V）、音频（A）、字幕（S）<br>（2）是否支持帧级别的多线程<br>（3）是否支持分片级别的多线程<br>（4）是否为实验版本<br>（5）是否支持 draw horiz band 模式<br>（6）是否支持直接渲染
2|解码格式
3|解码格式的说明

### **查看支持的滤镜列表**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -filters
{% endcodeblock %}

2、输出结果（只截取了部分）：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD4.png)

3、输出内容包括：

列数|说明
--|--
1|包含 3 个字段：<br>（1）是否支持时间轴<br>（2）是否支持分片级别的多线程<br>（3）是否支持命令
2|滤镜名
3|转换方式：音频转音频（A->A）、视频转视频（V->V）等
4|滤镜作用说明

### **查看 FLV 封装支持的参数**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -h muxer=flv
{% endcodeblock %}

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD5.png)

3、输出内容包括：

部分|说明
--|--
1|包括 4 个内容：<br>（1）扩展名<br>（2）MIME 类型<br>（3）默认的视频编码格式<br>（4）默认的音频编码格式
2|封装时支持的配置参数和说明

### **查看 FLV 解封装支持的参数**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -h demuxer=flv
{% endcodeblock %}

2、输出结果（只截取了部分）：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD6.png)

3、输出内容包括：

部分|说明
--|--
1|扩展名
2|解封装时支持的配置参数和说明

### **查看 AVC 编码支持的参数**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -h encoder=h264
{% endcodeblock %}

2、输出结果（只截取了部分）：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD7.png)

3、输出内容包括：

部分|说明
--|--
1|包括 3 个内容：<br>（1）支持的基本编码方式<br>（2）支持的多线程编码方式<br>（3）支持的像素的色彩格式
2|编码时支持的配置参数和说明

### **查看 AVC 解码支持的参数**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -h decoder=h264
{% endcodeblock %}

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD8.png)

3、输出内容包括：

部分|说明
--|--
1|包括 2 个内容：<br>（1）支持的基本解码方式<br>（2）支持的多线程解码方式
2|解码时支持的配置参数和说明

### **查看 colorkey 滤镜支持的参数**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -h filter=colorkey
{% endcodeblock %}

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD9.png)

3、输出内容包括：

部分|说明
--|--
1|包括 2 个内容：<br>（1）支持的色彩格式<br>（2）支持的多线程处理方式<br>（3）支持的输入和输出
2|支持的参数和说明

## **公共操作部分**

## **文件操作部分**

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

{% codeblock lang:shell %}
ffmpeg -i ~/sample.mkv -vcodec mpeg4 -b:v 200k -r 15 -an output.mp4
{% endcodeblock %}

2、输出的信息：

![](http://otkw6sse5.bkt.clouddn.com/ffmpeg-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD10.png)

---