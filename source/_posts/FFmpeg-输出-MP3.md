---
layout: article
title: FFmpeg 输出 MP3
date: 2018-07-29 11:11:11
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **实现方式**

FFmpeg 使用第三方库 libmp3lame 编码 MP3。

---

# **获取 libmp3lame 编码 MP3 的参数**

1、输入命令：

```shell
ffmpeg -h encoder=libmp3lame
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-MP31.png)

## **参数**

参数|类型|说明
--|--
b|布尔|设置 MP3 编码的码率
joint_stereo|布尔|设置环绕立体声模式
abr|布尔|设置编码为 ABR 状态，自动调整码率
compression_level|整数|设置压缩算法质量，参数设置为 0~9 区间的值即可，数值越大质量越差，但是编码速度越快
q|整型|设置恒质量的 VBR。调用 lame 接口的话，设置 global_quality 变量具有同样的效果


---

# **转编码 MP3**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -acodec libmp3lame output.mp3
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-MP32.png)

---

# **设置编码 MP3 的质量**

lame 操作参数|平均码率(kbit/s)|码率区间(kbit/s)|ffmpeg 操作参数
--|--
-b 320|320|320(CBR)|-b:a 320k
-v 0|245|220~260|-q:a 0
-v 1|225|190~250|-q:a 1
-v 2|190|170~210|-q:a 2
-v 3|175|150~195|-q:a 3
-v 4|165|140~185|-q:a 4
-v 5|130|120~150|-q:a 5
-v 6|115|100~130|-q:a 6
-v 7|100|80~120|-q:a 7
-v 8|85|70~105|-q:a 8
-v 9|65|45~85|-q:a 9

1、输入命令：

```shell
ffmpeg -i sample.mp4 -acodec libmp3lame -q:a 8 output.mp3
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-MP33.png)

---

# **使用平均码率编码参数 ABR**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -acodec libmp3lame -b:a 64k -abr 1 output.mp3
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E8%BE%93%E5%87%BA-MP34.png)

---