---
layout: article
title: FFmpeg 录制 RTSP 流
date: 2018-07-29 15:38:50
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **FFmpeg 操作 RTSP 的参数**

参数|类型|说明
--|--
initial_pause|布尔|建立连接后暂停播放
rtsp_transport|标记|设置 RTSP 传输协议，具体如下：<br>udp：UDP<br>tcp：TCP<br>udp_multicast：UDP 多播协议<br>http：HTTP 隧道
rtsp_flags|标记|RTSP 使用标记，具体如下：<br>filter_src：只接收指定 IP 的流<br>listen：设置为被动接收模式<br>prefer_tcp：TCP 亲和模式，如果 TCP 可用则首选 TCP 传输
allowed_media_types|标记|设置允许接收的数据模式（默认为全部开启）：<br>video：只接收视频<br>audio：只接收音频<br>data：只接收数据<br>subtitle：只接收字幕
min_port|整数|设置最小本地 UDP 端口，默认为 5000
max_port|整数|设置最大本地 UDP 端口，默认为 65000
timeout|整数|设置监听端口超时时间
reorder_queue_size|整数|设置录制数据 Buffer 的大小
buffer_size|整数|设置底层传输包 Buffer 的大小
user-agent|字符串|用户客户端标识


---

# **参数使用示例**

## **rtsp_transport**

FFmpeg 默认使用的 RTSP 拉流的方式是 UDP 传输。

1、输入命令：

```shell
ffmpeg -rtsp_transport tcp -i rtsp://184.72.239.149/vod/mp4://BigBuckBunny_175k.mov -c copy -f mp4 output.mp4
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E5%BD%95%E5%88%B6-RTSP-%E6%B5%811.png)

---