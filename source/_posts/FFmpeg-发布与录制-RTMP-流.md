---
layout: article
title: FFmpeg 发布与录制 RTMP 流
date: 2018-07-29 15:34:57
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **FFmpeg 操作 RTMP 的参数**

参数|类型|说明
--|--
rtmp_app|字符串|RTMP 流发布点，又称 APP
rtmp_buffer|整数|客户端 buffer 大小（单位：毫秒），默认为 3 秒
rtmp_conn|字符串|在 RTMP 的 Connect 命令中增加自定义 AMF 数据
rtmp_flashver|字符串|设置模拟的 flashplugin 的版本号
rtmp_live|整数|指定 RTMP 流媒体播放类型，具体如下：<br>any：直播或点播随意<br>live：直播<br>recorded：点播
rtmp_pageurl|字符串|RTMP 在 Connect 命令中设置的 PageURL 字段，其为播放时所在的 Web 页面 URL
rtmp_playpath|字符串|RTMP 流播放的 Stream 地址，或者称为密钥，或者称为发布流
rtmp_subscribe|字符串|直播流名称，默认设置为 rtmp_playpath 的值
rtmp_swfhash|二进制数据|解压 swf 文件后的 SHA256 的 hash 值
rtmp_swfsize|整数|swf 文件解压后的大小，用于 swf 认证
rtmp_swfurl|字符串|RTMP 的 Connect 命令中设置的 swfURL 播放器的 URL
rtmp_swfverify|字符串|设置 swf 认证时 swf 文件的 URL 地址
rtmp_tcurl|字符串|RTMP 的 Connect 命令中设置的 tcURL 目标发布点地址，一般形如 rtmp://xxx.xxx.xxx/app
rtmp_listen|整数|开启 RTMP 服务时所监听的端口
listen|整数|与 rtmp_listen 相同
timeout|整数|监听 rtmp 端口时设置的超时时间，以秒为单位


---

# **参数使用示例**

## **rtmp_app**

### **推流**

1、输入命令：

```shell
ffmpeg -re -i sample.mp4 -c copy -f flv -rtmp_app live rtmp://publish.chinaffmpeg.com
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/FFmpeg-%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BD%95%E5%88%B6-RTMP-%E6%B5%811.png)

### **拉流**

1、输入命令：

```shell
ffmpeg -rtmp_app live -i rtmp://publish.chinaffmpeg.com -c copy -f flv output.flv
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/FFmpeg-%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BD%95%E5%88%B6-RTMP-%E6%B5%812.png)

## **rtmp_playpath**

### **推流**

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f flv -rtmp_app live -rtmp_playpath class rtmp://publish.chinaffmpeg.com
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/FFmpeg-%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BD%95%E5%88%B6-RTMP-%E6%B5%813.png)

### **拉流**

1、输入命令：

```shell
ffmpeg -rtmp_app live -rtmp_playpath class -i rtmp://publish.chinaffmpeg.com -c copy -f flv output.flv
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/FFmpeg-%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BD%95%E5%88%B6-RTMP-%E6%B5%814.png)

## **省略 rtmp_app 和 rtmp_playpath 的写法**

### **推流**

1、输入命令：

```shell
ffmpeg -i guibu.flv -c copy -f flv rtmp://publish.chinaffmpeg.com/live/class
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/FFmpeg-%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BD%95%E5%88%B6-RTMP-%E6%B5%815.png)

### **拉流**

1、输入命令：

```shell
ffmpeg -i rtmp://publish.chinaffmpeg.com/live/class -c copy -f flv output.flv
```

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/FFmpeg-%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BD%95%E5%88%B6-RTMP-%E6%B5%816.png)

---