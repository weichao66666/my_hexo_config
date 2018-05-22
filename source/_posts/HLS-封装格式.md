---
layout: article
title: HLS 封装格式
date: 2018-05-20 23:06:13
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **FFmpeg 转封装 HLS 可配置的参数**

参数|类型|说明
--|--
start_number|整数|设置 M3U8 列表中的第一片的序列数
hls_time|浮点数|设置每一片时长<br>该切片规则采用的方式是从关键帧处开始切片，所以时间不一定不均匀，如果先转码再进行切片，则时间会比较均匀；或者使用 hls_flags 的 split_by_time（可能导致花屏等问题，因为第一帧不一定为关键帧）
hls_list_size|整数|设置 M3U8 中分片的个数
hls_ts_options|字符串|设置 TS 切片的参数
hls_wrap|整数|设置切片索引回滚的边界值
hls_allow_cache|整数|设置 M3U8 中 EXT-X-ALLOW-CACHE 的标签
hls_base_url|字符串|设置 M3U8 中每一片的前置路径
hls_segment_filename|字符串|设置切片名模板
hls_key_info_file|字符串|设置 M3U8 加密的 key 文件路径
hls_subtitle_path|字符串|设置 M3U8 字幕路径
hls_flags|标签（整数）|设置 M3U8 文件列表的操作：<br>single_file：生成一个媒体文件索引与字节范围<br>delete_segments：删除 M3U8 文件中不包含的过期的 TS 切片文件<br>round_durations：生成的 M3U8 切片信息的 duration 为整数<br>discont_start：生成 M3U8 的时候在列表前边加上 discontinuity 标签<br>omit_endlist：在 M3U8 末尾不追加 endlist 标签
use_localtime|布尔|设置 M3U8 文件序号为本地时间戳
use_localtime_mkdir|布尔|根据本地时间戳生成目录
hls_playlist_type|整数|设置 M3U8 列表为事件或者点播列表
method|字符串|设置 HTTP 属性

# **FFmpeg 转封装 HLS**

## **MP4 转封装 HLS（不加多余参数）**

1、输入命令：

```shell
ffmpeg -re -i guibu.mp4 -c copy -f hls -bsf:v h264_mp4toannexb output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F2.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:10
#EXTINF:9.800000,
output10.ts
#EXTINF:10.000000,
output11.ts
#EXTINF:5.280000,
output12.ts
#EXTINF:10.000000,
output13.ts
#EXTINF:2.000000,
output14.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output14.ts

## **FLV 转封装 HLS**

`-bsf:v h264_mp4toannexb` 的作用是将 MP4 中的 H264 编码格式的数据转换为 H264AnnexB 编码格式的数据，AnnexB 编码格式常见于实时传输流中。如果源文件为 FLV、TS 等可作为直播传输流的视频，则不需要这个参数。

### **不加多余参数**

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F3.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F4.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6.000000,
output18.ts
#EXTINF:6.000000,
output19.ts
#EXTINF:1.200000,
output20.ts
#EXTINF:6.000000,
output21.ts
#EXTINF:6.000000,
output22.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output22.ts

### **start_number**

比如设置 M3U8 列表中的第一片的序列数为 100。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -start_number 100 output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F5.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F6.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:118
#EXTINF:6.000000,
output118.ts
#EXTINF:6.000000,
output119.ts
#EXTINF:1.200000,
output120.ts
#EXTINF:6.000000,
output121.ts
#EXTINF:6.000000,
output122.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output100.ts～output122.ts

### **hls_time**

比如设置每片长度为 10 秒。
该切片规则采用的方式是从关键帧处开始切片，所以时间不一定不均匀，如果先转码再进行切片，则时间会比较均匀；或者使用 hls_flags 的 split_by_time。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_time 10 output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F7.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F8.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:12
#EXT-X-MEDIA-SEQUENCE:8
#EXTINF:11.200000,
output8.ts
#EXTINF:5.520000,
output9.ts
#EXTINF:12.000000,
output10.ts
#EXTINF:7.200000,
output11.ts
#EXTINF:6.000000,
output12.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output12.ts

### **hls_list_size**

比如设置 M3U8 列表中 TS 切片的个数为 3。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_list_size 3 output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F9.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F10.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:20
#EXTINF:1.200000,
output20.ts
#EXTINF:6.000000,
output21.ts
#EXTINF:6.000000,
output22.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output22.ts

### **hls_wrap**

比如设置 M3U8 列表中 TS 分片序号大于 2 时回滚为 0。
**注意：该参数对 CDN 支持不友好，会引起兼容性问题，新版本会被弃用。**

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_wrap 3 output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F11.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F12.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6.000000,
output0.ts
#EXTINF:6.000000,
output1.ts
#EXTINF:1.200000,
output2.ts
#EXTINF:6.000000,
output0.ts
#EXTINF:6.000000,
output1.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output2.ts

### **hls_base_url**

设置 M3U8 列表中的文件路径。
该路径可以为本地绝对路径、相对路径、网络路径（比如`http://192.168.0.1/live/`）。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_base_url /home/weichao/ output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F13.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F14.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6.000000,
/home/weichao/output18.ts
#EXTINF:6.000000,
/home/weichao/output19.ts
#EXTINF:1.200000,
/home/weichao/output20.ts
#EXTINF:6.000000,
/home/weichao/output21.ts
#EXTINF:6.000000,
/home/weichao/output22.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output22.ts

### **hls_segment_filename**

比如设置 M3U8 列表切片文件名以 live 开头。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_segment_filename live-%d.ts output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F15.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F16.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6.000000,
live-18.ts
#EXTINF:6.000000,
live-19.ts
#EXTINF:1.200000,
live-20.ts
#EXTINF:6.000000,
live-21.ts
#EXTINF:6.000000,
live-22.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

live-0.ts～live-22.ts

### **hls_flags**

#### **delete_segments**

删除旧文件（以 hls_list_size 的 2 倍作为删除的依据）。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_flags delete_segments output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F17.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F18.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6.000000,
output18.ts
#EXTINF:6.000000,
output19.ts
#EXTINF:1.200000,
output20.ts
#EXTINF:6.000000,
output21.ts
#EXTINF:6.000000,
output22.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output17.ts～output22.ts

#### **round_durations**

实现切片信息的 duration 为整型。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_flags round_durations output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F19.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F20.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6,
output18.ts
#EXTINF:6,
output19.ts
#EXTINF:1,
output20.ts
#EXTINF:6,
output21.ts
#EXTINF:6,
output22.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output22.ts

#### **discont_start**

输出的 M3U8 在第一片**（后续会刷掉）** TS 信息的前面有一个 EXT-X-DISCONTINUTY 标签。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_flags discont_start output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F21.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-DISCONTINUITY
#EXTINF:6.000000,
output0.ts
#EXTINF:5.760000,
output1.ts
#EXTINF:4.760000,
output2.ts
#EXTINF:5.600000,
output3.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output22.ts

#### **omit_endlist**

生成 M3U8 结束后不在文件末尾追加 endlist 标签。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_flags omit_endlist output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F23.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F24.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6.000000,
output18.ts
#EXTINF:6.000000,
output19.ts
#EXTINF:1.200000,
output20.ts
#EXTINF:6.000000,
output21.ts
#EXTINF:6.000000,
output22.ts
```

4、生成的 ts 文件：

output0.ts～output22.ts

#### **split_by_time**

更精确地设置每一片时长**（可能导致花屏等问题，因为第一帧不一定为关键帧）**。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -hls_time 10 -hls_flags split_by_time output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F25.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F26.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:8
#EXTINF:9.960000,
output8.ts
#EXTINF:10.000000,
output9.ts
#EXTINF:9.920000,
output10.ts
#EXTINF:10.080000,
output11.ts
#EXTINF:6.480000,
output12.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

output0.ts～output12.ts

### **use_localtime**

以时间戳为切片文件名。

1、输入命令：

```shell
ffmpeg -re -i guibu.flv -c copy -f hls -use_localtime 1 output.m3u8
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F27.png)
![](http://otkw6sse5.bkt.clouddn.com/HLS%20%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F28.png)

3、输出的 M3U8 文件的内容：

```xml
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:18
#EXTINF:6.000000,
output-1526809032.ts
#EXTINF:6.000000,
output-1526809038.ts
#EXTINF:1.200000,
output-1526809044.ts
#EXTINF:6.000000,
output-1526809046.ts
#EXTINF:6.000000,
output-1526809052.ts
#EXT-X-ENDLIST
```

4、生成的 ts 文件：

22 个 output-时间戳.ts

### **method**

将 M3U8 及 TS 文件上传至 HTTP 服务器。

#### **配置 Nginx**

```xml
TODO
```

#### **测试**

1、输入命令：

```shell
ffmpeg -i guibu.flv -c copy -f hls -method PUT http://127.0.0.1/test/output_test.m3u8
```

2、输出结果：

```xml
TODO
```

3、输出的 M3U8 文件的内容：

```xml
TODO
```

4、生成的 ts 文件：

```xml
TODO
```

---