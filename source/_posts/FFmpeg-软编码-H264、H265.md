---
layout: article
title: FFmpeg 软编码 H264、H265
date: 2018-05-27 16:38:56
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **x264**

## **参数**

参数|类型|说明
--|--
preset|字符串|编码器预设参数
tune|字符串|调优编码参数
profile|字符串|编码 profile 档级设置
level|字符串|编码 level 层级设置
wpredp|字符串|P 帧预测设置
x264opts|字符串|设置 x264 专有参数
crf|浮点数|选择质量恒定质量模式
crf_max|浮点数|选择质量恒定质量模式最大值
qp|整数|恒定量化参数控制
psy|浮点数|只用 psychovisual 优化
rc-lookahead|整数|设置预读帧设置
weightb|浮点数|B 帧预测设置
weightp|整数|设置预测分析方法：none、simple、smart 三种模式
ssim|布尔|计算打印 SSIM 状态
intra-refresh|布尔|定时刷 I 帧以替代 IDR 帧
bluray-compat|布尔|蓝光兼容参数
b-bias|整数|B 帧使用频率设置
mixed-refs|布尔|每个 partition 一个参考，而不是每个宏块一个参考
8x8dct|布尔|8x8 矩阵变换，用在 high profile
aud|布尔|带 AUD 分隔标识
mbtree|布尔|宏块树频率控制
deblock|字符串|环路滤波参数
cplxblur|浮点数|减少波动 QP 参数
partitions|字符串|逗号分隔的 partition 列表，可以包含的值有 p8x8、p4x4、b8x8、i8x8、i4x4、none、all
direct-pred|整数|运动向量预测模式
slice-max-size|整数|Slice 的最大值
nal-hrd|整数|HRD 信号信息设置：None、VBR、CBR 设置
motion-est|整数|运动估计方法
forced-idr|布尔|强行设置关键帧为 IDR 帧
coder|整数|编码器类型包括 default、cavlc、cabac、vlc、ac
b_strategy|整数|I/P/B 帧选择策略
chromaoffset|整数|QP 色度和亮度之间的差异参数
sc_threshold|整数|场景切换阈值参数
noise_reduction|整数|降噪处理参数
x264-params|字符串|与 x264opts 操作相同

### **preset**

#### **ultrafast**

最快的编码方式。

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -vcodec libx264 -preset ultrafast -b:v 2000k output.mp4
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2651.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2652.png)

#### **superfast**

#### **veryfast**

#### **faster**

#### **fast**

#### **medium**

#### **slow**

#### **slower**

#### **veryslow**

#### **placebo**

最慢的编码方式。

### **tune**

在使用 FFmpeg 与 x264 进行 H264 直播编码并进行推流时，只用 tune 参数的 zerolatency 将会提升效率，因为其降低了因编码导致的延迟。

#### **zerolatency**

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -vcodec libx264 -tune zerolatency -b:v 2000k output_tune_zerolatency.mp4
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2653.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2654.png)

### **profile**

x264 共支持 7 种 profile。

|Baseline|Extented|Main|High|High10|High4:2:2|High4:4:4（Predictive）
--|--
I 与 P 分片|支持|支持|支持|支持|支持|支持|支持
B 分片|不支持|支持|支持|支持|支持|支持|支持
SI 与 SP 分片|不支持|支持|不支持|不支持|不支持|不支持|不支持
多参考帧|支持|支持|支持|支持|支持|支持|支持
环路去块滤波|支持|支持|支持|支持|支持|支持|支持
CAVLC 熵编码|支持|支持|支持|支持|支持|支持|支持
CABAC 熵编码|不支持|不支持|支持|支持|支持|支持|支持
FMO|不支持|支持|不支持|不支持|不支持|不支持|不支持
ASO|不支持|支持|不支持|不支持|不支持|不支持|不支持
RS|不支持|支持|不支持|不支持|不支持|不支持|不支持
数据分区|支持|支持|不支持|不支持|不支持|不支持|不支持
场编码 PAFF/MBAFF|不支持|支持|支持|支持|支持|支持|支持
4:2:0 色度格式|支持|支持|支持|支持|支持|支持|支持
4:0:0 色度格式|不支持|不支持|不支持|支持|支持|支持|支持
4:2:2 色度格式|不支持|不支持|不支持|不支持|不支持|支持|支持
4:4:4 色度格式|不支持|不支持|不支持|不支持|不支持|不支持|支持
8 位采样深度|支持|支持|支持|支持|支持|支持|支持
9 和 10 位采样深度|不支持|不支持|不支持|不支持|支持|支持|支持
11 至 14 位采样深度|不支持|不支持|不支持|不支持|不支持|不支持|支持
8x8 与 4x4 转换适配|不支持|不支持|不支持|支持|支持|支持|支持
量化计算矩阵|不支持|不支持|不支持|支持|支持|支持|支持
分离 Cb 和 Cr 量化参数控制|不支持|不支持|不支持|支持|支持|支持|支持
分离色彩平面编码|不支持|不支持|不支持|不支持|不支持|不支持|支持
分离无损编码|不支持|不支持|不支持|不支持|不支持|不支持|支持

### **level**

<table><tr><td rowspan="2"></td><td colspan="2">最大解码速度</td><td colspan="2">帧最大尺寸</td><td colspan="3">视频编码层最大码率</td><td rowspan="2">最大分辨率@最大帧率(最大存储帧数)切换其他细节</td></tr><tr><td>亮度采样</td><td>宏块</td><td>亮度采样</td><td>宏块</td><td>Baseline、Extended 和 Main Profile</td><td>High Profile</td><td>High 10 Profile</td></tr><tr><td>1</td><td>380160</td><td>1485</td><td>25344</td><td>99</td><td>64</td><td>80</td><td>192</td><td>128x96@30.9(8)<br>176x144@15.0(4)</td></tr><tr><td>1b</td><td>380160</td><td>1485</td><td>25344</td><td>99</td><td>128</td><td>160</td><td>384</td><td>128x96@30.9(8)<br>176x144@15.0(4)</td></tr><tr><td>1.1</td><td>768000</td><td>3000</td><td>101376</td><td>396</td><td>192</td><td>240</td><td>576</td><td>176x144@30.3(9)<br>320x240@10.0(3)<br>352x288@7.5(2)</td></tr><tr><td>1.2</td><td>1536000</td><td>6000</td><td>101376</td><td>396</td><td>384</td><td>480</td><td>1152</td><td>320x240@20.0(7)<br>352x288@15.2(6)</td></tr><tr><td>1.3</td><td>3041280</td><td>11880</td><td>101376</td><td>396</td><td>768</td><td>960</td><td>2304</td><td>320x240@36.0(7)<br>352x288@30.0(6)</td></tr><tr><td>2</td><td>3041280</td><td>11880</td><td>101376</td><td>396</td><td>2000</td><td>2500</td><td>6000</td><td>320x240@36.0(7)<br>352x288@30.0(6)</td></tr><tr><td>2.1</td><td>3041280</td><td>19800</td><td>202752</td><td>396</td><td>4000</td><td>5000</td><td>12000</td><td>352x480@30.0(7)<br>352x576@25.0(6)</td></tr><tr><td>2.2</td><td>5068800</td><td>20250</td><td>414720</td><td>1620</td><td>4000</td><td>5000</td><td>12000</td><td>352x480@30.7(12)<br>352x576@25.6(10)<br>720x480@15.0(6)<br>720x576@12.5(5)</td></tr><tr><td>3</td><td>5184000</td><td>40500</td><td>414720</td><td>1620</td><td>10000</td><td>12500</td><td>30000</td><td>352x480@61.4(12)<br>352x576@51.1(10)<br>720x480@30.0(6)<br>720x576@25.0(5)</td></tr><tr><td>3.1</td><td>10368000</td><td>108000</td><td>921600</td><td>3600</td><td>14000</td><td>17500</td><td>42000</td><td>720x480@80.0(13)<br>720x576@66.7(11)<br>1280x720@30.0(5)</td></tr><tr><td>3.2</td><td>27648000</td><td>216000</td><td>1310720</td><td>5120</td><td>20000</td><td>25000</td><td>60000</td><td>1280x720@60.0(5)<br>1280x1024@42.2(4)</td></tr><tr><td>4</td><td>55296000</td><td>245760</td><td>2097152</td><td>8192</td><td>20000</td><td>25000</td><td>60000</td><td>1280x720@68.3(9)<br>1920x1080@30.1(4)<br>2048x1024@30.0(4)</td></tr><tr><td>4.1</td><td>62914560</td><td>245760</td><td>2097152</td><td>8192</td><td>50000</td><td>62500</td><td>150000</td><td>1280x720@68.3(9)<br>1920x1080@30.1(4)<br>2048x1024@30.0(4)</td></tr><tr><td>4.2</td><td>133693440</td><td>522240</td><td>2228224</td><td>8740</td><td>50000</td><td>62500</td><td>150000</td><td>1280x720@145.1(9)<br>1920x1080@64.0(4)<br>2048x1080@60.0(4)</td></tr><tr><td>5</td><td>150994944</td><td>589824</td><td>5652480</td><td>22080</td><td>135000</td><td>168750</td><td>405000</td><td>1920x1080@72.3(13)<br>2048x1024@72.0(13)<br>2048x1080@67.8(12)<br>2560x1920@30.7(5)<br>3672x1536@26.7(5)</td></tr><tr><td>5.1</td><td>251658240</td><td>983040</td><td>9437184</td><td>36864</td><td>240000</td><td>300000</td><td>720000</td><td>1920x1080@120.5(16)<br>2560x1920@51.2(9)<br>3840x2160@31.7(5)<br>4096x2048@30.0(5)<br>4096x2160@28.5(5)<br>4096x2304@26.7(5)</td></tr><tr><td>5.2</td><td>530841600</td><td>2073600</td><td>9437184</td><td>36864</td><td>240000</td><td>300000</td><td>720000</td><td>1920x1080@172.0(16)<br>2560x1920@108.0(9)<br>3840x2160@66.8(5)<br>4096x2048@63.3(5)<br>4096x2160@60.0(5)<br>4096x2304@56.3(5)</td></tr></table>

#### **生成 baseline 视频**

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -vcodec libx264 -profile:v baseline -level 3.1 -s 352x288 -an -y -t 10 output_baseline.ts
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2655.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2656.png)

#### **生成 high 视频**

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -vcodec libx264 -profile:v high -level 3.1 -s 352x288 -an -y -t 10 output_baseline.ts
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2657.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2658.png)

### **sc_threshold**

在 FFmpeg 中，可以通过命令行的 -g 参数设置以帧数间隔为 GOP 的长度，但是当遇到场景切换时，例如从一个画面突然变成另外一个画面时，会强行插入一个关键帧，这时 GOP 的间隔将会重新开始。可以通过 sc_threshold 参数设置是否在场景切换时插入关键帧。

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -c:v libx264 -g 50 -t 60 -sc_threshold 0 output_sc_threshold.mp4
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H2659.png)

### **x264opts**

可以通过该参数设置 x264 内部私有参数，比如设置 I 帧、P 帧、B 帧的顺序及规律等。

#### **不允许 B 帧**

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -c:v libx264 -x264opts "bframes=0" -g 50 -sc_threshold 0 output_no_bframe.mp4
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H26510.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H26511.png)

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H26512.png)

#### **每两个 P 帧之间存放 3 个 B 帧**

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -c:v libx264 -x264opts "bframes=3:b-adapt=0" -g 50 -sc_threshold 0 output_pbframe.mp4
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H26513.png)

### **nal-hrd**

编码可以设置为 VBR 或 CBR 模式，VBR 为可变码率，CBR 为恒定码率。
FFmpeg 可以通过参数 -b:v 指定视频的编码码率，但是设定的码率是平均码率，并不能很好的控制最大码率和最小码率的波动，如果需要控制最大码率和最小码率，则需要使用 FFmpeg 的三个参数 -b:v、maxrate、minrate。
同时，为了更好地控制编码时的波动，还可以设置编码时 buffer 的大小，buffer 的大小使用参数 -bufsize 设置即可，buffer 的设置不是越小越好，而是要设置得恰到好处。

比如设置 1M bit/s 码率，bufsize 为 50 KB。

1、输入命令：

```shell
ffmpeg -i guibu.mp4 -c:v libx264 -x264opts "bframes=10:b-adapt=0" -b:v 1000k -maxrate 1000k -minrate 1000k -bufsize 50k -nal-hrd cbr -g 50 -sc_threshold 0 output_nal_hrd.mp4
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BD%AF%E7%BC%96%E7%A0%81-H264%E3%80%81H26514.png)

---