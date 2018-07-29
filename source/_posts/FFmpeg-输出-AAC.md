---
layout: article
title: FFmpeg 输出 AAC
date: 2018-07-29 15:25:31
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **AAC**

与 MP3 相比，AAC 是一种编码效率更高、编码音质更好的音频编码格式，常见的使用 AAC 编码后的文件存储格式为 m4a。

---

# **抽取 MP4 文件中的音频为 AAC**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c:a aac -b:a 160k output.aac
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-AAC1.png)

## **使用 q 参数**

q 表示 qscale，有效范围为 0.1~2，用于设置 AAC 音频的 VBR 质量。

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c:a aac -q:a 2 output.aac
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-AAC2.png)

---

# **FDK AAC 第三方的 AAC 编解码 Codec 库**

FDK AAC 是 FFmpeg 支持的第三方编码库中质量最高的 AAC 编码库。

## **恒定码率（CBR）模式**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c:a libfdk_aac -b:a 128k output.aac
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-AAC3.png)

## **动态码率（VBR）模式**

VBR|每声道码率（kbit/s）|码率信息
--|--
1|20~32|LC、HE、HEv2
2|32~40|LC、HE、HEv2
3|48~56|LC、HE、HEv2
4|64~72|LC
5|96~112|LC


* LC：Low Complexity AAC，这种编码相对来说体积比较大，质量稍差
* HE：High-Efficiency AAC，这种编码相对来说体积稍小，质量较好
* HEv2：High-Efficiency AAC version2，这种编码相对来说体积小，质量优

### **AAC 编码 LC、HE、HEv2 推荐参数**

<table><tr><td>编码类型</td><td>码率范围（bit/s）</td><td>支持的采样率/kHz</td><td>推荐的采样率/kHz</td><td>声道数</td></tr><tr><td rowspan="4">HE-AAC v2（AAC LC + SBR + P）</td><td>8000 ~ 11999</td><td>22.05, 24.00</td><td>24.00</td><td>2</td></tr><tr><td>12000 ~ 17999</td><td>32.00</td><td>32.00</td><td>2</td></tr><tr><td>18000 ~ 39999</td><td>32.00, 44.10, 48.00</td><td>44.10</td><td>2</td></tr><tr><td>40000 ~ 56000</td><td>32.00, 44.10, 48.00</td><td>48.00</td><td>2</td></tr><tr><td rowspan="7">HE-AAC（AAC LC + SBR）</td><td>8000 ~ 11999</td><td>22.05, 24.00</td><td>24.00</td><td>1</td></tr><tr><td>12000 ~ 17999</td><td>32.00</td><td>32.00</td><td>1</td></tr><tr><td>18000 ~ 39999</td><td>32.00, 44.10, 48.00</td><td>44.10</td><td>1</td></tr><tr><td>40000 ~ 56000</td><td>32.00, 44.10, 48.00</td><td>48.00</td><td>1</td></tr><tr><td>16000 ~ 27999</td><td>32.00, 44.10, 48.00</td><td>32.00</td><td>2</td></tr><tr><td>28000 ~ 63999</td><td>32.00, 44.10, 48.00</td><td>44.10</td><td>2</td></tr><tr><td>64000 ~ 128000</td><td>32.00, 44.10, 48.00</td><td>48.00</td><td>2</td></tr><tr><td rowspan="4">HE-AAC（AAC LC + SBR）</td><td>64000 ~ 69999</td><td>32.00, 44.10, 48.00</td><td>32.00</td><td>5, 5.1</td></tr><tr><td>70000 ~ 159999</td><td>32.00, 44.10, 48.00</td><td>44.10</td><td>5, 5.1</td></tr><tr><td>160000 ~ 245999</td><td>32.00, 44.10, 48.00</td><td>48.00</td><td>5</td></tr><tr><td>160000 ~ 265999</td><td>32.00, 44.10, 48.00</td><td>48.00</td><td>5.1</td></tr><tr><td rowspan="6">AAC LC</td><td>8000 ~ 15999</td><td>11.025, 12.00, 16.00</td><td>12.00</td><td>1</td></tr><tr><td>16000 ~ 23999</td><td>16.00</td><td>16.00</td><td>1</td></tr><tr><td>24000 ~ 31999</td><td>16.00, 22.05, 24.00</td><td>24.00</td><td>1</td></tr><tr><td>32000 ~ 55999</td><td>32.00</td><td>32.00</td><td>1</td></tr><tr><td>56000 ~ 160000</td><td>32.00, 44.10, 48.00</td><td>44.10</td><td>1</td></tr><tr><td>160001 ~ 288000</td><td>48.00</td><td>48.00</td><td>1</td></tr><tr><td rowspan="7">AAC LC</td><td>16000 ~ 23999</td><td>11.025, 12.00, 16.00</td><td>12.00</td><td>2</td></tr><tr><td>24000 ~ 31999</td><td>16.00</td><td>16.00</td><td>2</td></tr><tr><td>32000 ~ 39999</td><td>16.00, 22.05, 24.00</td><td>22.05</td><td>2</td></tr><tr><td>40000 ~ 95999</td><td>32.00</td><td>32.00</td><td>2</td></tr><tr><td>96000 ~ 111999</td><td>32.00, 44.10, 48.00</td><td>32.00</td><td>2</td></tr><tr><td>112000 ~ 320001</td><td>32.00, 44.10, 48.00</td><td>44.10</td><td>2</td></tr><tr><td>320002 ~ 576000</td><td>48.00</td><td>48.00</td><td>2</td></tr><tr><td rowspan="3">AAC LC</td><td>160000 ~ 239999</td><td>32.00</td><td>32.00</td><td>5, 5.1</td></tr><tr><td>240000 ~ 279999</td><td>32.00, 44.10, 48.00</td><td>32.00</td><td>5, 5.1</td></tr><tr><td>280000 ~ 800000</td><td>32.00, 44.10, 48.00</td><td>44.10</td><td>5, 5.1</td></tr></table>

## **压缩为 AAC 编码的 m4a 容器**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c:a libfdk_aac output.m4a
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-AAC4.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-AAC5.png)

### **高质量 AAC 设置**

#### **HE-AAC 音频编码设置**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c:a libfdk_aac -profile:a aac_he -b:a 64k output.aac
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-AAC6.png)

#### **HEv2-AAC 音频编码设置**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c:a libfdk_aac -profile:a aac_he_v2 -b:a 32k output.aac
```

2、输出结果（失败）：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-AAC7.png)

---