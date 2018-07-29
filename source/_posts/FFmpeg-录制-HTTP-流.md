---
layout: article
title: FFmpeg 录制 HTTP 流
date: 2018-07-29 15:41:15
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **简介**

使用 HTTP 可以传输 FLV 直播流、TS 直播流、M3U8 及 TS 文件。

---

# **HTTP 参数说明**

参数|类型|说明
--|--
seekable|布尔|设置 HTTP 链接为可以 seek 操作
chunked_post|布尔|使用 Chunked 模式 post 数据
http_proxy|字符串|设置 HTTP 代理传输数据
headers|字符串|自定义 HTTP Header 数据
content_type|字符串|设置 POST 的内容类型
user_agent|字符串|设置 HTTP 请求客户端信息
multiple_requests|布尔|HTTP 长连接开启
post_data|二进制数据|设置将要 POST 的数据
cookies|字符串|设置 HTTP 请求时写代码的 Cookies
icy|布尔|请求 ICY 元数据：默认打开
auth_type|整数|HTTP 验证类型设置
offset|整数|初始化 HTTP 请求时的偏移位置
method|字符串|发起 HTTP 请求时使用的 HTTP 的方法
reconnect|布尔|在 EOF 之前断开发起重连
reconnect_at_eof|布尔|在得到 EOF 时发起重连
reply_code|整数|作为 HTTP 服务时向客户端反馈状态码


---

# **参数使用示例**

## **seekable**

### **阻塞**

1、输入命令：

```shell
ffmpeg -ss 300 -seekable 0 -i http://live.hkstv.hk.lxdns.com/live/hks/playlist.m3u8 -c copy output.mp4
```

2、输出结果（实际没播放成功）：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E5%BD%95%E5%88%B6-HTTP-%E6%B5%811.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E5%BD%95%E5%88%B6-HTTP-%E6%B5%812.png)

### **非阻塞**

1、输入命令：

```shell
ffmpeg -ss 300 -seekable 1 -i http://live.hkstv.hk.lxdns.com/live/hks/playlist.m3u8 -c copy -y output.mp4
```

2、输出结果（实际没播放成功）：

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E5%BD%95%E5%88%B6-HTTP-%E6%B5%813.png)
![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E5%BD%95%E5%88%B6-HTTP-%E6%B5%814.png)

---