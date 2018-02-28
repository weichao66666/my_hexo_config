---
layout: article
title: FLV 封装格式
date: 2018-02-28 21:25:09
tags:
categories: 
copyright: true
---

# **Reference**

* [F4V/FLV file format spec v10.1](https://wwwimages2.adobe.com/content/dam/acom/en/devnet/flv/video_file_format_spec_v10_1.pdf "https://wwwimages2.adobe.com/content/dam/acom/en/devnet/flv/video_file_format_spec_v10_1.pdf")
* [FLV 文件格式简析](https://www.jianshu.com/p/74a13250fa1b "https://www.jianshu.com/p/74a13250fa1b")
* [FLV文件的第一个Tag: onMetaData](https://www.jianshu.com/p/f2b31ddcf200 "https://www.jianshu.com/p/f2b31ddcf200")
* [H.264/AVC编码的FLV文件的第二个Tag: AVCDecoderConfigurationRecord](https://www.jianshu.com/p/e1e417eee2e7 "https://www.jianshu.com/p/e1e417eee2e7")

# **FLV 格式**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1.png)

---

# **FLV header**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F2.png)

## **从 FLV 报文看 FLV header 数据结构**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F3.png)

数据|数据位置的说明
-|-
0x46 0x4c 0x56|字符`FLV`
0x01|版本号
0x05|右起第 0 位和第 2 位分别表示 video 与 audio 存在的情况（1 表示存在，0 表示不存在）
0x00 0x00 0x00 0x09|报头数据长度


---

# **FLV body**

## **FLV TAG header**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8FQQ%E6%88%AA%E5%9B%BE20180228213618.png)

## **第一个 TAG**

`onMetaData`和`header`之间存在 4 个空字节。

### **rtmp 中，FLV body 的第一个 TAG 必须是 onMetaData**

首先是 11 个字节的`FLV TAG header`，`TagType=0x12`。然后是`onMetaData`的`header`。最后是`onMetaData`的`body`。

#### **onMetaData 的 header**

1个字节的`2` + 12个字节的`onMetaData`字符串。

#### **onMetaData 的 body**

将数据按照写入规则排在`header`后面。

参数包括：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F6.png)

其中，`audiocodecid`范围：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F7.png)

`videocodecid`范围：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F8.png)

参数写入规则根据数据类型而定。

数据类型包括：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F9.png)

数据类型|参数写入规则
--|--
String|1个字节的`2` + 2个字节的value长度 + value长度个字节的value
ECMA array|4个字节的`8` + 每个元素（key长度个字节的key + 1个字节的数据类型 + value长度个字节的value）

`onMetaData`结尾是 3 个字节的`0x00 0x00 0x09`。

## **第二个 TAG**

第二个 TAG 和前面的`onMetaData`之间存在 4 个字节（用于表示`onMetaData`长度）。

### **如果有 AVC，rtmp 中，FLV body 下一个 TAG 必须是 AVCDecoderConfigurationRecord**

首先是 11 个字节的`FLV TAG header`，`TagType=0x09`。然后是`AVC`编码格式的`header`，`AVCPacketType=0x00`。最后是`AVCDecoderConfigurationRecord`。

---

### **如果有 AAC，rtmp 中，FLV body 下一个 TAG 必须是 AudioSpecificConfig**

首先是 11 个字节的`FLV TAG header`，`TagType=0x08`。然后是`AAC`编码格式的`header`，`AACPacketType=0x00`。最后是`AudioSpecificConfig`。

## **后续 TAG**

每个 TAG 都和前面的 TAG 之间存在 4 个字节（用于表示前一个 TAG 长度）。

首先是 11 个字节的`FLV TAG header`。然后是`AVC/AAC`编码格式的`header`，最后是`AVC/AAC`编码格式的`body`。

### **TAG header**

#### **AVC 编码格式的 header**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F10.png)

#### **AAC 编码格式的 header**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8FQQ%E6%88%AA%E5%9B%BE20180228213649.png)

### **TAG body**

#### **AVC 编码格式的 body**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F13.png)

#### **AAC 编码格式的 body**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8FQQ%E6%88%AA%E5%9B%BE20180228213710.png)

---