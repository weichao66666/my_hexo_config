---
layout: article
title: FLV 封装格式
date: 2018-02-28 21:25:09
tags:
categories: 
copyright: true
---

# **Reference**

* [video_file_format_spec_v10_1.pdf](https://wwwimages2.adobe.com/content/dam/acom/en/devnet/flv/video_file_format_spec_v10_1.pdf "https://wwwimages2.adobe.com/content/dam/acom/en/devnet/flv/video_file_format_spec_v10_1.pdf")
* [FLV 文件格式简析](https://www.jianshu.com/p/74a13250fa1b "https://www.jianshu.com/p/74a13250fa1b")
* [FLV文件的第一个Tag: onMetaData](https://www.jianshu.com/p/f2b31ddcf200 "https://www.jianshu.com/p/f2b31ddcf200")
* [H.264/AVC编码的FLV文件的第二个Tag: AVCDecoderConfigurationRecord](https://www.jianshu.com/p/e1e417eee2e7 "https://www.jianshu.com/p/e1e417eee2e7")
* 《FFmpeg 从入门到精通》

---

# **FLV 封装格式**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1.png)

---

# **FLV header**

## **定义**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F2.png)

翻译：

字段|占用位数|说明
--|--
Signature|8|签名字段，字符 F（0x46）
Signature|8|签名字段，字符 L（0x4C）
Signature|8|签名字段，字符 V（0x56）
Version|8|文件版本（例如 0x01 为 FLV 版本 1）
TypeFlagsReserved|5|保留标记类型，固定为 0
TypeFlagsAudio|1|音频标记类型，1 为显示音频标签
TypeFlagsReserved|1|保留标记类型，固定为 0
TypeFlagsVideo|1|视频标记类型，1 为显示视频标签
DataOffset|32|这个头的字节数


---

# **FLV body**

字段|类型/大小|说明
--|--
保留字段|4 字节|一直是 0
TAG0|FLV TAG|第一个 TAG
Tag0Size|4 字节|TAG0 的大小，包括 TAG0 的 header+body，TAG 的 header 大小为 11 字节，所以这个字段大小为 11 字节+body 的大小
TAG1|FLV TAG|第二个 TAG
……|……|……
TagN-1Size|4 字节|TagN-1 的大小，包括 TagN-1 的 header+body，TAG 的 header 大小为 11 字节，所以这个字段大小为 11 字节+body 的大小
TAGN|FLV TAG|第 N 个 TAG

## **FLV TAG**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8FQQ%E6%88%AA%E5%9B%BE20180228213618.png)

翻译：

字段|类型/大小|说明
--|--
Reserved|2 位|为 FMS 保留，应该是 0
Filter|1 位|主要用于做文件内容加密处理：<br>0：不预处理<br>1：预处理
TagType|5 位|8（0x08）：音频 TAG<br>9（0x09）：视频 TAG<br>18（0x12）：脚本数据，例如 Metadata
DataSize|24 位|TAG 的 DATA 部分的大小
Timestamp|24 位|以毫秒为单位的展示时间 0x000000
TimestampExtended|8 位|针对时间戳增加的补充时间戳
StreamID|24 位|一直是 0
AudioTagHeader||
VideoTagHeader||
EncryptionHeader||
FilterParam||
Data|音频数据/视频数据/脚本数据|包含 startcode

每个 TAG 都和前面的 TAG 之间存在 4 个字节（用于表示前一个 TAG 长度）。

 4 个字节后，首先是 11 个字节的`FLV TAG header`，然后是音频/视频/脚本的`header`，最后是音频/视频/脚本的`body`。

## **TAG header**

### **脚本 TAG 的 header**

1个字节的`2` + 12个字节的`onMetaData`字符串。

### **视频 TAG 的 header**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F10.png)

翻译：

字段|类型/大小|说明
--|--
FrameType|4 位|视频帧的类型：<br>1：关键帧（H264使用，可以 seek 的帧）<br>2：P 或 B 帧（H264 使用，不可以 seek 的帧）<br>3：仅应用于 H263<br>4：生成关键帧（服务器端使用）<br>5：视频信息/命令帧
CodecID|4 位|Codec 类型：<br>2：Sorenson H263<br>3：Screen Video<br>4：On2 VP6<br>5：带 Alpha 通道的 On2 VP6<br>6：Screen Video 2<br>7：H264
AVCPacketType|当 Codec 为 H264 编码时占用这个 8 位|当 H264 编码封装在 FLV 中时，需要三类 H264 的数据：<br>0：H264 的 Sequence Header<br>1：NALU<br>2：H264 的 Sequence 的 end
CompositionTime|当 Codec 为 H264 编码时占用这个 24 位|当编码使用 B 帧时，DTS 和 PTS 不相等，CTS 用于表示 PTS 和 DTS 之间的差值

### **音频 TAG 的 header**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8FQQ%E6%88%AA%E5%9B%BE20180228213649.png)

翻译：

字段|类型/大小|说明
--|--
SoundFormat|4 位|0：线性 PCM，大小端取决于平台<br>1：ADPCM 音频格式<br>2：MP3<br>3：线性 PCM，小端<br>4：Nellymoser 16kHz Mono<br>5：Nellymoser 8kHz Mono<br>6：Nellymoser<br>7：G711 A-law<br>8：G711 mu-law<br>9：保留<br>10：AAC<br>11：Speex<br>14：MP3 8kHz<br>15：设备支持的声音<br>格式 7、8、14、15 均为保留；使用频率非常高的为 AAC、MP3、Speex
SoundRate|2 位|0：5.5 kHz<br>1：11 kHz<br>2：22 kHz<br>3： 44 kHz<br>有些音频为 48 kHz 的 AAC 也可以被包含进来，不过也是采用 44 kHz 的方式存储，因为音频采样率在标准中只用 2 位来表示不同的采样率，所以一般为 4 种
SoundSize|1 位|0：8 位采样<br>1：16 位采样
SoundSize|1 位|0：Mono sound<br>1：Stereo sound
AACPacketType|当音频为 AAC 时占用这 8 位|0：AAC Sequence Header<br>1：AAC raw 数据

## **TAG body**

### **脚本 TAG 的 body**

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

### **视频 TAG 的 body（AVC 编码格式）**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F13.png)

翻译：

字段|说明
--|--
Data|AVC 序列头或视频数据

#### **AVCDecoderConfigurationRecord**

##### **定义**

```
aligned(8) class AVCDecoderConfigurationRecord {
	unsigned int(8) configurationVersion = 1;
	unsigned int(8) AVCProfileIndication;
	unsigned int(8) profile_compatibility;
	unsigned int(8) AVCLevelIndication; 
	bit(6) reserved = ‘111111’b;
	unsigned int(2) lengthSizeMinusOne; 
	bit(3) reserved = ‘111’b;
	unsigned int(5) numOfSequenceParameterSets;
	for (i=0; i< numOfSequenceParameterSets;  i++) {
		unsigned int(16) sequenceParameterSetLength ;
		bit(8*sequenceParameterSetLength) sequenceParameterSetNALUnit;
	}
	unsigned int(8) numOfPictureParameterSets;
	for (i=0; i< numOfPictureParameterSets;  i++) {
		unsigned int(16) pictureParameterSetLength;
		bit(8*pictureParameterSetLength) pictureParameterSetNALUnit;
	}
	if( profile_idc  ==  100  ||  profile_idc  ==  110  ||
	    profile_idc  ==  122  ||  profile_idc  ==  144 )
	{
		bit(6) reserved = ‘111111’b;
		unsigned int(2) chroma_format;
		bit(5) reserved = ‘11111’b;
		unsigned int(3) bit_depth_luma_minus8;
		bit(5) reserved = ‘11111’b;
		unsigned int(3) bit_depth_chroma_minus8;
		unsigned int(8) numOfSequenceParameterSetExt;
		for (i=0; i< numOfSequenceParameterSetExt; i++) {
			unsigned int(16) sequenceParameterSetExtLength;
			bit(8*sequenceParameterSetExtLength) sequenceParameterSetExtNALUnit;
		}
	}
}
```

##### **从报文看结构**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1526737798220_32.png)

图中阴影部分对应了`header`全部数据

数据|数据位置的说明
-|-
0x61 0x76 0x63 0x43|字符`avcC`
0x01|configurationVersion
0x42|AVCProfileIndication
0x00|profile_compatibility
0x1F|AVCLevelIndication
0xFF|6bit的reserved + 2bit的lengthSizeMinusOne（0x03 表示使用 4 个字节表示 NALU 的长度）
0xE1|3bit的reserved + 5bit的numOfSequenceParameterSets
0x00 0x09|`SPS`的长度
0x67 0x42 0x00 0x1f 0xe9 0x02 0xc1 0x2c 0x80|`SPS`的内容（和长度对应）
0x01|numOfPictureParameterSets
0x00 0x04|`PPS`的长度
0x68 0xce 0x06 0xf2|`PPS`的内容（和长度对应）

### **音频 TAG 的 body（AAC 编码格式）**

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8FQQ%E6%88%AA%E5%9B%BE20180228213710.png)

翻译：

字段|说明
--|--
Data|AAC 序列头或音频数据

#### **AudioSpecificConfig**

## **RTMP 与 FLV 相比特殊的地方**

1、FLV body 的第一个 TAG 必须是 onMetaData

2、如果有 AVC，FLV body 下一个 TAG 必须是 AVCDecoderConfigurationRecord

首先是 11 个字节的`FLV TAG header`（其中`TagType=0x09`），然后是`AVC`编码格式的`header`（其中`AVCPacketType=0x00`），最后是`AVCDecoderConfigurationRecord`。

3、如果有 AAC，FLV body 下一个 TAG 必须是 AudioSpecificConfig

首先是 11 个字节的`FLV TAG header`（其中`TagType=0x08`），然后是`AAC`编码格式的`header`（其中`AACPacketType=0x00`），最后是`AudioSpecificConfig`。

---

# **将 FLV 文件按字节转十六进制**

参考【MP4 封装格式#将 MP4 文件按字节转十六进制】

---

# **实际 FLV 文件的数据结构**

## **FLV header**

```xml
00000001    46 4c 56 01 05 00 00 00  09                         FLV
```

## **FLV body**

```xml
00000001    46 4c 56 01 05 00 00 00  09 00 00 00 00 12 00 01    FLV             
00000002    24 00 00 00 00 00 00 00  02 00 0a 6f 6e 4d 65 74    $          onMet
00000003    61 44 61 74 61 08 00 00  00 0c 00 08 64 75 72 61    aData       dura
.
（内容超多，会导致编辑器渲染时间过长，此处省略）
.
00018956    52 72 71 4a 7c 41 1f 3f  cf 6c 6e dc 15 cf c9 45    RrqJ|A ? ln    E
00018957    49 70 85 7b 2f 9b b5 ba  c1 2d f5 be d5 0d ae ec    Ip {/    -      
00018958    f7 a3 9a eb bf ff ff ff  ff ff 80 00 00 0c 97 d0                    
```

### **保留字段**

```xml
00000001                                00 00 00 00
```

### **第 1 个 FLV TAG（onMetadata）**

```xml
00000001                                            12 00 01                    
00000002    24 00 00 00 00 00 00 00  02 00 0a 6f 6e 4d 65 74    $          onMet
00000003    61 44 61 74 61 08 00 00  00 0c 00 08 64 75 72 61    aData       dura
00000004    74 69 6f 6e 00 40 16 3d  70 a3 d7 0a 3d 00 05 77    tion @ =p   =  w
00000005    69 64 74 68 00 40 74 00  00 00 00 00 00 00 06 68    idth @t        h
00000006    65 69 67 68 74 00 40 6e  00 00 00 00 00 00 00 0d    eight @n        
00000007    76 69 64 65 6f 64 61 74  61 72 61 74 65 00 40 88    videodatarate @ 
00000008    6a 00 00 00 00 00 00 09  66 72 61 6d 65 72 61 74    j       framerat
00000009    65 00 40 39 00 00 00 00  00 00 00 0c 76 69 64 65    e @9        vide
00000010    6f 63 6f 64 65 63 69 64  00 40 00 00 00 00 00 00    ocodecid @      
00000011    00 00 0d 61 75 64 69 6f  64 61 74 61 72 61 74 65       audiodatarate
00000012    00 40 4f e9 00 00 00 00  00 00 0f 61 75 64 69 6f     @O        audio
00000013    73 61 6d 70 6c 65 72 61  74 65 00 40 e5 88 80 00    samplerate @    
00000014    00 00 00 00 0f 61 75 64  69 6f 73 61 6d 70 6c 65         audiosample
00000015    73 69 7a 65 00 40 30 00  00 00 00 00 00 00 06 73    size @0        s
00000016    74 65 72 65 6f 01 00 00  0c 61 75 64 69 6f 63 6f    tereo    audioco
00000017    64 65 63 69 64 00 40 00  00 00 00 00 00 00 00 07    decid @         
00000018    65 6e 63 6f 64 65 72 02  00 0c 4c 61 76 66 35 32    encoder   Lavf52
00000019    2e 31 30 33 2e 30 00 08  66 69 6c 65 73 69 7a 65    .103.0  filesize
00000020    00 41 12 83 7c 00 00 00  00 00 00 09                 A  |
```

#### **onMetadata header**

```xml
00000002                             02 00 0a 6f 6e 4d 65 74               onMet
00000003    61 44 61 74 61                                      aData
```

#### **onMetadata body**

```xml
00000003                   08 00 00  00 0c 00 08 64 75 72 61                dura
00000004    74 69 6f 6e 00 40 16 3d  70 a3 d7 0a 3d 00 05 77    tion @ =p   =  w
00000005    69 64 74 68 00 40 74 00  00 00 00 00 00 00 06 68    idth @t        h
00000006    65 69 67 68 74 00 40 6e  00 00 00 00 00 00 00 0d    eight @n        
00000007    76 69 64 65 6f 64 61 74  61 72 61 74 65 00 40 88    videodatarate @ 
00000008    6a 00 00 00 00 00 00 09  66 72 61 6d 65 72 61 74    j       framerat
00000009    65 00 40 39 00 00 00 00  00 00 00 0c 76 69 64 65    e @9        vide
00000010    6f 63 6f 64 65 63 69 64  00 40 00 00 00 00 00 00    ocodecid @      
00000011    00 00 0d 61 75 64 69 6f  64 61 74 61 72 61 74 65       audiodatarate
00000012    00 40 4f e9 00 00 00 00  00 00 0f 61 75 64 69 6f     @O        audio
00000013    73 61 6d 70 6c 65 72 61  74 65 00 40 e5 88 80 00    samplerate @    
00000014    00 00 00 00 0f 61 75 64  69 6f 73 61 6d 70 6c 65         audiosample
00000015    73 69 7a 65 00 40 30 00  00 00 00 00 00 00 06 73    size @0        s
00000016    74 65 72 65 6f 01 00 00  0c 61 75 64 69 6f 63 6f    tereo    audioco
00000017    64 65 63 69 64 00 40 00  00 00 00 00 00 00 00 07    decid @         
00000018    65 6e 63 6f 64 65 72 02  00 0c 4c 61 76 66 35 32    encoder   Lavf52
00000019    2e 31 30 33 2e 30 00 08  66 69 6c 65 73 69 7a 65    .103.0  filesize
00000020    00 41 12 83 7c 00 00 00  00 00 00 09                 A  |
```

### **第 1 个 TAG 的长度**

```xml
00000020                                         00 00 01 2f                   /
```

### **第 2 个 FLV TAG（AudioTag）**

```xml
00000021    08 00 00 d1 00 00 00 00  00 00 00 2e ff fb 50 c4               .  P 
00000022    00 00 09 01 23 15 20 8c  6b c9 79 29 e5 26 86 80        #   k y) &  
00000023    01 06 04 0c 05 52 2f 04  94 c6 48 5e 8a 01 5e 22         R/   H^  ^"
00000024    57 e8 19 97 e8 40 05 c2  af 96 69 9a 58 44 e9 ee    W    @    i XD  
00000025    73 4f ae f9 7f 4d 0a bc  7f 79 4d 0a 15 3f ff 84    sO   M   yM  ?  
00000026    95 fe 93 f1 0b f4 27 ce  29 97 b9 a1 3c 4f ff 85          ' )   <O  
00000027    88 07 80 3e 72 ce ef bf  7d 32 10 00 01 02 34 a8       >r   }2    4 
00000028    f1 c1 08 4c 7a 07 2b 05  11 3c 7d 1e 49 40 28 23       Lz +  <} I@(#
00000029    c5 58 70 29 57 3d 7b f1  15 49 2f 63 03 cb 1d c1     Xp)W={  I/c    
00000030    70 34 5c 7c 25 a7 57 dc  dc 25 5d a8 bb 4a 7f 7f    p4\|% W  %]  J  
00000031    ff f2 f7 bf 15 03 b8 ac  bb d2 62 90 b1 45 21 14              b  E! 
00000032    f9 ec 7d 70 ef 08 ef f2  ff f3 58 c5 e1 6e f8 1c      }p      X  n  
00000033    f2 d5 77 57 06 74 7e 95  c4 00 10 00 00 10 04 00      wW t~         
00000034    06 63 5e 6e 50 11 a0 d5  c2 86 f0 e0                 c^nP           
```

#### **AudioTagHeader**

```xml
00000021                                      2e                           .
```

#### **AUDIODATA**

```xml
00000021                                         ff fb 50 c4                  P 
00000022    00 00 09 01 23 15 20 8c  6b c9 79 29 e5 26 86 80        #   k y) &  
00000023    01 06 04 0c 05 52 2f 04  94 c6 48 5e 8a 01 5e 22         R/   H^  ^"
00000024    57 e8 19 97 e8 40 05 c2  af 96 69 9a 58 44 e9 ee    W    @    i XD  
00000025    73 4f ae f9 7f 4d 0a bc  7f 79 4d 0a 15 3f ff 84    sO   M   yM  ?  
00000026    95 fe 93 f1 0b f4 27 ce  29 97 b9 a1 3c 4f ff 85          ' )   <O  
00000027    88 07 80 3e 72 ce ef bf  7d 32 10 00 01 02 34 a8       >r   }2    4 
00000028    f1 c1 08 4c 7a 07 2b 05  11 3c 7d 1e 49 40 28 23       Lz +  <} I@(#
00000029    c5 58 70 29 57 3d 7b f1  15 49 2f 63 03 cb 1d c1     Xp)W={  I/c    
00000030    70 34 5c 7c 25 a7 57 dc  dc 25 5d a8 bb 4a 7f 7f    p4\|% W  %]  J  
00000031    ff f2 f7 bf 15 03 b8 ac  bb d2 62 90 b1 45 21 14              b  E! 
00000032    f9 ec 7d 70 ef 08 ef f2  ff f3 58 c5 e1 6e f8 1c      }p      X  n  
00000033    f2 d5 77 57 06 74 7e 95  c4 00 10 00 00 10 04 00      wW t~         
00000034    06 63 5e 6e 50 11 a0 d5  c2 86 f0 e0                 c^nP           
```

### **第 2 个 TAG 的长度**

```xml
00000034                                         00 00 00 dc
```

### **第 3 个 FLV TAG（VideoTag）**

```xml
00000035    09 00 34 b2 00 00 00 00  00 00 00 12 00 00 84 02      4             
00000036    92 26 02 02 02 03 ff ff  30 10 10 10 1f ff f9 80     &      0       
00000037    80 80 80 ff ff cc 04 04  04 07 ff fe 60 20 20 20                `   
00000038    3f ff f3 01 01 01 01 ff  ff 98 08 08 08 0f ff fc    ?               
00000039    c0 40 40 40 7f ff e6 02  02 02 03 ff ff 30 10 10     @@@         0  
00000040    10 1f ff f9 80 80 80 80  ff ff cc 04 04 04 07 ff                    
00000041    fe 60 20 20 20 3f ff f3  01 01 01 01 ff ff 98 08     `   ?          
00000042    08 08 0f ff fc c0 40 40  40 7f ff e6 02 02 02 03          @@@       
00000043    ff ff 30 10 10 10 1f ff  f9 80 80 80 80 ff ff cc      0             
00000044    04 04 04 07 ff fd c8 08  08 a8 30 1d 21 00 14 c2              0 !   
00000045    4a 02 e0 69 bf 05 41 80  e8 08 00 a6 12 50 2a 06    J  i  A      P* 
00000046    9b f1 f0 10 95 ac 5a e8  40 3e a9 72 c7 b9 01 01          Z @> r    
00000047    16 06 03 a0 20 02 98 49  40 a8 1a 6f c1 60 60 3a           I@  o ``:
00000048    02 00 29 84 94 0a 81 a6  fc 7c 04 25 6b 16 ba 10      )      | %k   
00000049    0f aa 5c b1 ee 40 40 45  81 80 e8 08 00 a6 12 40      \  @@E       @
00000050    82 a0 69 bf 05 81 80 e8  08 00 a6 12 40 82 a0 69      i         @  i
00000051    bf 1f 01 09 5a c5 ae 84  03 ea 97 2c 7b 90 10 11        Z      ,{   
00000052    60 60 3a 02 00 29 84 94  0a 81 a6 fc 16 06 03 a0    ``:  )          
00000053    20 02 98 49 02 0a 81 a6  fc 7c 04 25 6b 16 ba 10       I     | %k   
00000054    0f aa 5c b1 ee 40 40 45  81 80 e8 08 00 a6 12 40      \  @@E       @
00000055    82 a0 69 bf 05 81 80 e8  08 00 a6 12 40 82 a0 69      i         @  i
00000056    bf 1f 01 09 5a c5 ae 84  03 ea 97 2c 7b 90 10 11        Z      ,{   
00000057    60 60 3a 02 00 29 84 90  20 a8 1a 6f c1 60 60 3a    ``:  )     o ``:
00000058    02 00 29 84 94 0a 81 a6  fc 7c 04 25 6b 16 ba 10      )      | %k   
00000059    0f aa 5c b1 ee 40 40 45  81 80 e8 08 00 a6 12 50      \  @@E       P
00000060    2a 06 9b f0 58 18 0e 80  80 0a 61 25 02 a0 69 bf    *   X     a%  i 
00000061    1f 01 09 5a c5 ae 84 03  ea 97 2c 7b 90 10 11 60       Z      ,{   `
00000062    60 3a 02 00 29 84 94 0a  81 a6 fc 15 06 03 a4 20    `:  )           
00000063    02 98 49 40 5c 0d 37 e3  e0 1e 56 b1 6b a1 00 fa      I@\ 7   V k   
00000064    a5 cb 1e e4 04 04 54 18  0e 90 80 0a 61 25 01 70          T     a% p
00000065    34 df 82 a0 c0 74 84 00  53 09 68 15 03 4d f8 f8    4    t  S h  M  
00000066    07 95 ac 5a e8 40 3e a9  72 c7 b9 01 01 15 06 03       Z @> r       
00000067    a4 20 02 98 4b 40 a8 1a  6f c1 50 60 3a 42 00 29        K@  o P`:B )
00000068    84 b4 0a 81 a6 fc 7c 03  ca d6 2d 74 20 1f 54 b9          |   -t  T 
00000069    63 dc 80 80 8a 03 01 d4  10 81 4c 25 a0 54 0d 37    c         L% T 7
00000070    e0 a0 30 1d 41 08 14 c2  5a 05 40 d3 7e 3e 81 ef      0 A   Z @ ~>  
00000071    ac 30 84 03 ea 97 2c 7b  90 10 11 40 60 3a c2 10     0    ,{   @`:  
00000072    18 12 d0 2a 06 9b f0 4c  18 0e b0 84 06 04 b4 0a       *   L        
00000073    81 a6 fc 7d 03 df 58 61  06 07 fc b8 c5 c8 08 08       }  Xa        
00000074    98 30 1d 81 08 0c 09 68  15 03 4d f8 26 0c 07 60     0     h  M &  `
00000075    42 03 02 5a 05 40 d3 7e  3e 8f 3e 4d 06 07 fc b8    B  Z @ ~> >M    
00000076    c5 c8 08 08 90 30 1d 81  08 0c 0f d0 2a 06 9b f0         0      *   
00000077    48 18 0e c1 20 0c 0f d0  2a 06 9b f5 f4 79 f2 68    H       *    y h
00000078    30 3f e5 c6 2e 40 40 44  81 80 ed 12 00 c0 fd 02    0?  .@@D        
00000079    a0 69 bf 44 41 80 ed 12  00 c0 fd 02 a0 69 bf 5f     i DA        i _
00000080    47 9f 26 83 1e f9 71 8b  90 10 11 10 60 3b 84 80    G &   q     `;  
00000081    30 3f 41 e0 69 bf 44 01  80 ee 12 00 c0 fd 07 81    0?A i D         
00000082    a6 fd 7d 1e 7c 9a 0c 7b  e2 77 20 20 22 00 c0 77      } |  { w  "  w
00000083    09 00 60 7e 83 c0 d3 7e  88 03 01 de 25 2c 3f 41      `~   ~    %,?A
00000084    e0 69 bf 5f 47 9f 26 83  1e f8 9d c8 08 08 78 30     i _G &       x0
00000085    1e 02 52 c3 f4 1e 06 9b  f4 3c 18 0f 01 29 62 f4      R      <   )b 
00000086    1e 06 9b f5 f9 47 c9 a0  c7 be 27 72 02 02 1c 0c         G    'r    
00000087    07 88 94 b1 7a 0f 03 4d  fa 1c 0c 07 88 94 b1 7a        z  M       z
00000088    0f 03 4d fa fc a1 f0 43  d7 39 01 01 0e 06 03 c4      M    C 9      
00000089    4a 58 bd 07 81 a6 fd 0d  06 03 c8 7c b1 7a 0f 0b    JX         | z  
00000090    5f 94 3e 09 4b b9 e1 f3  c3 e8 27 d0 4f b7 c8 73    _ > K     ' O  s
00000091    d3 e7 a7 d0 8f a1 1f 6f  90 e7 a7 cf 4f a1 1f 43           o    O  C
00000092    3e df 21 cf 8f 9f 1f 43  3e 86 7d be 43 9f 1f 3e    > !    C> } C  >
00000093    3e 86 7d 0c fb 7c 8f 3e  3e 7c 7d 0c fa 21 f6 f9    > }  | >>|}  !  
00000094    0e 7c 7c f8 fa 19 f4 33  ed f2 1c f4 f9 e1 f4 23     ||    3       #
00000095    e8 47 db e4 39 e1 f3 b3  e8 27 d0 0f b7 c8 3c ec     G  9    '    < 
00000096    f9 d1 f3 d5 1f d9 ec bb  4c 1e dd f5 cb c7 ba f8            L       
00000097    7e 25 2b fd f7 8b fd 7f  18 86 e2 e8 f8 7e ab d3    ~%+          ~  
00000098    ff 2e fc f5 6a 98 6e 72  70 7c f4 f2 ec 6e 58 46     .  j nrp|   nXF
00000099    e9 15 97 5f df 45 7e bf  8d fa 58 7e 32 aa 2f 92       _ E~   X~2 / 
00000100    7e aa fc f5 6b f9 4f 3a  37 3e 6e 78 7c e2 c5 70    ~   k O:7>nx|  p
00000101    b8 0f a8 b6 6e 48 23 6a  3a 61 d4 bc bd 50 fb d7        nH#j:a   P  
00000102    ea ee 47 45 97 17 7c 7f  f9 e5 53 6b de 6a 7c d4      GE  |   Sk j| 
00000103    f9 af a8 fb e5 df c5 72  ab 6e 28 62 38 e4 b9 5d           r n(b8  ]
00000104    56 3a be f4 cd 1d ed dc  8b 98 82 06 03 bc 7c 3d    V:            |=
00000105    92 4b 55 5c 05 1a c8 b0  c3 f0 30 12 05 e3 cb 6d     KU\      0    m
00000106    91 5c 05 21 56 52 59 4d  0c cd cd 8f b9 c6 cc c8     \ !VRYM        
00000107    c0 d8 d5 ce 3b 25 e5 c6  87 cc 8f b9 c6 e4 b4 b0        ;%          
00000108    c4 f9 81 f7 48 c9 95 94  97 16 ba c6 4c a0 9c b0        H       L   
00000109    ad da 2e 64 c4 a5 25 0e  d1 5d 18 fa 31 f4 b3 e9      .d  %  ]  1   
00000110    87 db 64 ba 39 f4 73 e9  a7 d3 8f b6 49 74 83 e9      d 9 s     It  
00000111    07 d3 8f a7 1f 6c 92 e9  07 d2 4f a7 9f 4f 3e d9         l    O  O> 
00000112    27 d2 4f a5 1f 4f 3e a0  7d ae 4b a5 1f 4a 3e a0    ' O  O> } K  J> 
00000113    7d 40 fb 64 97 49 3e 92  7d 40 fa 79 f6 c9 2e 90    }@ d I> }@ y  . 
00000114    7d 20 fa 79 f4 d3 ed b2  4f 47 3e 76 08 12 17 8f    }  y    OG>v    
00000115    ee c9 67 95 5f e7 e6 c4  6e 4d 3e 90 5c 25 ab 1f      g _   nM> \%  
00000116    5d 55 72 e6 10 38 2b f2  95 5f f3 5b 2b a3 d5 59    ]Ur  8+  _ [+  Y
00000117    ef ca b3 de 7a 08 2a c0  ff da 91 21 a3 d1 20 7e        z *    !   ~
00000118    3e 2e 90 7a a8 7a 97 d2  ca 5b 1e 91 f2 fa ae eb    >. z z   [      
00000119    d2 8f c6 50 60 04 80 36  7e ff df 2e d5 d4 dd 18       P`  6~  .    
00000120    3a 40 60 1b c1 02 fa 7b  fe 2f c5 94 0a de 76 10    :@`    { /    v 
00000121    42 1a a1 fe 01 ef 51 f6  4c 93 db 10 77 5c 7c 3e    B     Q L   w\|>
00000122    56 ab 28 28 7f 7f 58 52  07 fe 5b 29 f4 93 e9 27    V ((  XR  [)   '
00000123    e4 01 80 8f 06 01 ca 7c  7a 3a 52 bc 56 e7 10 60           |z:R V  `
00000124    3c 41 80 0e 50 a0 0e e2  c3 c7 bc a8 21 0f 84 8f    <A  P       !   
00000125    c8 ad a9 6f 2c f5 fc b1  60 3b 79 49 4a 41 80 71       o,   `;yIJA q
00000126    9f 54 ab d3 75 52 7a d4  4d 87 c8 c1 80 0e 2e f8     T  uRz M     . 
00000127    30 11 85 df cd 9f 52 57  61 a2 51 f8 42 54 5f 09    0     RWa Q BT_ 
00000128    a3 7e 25 17 8f 2f bf 83  d4 fd ba cf 4c ba 41 2c     ~%  /      L A,
00000129    ba 2b df 68 f3 51 61 13  a3 f5 43 cc d5 43 dd 64     + h Qa   C  C d
00000130    0f fb 30 66 78 6a 24 c5  42 50 95 07 c3 c9 2c bb      0fxj$ BP    , 
00000131    b3 6c a8 4e a3 17 17 ee  f9 5a b5 76 31 29 d9 b8     l N     Z v1)  
00000132    3c f1 7c 85 fb 7d b1 4e  1c 66 03 bf 1f 7e 8f b2    < |  } N f   ~  
00000133    7f 2a 8d 3a b3 d3 e7 87  d0 cf a0 9f 70 8d 4c 6b     * :        p Lk
00000134    1d 1f ce 8f 9c 9f 40 3e  7e 7d c2 43 9c 1f 36 3e          @>~} C  6>
00000135    7a 7c ec fb 8c 7e 0d 4f  9a 1f 3a 38 3e e5 1f 93    z|   ~ O  :8>   
00000136    23 03 63 e6 a7 dc e3 72  5e 5c 66 7c c8 fb 9c 64    # c    r^\f|   d
00000137    cb 0a cc 0b dd 63 3a 81  f5 23 ea e7 d6 0f b5 49         c:  #     I
00000138    f5 33 ea 67 d6 0f ac 9f  6a 94 ea 87 d5 0f ad 1f     3 g    j       
00000139    5b 3e d5 29 d5 4f ab 1f  5b 3e b8 7d a6 57 ab 1f    [> ) O  [> } W  
00000140    56 3e b8 7d 70 fb 4c a7  56 3e ac 7d 70 fa e1 f6    V> }p L V> }p   
00000141    a9 4e ac 7d 54 fa e1 f5  b3 ed 52 9d 50 fa 99 f5     N }T     R P   
00000142    a3 eb 47 da e5 3a 99 f5  23 eb 27 d6 0f b5 ca 75      G  :  # '    u
00000143    13 ea 07 d5 cf ab 1f 6b  93 e9 e7 d3 8f aa 9f 52           k       R
00000144    2e be aa e4 57 6f 95 33  7d 36 c8 9a 3d b2 4d e2    .   Wo 3}6  = M 
00000145    c0 c0 0a 03 03 1c 10 00  a5 a8 7e f2 90 60 1a 04              ~  `  
00000146    92 f1 20 21 8f 80 f5 c9  47 db 63 16 8f 69 b1 60       !    G c  i `
00000147    0c 12 c0 30 7c 10 84 a1  f4 08 73 f1 52 a9 25 fc       0|     s R % 
00000148    cf a3 90 d1 68 30 0c 9e  1f 17 a8 08 7e 12 65 be        h0      ~ e 
00000149    f4 55 07 d8 de fe db 54  ea 76 8f 3b fa 50 43 ef     U     T v ; PC 
00000150    ae 5b 4c 44 5b 00 f7 3f  36 61 97 a1 89 61 06 89     [LD[  ?6a   a  
00000151    6d c9 1e 99 ee 3d 40 fa  79 f7 6a 08 01 02 dd a2    m    =@ y j     
00000152    57 ad 21 89 80 82 10 7d  14 41 2f f2 10 74 83 e8    W !    } A/  t  
00000153    e7 d3 8f a6 9f 6d 92 e8  c7 d1 0f a6 1f 4a 3e dd         m       J> 
00000154    25 d0 cf a0 9f 48 3e 8c  7d ba 47 9f 9f 3d 3e 8a    %    H> } G  => 
00000155    7d 0a 3d be 47 27 67 28  07 cf 8f b8 48 48 6e 6c    } = G'g(    HHnl
00000156    78 7c e9 c6 3e 66 a6 67  06 ee 51 dd 70 fa e9 f6    x|  >f g  Q p   
00000157    23 ec 67 da 65 7a f9 f6  03 ec 87 d9 4f b4 4b 76    # g ez      O Kv
00000158    13 ec 27 d9 8f b3 9f 67  95 ec 47 d8 cf b4 1f 69      '    g  G    i
00000159    3e cf 2d d8 cf b1 9f 69  3e d2 7d 9e 5b b1 9f 63    > -    i> } [  c
00000160    3e d2 7d a4 fb 3c b7 62  3e c2 7d a4 fb 39 f6 79    > }  < b> }  9 y
00000161    6e c0 7d 80 fb 39 f6 73  ed 12 ab 60 3e be 7d 98    n }  9 s   `> } 
00000162    fb 11 77 be 5f f8 a9 54  bf fd 6a 39 a0 f4 b3 d7      w _  T  j9    
00000163    4f ae 9f 5f a0 a0 12 04  90 50 03 00 e2 10 c1 80    O  _     P      
00000164    8d f4 55 6d 1f fc 4a 56  25 81 d6 3c a1 55 96 42      Um  JV%  < U B
00000165    56 81 2c 10 41 80 14 55  55 40 84 0c 03 7f 80 35    V , A  UU@     5
00000166    4e ec 82 41 78 fb ea ab  11 47 d4 a9 9f 62 fa f1    N  Ax    G   b  
00000167    98 e6 94 55 29 89 54 6e  eb 87 d6 a5 eb 98 cf af       U) Tn        
00000168    aa 2f 51 f9 f5 93 4b 97  39 73 f2 fa e2 77 34 ca     /Q   K 9s   w4 
00000169    bd 33 c0 83 f9 f0 60 19  f6 17 60 06 03 00 28 25     3    `   `   (%
00000170    89 22 52 95 52 e0 ff d6  1e 66 06 00 38 18 06 e0     "R R    f  8   
00000171    86 0c 03 81 77 fe 24 a8  05 15 93 ca 3d 3c dc e9        w $     =<  
00000172    a5 a1 2c 18 01 a0 60 17  30 20 d0 81 00 38 7f e0      ,   ` 0    8  
00000173    60 1c 40 3a 89 43 fb 4b  c7 ca af c4 a2 fc fb 5f    ` @: C K       _
00000174    c1 e2 b4 fe 25 6c 08 00  c0 08 83 00 e4 0c 00 a1        %l          
00000175    7a b1 20 10 41 09 55 c9  fa 5c 5d f0 3f db 39 6a    z   A U  \] ? 9j
00000176    b9 c6 98 95 ed 52 97 5d  cb 2d b9 c2 29 3b 3f 33         R ] -  );?3
00000177    65 8e ea e7 d5 8f ae 9f  5c 3e d5 29 d5 0f a9 9f    e       \> )    
00000178    5a 3e b2 7d aa 53 a9 1f  50 3e b0 7d 58 fb 5c a7    Z> } S  P> }X \ 
00000179    4e 3e 98 7d 50 fa 91 f6  c9 3e 94 7d 22 3d 42 3d    N> }P    > }"=B=
00000180    38 fb 64 9f 45 3e 86 7d  2c fa 49 f6 e9 2e 82 7c    8 d E> }, I  . |
00000181    fc fa 39 f4 48 f6 f9 1e  7a 7c ec fa 11 f4 03 ee      9 H   z|      
00000182    11 fd a0 fb 49 f6 e3 ed  e7 d9 e5 fb 51 f6 b3 ee        I       Q   
00000183    07 dc 8f b3 4b f6 d3 ed  c7 dc cf ba 1f 66 97 ed        K        f  
00000184    e7 db cf ba 9f 76 3e cb  2f dc 0f b8 1f 77 3e ee         v> /    w> 
00000185    7d 96 63 b8 1f 70 3e ee  7d d8 fb 2c bb db cf b7    } c  p> }  ,    
00000186    1f 76 3f 10 10 c1 80 69  08 02 47 3d 68 43 2f dc     v?    i  G=hC/ 
00000187    9e 1f 77 da b5 2f 03 51  16 bd 78 4a 00 e1 24 21      w  / Q  xJ  $!
00000188    dc 1f 17 5f a7 6b d1 2f  e9 c9 af ab f7 9b 7b db       _ k /      { 
00000189    8f b6 7b 7f 73 d3 5f 20  08 01 0c 18 06 60 60 1b      { s _      `` 
00000190    87 e1 06 02 00 90 3e 56  ac 10 3d 07 ea c1 80 0f          >V  =     
00000191    51 7e 25 8f c7 ca 54 7e  aa bb 2f d4 5b 14 a8 b2    Q~%   T~  / [   
00000192    f6 63 9b 01 80 0e 12 55  03 00 dc 24 2b f0 fb d4     c     U   $+   
00000193    48 08 32 d1 f5 12 bf 73  e2 50 43 93 41 0a af 15    H 2    s PC A   
00000194    b4 ab e8 ae 3d 58 bc 18  00 e0 42 b6 7c 4b 2e 57        =X    B |K.W
00000195    4b 94 51 2f c5 d7 7e 3e  8a da b7 5b f5 5e 19 ab    K Q/  ~>   [ ^  
00000196    04 10 60 1c 87 e2 42 a2  e5 63 e1 2b 55 2b 57 3d      `   B  c +U+W=
00000197    47 ff bc f6 d9 37 19 43  c3 2f 6b 12 c1 80 11 06    G    7 C /k     
00000198    01 a4 03 7f e1 20 18 01  38 ab 6f af c7 e2 5f ec            8 o   _ 
00000199    50 a9 62 ef 56 7e f6 e0  60 07 c2 10 06 83 00 28    P b V~  `      (
00000200    0c 03 44 2e 06 01 b6 fc  10 fe a5 57 ed 2f 1e aa      D.       W /  
00000201    53 f9 a5 d8 06 35 a3 c9  40 c0 11 80 78 fc 48 f4    S    5  @   x H 
00000202    a3 f9 73 54 59 9d 21 29  06 01 a8 21 04 20 41 08      sTY !)   !  A 
00000203    03 f1 f9 72 af 50 86 07  80 e7 da b7 fe ef ba bb       r P          
00000204    11 eb 7f 00 f5 62 47 fe  01 d2 2a f8 06 2b 83 cb         bG   *  +  
00000205    7d 22 9d a0 a5 aa db 7d  29 72 81 24 ba 97 8f 6c    }"     })r $   l
00000206    12 55 5d ca 23 59 18 73  db 01 80 63 06 01 a9 5f     U] #Y s   c   _
00000207    c7 ea d5 02 18 91 ce 61  78 90 aa 7e 23 fc be a9           ax  ~#   
00000208    4f cd aa 06 01 c8 bd 51  7e 97 d2 eb de 79 57 8b    O      Q~    yW 
00000209    ad a9 01 40 aa 65 21 7c  06 01 9c 18 01 40 60 1c       @ e!|     @` 
00000210    a8 07 ab 57 e1 22 97 cb  20 1a be fd 02 1a a4 65       W "         e
00000211    3e 5e 24 17 89 57 04 b1  f1 78 fa ac 5e ab e5 d4    >^$  W   x  ^   
00000212    b6 39 20 48 08 00 c0 08  ff c5 d4 7b 14 e4 df 52     9 H       {   R
00000213    f5 1b 52 53 d5 d0 18 07  1b fb 7d 15 aa b6 72 c6      RS      }   r 
00000214    5c f7 60 60 04 41 80 6d  06 00 3c 18 07 0f f8 4a    \ `` A m  <    J
00000215    f0 07 f8 4a 08 7f e4 12  84 bf 2b 96 d5 a2 b5 53       J      +    S
00000216    e3 ea 04 74 91 51 50 30  02 01 0f 6d f7 95 0f c7       t QP0   m    
00000217    9d 51 62 99 56 53 e6 43  2a 12 ed 1f 17 f8 49 1f     Qb VS C*     I 
00000218    58 5c 25 f9 a1 fc 9e 53  11 d8 f8 81 28 18 06 20    X\%    S    (   
00000219    60 1b cb 95 2b 12 87 e5  f0 03 c7 c5 f7 68 21 d5    `   +        h! 
00000220    70 7f e6 01 44 3f f7 e5  f4 65 46 5f 69 84 60 86    p   D?   eF_i ` 
00000221    0c 00 aa a5 7d 08 18 a8  0b 7e df f9 15 31 59 02        }    ~   1Y 
00000222    1a b5 33 55 97 c9 18 f2  8e a1 8f 7a 61 78 30 0a      3U       zax0 
00000223    a0 c0 38 02 04 85 c2 58  30 03 55 5f d5 72 51 20      8    X0 U_ rQ 
00000224    7e a1 56 24 ff 95 e6 11  b3 2b 06 01 b4 21 83 00    ~ V$     +   !  
00000225    da 10 cb c2 08 fc 03 20  07 7a cf 49 07 a3 de fa             z I    
00000226    47 4e 80 68 97 44 9b b2  17 b7 bd 57 15 95 9e 9d    GN h D     W    
00000227    08 40 c0 37 83 00 e0 0c  00 74 ff c0 3a 4e c5 0a     @ 7     t  :N  
00000228    d5 2b 9e 1f 8f 7c 9b d7  27 b2 b3 ed 56 4c a6 a8     +   |  '   VL  
00000229    18 01 81 2f 5a 12 8b 80  be 8b 28 7e 0c 03 97 38       /Z     (~   8
00000230    ae da c4 57 cf 23 ae 7b  21 f6 4a 5f ef 4d ff e7       W # {! J_ M  
00000231    a2 ff b2 41 9b b0 30 02  e0 c0 35 09 60 c0 38 6d       A  0   5 ` 8m
00000232    04 10 80 0c 00 70 06 09  4a b9 47 ea d5 d1 f0 ff         p  J G     
00000233    f5 37 95 d9 31 7f db 1c  b4 0c 03 79 70 07 04 2f     7  1      yp  /
00000234    2a 56 a8 03 68 07 09 7e  c2 f2 ea 3c 56 ae ca dc    *V  h  ~   <V   
00000235    bb 95 52 79 d7 af 04 30  0c 56 10 3f 9f ba 05 fd      Ry   0 V ?    
00000236    3f 7c 87 1d 46 25 00 7a  b5 7f 51 f1 f4 8b ff c3    ?|  F% z  Q     
00000237    f9 62 0b 3c f5 b2 78 4b  2e ff 3c ad 57 ea d3 f1     b <  xK. < W   
00000238    ec 07 e1 02 18 30 0c a0  c0 37 89 76 89 05 e0 c0         0   7 v    
00000239    34 fc bc 7c b5 00 cf 49  bb cb a3 cf 75 27 c8 db    4  |   I    u'  
00000240    42 15 80 a0 55 85 f4 7f  b3 40 fd 32 ce 7a 5b af    B   U    @ 2 z[ 
00000241    1f 5c 3e ca 7d 90 fb 4c  b7 5b 3e b2 7d 88 fb 01     \> }  L [> }   
00000242    f6 99 5e ac 7d 52 3d 76  3d 6e 3d aa 57 a9 1f 4f      ^ }R=v=n= W  O
00000243    3e af 1e ab 1e d7 29 d3  63 d2 a3 d4 a3 d4 0f b6    >     ) c       
00000244    49 74 73 e8 a7 d3 4f a5  c7 b7 49 77 43 ee a7 de    Its   O   IwC   
00000245    8f bd 9f 65 97 ee e7 de  0f be 9f 7d 94 eb 24 c3       e       }  $ 
00000246    de 4f bd 1f 67 08 02 41  71 79 95 02 e0 60 06 61     O  g  Aqy   ` a
00000247    7b 55 cc f7 f1 d2 d3 d5  ee df 1f 0f a5 03 01 46    {U             F
00000248    f4 ca 0c 01 18 05 b5 02  84 02 a4 c1 08 02 de fa                    
00000249    7d f4 fa 9d 06 00 8c 1a  07 b6 40 41 08 00 18 0c    }         @A    
00000250    00 ac 94 18 07 21 2c 79  7e 06 8b a4 aa c7 ff 4f         !,y~      O
00000251    15 46 13 dc 73 31 75 05  0f c7 85 d1 4f d6 85 76     F  s1u     O  v
00000252    de 48 9d f2 bf f5 55 37  fc 73 de 3c 5d f7 b8 89     H    U7 s <]   
00000253    2a a0 06 00 60 96 01 e3  fc 05 11 77 ff e5 42 29    *   `      w  B)
00000254    7f e4 e4 4d 35 bf d4 51  ed 60 c0 0a 03 00 c4 0c       M5  Q `      
00000255    00 b0 96 0c 03 94 00 cf  89 05 f2 7f d4 10 be 5f                   _
00000256    c5 25 f3 b9 ff 46 b6 6e  1d 8a 06 01 94 18 07 18     %   F n        
00000257    25 80 78 20 89 52 fb 21  7d 55 55 2a 96 7b d2 fa    % x  R !}UU* {  
00000258    64 cf a6 45 6e 9e 50 08  40 c0 09 0f e0 43 9e 08    d  En P @    C  
00000259    6a b3 d3 d0 7f cb 2c fc  d6 b8 33 a4 1f 84 19 98    j     ,   3     
00000260    5d ba 9e 30 bb 9f 5c 5e  10 c1 04 10 6a af 8f fe    ]  0  \^    j   
00000261    ad 58 91 f6 cb 84 8d 54  5c 5c 9a c6 6c 5f f6 d2     X     T\\  l_  
00000262    59 f2 f0 60 04 bd 41 80  10 08 41 04 4a cb 7e 3d    Y  `  A   A J ~=
00000263    1f 17 ab 9a a5 50 41 2e  a3 b1 f0 97 37 33 fe 50         PA.    73 P
00000264    5e a9 75 5f 96 65 75 b1  70 93 f1 2b 4b 87 ff fc    ^ u_ eu p  +K   
00000265    2e 1e 62 a6 7d 3f 35 47  f0 af 0d d5 83 00 1e 25    . b }?5G       %
00000266    83 00 dc 5f 3e 5c 01 9e  fa bd ba 24 df e5 12 2c       _>\     $   ,
00000267    f0 1a f9 7c fe 2a f5 2e  db 85 fb 33 dc ec 38 58       | * .   3  8X
00000268    ae 64 f6 7f c5 d1 55 d9  18 b3 31 46 42 83 36 5f     d    U   1FB 6_
00000269    9f ff 54 c4 cf 7c 20 30  03 00 c0 2a f8 18 01 90      T  | 0   *    
00000270    60 04 0b 81 40 0c 00 97  8b c1 04 b8 7e 5e a8 03    `   @       ~^  
00000271    4b d4 f8 7d 02 08 42 fe  02 15 12 55 8f 3e 24 ab    K  }  B    U >$ 
00000272    12 c7 d6 7d 52 ac aa 95  4c 9f fa 9b fb 1c fc 0c       }R   L       
00000273    00 88 06 c0 60 1c 41 80  63 06 01 a8 18 07 20 0e        ` A c       
00000274    53 40 38 21 a8 56 ab 07  d4 bb 07 f6 cb e5 58 c9    S@8! V        X 
00000275    7f 87 7b 59 37 20 01 80  c0 0d 02 05 08 05 e0 1a      {Y7           
00000276    10 c2 0a a0 83 44 ab 60  fc b8 7d f9 ff 79 5b 79         D `  }  y[y
00000277    85 d9 fb ba dd ff a5 ee  5c 75 28 30 1d 02 48 94            \u(0  H 
00000278    24 97 50 81 82 41 70 40  2f 9f 2e 55 ec 9c 97 bc    $ P  Ap@/ .U    
00000279    f6 7a 6c 6f 96 d6 7e f3  aa 10 04 a1 fd 08 70 4a     zlo  ~       pJ
00000280    12 a0 f8 bd 5c be 1f 89  7f 96 5f 8f 6c 56 d3 3b        \     _ lV ;
00000281    64 b2 98 b4 12 40 f8 43  12 a8 40 b4 0e 58 cb 11    d    @ C  @  X  
00000282    6f 79 74 ef 79 b8 30 01  d4 4a 08 63 ed fa aa 5c    oyt y 0  J c   \
00000283    5e a6 cf c5 eb 34 e9 57  c7 85 d2 61 0a ff c1 80    ^    4 W   a    
00000284    71 03 c0 c0 39 04 00 60  1c bd fb 44 80 60 1c 44    q   9  `   D ` D
00000285    81 f0 96 dd c9 6c c5 68  3e e2 b5 4a bc 25 c9 fd         l h>  J %  
00000286    d9 64 7b 39 7f 94 7e a9  dd 9f b3 f1 6f ba 8c 48     d{9  ~     o  H
00000287    2e f1 70 f9 57 bd d8 a2  49 7f df a3 7b d1 c1 04    . p W   I   {   
00000288    10 00 34 7f 54 8f 84 82  f2 f9 5a aa a2 91 83 80      4 T     Z     
00000289    30 0c 20 c0 34 83 00 da  0c 00 9c 00 e0 60 03 b3    0   4        `  
00000290    d1 5f d5 51 2c 7e a8 20  8f a1 7e 97 5b 55 0f 8b     _ Q,~    ~ [U  
00000291    ac f8 14 ff e9 01 20 20  0f e8 42 aa cb 84 bb b3              B     
00000292    df bf 55 41 50 b5 22 46  06 01 a1 50 42 08 20 c0      UAP "F   PB   
00000293    39 89 60 1f 24 fd 8a 82  12 a9 fe 35 15 5b 9a 9a    9 ` $      5 [  
00000294    d7 b0 02 00 92 ad 50 93  3e ac 7d e1 f8 fb d2 e6          P > }     
00000295    c6 6c cf 1f b5 00 d0 40  04 00 40 2e 56 08 23 e0     l     @  @.V # 
00000296    3f 42 1c bd 2e fa b0 51  74 47 b9 9a 33 7b c0 30    ?B  .  QtG  3{ 0
00000297    03 3e 12 81 80 69 54 3f  1f 82 08 30 02 01 00 ba     >   iT?   0    
00000298    42 f8 3e 55 f1 22 17 89  40 7e c9 2a bb 9e b2 4c    B >U "  @~ *   L
00000299    b7 d3 77 b7 49 9f 87 e0  c0 0b 84 2a 08 34 20 17      w I      * 4  
00000300    7c bc bb 44 9f 0f a5 e2  bc 85 ff 92 37 96 e4 fa    |  D        7   
00000301    e7 57 73 2d b2 4e af 2f  13 fa cb 2b 99 2a aa a8     Ws- N /   + *  
00000302    7f 04 b5 5e d5 7e fc 57  ef fc 77 f9 ff 76 5b bc       ^ ~ W  w  v[ 
00000303    b9 63 31 ea a5 e2 5f ff  fa ac bb f4 48 b5 a8 5d     c1   _     H  ]
00000304    6c f3 b2 44 a0 60 1a 21  75 b4 bc 20 79 5c a2 27    l  D ` !u   y\ '
00000305    bf 98 85 cf 67 08 20 c0  1f 03 00 dc ab 68 40 00        g        h@ 
00000306    d0 0c 08 72 6f 15 81 f0  84 5e 54 3f 54 0d 2b ec       ro    ^T?T + 
00000307    38 96 0c 00 f0 30 0d 01  02 ff c5 e0 c0 07 0f fe    8    0          
00000308    a1 ba 25 60 93 10 97 5b  46 48 c2 40 30 0c 00 83      %`   [FH @0   
00000309    e0 60 7d c1 0b e9 55 5f  54 2a ae 0c 94 c1 80 70     `}   U_T*     p
00000310    06 01 ac 4b 08 20 18 10  55 04 09 40 38 7e 24 cb       K    U  @8~$ 
00000311    e8 25 8f 7e 5e 08 4a 6c  1e fd 53 56 55 1b 15 63     % ~^ Jl  SVU  c
00000312    4f 55 12 00 34 18 06 f5  5f c5 62 4a a2 f5 5c e9    OU  4   _ bJ  \ 
00000313    7f a5 b1 7b c9 3f 46 b7  6a c1 80 6c 06 00 4c 7f       { ?F j  l  L 
00000314    e9 7c 3e 12 c2 12 bd ec  b3 7d 62 6b 2b df 39 f2     |>      }bk+ 9 
00000315    ec a0 74 ba ca 05 fd 7d  46 72 ff 00 f0 82 a8 48      t    }Fr     H
00000316    be 82 49 7d 54 c2 a2 fa  08 7e 89 21 7a 9c 44 79      I}T    ~ !z Dy
00000317    c0 48 1f 04 31 28 7d 22  b0 41 12 40 3f 58 08 76     H  1(}" A @?X v
00000318    7e 22 f7 fd 4a c3 36 aa  25 03 00 1e 5f eb 02 08    ~"  J 6 %   _   
00000319    07 09 4a ed 80 a4 f5 1f  7e d0 2a 25 7c 02 90 e8      J     ~ *%|   
00000320    30 0e 1d c0 3d 0b d5 01  0d 45 54 bb 34 18 0e ef    0   =    ET 4   
00000321    45 73 3d f1 e0 19 dc a8  05 6f b9 2f 04 11 2d 4e    Es=      o /  -N
00000322    82 89 11 73 9f 01 80 12  fd 06 02 38 20 5f b7 4b       s       8 _ K
00000323    b6 d4 b5 f0 60 c0 4e 03  00 25 7e a6 84 2a 92 04        ` N  %~  *  
00000324    31 2e c1 73 71 7a a5 43  e5 42 57 8b bd 3d ef 78    1. sqz C BW  = x
00000325    79 3d 1a 93 c6 93 01 80  6f 00 f1 f0 43 12 bc a8    y=      o   C   
00000326    b8 7c 07 94 2b 53 6e 7b  bc cc e3 ae 01 80 0e 06     |  +Sn{        
00000327    01 b1 50 92 10 4b 84 af  17 2b 83 d9 f9 b6 c6 22      P  K   +     "
00000328    91 77 6d 3e d6 7d cc fb  91 f6 79 7e d0 7d 98 fb     wm> }    y~ }  
00000329    79 f6 d3 ec f2 fd 92 3d  86 3d ac fb 44 7b 3c b7    y      = =  D{< 
00000330    5f 3e b9 1e c9 1e c5 1e  d1 2d d6 63 d5 a3 d7 e3    _>       - c    
00000331    d7 0f b5 4a f5 38 f4 f3  eb 31 ea c7 da e5 36 ff       J 8   1    6 
00000332    02 7e 10 fc 31 f6 39 8f  06 7e 14 fc 41 f8 aa f6     ~  1 9  ~  A   
00000333    29 85 c0 aa 12 55 97 5d  f3 db 47 c0 c0 33 09 65    )    U ]  G  3 e
00000334    c6 63 4f c7 1f 63 3d 30  f5 ff 03 00 b6 24 c3 2b     cO  c=0     $ +
00000335    40 c0 6c 03 40 f5 20 7e  40 fb 20 f3 cb 3a 5d 4d    @ l @  ~@    :]M
00000336    27 7a b4 06 01 58 bc 9d  68 18 01 d5 70 18 06 a0    'z   X  h   p   
00000337    60 18 41 80 6f 00 f0 83  0b 84 b9 01 01 57 8b 95    ` A o        W  
00000338    17 cf 5b 27 ec 92 8f a7  e5 93 27 bf ef d0 54 92      ['      '   T 
00000339    c3 89 40 1c 25 89 37 15  2a af 7a 06 01 c4 18 01      @ % 7 * z     
00000340    90 60 1b c1 80 0f 04 0f  83 00 d6 3e a2 59 75 08     `         > Yu 
00000341    52 02 1d 12 bf ff 17 c1  2b fc bf 1d 59 6f bd 71    R       +   Yo q
00000342    4c 5b a6 17 c4 a0 50 29  f8 91 ff c1 f4 08 3e a2    L[    P)      > 
00000343    40 f7 9b eb fb 2c 45 6c  c7 cc fc 0f e4 57 25 56    @    ,El     W%V
00000344    07 a8 f7 17 3c f8 20 60  12 41 80 11 06 00 3c 18        <  ` A    < 
00000345    01 00 0f 08 03 f0 60 05  a4 12 07 fe 97 ff 12 8b          `         
00000346    94 d8 5c ac 4a fb 2a ff  ff e9 7d fe ea a9 99 27      \ J *   }    '
00000347    97 d7 ce c0 0d 06 00 54  18 06 e5 40 a1 08 25 f0           T   @  % 
00000348    10 01 06 4f 45 6a fb 85  f3 cb 37 ef cb dc 94 0e       OEj    7     
00000349    ba 0c 20 03 00 d0 a8 10  40 38 03 81 80 ec aa c1            @8      
00000350    80 68 00 d1 f2 b1 2e 00  77 be aa 09 0a c1 0a 8e     h    . w       
00000351    a0 21 fe dc 12 e5 8d 51  e5 db 37 36 57 de 03 00     !     Q  76W   
00000352    e0 01 8a c1 00 20 ab 55  f1 28 bc 48 f2 bb f2 f1           U ( H    
00000353    29 51 7b 63 cb 5a d6 bf  8b b8 e0 10 07 c0 84 a8    )Q{c Z          
00000354    b8 10 04 9b d2 ed 83 e1  f8 f7 15 ab 1d db c9 b6                    
00000355    c7 56 ab 57 e0 86 3f 57  b0 79 b5 4f a4 b9 53 a2     V W  ?W y O  S 
00000356    7b df c1 80 17 06 00 50  10 02 08 20 09 20 c0 38    {      P       8
00000357    04 08 5e 0c 03 84 12 95  fd 52 b1 ea a0 3d e1 f5      ^      R   =  
00000358    99 93 fe 97 73 f2 cf cf  f3 7d 5a 87 6a 81 80 69        s    }Z j  i
00000359    08 60 83 f5 74 b9 5e 0f  82 17 b9 02 00 95 e8 3e     `  t ^        >
00000360    1f 48 9e 7a 41 ed 4f f3  f2 a0 c0 1c 03 00 c7 01     H zA O         
00000361    80 19 06 01 aa 0f 82 1d  06 04 40 18 06 e5 01 00              @     
00000362    03 d4 81 a5 5f b7 04 81  26 a6 f7 ed 54 ad 72 e5        _   &   T r 
00000363    7f a6 17 41 80 57 06 00  99 57 42 18 42 9a 3e fa       A W   WB B > 
00000364    30 3d fa 22 50 2d 41 49  48 10 0b c4 90 60 23 c1    0= "P-AIH    `# 
00000365    80 70 50 24 f4 48 08 32  51 20 4b ff 6a ab 39 f1     pP$ H 2Q K j 9 
00000366    f5 b5 6b d8 9a ba 9e 09  10 0f 01 f1 f7 55 97 01      k          U  
00000367    b5 4d ef 98 fc 2a 7b ee  80 30 03 cb c0 33 ea 4b     M   *{  0   3 K
00000368    a8 1c 57 e1 e2 b5 4c 2b  54 06 3e 91 bf 56 1f 74      W   L+T >  V t
00000369    5e 07 47 fa 5d 2f c7 ad  49 2d a9 31 d7 62 48 07    ^ G ]/  I- 1 bH 
00000370    0f c2 10 fe 97 28 b7 7f  93 d9 6d db 72 49 17 b4         (    m rI  
00000371    d3 80 30 03 c0 c0 16 83  00 b4 08 0a 81 0d 54 04      0           T 
00000372    00 81 40 3d 52 af fc 18  0f 8b f5 62 58 95 26 df      @=R      bX & 
00000373    4a 3e 93 22 ba 22 cd d8  d5 ae 32 00 d0 60 1c 42    J> " "    2  ` B
00000374    10 07 2a 9f 56 3f 2e f2  b5 70 14 5b b5 4c 8c d8      * V?.  p [ L  
00000375    aa 5b 90 63 54 a8 7c ac  4b e8 fb ca 6c 45 9a bb     [ cT | K   lE  
00000376    df 22 01 81 04 10 60 20  0f 81 00 03 42 10 fc ba     "    `     B   
00000377    00 76 d0 84 5e ac b9 5f  b5 52 b6 d5 6d 99 cb 97     v  ^  _ R  m   
00000378    c4 29 80 c0 11 83 00 df  42 18 95 42 07 fe 0a 31     )      B  B   1
00000379    29 a4 ea 8c 28 83 00 e4  0c 00 e8 30 0c e3 f1 22    )   (      0   "
00000380    09 4a ef 84 be cf ab 57  3d 3c a6 c5 bd b5 47 91     J     W=<    G 
00000381    d7 b6 84 00 60 04 01 80  6d 06 00 50 03 4b 84 90        `   m  P K  
00000382    50 7f 3d 73 fc bd 55 4d  b0 84 10 0f 1f 04 11 ff    P =s  UM        
00000383    d5 ca aa 5f ad be 96 81  9c b7 95 f3 6a 82 10 fd       _        j   
00000384    52 ac ff ba 48 f2 e1 25  56 aa 57 19 fc 36 e0 01    R   H  %V W  6  
00000385    e0 1a 25 83 00 38 3e 2f  1f 2b 2e 81 0f c0 87 7f      %  8>/ +.     
00000386    d5 77 ca d4 7e 4b ec f8  ee 72 72 4a e2 ab f8 af     w  ~K   rrJ    
00000387    de c6 64 b5 34 7b 80 30  0a 20 a0 53 61 78 21 c9      d 4{ 0   Sax! 
00000388    9e e8 fa 2a fb 5e 03 0a  ea a2 c5 64 0b 40 c0 09       * ^     d @  
00000389    74 4b 83 ef c5 2a 95 f6  f9 7c 35 50 01 fe f5 99    tK   *   |5P    
00000390    64 b1 ab 1e f7 60 60 03  81 80 63 00 e0 60 1c 76    d    ``   c  ` v
00000391    7b df 1f 17 f9 b2 ef d9  6a 59 79 ae 61 82 5a aa    {       jYy a Z 
00000392    72 ec 7c 5e 24 09 02 4c  12 d5 09 21 0e 55 5f de    r |^$  L   ! U_ 
00000393    04 22 e9 6f 5a ba 10 73  1e 82 0c 00 e0 30 01 c0     " oZ  s     0  
00000394    80 10 c1 80 0e 00 e0 60  1c 22 aa 5f f1 2c b8 7b           ` " _ , {
00000395    29 7d 53 27 6c e2 a9 c9  78 b5 23 48 06 01 b4 7c    )}S'l   x #H   |
00000396    25 00 67 a0 fe 0f bf 9f  aa 32 5c 93 fd ab be e8    % g      2\     
00000397    18 09 70 84 25 ab cf 40  42 96 6d ca d2 49 b5 cf      p %  @B m  I  
00000398    5d 3c c8 3f b7 d0 e1 05  2e 99 95 52 af a6 c1 52    ]< ?    .  R   R
00000399    10 30 0d a0 c0 07 00 65  1f a9 f6 f8 be fa 45 65     0     e      Ee
00000400    de e0 fb cc 8f 54 66 82  1d 6a de 45 10 d3 18 f8         Tf  j E    
00000401    4b a5 ea d4 28 46 ec 00  0f 06 00 78 b8 7c aa ab    K   (F     x |  
00000402    f0 07 d4 d0 bd 54 ed 5b  7d 45 cf 6d 1e fe f2 bd         T [}E m    
00000403    98 21 00 60 43 00 d5 63  c1 21 55 57 fc d6 e7 a4     ! `C  c !UW    
00000404    8b db 4c 35 7c 4b 06 00  43 c2 4f c4 8f 04 31 f5      L5|K  C O   1 
00000405    2f f7 ad 57 2f e6 2a 93  27 6b 5b b6 64 36 d8 0c    /  W/ * 'k[ d6  
00000406    07 42 a1 f0 f8 be 03 00  20 24 84 21 24 10 66 41     B       $ !$ fA
00000407    f7 be 5f 8c 81 d9 2e fb  79 2e 7a 7c da 80 30 11      _   . y.z|  0 
00000408    82 4d 9c f7 e5 e7 7e aa  e4 d6 5f 96 25 03 00 2b     M    ~   _ %  +
00000409    e6 c4 b5 77 ec a8 f4 be  96 2d 7f db 36 2c e7 a5       w     -  6,  
00000410    17 83 00 3b f0 0e 08 54  49 08 62 4f af f9 83 e2       ;   TI bO    
00000411    ee c4 6a eb 68 9c de 01  b7 07 84 2e a0 80 0c 02      j h      .    
00000412    e8 fc 4b 1f fc 03 81 80  6c 2f f8 95 ff ee 40 87      K     l/    @ 
00000413    cf 7b ec 48 af 93 2b 0a  71 2b d8 82 18 07 41 ff     { H  + q+    A 
00000414    a7 73 f3 be fc ef ac a5  71 eb 4a c2 10 92 25 4b     s      q J   %K
00000415    04 be ed e2 be 0f 2a 4b  38 88 de 21 7f c4 a0 60          *K8  !   `
00000416    03 81 80 70 12 04 9f 09  10 b8 03 3c ab 34 7c a7       p       < 4| 
00000417    fd b3 f2 fd 5d ff 6a bd  a8 24 3c 77 b3 ef 27 df        ] j  $<w  ' 
00000418    cb d5 2b 12 be a0 be 65  2c a4 50 47 d9 27 0b 87      +    e, PG '  
00000419    ea 87 ea a1 79 77 e2 b5  5f 6a bb bb 1f 74 3e fe        yw  _j   t> 
00000420    01 2f a7 d9 66 3b 91 f6  f8 f7 a3 ee f1 ec d3 1d     /  f;          
00000421    b2 3d a4 fb a4 7b 89 f6  69 7e cd 1e c7 1e dd 1e     =   {  i~      
00000422    d5 1e cf 2f d8 23 d7 4f  b3 47 b1 9f 69 96 f1 27       / # O G  i  '
00000423    e2 cf c7 57 c8 d7 b1 4c  f8 e3 f2 07 e5 0f ca 4d       W   L       M
00000424    b7 1c c3 32 f9 23 f2 53  69 a7 10 60 03 c1 80 24       2 # Si  `   $
00000425    00 d0 41 cb 66 00 51 c0  30 01 e0 c0 15 00 70 42      A f Q 0     pB
00000426    ca 5e 3b 7b 42 b1 25 52  a4 ef 94 54 3e fa b2 27     ^;{B %R   T>  '
00000427    ff 80 60 30 02 c0 1a 01  f6 28 2e fe 3d 34 18 00      `0     (. =4  
00000428    f0 60 0b 9a f8 34 0f 22  22 50 40 12 84 89 85 ca     `   4 ""P@     
00000429    bc e0 d2 f1 f9 70 fa 29  2f fc 55 e5 ed 3c e9 f0         p )/ U  <  
00000430    86 af fc f1 c8 b8 24 2a  f3 67 5e 3a 01 c0 c0 0b          $* g^:    
00000431    00 60 07 48 a1 54 af 0d  56 5c 3e 56 5c ac 7c 5f     ` H T  V\>V\ |_
00000432    ef 7c 7c 5e ab e5 c5 fe  b3 9f 55 f9 9f 95 35 71     ||^      U   5q
00000433    80 06 83 00 aa 0c 00 a0  07 7f 07 b4 bd 71 e1 82                 q  
00000434    d0 84 0c 02 b8 30 03 40  1b c8 01 fe 55 56 2f 1d         0 @    UV/ 
00000435    0c 5d a8 90 25 de a8 7c  48 21 8f a7 25 73 e6 81     ]  %  |H!  %s  
00000436    80 6e 08 40 c0 51 03 00  b9 4b b4 0f 80 62 a5 73     n @ Q   K   b s
00000437    e2 42 b6 f4 10 15 ec 02  22 53 4a e2 fe d3 3f e0     B      "SJ   ? 
00000438    c0 2e c1 f0 06 83 00 2a  aa f9 42 b2 f8 3f 2e 56     .     *  B  ?.V
00000439    10 3f 3b 7b 2c fc f2 fe  2f f3 71 34 a6 13 c1 80     ?;{,   / q4    
00000440    56 06 00 84 18 00 e0 60  1b 01 80 11 12 c1 04 18    V      `        
00000441    01 0c 2f 04 10 81 28 43  2e 85 c5 f3 e0 87 e9 34      /   (C.      4
00000442    7e 5d 19 9f be ef 93 cc  33 3c 0c 00 88 41 f6 09    ~]      3<   A  
00000443    03 e2 e1 f7 e8 43 aa ff  44 8f 04 35 73 e5 ca b9         C  D  5s   
00000444    94 be 17 17 4f 51 e8 8f  20 f6 8e a6 e1 b4 30 60        OQ        0`
00000445    03 84 90 60 3c c2 10 f9  5f e7 e0 41 04 0c f1 7e       `<   _  A   ~
00000446    f1 4f a4 df 2b a5 36 33  1f 56 0c 03 90 94 07 87     O  + 63 V      
00000447    f0 bc bb 33 b6 59 79 8b  50 28 30 77 90 8e 0c 01       3 Yy P(0w    
00000448    b8 30 0c ca 81 80 64 06  00 62 82 00 06 e1 78 06     0    d  b    x 
00000449    83 00 1f ef 04 20 87 3f  da 24 84 0b 30 21 09 29           ? $  0! )
00000450    ed 90 0f 2b ad 81 f5 4a  5b 0c ec 01 80 67 06 00       +   J[    g  
00000451    48 20 83 00 1e 24 c1 f8  96 ab d0 b8 03 ef ec f8    H    $          
00000452    fa e9 74 2f 6e fa 33 e4  94 67 5c 0c 00 f8 30 0c      t/n 3  g\   0 
00000453    e2 58 30 03 a0 c0 36 89  40 c0 39 0f bb ec a3 e0     X0   6 @ 9     
00000454    86 3e 60 7e 10 95 e2 9f  25 f5 c2 f5 4b 7f ce 35     >`~    %   K  5
00000455    04 10 84 a8 18 06 c0 60  03 bf ef 59 4b e0 ec 7d           `   YK  }
00000456    f5 87 fe 2d 93 f0 74 fa  50 80 25 a8 be 1f c6 af       -  t P %     
00000457    3f 06 2f 28 08 40 c0 11  03 00 2a 10 82 0c 04 18    ? /( @    *     
00000458    24 01 f1 27 d7 47 df 12  e8 f6 0f aa ea a6 62 95    $  ' G        b 
00000459    bf 76 c3 01 97 e7 91 44  80 60 19 41 80 eb fa af     v     D ` A    
00000460    82 00 07 c1 f5 b9 68 f9  58 b8 68 4a 85 ea e1 75          h X hJ   u
00000461    2f ff 55 5b 55 e3 5f 26  74 12 cb 95 fa 82 8e db    / U[U _&t       
00000462    f9 cb 3f b6 cc e6 10 46  17 2b f5 e1 77 a4 8d f0      ?    F +  w   
00000463    e3 c2 8f 96 83 00 d2 0c  03 39 70 20 5f 89 5e 8a             9p _ ^ 
00000464    be af de 52 5f f5 73 62  a9 9a d3 37 0e 86 4c 7b       R_ sb   7  L{
00000465    40 30 0e 21 04 18 0f 21  2c 20 84 31 e5 aa 82 1d    @0 !   !,  1    
00000466    12 55 4f 4b d1 e6 ab 12  26 5b 93 72 00 4b 78 20     UOK    &[ r Kx 
00000467    0f bc ad 5d fa b9 aa 95  28 d0 27 47 11 f4 60 c0       ]    ( 'G  ` 
00000468    30 00 78 fc 20 00 70 fa  0f 95 d2 ea 3f 12 e7 95    0 x   p     ?   
00000469    ff dc f8 1e 92 db 25 63  49 de bf e0 60 19 84 b1          %cI   `   
00000470    26 40 86 24 04 38 ac bb  f6 f2 4c 9a b7 fd 9b 29    &@ $ 8    L    )
00000471    1b f0 30 0b 61 06 04 12  e8 5f 99 6c ec 4f 1d 04      0 a    _ l O  
00000472    0c 00 98 30 01 c0 1a 0c  00 7c 08 25 c3 f5 5a 24       0     | %  Z$
00000473    2b f7 8b e8 96 da a2 f1  fc b8 a8 10 aa de f5 fc    +               
00000474    be e5 6e fc c4 78 30 0c  df d1 21 c9 40 19 07 d4      n  x0   ! @   
00000475    10 60 21 4a 07 a7 ec f6  b6 7b 28 7d 85 c0 c0 39     `!J     {(}   9
00000476    4f 09 02 4d 1f 89 42 59  74 57 47 77 f9 6c dc ba    O  M  BYtWGw l  
00000477    22 c4 3d c7 bf 23 1b 4a  2f 3a 06 00 b0 21 7c 18    " =  # J/:   !| 
00000478    0a 40 60 06 2c 9b a1 01  8b 97 28 fc bd f8 de ed     @` ,     (     
00000479    e7 4e a3 83 00 4a 0c 02  c0 30 03 60 1c 0c 00 e0     N   J   0 `    
00000480    20 00 70 92 01 f4 48 1f  83 00 1e 08 1e f0 97 0b      p   H         
00000481    cb 95 c9 ff cd bf 82 57  f2 f8 7f 75 88 f4 40 60           W   u  @`
00000482    17 d4 ab 12 04 81 f0 95  14 97 d1 27 47 93 bf b1               'G   
00000483    47 fd b8 95 f3 e0 1d 55  42 fb 55 dc f4 55 7a 9c    G      UB U  Uz 
00000484    e3 cc 41 80 65 06 01 a8  21 03 00 3a 01 df 08 60      A e   !  :   `
00000485    1f 55 ea b0 38 10 04 a9  6a 91 f9 7e 31 d1 19 53     U  8   j  ~1  S
00000486    6a eb 13 23 75 d3 20 c0  35 83 00 b4 0c 03 90 20    j  #u   5       
00000487    a9 8a bf 44 82 e5 7b 7e  a2 82 8f fa dd 56 ad b4       D  {~     V  
00000488    4e a0 06 01 7c 18 06 31  24 18 06 30 60 1c 0b c1    N   |  1$  0`   
00000489    04 7f aa 84 af 69 70 06  09 0c 09 42 44 bc 8b 01         ip    BD   
00000490    f2 f1 1c be b3 fe 35 1d  7e ab ca c4 95 6a 0b a5          5 ~    j  
00000491    57 72 fe e3 5e f7 9b b6  b1 3c 64 e8 18 01 10 60    Wr  ^    <d    `
00000492    04 07 c0 80 3e f5 2f 51  42 0d 8a 4b fe b2 65 49        > /QB  K  eI
00000493    bc d6 c6 df 52 24 ab 82  5a bb f6 8b bd aa 34 ad        R$  Z     4 
00000494    2b df 72 a9 51 7a b1 f0  f4 b9 52 b5 79 b0 bd 44    + r Qz    R y  D
00000495    bf cb bb 2a bc de 11 df  2a dd f1 77 a5 56 af ff       *    *  w V  
00000496    c9 e9 db 7f 5a 8e ba 08  43 e2 f0 43 57 2e 8f 15        Z   C  CW.  
00000497    49 55 c5 5f b8 af 6c 8a  a5 be bd f5 dd 48 e9 b2    IU _  l      H  
00000498    e1 f2 af 55 4a e2 a9 e1  f9 76 a8 2f f7 a4 b2 d9       UJ    v /    
00000499    95 a6 e6 20 30 6a 10 41  80 71 f0 40 1f c2 e5 0a        0j A q @    
00000500    77 db 68 29 b5 a4 ab 98  a9 85 db 2d 94 89 f3 e0    w h)       -    
00000501    c0 08 04 20 60 1c 81 80  6e 06 01 c4 10 01 80 14        `   n       
00000502    2f fa 95 42 48 fe 7b ff  b3 c3 f5 62 40 f9 54 bf    /  BH {    b@ T 
00000503    12 ef 8b fb 66 2a f7 f9  4c 93 17 03 00 2e 0c 00        f*  L    .  
00000504    9a a0 40 54 08 02 50 91  04 8f 2a f4 8a bf 20 28      @T  P   *    (
00000505    fd 57 b6 f7 f6 88 a7 de  81 80 23 f8 30 0d 60 c0     W        # 0 ` 
00000506    07 03 00 e0 01 f0 18 01  41 20 b8 18 07 11 24 49            A     $I
00000507    12 a9 78 41 54 24 c1 e0  21 2a 2e fe e7 a8 1f f0      xAT$  !*.     
00000508    28 ff ff e3 64 04 00 82  0c 03 68 20 03 00 22 3e    (   d     h   ">
00000509    12 0b c1 80 72 f8 21 51  26 4c 2e f7 aa b5 73 f9        r !Q&L.   s 
00000510    2a be 7b b8 aa f2 bd 7c  18 06 90 0d a0 1e 10 cb    * {    |        
00000511    bc 3e ba a9 57 87 52 6f  be 3d bd fa bf c2 c8 49     >  W Ro =     I
00000512    30 01 82 52 a8 25 2b 05  0f bd 47 bc 56 a6 4b ef    0  R %+   G V K 
00000513    f9 a5 04 af 36 1f 83 00  48 10 95 8e 84 90 87 55        6   H      U
00000514    aa b0 0e 4f 7a 78 7e a6  ad 6d b3 08 1c 81 80 6a       Ozx~  m     j
00000515    06 00 50 bc 18 01 31 26  aa da 5e 25 5c aa 87 f6      P   1&  ^%\   
00000516    65 ca c2 b9 1a b2 a2 79  6c 06 01 8c 18 01 71 2d    e      yl     q-
00000517    5c 04 20 60 1c 84 a9 f1  e7 f2 17 e9 72 b6 39 07    \  `        r 9 
00000518    be 83 15 50 60 18 01 80  68 04 01 f0 30 0e 4a 84       P`   h   0 J 
00000519    a1 25 58 e9 58 40 2e aa  78 ad 56 7e fe 76 cb 49     %X X@. x V~ v I
00000520    9a 42 02 a0 86 3f 12 d4  55 34 75 44 7b 23 43 2a     B   ?  U4uD{#C*
00000521    00 0d f1 70 20 00 75 a0  c0 08 84 22 e5 20 a1 55       p  u    "   U
00000522    68 f8 ba 62 9f 50 36 9e  71 a2 67 c3 9f 85 3e f2    h  b P6 q g   > 
00000523    08 20 81 ef 7a 04 30 81  f5 22 55 cb 64 bb 92 b3        z 0  "U d   
00000524    ea cf 86 11 a7 d8 af af  ae 3e 8c 03 80 3c b8 03             >   <  
00000525    4b 87 aa b6 65 50 ad 57  f9 f2 ef ac ff 05 1e fd    K   eP W        
00000526    1f 15 1f 0d 1e c9 33 de  e3 de 23 e0 e3 df e3 d9          3   #     
00000527    66 7b a4 7b 8c 7b e4 7b  c4 7b 2c c7 6d 8f 6a 3e    f{ { { { {, m j>
00000528    eb 1e e2 7d 9e 5f c9 d7  cb 1f 98 af 9a af 61 99       } _        a 
00000529    f3 07 e6 8f ce 9f a0 af  60 99 7c 08 30 01 e0 c0            ` | 0   
00000530    2f 83 00 dc 0c 00 8d 80  84 3f 52 a8 14 c5 fd 72    /        ?R    r
00000531    58 30 01 c0 c0 21 84 05  79 e0 60 1c 44 8f 89 15    X0   !  y ` D   
00000532    92 f2 e1 da a1 6d 15 0c  e9 00 25 8f e5 ca 95 b7         m    %     
00000533    23 e6 27 ef 9a 7b c6 40  30 18 06 d0 60 03 81 80    # '  { @0   `   
00000534    72 f6 80 70 41 1f 0f d3  4f c5 28 fc 6c 5c 03 40    r  pA   O ( l\ @
00000535    38 48 04 12 ea 3e 12 7d  be 4d 72 ba 93 c3 f5 65    8H   > } Mr    e
00000536    d6 2b 1f 2b f9 77 e3 5f  55 fb ea 85 cf c0 c0 07     + + w _U       
00000537    03 00 9a 0c 03 40 20 67  8b cb fc 08 23 e8 22 04         @ g    # " 
00000538    30 37 89 8b fd d4 27 da  fe 01 91 b5 57 08 e4 60    07    '     W  `
00000539    43 a2 2c 3e f5 60 60 1b  c1 80 38 9f bf 56 0c 00    C ,> ``   8  V  
00000540    70 07 d9 58 2e 12 4a 3e  e8 e0 60 1c 81 80 1c 06    p  X. J>  `     
00000541    01 b0 18 01 0d a1 0c 03  0b c7 93 80 7c ba 01 f2                |   
00000542    e4 3e dc 21 4d 04 00 60  11 81 80 13 00 fc f0 06     > !M  `        
00000543    c2 f0 60 03 cb d7 56 0a  31 29 11 7a ae c0 27 e7      `   V 1) z  ' 
00000544    13 17 04 31 20 20 97 01  f1 20 21 97 09 25 df 11       1      !  %  
00000545    4b 8b fc 5f ea 82 d1 9b  5c a2 49 73 7d 83 59 3b    K  _    \ Is} Y;
00000546    4b be d1 c7 cf 97 00 68  43 08 57 44 a1 24 7e 3e    K      hC WD $~>
00000547    b0 14 ca fe eb 3f 89 65  e1 00 18 07 21 f0 20 fb         ? e    !   
00000548    b4 7b 15 d6 25 ba 07 7c  98 7b 2b cf 81 04 18 02     {  %  | {+     
00000549    00 60 05 81 80 70 da 3f  08 5e f1 7d e8 21 89 58     `   p ? ^ } ! X
00000550    3e 2f 2c e5 47 1e e0 0c  03 80 30 07 62 5e 7e 83    >/, G     0 b^~ 
00000551    00 20 01 ea 84 b9 97 04  80 83 14 2b 4d 21 7c d5               +M!| 
00000552    56 ac a6 06 4b 60 c0 07  03 00 e3 02 10 97 3f f9    V   K`        ? 
00000553    9e a0 70 be 37 2a aa da  03 73 63 f1 f2 9f 56 9e      p 7*   sc   V 
00000554    f8 20 60 1a c2 08 07 03  00 d6 0c 00 a4 54 3e cf      `          T> 
00000555    97 97 97 8f 81 41 28 1b  54 3f 5b 18 b9 2e ea c7         A( T?[  .  
00000556    f2 3f 29 77 ea a5 73 3e  aa df 7e 2d 5d 3e 0c 00     ?)w  s>  ~-]>  
00000557    88 96 5f e1 25 50 30 01  e0 84 5c 10 80 3c bc 7f      _ %P0   \  <  
00000558    47 d4 be 4b 47 e2 56 6f  bf ea 0a 1b ff 7b ff be    G  KG Vo     {  
00000559    b6 a8 8d 63 a3 c1 80 65  06 00 a0 4b 1f 7c b8 10       c   e   K |  
00000560    61 7c 08 7e 08 63 f2 e1  28 bf f5 55 95 50 fc be    a| ~ c  (  U P  
00000561    e2 aa 5e a8 ba 5d f7 ef  07 b2 71 98 64 d5 50 96      ^  ]    q d P 
00000562    25 02 02 b8 aa d9 e2 e1  27 55 97 56 e5 5a a9 63    %       'U V Z c
00000563    4c 59 97 50 50 5f fa 81  a9 2d 1b eb de e2 0c 01    LY PP_   -      
00000564    e0 20 82 00 41 12 15 82  00 20 aa 55 02 18 07 7e        A      U   ~
00000565    7c 03 82 1d e0 1f 12 8b  ef 44 a1 eb 57 e3 ef ee    |        D  W   
00000566    46 4b bf 21 81 00 0e 55  e1 e5 b3 ae 8a 06 00 44    FK !   U       D
00000567    18 01 a0 60 1b 41 00 4b  8a e0 07 97 04 09 2b 65       ` A K      +e
00000568    d7 e5 f5 5a 61 e4 51 30  0a d8 44 9b 00 34 03 55       Za Q0  D  4 U
00000569    2a 1e 02 84 0f 73 b6 73  f5 14 34 b0 0c 03 58 94    *    s s  4   X 
00000570    10 02 18 93 f1 f8 90 3f  03 79 55 c9 eb 8a 59 b3           ? yU   Y 
00000571    57 33 69 01 80 61 2e 08  57 3d e8 24 ad 26 aa c4    W3i  a. W= $ &  
00000572    bf a5 0e 78 88 41 08 20  1c 10 bd 41 0c 7e a9 52       x A     A ~ R
00000573    6d 1d a0 36 be a9 50 94  3e b4 4a 92 0f d5 76 2a    m  6  P > J   v*
00000574    db 7d fd bf b6 58 65 55  56 17 97 81 dd 57 6b 3f     }   XeUV    Wk?
00000575    7a fd 1f fc 18 00 e0 60  1c 20 fc 0f 0f 47 9f 8a    z      `     G  
00000576    e7 98 b1 ec 62 50 fa 09  5b e9 6d bf 6f 6f 2d 86        bP  [ m oo- 
00000577    b3 01 80 71 06 01 b4 4b  06 01 bc 0f 51 2e 2a 54       q   K    Q.*T
00000578    24 17 0f c0 d1 74 ef a5  6e df de db 89 1c f6 00    $    t  n       
00000579    87 80 a0 b2 79 5f ac fd  9d bb 2e 2d c7 47 83 00        y_    .- G  
00000580    d6 5c ad 5c 9e cd 55 8d  92 a8 80 78 30 0d 0a 81     \ \  U    x0   
00000581    80 6c f5 08 5f 06 00 58  7e 0c 00 70 96 5c af d4     l  _  X~  p \  
00000582    10 f4 b9 58 42 57 3f cc  2e 2f 55 3c a7 5b 83 a6       XBW? ./U< [  
00000583    a1 d6 b1 2c 10 01 0a d9  7d be 7a a8 40 12 fe 10       ,    } z @   
00000584    15 cf fe 2a 52 b7 95 c5  7e b2 ac fd 31 2c 10 80       *R   ~   1,  
00000585    30 03 ef c1 03 f4 49 12  a7 bd ff 5a dc 9f 9d f5    0     I    Z    
00000586    22 7d 61 70 20 0f 21 7d  93 2f bd 27 80 a9 7f d4    "}ap  !} / '    
00000587    0b 67 c1 80 6b 08 5f 08  00 84 01 cd 82 11 72 a1     g  k _       r 
00000588    ed 03 23 fc 94 be 81 1b  91 34 73 60 20 03 00 22      #      4s`   "
00000589    0c 00 70 07 fe 97 17 ab  82 55 bf 55 9a d7 c7 9f      p      U U    
00000590    9f cd 2c c7 ad 03 00 d0  0c 01 39 78 41 1f d1 24      ,       9xA  $
00000591    48 cf cf dd c6 5c 9b 41  04 7c 0c 07 81 70 92 3f    H    \ A |   p ?
00000592    d9 e9 ec 96 ce 4b 1d 64  01 c2 4c 00 f0 0e f8 28         K d  L    (
00000593    2c aa fd 40 e9 76 c1 ec  44 cc 4e e7 bb 03 00 7a    ,  @ v  D N    z
00000594    0c 00 e0 90 08 22 58 43  06 01 bc 10 44 88 24 97         "XC    D $ 
00000595    00 72 b5 42 40 42 2e 57  27 d5 02 85 5c 52 ac 7f     r B@B.W'   \R  
00000596    ff 5e 50 42 f2 89 ed d9  aa f4 c1 d8 30 0d c0 c0     ^PB        0   
00000597    19 03 00 20 0c 03 88 93  42 18 95 41 80 90 12 fe            B  A    
00000598    5c 08 5f f8 20 17 0f 95  78 bf db a0 a3 2e 50 a3    \ _     x    .P 
00000599    f4 0b cc b7 9b 6e 38 e0  18 02 f0 60 04 47 e0 c0         n8    ` G  
00000600    09 03 00 1c 24 89 03 f9  9f f8 06 ab 12 d1 55 45        $         UE
00000601    de 83 00 be f8 c3 40 30  0e 00 82 ad 58 30 0e 03          @0    X0  
00000602    f1 2c 7c 3f b3 33 c3 ef  fa e6 2a 92 1a 92 00 e1     ,|? 3    *     
00000603    2b d4 03 95 42 f2 e6 f6  fe 58 c9 f7 aa 84 20 60    +   B    X     `
00000604    0f 41 80 11 08 41 08 bc  b8 18 0f 50 82 10 c2 0d     A   A     P    
00000605    fc d1 26 45 5e 55 e0 42  b6 fb c0 85 27 f8 c2 b1      &E^U B    '   
00000606    d8 d1 72 03 00 42 07 87  ea ad 96 aa 8a e5 b3 1b      r  B          
00000607    9e bb 84 23 df fd f5 a5  fe f6 a9 bf 9f ee 28 96       #          ( 
00000608    5e e1 92 f0 0c 1f 97 89  73 ea fe 10 04 91 23 2d    ^       s     #-
00000609    f2 a0 43 8a 00 fe 5c d5  52 f5 45 5e c9 16 3e dd      C   \ R E^  > 
00000610    02 18 fd 5d fa b1 f8 97  ff db 9b 55 ff 6b e4 8b       ]       U k  
00000611    c7 d2 28 f1 71 72 07 3d  14 18 02 f2 e2 e5 42 48      ( qr =      BH
00000612    30 0c e5 c5 f9 00 36 00  79 75 57 d9 b4 b9 5a ab    0     6 yuW   Z 
00000613    55 62 35 36 90 32 83 00  ad f5 63 f0 60 1a 04 9f    Ub56 2    c `   
00000614    51 ee 09 4a 87 c3 e2 e5  a4 1e 45 77 13 5f fe bc    Q  J      Ew _  
00000615    98 03 81 80 1c 00 e5 40  83 41 80 6c 82 50 1e 00           @ A l P  
00000616    d5 73 3e ac 7a ae 2a fa  7f 8f b7 3f f8 c7 f1 52     s> z *    ?   R
00000617    47 97 03 00 d6 0c 00 d8  42 06 01 bc 20 c1 ff 80    G       B       
00000618    3e 97 04 3b ef 2a b2 17  42 f5 5f 1e 35 ff f6 59    >  ; *  B _ 5  Y
00000619    ce 1b 72 82 5f a0 97 55  f6 cb ec 7c 70 21 7e 8f      r _  U   |p!~ 
00000620    bd ee 4b bc 73 e3 a1 70  1d 56 ac 7a a4 7b 50 49      K s  p V z {PI
00000621    24 9e 84 51 be 85 fb f1  ec 96 5b 64 9c ff a1 73    $  Q      [d   s
00000622    13 2a 6e d2 56 20 0d 06  00 80 bc 18 01 0b ff 2b     *n V          +
00000623    12 c0 38 be 4b 6c b3 f5  55 cf ed 9b ef c6 d3 4f      8 Kl  U      O
00000624    be 2c 14 20 18 aa 01 ef  84 11 2b c2 5c 96 28 a3     ,        + \ ( 
00000625    ff 6a 98 8d 97 b1 2a d2  f8 a7 d2 d8 45 33 f0 39     j    *     E3 9
00000626    72 d9 1e f8 b1 28 10 4b  c7 c3 ef ce 17 0f ae c0    r    ( K        
00000627    50 5f 49 ef 78 a2 d3 6e  23 e0 60 09 81 80 69 08    P_I x  n# `   i 
00000628    60 1c 3e 00 d0 84 10 bd  ff 09 7e 1e f8 7c 0a 19    ` >       ~  |  
00000629    07 f1 55 6a d8 5e a1 b9  d9 d7 c5 8f fd 41 40 3d      Uj ^       A@=
00000630    03 70 75 7e d5 df 6c 3e  30 08 01 0c 4a 12 c4 80     pu~  l>0   J   
00000631    85 44 aa 3e cf 89 03 ef  c2 e8 5f e5 3e f5 f4 c9     D >      _ >   
00000632    9a c1 a6 81 26 97 fa 41  e4 67 bc 3f 2a 25 78 7d        &  A g ?*%x}
00000633    6f d4 e4 41 0f be 08 49  06 00 48 18 06 e0 0e 06    o  A   I  H     
00000634    07 e0 18 00 e0 0f 1f 63  34 4a fd f0 11 f4 be 19           c4J      
00000635    c7 80 48 52 9c 6e 57 0a  89 20 c0 08 03 00 22 01      HR nW       " 
00000636    bc 08 60 80 01 c3 f4 ea  c7 ca 94 81 1c 9f 19 b9      `             
00000637    40 0d 57 1b f4 a3 28 da  10 3d 5a f8 bd f1 70 33    @ W   (  =Z   p3
00000638    87 81 99 80 92 0c 01 88  30 03 62 5f 4b 87 c3 f0            0 b_K   
00000639    81 57 12 07 f7 00 81 78  f5 cc 80 c0 32 03 00 3a     W     x    2  :
00000640    10 01 80 18 f8 95 3c 01  a5 e2 5d 1f ed 03 df 55          <   ]    U
00000641    25 9e 8c d5 7e a2 e6 b5  42 4f 8b ea 89 2c 7c 9a    %   ~   BO   ,| 
00000642    b1 2a aa 9b b2 bb c2 47  c0 c7 c1 d8 5d b3 d2 dd     *     G    ]   
00000643    6d 7c 74 10 f3 2c cf cb  a4 0c 93 1d f6 3d e6 3f    m|t  ,       = ?
00000644    fd 42 b8 63 fc fb 2c c4  e7 67 e7 e8 98 66 67 43     B c  ,  g   fgC
00000645    49 49 4e c1 33 3a 7a 9a  aa c5 f9 ac d5 d7 57 d7    IIN 3:z       W 
00000646    d9 57 af 4d 42 b0 ad 12  02 18 96 01 ca c0 f8 40     W MB          @
00000647    12 c7 e1 08 bc 0c 8f 95  d2 ea 8f 1b 49 4f da 57                IO W
00000648    db af 4d a8 81 06 00 38  18 04 90 60 1b 82 16 78      M    8   `   x
00000649    03 4b 84 a0 60 1c 95 46  fe 0c 0c 38 43 02 2a a5     K  `  F   8C * 
00000650    2e fd 02 a3 e2 15 00 40  06 01 44 18 01 90 41 cf    .      @  D   A 
00000651    7c 49 a0 80 3f a0 a4 04  18 3c f6 a6 12 66 89 48    |I  ?    <   f H
00000652    7e a8 8a e6 f2 bd 82 aa  55 4c 4d bd ec 18 05 60    ~       ULM    `
00000653    60 06 c2 0d 06 01 bc 48  06 01 a0 18 01 6f 02 1c    `      H     o  
00000654    12 6a aa 3f a2 55 9f 83  e0 83 eb 2f 84 9d 97 01     j ? U     /    
00000655    08 4b 80 6f f1 a5 72 e8  34 3f aa 05 c5 e2 49 78     K o  r 4?    Ix
00000656    94 10 07 c1 06 41 29 5a  ba aa 09 6a 2e fa 5b cb         A)Z   j. [ 
00000657    65 e6 c8 fa a0 60 04 c1  80 61 06 01 c8 18 01 95    e    `   a      
00000658    76 00 67 fd f2 e5 76 52  f0 84 3f f0 f4 4a 56 89    v g   vR  ?  JV 
00000659    b5 5f 4e aa c4 e7 dc 04  a0 60 12 c1 80 14 04 11     _N      `      
00000660    f0 21 03 00 e2 01 df 06  00 38 be e6 09 23 e5 50     !       8   # P
00000661    4b 1f 4a b8 1c 90 be 45  f5 55 23 58 56 25 8f e8    K J    E U#XV%  
00000662    94 3e 12 ae 7d 50 fb 47  96 f0 be 4a c6 ad 0c 55     >  }P G   J   U
00000663    82 08 40 05 0a a2 f9 04  7b eb 8b 5a de 56 73 f9      @     {  Z Vs 
00000664    8f 79 b8 06 83 00 2e a8  7e 01 c3 e5 41 00 78 07     y    . ~   A x 
00000665    fd 3a af fc a2 5e b3 6c  e4 ba 69 53 e0 1c 5e 01     :   ^ l  iS  ^ 
00000666    c5 e2 40 30 01 c1 0e fa  84 3f f6 ab f7 f7 3d f0      @0     ?    = 
00000667    35 2b 1a 65 73 c0 c0 33  89 00 82 24 17 04 21 20    5+ es  3   $  ! 
00000668    18 07 10 0e 2f 85 c2 30  1e 12 61 71 7f e3 13 ea        /  0  aq    
00000669    95 d5 4a f8 44 be 10 c1  0e d1 20 bf 15 cf 97 2a      J D          *
00000670    9e aa ee ff 6f ae fb f1  ec 65 ca be a9 b5 7b df        o    e    { 
00000671    f1 2b 3a 6f 28 18 06 ef  d1 fd 9f 06 02 4a 84 25     +:o(        J %
00000672    71 50 fb d5 5c 1e e5 1e  4c be 9e 9a 5c a1 89 5b    qP  \   L   \  [
00000673    3c f5 10 0d 00 c1 2c 10  24 08 6a bf 4b d5 7a c5    <     , $ j K z 
00000674    32 98 52 57 f2 e0 0f 9e  80 85 ff 8f f3 47 57 2b    2 RW         GW+
00000675    92 04 b0 60 1b d5 09 20  c0 07 fd 57 e6 0f ff 27       `       W   '
00000676    aa cd c3 ab 41 08 4b 81  00 18 07 0b 3f 75 55 aa        A K     ?uU 
00000677    2f e7 66 59 31 ec 32 7f  3a dc 35 b4 3f f8 94 07    / fY1 2 : 5 ?   
00000678    fd 44 a0 38 5c 5f ef 41  2f cb 5d ef 1e f6 1f fe     D 8\_ A/ ]     
00000679    aa 57 0b ac 56 5d 67 7d  3e 2e 52 54 3f f1 7f 87     W  V]g}>.RT?   
00000680    82 5e 25 7a 40 30 0d 42  51 72 b8 10 bf 3b 61 78     ^%z@0 BQr   ;ax
00000681    ea c6 ef b3 5e a4 0c 00  9f 8b 80 3c 10 e1 7c 96        ^      <  | 
00000682    cc c8 48 c1 bb e3 fb 43  c8 a8 20 c1 f1 72 b8 a3      H    C     r  
00000683    f9 8a ef 60 ee 19 7a f8  43 04 0a a8 0e 97 77 cb       `  z C     w 
00000684    b9 bc 7e 10 41 80 69 12  a0 f4 18 00 e0 60 1c 84      ~ A i      `  
00000685    b9 63 61 08 be c1 f9 75  02 90 be 55 15 31 a4 e0     ca    u   U 1  
00000686    42 f8 07 02 0d 12 bd aa  c2 11 79 75 57 e2 e6 ec    B         yuW   
00000687    b0 99 5b e0 c0 07 89 65  e3 e1 2c 03 c7 83 cf 17      [    e  ,     
00000688    5e cb 32 a2 5c f2 e2 a1  24 10 ac fa 98 c6 6e 5d    ^ 2 \   $     n]
00000689    d6 2c 33 ae a8 48 00 f8  3f fd 08 5e 56 08 62 52     ,3  H  ?  ^V bR
00000690    91 ed 2e 94 79 9e 8a 6d  49 86 1e ea 10 01 80 42      . y  mI      B
00000691    06 01 ac 10 3f 80 1e 25  04 01 27 39 ff 7d 57 c4        ?  %  '9 }W 
00000692    a2 f4 3e b8 0d 28 1a 18  30 03 60 c0 21 83 00 22      >  (  0 ` !  "
00000693    0c 03 30 fc 18 08 d1 f8  fb e0 c0 07 00 6a b9 41      0          j A
00000694    41 15 fe 7c 7c 25 dc 03  4a 8b f9 8a d6 f7 e4 d3    A  ||%  J       
00000695    11 40 c0 32 83 00 1c a8  03 c4 82 f1 fa af 60 41     @ 2          `A
00000696    12 a5 ba d0 f7 de 6f d5  7e 77 cf c7 12 8b d5 97          o ~w      
00000697    09 63 e2 f2 eb 61 79 71  7a a5 6a ac 6e 7f df 96     c   ayqz j n   
00000698    0c d0 55 00 70 30 01 e0  c0 37 89 0a ea b0 84 5c      U p0   7     \
00000699    3e 12 e7 2a b5 51 45 f5  cd 9f 96 37 13 3e c4 18    >  * QE    7 >  
00000700    01 40 60 1c 47 e2 59 72  91 ef e2 b2 eb 8a cb b7     @` G Yr        
00000701    7c 9a c9 58 a8 5c f2 18  ad 52 d5 e3 20 c0 7c 0f    |  X \   R    | 
00000702    bd 73 c0 74 bd 37 68 a9  14 10 41 80 65 12 01 80     s t 7h   A e   
00000703    0e 00 ea 25 8f fa ac 79  3f 0b a5 1d fd 5d f6 a8       %   y?    ]  
00000704    91 2b 2d 3a 5c 18 07 00  60 16 41 80 6b 06 01 c1     +-:\   ` A k   
00000705    4a aa 3f 80 18 10 be d8  42 00 ed 2e d8 b0 fc 7f    J ?     B  .    
00000706    a0 86 5e 9e 6c 54 0d 18  1b 7c 08 0a 6d d3 12 14      ^ lT   |  m   
00000707    49 c6 bb 5e f1 41 ea 98  cf f5 80 52 44 1c 78 af    I  ^ A     RD x 
00000708    e8 92 3e 12 95 d0 80 5c  3e f8 93 83 a8 25 ff e5      >    \>    %  
00000709    ca ad 02 f4 b9 57 ff 91  68 ea c1 20 18 07 10 41         W  h      A
00000710    08 16 50 84 01 a0 1e 24  63 40 84 10 c7 ea a2 09      P    $c@      
00000711    fc 19 45 09 00 c0 24 83  00 bc 24 64 a0 83 e0 87      E   $   $d    
00000712    e8 c0 41 12 07 ca f5 2f  fd 15 2b 46 a4 fb 6d 00      A    /  +F  m 
00000713    c0 31 f4 2e 92 81 0e b7  f9 a9 29 e7 b6 83 00 c4     1 .      )     
00000714    0c 00 c8 30 0a 40 83 f2  ff ab f0 94 3f 2e 2f b7       0 @      ?./ 
00000715    f7 c0 c0 08 0f f3 53 7c  10 ef 84 95 71 32 9b ec          S|    q2  
00000716    6b ce b7 08 20 c0 07 89  60 80 5f 4b 82 08 42 2f    k       ` _K  B/
00000717    08 45 f2 f7 c3 f5 57 c5  66 db c4 a0 70 1e ec 49     E    W f   p  I
00000718    e4 08 20 c0 09 0f 95 af  44 81 27 e5 d5 17 8b fc            D '     
00000719    ae 0c de 81 04 18 05 40  60 04 81 80 72 12 ed 12           @`   r   
00000720    c1 80 11 08 7f 1f 2b cd  f1 7c 08 45 f2 73 b9 2f          +  | E s /
00000721    fa 32 62 12 00 fa ae 2b  c5 18 62 64 7e a2 f7 d2     2b    +  bd~   
00000722    ee 99 7d 98 95 6c 6a d8  91 f4 d0 bd 5e ca 3d 57      }  lj     ^ =W
00000723    be 99 ea a6 cb e8 b4 b2  ec 73 88 42 06 02 ec 10             s B    
00000724    7d 21 7a 90 40 2f 5e c1  24 20 75 2f 80 f8 34 00    }!z @/^ $ u/  4 
00000725    cc a0 c0 58 83 00 2c 08  3d 06 00 50 18 06 4f 7d       X  , =  P  O}
00000726    6f 7e 89 01 0e 21 2e fa  a5 63 26 2f 8f f2 f5 52    o~   !.  c&/   R
00000727    b4 b2 1f 9b 54 a9 3b 9f  1a 10 01 80 60 06 01 9c        T ;     `   
00000728    03 b0 14 00 c0 37 00 7c  90 44 56 5e 0a 1f 20 1f         7 | DV^    
00000729    80 4b 58 42 06 01 10 18  06 a5 5c 08 63 e0 0c 00     KXB      \ c   
00000730    f8 bd 55 47 ca d0 aa b8  32 74 12 41 80 39 06 01      UG    2t A 9  
00000731    98 4a e8 95 40 38 03 e0  16 2f a3 ef a0 1f c9 06     J  @8   /      
00000732    70 c0 c0 53 03 00 c3 34  18 11 31 2f 8b 7c 7d 50    p  S   4  1/ |}P
00000733    62 b3 ac d5 58 95 f6 ef  80 9c 16 cb 4f 17 c6 a5    b   X       O   
00000734    20 7a 60 40 06 01 08 10  3f c0 82 08 01 0c 48 4e     z`@    ?     HN
00000735    25 2b f0 ff c8 2a 87 b2  03 00 ca 0c 03 80 42 06    %+   *        B 
00000736    01 a4 03 fc 24 80 61 7d  2f 1f 78 7d ef 62 bf 5f        $ a}/ x} b _
00000737    7d 57 fb b2 c7 ca 89 60  c0 07 03 00 e2 10 38 24    }W     `      8$
00000738    82 00 07 8f b1 ba ac 48  2f 91 0e 1e 9c f0 ff fe           H/       
00000739    11 bf 7d 53 3d 9f ca 84  ad 52 ab 35 33 e5 6f cb      }S=    R 53 o 
00000740    f3 3f a4 3d fc 78 3b a3  ae ce 57 c0 c5 51 4d ff     ? = x;   W  QM 
00000741    77 90 69 22 a8 18 07 21  25 53 33 fa bb a1 cb 81    w i"   !%S3     
00000742    80 70 9d 1e 8f b7 6e f1  ec 73 0f 7f 9e fc fd fe     p    n  s      
00000743    ec 60 91 40 18 01 d0 60  07 8b 81 80 6e 04 05 61     ` @   `    n  a
00000744    0c 20 fd 5f 87 bf aa 84  ac 62 7b 97 e3 57 ff 00       _     b{  W  
00000745    78 07 7e 5d b7 d6 eb 6e  4c 06 00 70 18 05 c0 60    x ~]   nL  p   `
00000746    04 01 80 0e 06 00 40 b8  21 04 1f fe 7c b8 bf fe          @ !   |   
00000747    83 c9 bc b7 67 c8 9a 47  f1 5d 54 07 1b 3f 26 5d        g  G ]T  ?&]
00000748    7d 28 ec d4 e8 a9 69 aa  18 26 67 4f 53 57 5a c1    }(    i  &gOSWZ 
00000749    33 3a ca fb 0b 26 09 99  d8 da 5a 5b 30 4d 4e da    3:   &    Z[0MN 
00000750    e2 de e5 82 69 dd d6 07  df 7d 5f 8a db d7 5d 38        i    }_   ]8
00000751    f8 78 b9 d3 f8 18 98 f0  33 5b 9e 2d 81 5d 74 3e     x      3[ - ]t>
00000752    00 d0 0d 08 5d 08 41 04  21 89 49 8b 84 b5 7e 88        ] A ! I   ~ 
00000753    37 06 58 27 f0 97 a8 6b  8a 52 8f 81 80 5c 06 01    7 X'   k R   \  
00000754    a8 7c d8 96 5c 10 e2 c1  0c 48 55 fa 04 55 42 e5     |  \    HU  UB 
00000755    63 39 45 40 c0 2a ee 50  86 0c 00 87 80 a8 42 12    c9E@ * P      B 
00000756    ef a8 10 1f c5 70 1a 57  dc 3c 45 ea 82 f0 80 24         p W <E    $
00000757    17 f1 58 f8 bb e9 7c e2  8f a2 50 30 0a e0 c0 09      X   |   P0    
00000758    2b aa 94 97 84 30 85 ff  34 0a 11 2a ff d3 78 d9    +    0  4  *  x 
00000759    75 56 30 6c 04 00 60 1c  00 3a 09 70 7e 01 c0 82    uV0l  `  : p~   
00000760    24 17 dd b7 20 fe 4b 95  8a ab f9 4f 62 63 2f 5a    $     K    Obc/Z
00000761    97 83 00 2e 08 1f 6b df  2e a0 a8 fd 85 94 fb d4       .  k .       
00000762    be 08 2a fe 10 e0 43 cc  be 1f 58 5d 67 60 04 9f      *   C   X]g`  
00000763    00 60 30 0d 3a 25 09 74  03 f3 ff 55 8a 15 02 9e     `0 :% t   U    
00000764    54 a7 2f 87 c0 1e 01 e1  0b a2 50 41 12 47 c0 50    T /       PA G P
00000765    7f fc 41 b5 d5 9e 06 01  84 18 06 b9 32 84 38 5e      A         2 8^
00000766    9d 59 77 b5 0c 3c c1 7c  ab f3 2b ae bc 0c 00 98     Yw  < |  +     
00000767    92 24 17 82 18 07 02 0f  fc af ba 3d 9b aa 66 5d     $         =  f]
00000768    cb d4 da e7 c1 83 00 c6  0c 03 78 94 24 51 27 32              x $Q'2
00000769    0f 67 f1 54 02 e6 31 c4  a0 82 a8 48 2f f1 7e 4b     g T  1    H/ ~K
00000770    ff 5d f4 cd ec 6a e0 b7  1c 21 ab 12 c4 9b 55 97     ]   j   !    U 
00000771    ab b4 ce 8f bd 25 72 b7  84 bf c2 ef 97 78 75 ea         %r      xu 
00000772    de 19 ab 04 00 40 54 10  3d 9d fa bb 75 5f d7 7d         @T =   u_ }
00000773    2f 07 e0 c0 37 80 78 07  fe fe fe f5 bf ab aa 7e    /   7 x        ~
00000774    38 75 f1 fc 4c 45 f9 b5  df 00 4d e1 fc 40 ca fc    8u  LE    M  @  
00000775    fb 03 e6 e4 bb ba be 3f  78 c1 35 8b 67 59 cb 97           ?x 5 gY  
00000776    67 2a f3 8f b9 b7 3e c1  34 f9 d0 80 0c 03 28 30    g*    > 4     (0
00000777    0d 40 19 90 21 03 00 dc  01 aa a4 6b c0 a1 fa b5     @  !      k    
00000778    40 40 bd 4b 9b 81 80 10  07 01 f0 01 80 72 12 a0    @@ K         r  
00000779    f0 18 07 00 84 a8 10 58  04 32 eb 04 81 fa 39 94           X 2    9 
00000780    ba 8c ad 4f d9 9f 62 be  9f 6e 47 cc cb 61 a7 a1       O  b  nG  a  
00000781    84 00 60 11 41 80 1c 12  66 09 34 21 80 75 67 c3      ` A   f 4! ug 
00000782    f1 f0 f7 c0 44 7c 3d 71  a7 be 0c 01 58 30 0d 52        D|=q    X0 R
00000783    d8 0c 04 a0 93 ad f0 7c  5c 3f 2d 1e 17 ba c0 fd           |\?-     
00000784    79 f6 8a 10 07 cc 6c 48  f9 48 25 ab 5e 54 ae 79    y     lH H% ^T y
00000785    f8 96 0c 01 b0 30 03 a2  4c b4 4a a1 0c 21 6b 54         0  L J  !kT
00000786    4a 12 47 be 88 95 a9 72  e8 41 06 00 f4 21 fa 5a    J G    r A   ! Z
00000787    08 00 c0 38 2a 12 d6 2e  12 4b 80 f5 a0 4f ca 60       8*  . K   O `
00000788    ca 80 21 03 00 dc 0c 03  88 07 70 21 02 08 41 12      !       p!  A 
00000789    a2 ca 84 b5 73 c8 34 f4  28 f8 18 05 10 60 18 4b        s 4 (    ` K
00000790    bc a0 18 0f 31 ea 72 f1  f1 77 fe 84 f3 50 28 0b        1 r  w   P( 
00000791    fc 23 09 75 19 d9 30 3f  ea d9 74 44 71 f1 01 0c     # u  0?  tDq   
00000792    18 01 80 60 1a c1 02 d8  24 83 00 1c 10 e6 08 b0       `    $       
00000793    ba ab 54 8b ea b5 d2 cd  5e cc ab 59 8f 73 08 00      T     ^  Y s  
00000794    c0 23 03 00 d0 3e e0 97  02 10 41 80 5d 50 f2 a1     #   >    A ]P  
00000795    55 45 cb 63 e0 60 13 c1  80 14 04 1b 3c 0c 03 78    UE c `      <  x
00000796    30 03 0a bb 9a 5f 20 f8  7f 22 7e cf 00 5b 46 89    0    _   "~  [F 
00000797    71 85 4a c6 52 a0 a2 02  f0 66 d0 f8 1c 08 45 06    q J R    f    E 
00000798    00 6c 18 2f 60 60 1b 01  84 fa 00 c0 69 a3 50 01     l /``      i P 
00000799    c0 84 70 60 06 c1 82 f6  06 01 ac 18 4f 90 0c 06      p`        O   
00000800    9a 38 04 05 f0 10 c7 e0  a7 56 04 5f 06 07 8b 81     8       V _    
00000801    4c a8 08 39 a2 08 38 10  90 0c 00 e0 30 5e c0 c0    L  9  8     0^  
00000802    35 83 09 f2 01 80 d3 47  21 03 81 09 20 c0 0e 03    5      G!       
00000803    05 ec 0c 03 50 30 9f 20  18 0d 34 70 08 0b e0 21        P0    4p   !
00000804    8f c1 4e 5e 04 5f 06 07  8b 81 4c a8 08 39 a2 10      N^ _    L  9  
00000805    38 10 94 0c 00 e0 30 5e  c0 c0 35 03 09 f2 08 00    8     0^  5     
00000806    d3 47 21 83 81 09 60 c0  0e 83 05 ea 0c 03 50 30     G!   `       P0
00000807    9f 20 80 0d 34 70 08 0b  e0 21 8f c1 4e ac 08 be        4p   !  N   
00000808    0c 0f 17 02 99 50 10 73  44 40 70 21 30 18 01 d0         P sD@p!0   
00000809    60 bd 41 80 6a 06 13 e4  10 01 a6 8f 45 07 02 13    ` A j       E   
00000810    01 80 1d 06 0b d4 18 06  90 61 3e 41 00 1a 68 f0             a>A  h 
00000811    10 17 c0 43 1f 82 9d 58  11 7c 18 1e 2e 05 32 a0       C   X |  . 2 
00000812    20 e6 88 c0 e0 42 68 30  03 c0 c1 7a 83 00 d2 0c         Bh0   z    
00000813    27 c8 20 03 4d 1e 8e 0e  04 27 03 00 3c 0c 17 a8    '   M    '  <   
00000814    30 0d 20 c2 7c 82 00 34  d1 e0 20 2f 80 86 3f 05    0   |  4   /  ? 
00000815    39 78 11 7c 18 1e 2e 05  32 a0 20 e6 89 20 e0 42    9x |  . 2      B
00000816    88 30 03 f5 b0 41 06 01  a0 18 4f 90 40 06 9a 41     0   A    O @  A
00000817    28 1c 08 52 06 00 80 18  2f 40 60 1a 01 84 f8 04    (  R    /@`     
00000818    00 69 a4 00 40 5f 01 42  3f 60 7e 5e 04 7e 2d 84     i  @_ B?`~^ ~- 
00000819    03 c5 c0 a6 54 04 1c d1  2c 1c 08 53 06 00 80 18        T   ,  S    
00000820    2f 40 60 19 c1 84 f8 04  00 69 a4 13 01 c0 85 40    /@`      i     @
00000821    60 08 41 82 f4 06 01 9c  18 4f 80 40 06 9a 40 04    ` A      O @  @ 
00000822    05 ec 14 23 f0 60 be cb  c0 8f c5 b0 80 84 5c 0a       # `        \ 
00000823    65 40 41 cd 12 e0 38 00  a6 0c 01 04 68 10 41 80    e@A   8     h A 
00000824    67 06 13 e0 10 01 a6 90  4a 07 02 14 81 80 1f 06    g       J       
00000825    0b d0 18 06 80 61 3e 01  00 1a 69 00 10 17 c0 43         a>   i    C
00000826    1f b2 3f 2f 02 3f 16 c2  02 10 f8 14 ca 80 83 9a      ?/ ?          
00000827    25 03 81 0a 40 c0 0f 83  05 e8 0c 03 40 30 9f 00    %   @       @0  
00000828    80 0d 34 82 50 38 10 a4  0c 00 f8 30 5e 80 c0 34      4 P8     0^  4
00000829    03 09 f0 08 00 d3 48 00  80 be 02 18 fc 14 e5 e0          H         
00000830    45 f0 80 84 3e 05 32 a0  20 e6 89 40 e0 42 90 30    E   > 2    @ B 0
00000831    04 00 c1 7a 03 00 d0 0c  27 c0 20 03 4d 20 94 0e       z    '   M   
00000832    04 29 03 00 40 0c 17 a0  30 0d 00 c2 7c 02 00 34     )  @   0   |  4
00000833    d2 00 20 2f 80 86 3f 05  39 78 11 7c 20 21 0f 81       /  ? 9x | !  
00000834    4c a8 08 39 a2 50 38 10  a4 0c 01 00 30 5e 80 c0    L  9 P8     0^  
00000835    34 03 09 f0 08 00 d3 48  25 03 81 0a 40 c0 10 03    4      H%   @   
00000836    05 e8 0c 03 40 30 9f 00  80 0d 34 80 08 0b e0 21        @0    4    !
00000837    8f c1 4e 5e 04 5f 08 08  45 c0 a6 54 04 1c d1 2c      N^ _  E  T   ,
00000838    1c 08 53 06 00 80 18 2f  40 60 19 c1 84 f8 04 00      S    /@`      
00000839    69 a4 13 41 c0 85 60 60  08 41 82 f4 06 01 9c 18    i  A  `` A      
00000840    4f 80 40 06 9a 44 04 05  f0 14 23 f6 07 e5 e0 47    O @  D    #    G
00000841    e2 d8 40 42 1f 02 99 50  10 73 44 90 70 21 44 18      @B   P sD p!D 
00000842    01 f0 60 bd 01 80 68 06  13 e0 10 01 a6 90 4a 07      `   h       J 
00000843    02 14 81 80 20 06 0b d0  18 06 80 61 3e 01 00 1a               a>   
00000844    69 00 10 17 c0 43 1f 82  9d 58 11 7c 18 1e 2e 05    i    C   X |  . 
00000845    32 a0 20 e6 89 40 e0 42  90 30 04 00 c1 7a 03 00    2    @ B 0   z  
00000846    d0 0c 27 c0 20 03 4d 20  92 0e 04 28 83 00 3e 0c      '   M    (  > 
00000847    17 a0 30 0d 00 c2 7c 02  00 34 d2 00 20 2f 80 86      0   |  4   /  
00000848    3f 64 bd 58 11 7c 18 1e  2e 05 32 a0 20 e6 89 20    ?d X |  . 2     
00000849    e0 42 88 30 03 e0 c1 7a  03 00 d0 0c 27 c0 20 03     B 0   z    '   
00000850    4d 20 90 0e 04 28 03 00  3e 0c 17 a0 30 0d 00 c2    M    (  >   0   
00000851    7c 82 00 34 d2 00 20 2f  80 86 3f 05 3a b0 22 f8    |  4   /  ? : " 
00000852    30 3c 5c 0a 65 40 41 cd  11 c1 c0 84 f0 60 07 81    0<\ e@A      `  
00000853    82 f5 06 01 a0 18 4f 90  40 06 9a 3d 18 1c 08 4e          O @  =   N
00000854    06 00 78 18 2f 50 60 1a  41 84 f9 04 00 69 a3 c0      x /P` A    i  
00000855    40 5f 01 0c 7e 0a 75 60  45 f0 60 78 b8 14 ca 80    @_  ~ u`E `x    
00000856    83 9a 22 83 81 09 80 c0  0e 83 05 ea 0c 03 48 30      "           H0
00000857    9f 20 80 0d 34 7a 20 38  10 98 0c 00 e8 30 5e a0        4z 8     0^ 
00000858    c0 35 03 09 f2 08 00 d3  47 80 80 be 02 19 78 29     5      G     x)
00000859    d5 81 17 c1 81 e2 e0 53  2a 02 0e 68 86 0e 04 25           S*  h   %
00000860    03 00 3a 0c 17 a8 30 0d  40 c2 7c 82 00 34 d1 e8      :   0 @ |  4  
00000861    60 e0 42 50 30 03 80 c1  7a 83 00 d4 0c 27 c8 20    ` BP0   z    '  
00000862    03 4d 1c 02 02 f8 08 65  e0 a7 56 04 5f 06 07 8b     M     e  V _   
00000863    81 4c a8 08 39 a2 10 38  10 92 0c 00 e0 30 5e c0     L  9  8     0^ 
00000864    c0 35 03 09 f2 01 80 d3  47 20 83 81 09 00 c0 0e     5      G       
00000865    03 05 ec 0c 03 58 30 9f  20 18 0d 34 70 08 0b e0         X0    4p   
00000866    21 8f c1 4e ac 08 be 0c  0f 17 02 99 50 10 73 44    !  N        P sD
00000867    00 70 21 18 18 01 b0 60  bd 81 80 6b 06 13 e8 03     p!    `   k    
00000868    01 a6 8e 3e 07 02 11 41  80 1a 06 0b d8 18 06 c0       >   A        
00000869    61 3e 80 30 1a 68 d0 10  17 c0 43 2f 05 3a b0 22    a> 0 h    C/ : "
00000870    f8 30 3c 5c 0a 65 40 41  d3 01 01 01 01 ff ff 98     0<\ e@A        
00000871    08 08 08 0f ff fc c0 40  40 40 7f ff e6 02 02 02           @@@      
00000872    03 ff ff 30 10 10 10 1f  ff f9 80 80 80 80 ff ff       0            
00000873    cc 04 04 04 07 ff fe 60  20 20 20 3f ff f3 01 01           `   ?    
00000874    01 01 ff ff 98 08 08 08  0f ff fc c0 40 40 40 7f                @@@ 
00000875    ff e6 02 02 02 03 ff ff  30 10 10 10 1f ff f9 80            0       
00000876    80 80 80 ff ff cc 04 04  04 07 ff fe 60 20 20 20                `   
00000877    3f ff f3 01 01 01 01 ff  ff 98 08 08 08 0f ff fc    ?               
00000878    c0 40 40 40 7f ff e6 02  02 02 03 ff fe              @@@           
```

#### **VideoTagHeader**

```xml
00000035                                      12
```

#### **VIDEODATA**

```xml
00000035                                         00 00 84 02                    
00000036    92 26 02 02 02 03 ff ff  30 10 10 10 1f ff f9 80     &      0       
00000037    80 80 80 ff ff cc 04 04  04 07 ff fe 60 20 20 20                `   
00000038    3f ff f3 01 01 01 01 ff  ff 98 08 08 08 0f ff fc    ?               
00000039    c0 40 40 40 7f ff e6 02  02 02 03 ff ff 30 10 10     @@@         0  
00000040    10 1f ff f9 80 80 80 80  ff ff cc 04 04 04 07 ff                    
00000041    fe 60 20 20 20 3f ff f3  01 01 01 01 ff ff 98 08     `   ?          
00000042    08 08 0f ff fc c0 40 40  40 7f ff e6 02 02 02 03          @@@       
00000043    ff ff 30 10 10 10 1f ff  f9 80 80 80 80 ff ff cc      0             
00000044    04 04 04 07 ff fd c8 08  08 a8 30 1d 21 00 14 c2              0 !   
00000045    4a 02 e0 69 bf 05 41 80  e8 08 00 a6 12 50 2a 06    J  i  A      P* 
00000046    9b f1 f0 10 95 ac 5a e8  40 3e a9 72 c7 b9 01 01          Z @> r    
00000047    16 06 03 a0 20 02 98 49  40 a8 1a 6f c1 60 60 3a           I@  o ``:
00000048    02 00 29 84 94 0a 81 a6  fc 7c 04 25 6b 16 ba 10      )      | %k   
00000049    0f aa 5c b1 ee 40 40 45  81 80 e8 08 00 a6 12 40      \  @@E       @
00000050    82 a0 69 bf 05 81 80 e8  08 00 a6 12 40 82 a0 69      i         @  i
00000051    bf 1f 01 09 5a c5 ae 84  03 ea 97 2c 7b 90 10 11        Z      ,{   
00000052    60 60 3a 02 00 29 84 94  0a 81 a6 fc 16 06 03 a0    ``:  )          
00000053    20 02 98 49 02 0a 81 a6  fc 7c 04 25 6b 16 ba 10       I     | %k   
00000054    0f aa 5c b1 ee 40 40 45  81 80 e8 08 00 a6 12 40      \  @@E       @
00000055    82 a0 69 bf 05 81 80 e8  08 00 a6 12 40 82 a0 69      i         @  i
00000056    bf 1f 01 09 5a c5 ae 84  03 ea 97 2c 7b 90 10 11        Z      ,{   
00000057    60 60 3a 02 00 29 84 90  20 a8 1a 6f c1 60 60 3a    ``:  )     o ``:
00000058    02 00 29 84 94 0a 81 a6  fc 7c 04 25 6b 16 ba 10      )      | %k   
00000059    0f aa 5c b1 ee 40 40 45  81 80 e8 08 00 a6 12 50      \  @@E       P
00000060    2a 06 9b f0 58 18 0e 80  80 0a 61 25 02 a0 69 bf    *   X     a%  i 
00000061    1f 01 09 5a c5 ae 84 03  ea 97 2c 7b 90 10 11 60       Z      ,{   `
00000062    60 3a 02 00 29 84 94 0a  81 a6 fc 15 06 03 a4 20    `:  )           
00000063    02 98 49 40 5c 0d 37 e3  e0 1e 56 b1 6b a1 00 fa      I@\ 7   V k   
00000064    a5 cb 1e e4 04 04 54 18  0e 90 80 0a 61 25 01 70          T     a% p
00000065    34 df 82 a0 c0 74 84 00  53 09 68 15 03 4d f8 f8    4    t  S h  M  
00000066    07 95 ac 5a e8 40 3e a9  72 c7 b9 01 01 15 06 03       Z @> r       
00000067    a4 20 02 98 4b 40 a8 1a  6f c1 50 60 3a 42 00 29        K@  o P`:B )
00000068    84 b4 0a 81 a6 fc 7c 03  ca d6 2d 74 20 1f 54 b9          |   -t  T 
00000069    63 dc 80 80 8a 03 01 d4  10 81 4c 25 a0 54 0d 37    c         L% T 7
00000070    e0 a0 30 1d 41 08 14 c2  5a 05 40 d3 7e 3e 81 ef      0 A   Z @ ~>  
00000071    ac 30 84 03 ea 97 2c 7b  90 10 11 40 60 3a c2 10     0    ,{   @`:  
00000072    18 12 d0 2a 06 9b f0 4c  18 0e b0 84 06 04 b4 0a       *   L        
00000073    81 a6 fc 7d 03 df 58 61  06 07 fc b8 c5 c8 08 08       }  Xa        
00000074    98 30 1d 81 08 0c 09 68  15 03 4d f8 26 0c 07 60     0     h  M &  `
00000075    42 03 02 5a 05 40 d3 7e  3e 8f 3e 4d 06 07 fc b8    B  Z @ ~> >M    
00000076    c5 c8 08 08 90 30 1d 81  08 0c 0f d0 2a 06 9b f0         0      *   
00000077    48 18 0e c1 20 0c 0f d0  2a 06 9b f5 f4 79 f2 68    H       *    y h
00000078    30 3f e5 c6 2e 40 40 44  81 80 ed 12 00 c0 fd 02    0?  .@@D        
00000079    a0 69 bf 44 41 80 ed 12  00 c0 fd 02 a0 69 bf 5f     i DA        i _
00000080    47 9f 26 83 1e f9 71 8b  90 10 11 10 60 3b 84 80    G &   q     `;  
00000081    30 3f 41 e0 69 bf 44 01  80 ee 12 00 c0 fd 07 81    0?A i D         
00000082    a6 fd 7d 1e 7c 9a 0c 7b  e2 77 20 20 22 00 c0 77      } |  { w  "  w
00000083    09 00 60 7e 83 c0 d3 7e  88 03 01 de 25 2c 3f 41      `~   ~    %,?A
00000084    e0 69 bf 5f 47 9f 26 83  1e f8 9d c8 08 08 78 30     i _G &       x0
00000085    1e 02 52 c3 f4 1e 06 9b  f4 3c 18 0f 01 29 62 f4      R      <   )b 
00000086    1e 06 9b f5 f9 47 c9 a0  c7 be 27 72 02 02 1c 0c         G    'r    
00000087    07 88 94 b1 7a 0f 03 4d  fa 1c 0c 07 88 94 b1 7a        z  M       z
00000088    0f 03 4d fa fc a1 f0 43  d7 39 01 01 0e 06 03 c4      M    C 9      
00000089    4a 58 bd 07 81 a6 fd 0d  06 03 c8 7c b1 7a 0f 0b    JX         | z  
00000090    5f 94 3e 09 4b b9 e1 f3  c3 e8 27 d0 4f b7 c8 73    _ > K     ' O  s
00000091    d3 e7 a7 d0 8f a1 1f 6f  90 e7 a7 cf 4f a1 1f 43           o    O  C
00000092    3e df 21 cf 8f 9f 1f 43  3e 86 7d be 43 9f 1f 3e    > !    C> } C  >
00000093    3e 86 7d 0c fb 7c 8f 3e  3e 7c 7d 0c fa 21 f6 f9    > }  | >>|}  !  
00000094    0e 7c 7c f8 fa 19 f4 33  ed f2 1c f4 f9 e1 f4 23     ||    3       #
00000095    e8 47 db e4 39 e1 f3 b3  e8 27 d0 0f b7 c8 3c ec     G  9    '    < 
00000096    f9 d1 f3 d5 1f d9 ec bb  4c 1e dd f5 cb c7 ba f8            L       
00000097    7e 25 2b fd f7 8b fd 7f  18 86 e2 e8 f8 7e ab d3    ~%+          ~  
00000098    ff 2e fc f5 6a 98 6e 72  70 7c f4 f2 ec 6e 58 46     .  j nrp|   nXF
00000099    e9 15 97 5f df 45 7e bf  8d fa 58 7e 32 aa 2f 92       _ E~   X~2 / 
00000100    7e aa fc f5 6b f9 4f 3a  37 3e 6e 78 7c e2 c5 70    ~   k O:7>nx|  p
00000101    b8 0f a8 b6 6e 48 23 6a  3a 61 d4 bc bd 50 fb d7        nH#j:a   P  
00000102    ea ee 47 45 97 17 7c 7f  f9 e5 53 6b de 6a 7c d4      GE  |   Sk j| 
00000103    f9 af a8 fb e5 df c5 72  ab 6e 28 62 38 e4 b9 5d           r n(b8  ]
00000104    56 3a be f4 cd 1d ed dc  8b 98 82 06 03 bc 7c 3d    V:            |=
00000105    92 4b 55 5c 05 1a c8 b0  c3 f0 30 12 05 e3 cb 6d     KU\      0    m
00000106    91 5c 05 21 56 52 59 4d  0c cd cd 8f b9 c6 cc c8     \ !VRYM        
00000107    c0 d8 d5 ce 3b 25 e5 c6  87 cc 8f b9 c6 e4 b4 b0        ;%          
00000108    c4 f9 81 f7 48 c9 95 94  97 16 ba c6 4c a0 9c b0        H       L   
00000109    ad da 2e 64 c4 a5 25 0e  d1 5d 18 fa 31 f4 b3 e9      .d  %  ]  1   
00000110    87 db 64 ba 39 f4 73 e9  a7 d3 8f b6 49 74 83 e9      d 9 s     It  
00000111    07 d3 8f a7 1f 6c 92 e9  07 d2 4f a7 9f 4f 3e d9         l    O  O> 
00000112    27 d2 4f a5 1f 4f 3e a0  7d ae 4b a5 1f 4a 3e a0    ' O  O> } K  J> 
00000113    7d 40 fb 64 97 49 3e 92  7d 40 fa 79 f6 c9 2e 90    }@ d I> }@ y  . 
00000114    7d 20 fa 79 f4 d3 ed b2  4f 47 3e 76 08 12 17 8f    }  y    OG>v    
00000115    ee c9 67 95 5f e7 e6 c4  6e 4d 3e 90 5c 25 ab 1f      g _   nM> \%  
00000116    5d 55 72 e6 10 38 2b f2  95 5f f3 5b 2b a3 d5 59    ]Ur  8+  _ [+  Y
00000117    ef ca b3 de 7a 08 2a c0  ff da 91 21 a3 d1 20 7e        z *    !   ~
00000118    3e 2e 90 7a a8 7a 97 d2  ca 5b 1e 91 f2 fa ae eb    >. z z   [      
00000119    d2 8f c6 50 60 04 80 36  7e ff df 2e d5 d4 dd 18       P`  6~  .    
00000120    3a 40 60 1b c1 02 fa 7b  fe 2f c5 94 0a de 76 10    :@`    { /    v 
00000121    42 1a a1 fe 01 ef 51 f6  4c 93 db 10 77 5c 7c 3e    B     Q L   w\|>
00000122    56 ab 28 28 7f 7f 58 52  07 fe 5b 29 f4 93 e9 27    V ((  XR  [)   '
00000123    e4 01 80 8f 06 01 ca 7c  7a 3a 52 bc 56 e7 10 60           |z:R V  `
00000124    3c 41 80 0e 50 a0 0e e2  c3 c7 bc a8 21 0f 84 8f    <A  P       !   
00000125    c8 ad a9 6f 2c f5 fc b1  60 3b 79 49 4a 41 80 71       o,   `;yIJA q
00000126    9f 54 ab d3 75 52 7a d4  4d 87 c8 c1 80 0e 2e f8     T  uRz M     . 
00000127    30 11 85 df cd 9f 52 57  61 a2 51 f8 42 54 5f 09    0     RWa Q BT_ 
00000128    a3 7e 25 17 8f 2f bf 83  d4 fd ba cf 4c ba 41 2c     ~%  /      L A,
00000129    ba 2b df 68 f3 51 61 13  a3 f5 43 cc d5 43 dd 64     + h Qa   C  C d
00000130    0f fb 30 66 78 6a 24 c5  42 50 95 07 c3 c9 2c bb      0fxj$ BP    , 
00000131    b3 6c a8 4e a3 17 17 ee  f9 5a b5 76 31 29 d9 b8     l N     Z v1)  
00000132    3c f1 7c 85 fb 7d b1 4e  1c 66 03 bf 1f 7e 8f b2    < |  } N f   ~  
00000133    7f 2a 8d 3a b3 d3 e7 87  d0 cf a0 9f 70 8d 4c 6b     * :        p Lk
00000134    1d 1f ce 8f 9c 9f 40 3e  7e 7d c2 43 9c 1f 36 3e          @>~} C  6>
00000135    7a 7c ec fb 8c 7e 0d 4f  9a 1f 3a 38 3e e5 1f 93    z|   ~ O  :8>   
00000136    23 03 63 e6 a7 dc e3 72  5e 5c 66 7c c8 fb 9c 64    # c    r^\f|   d
00000137    cb 0a cc 0b dd 63 3a 81  f5 23 ea e7 d6 0f b5 49         c:  #     I
00000138    f5 33 ea 67 d6 0f ac 9f  6a 94 ea 87 d5 0f ad 1f     3 g    j       
00000139    5b 3e d5 29 d5 4f ab 1f  5b 3e b8 7d a6 57 ab 1f    [> ) O  [> } W  
00000140    56 3e b8 7d 70 fb 4c a7  56 3e ac 7d 70 fa e1 f6    V> }p L V> }p   
00000141    a9 4e ac 7d 54 fa e1 f5  b3 ed 52 9d 50 fa 99 f5     N }T     R P   
00000142    a3 eb 47 da e5 3a 99 f5  23 eb 27 d6 0f b5 ca 75      G  :  # '    u
00000143    13 ea 07 d5 cf ab 1f 6b  93 e9 e7 d3 8f aa 9f 52           k       R
00000144    2e be aa e4 57 6f 95 33  7d 36 c8 9a 3d b2 4d e2    .   Wo 3}6  = M 
00000145    c0 c0 0a 03 03 1c 10 00  a5 a8 7e f2 90 60 1a 04              ~  `  
00000146    92 f1 20 21 8f 80 f5 c9  47 db 63 16 8f 69 b1 60       !    G c  i `
00000147    0c 12 c0 30 7c 10 84 a1  f4 08 73 f1 52 a9 25 fc       0|     s R % 
00000148    cf a3 90 d1 68 30 0c 9e  1f 17 a8 08 7e 12 65 be        h0      ~ e 
00000149    f4 55 07 d8 de fe db 54  ea 76 8f 3b fa 50 43 ef     U     T v ; PC 
00000150    ae 5b 4c 44 5b 00 f7 3f  36 61 97 a1 89 61 06 89     [LD[  ?6a   a  
00000151    6d c9 1e 99 ee 3d 40 fa  79 f7 6a 08 01 02 dd a2    m    =@ y j     
00000152    57 ad 21 89 80 82 10 7d  14 41 2f f2 10 74 83 e8    W !    } A/  t  
00000153    e7 d3 8f a6 9f 6d 92 e8  c7 d1 0f a6 1f 4a 3e dd         m       J> 
00000154    25 d0 cf a0 9f 48 3e 8c  7d ba 47 9f 9f 3d 3e 8a    %    H> } G  => 
00000155    7d 0a 3d be 47 27 67 28  07 cf 8f b8 48 48 6e 6c    } = G'g(    HHnl
00000156    78 7c e9 c6 3e 66 a6 67  06 ee 51 dd 70 fa e9 f6    x|  >f g  Q p   
00000157    23 ec 67 da 65 7a f9 f6  03 ec 87 d9 4f b4 4b 76    # g ez      O Kv
00000158    13 ec 27 d9 8f b3 9f 67  95 ec 47 d8 cf b4 1f 69      '    g  G    i
00000159    3e cf 2d d8 cf b1 9f 69  3e d2 7d 9e 5b b1 9f 63    > -    i> } [  c
00000160    3e d2 7d a4 fb 3c b7 62  3e c2 7d a4 fb 39 f6 79    > }  < b> }  9 y
00000161    6e c0 7d 80 fb 39 f6 73  ed 12 ab 60 3e be 7d 98    n }  9 s   `> } 
00000162    fb 11 77 be 5f f8 a9 54  bf fd 6a 39 a0 f4 b3 d7      w _  T  j9    
00000163    4f ae 9f 5f a0 a0 12 04  90 50 03 00 e2 10 c1 80    O  _     P      
00000164    8d f4 55 6d 1f fc 4a 56  25 81 d6 3c a1 55 96 42      Um  JV%  < U B
00000165    56 81 2c 10 41 80 14 55  55 40 84 0c 03 7f 80 35    V , A  UU@     5
00000166    4e ec 82 41 78 fb ea ab  11 47 d4 a9 9f 62 fa f1    N  Ax    G   b  
00000167    98 e6 94 55 29 89 54 6e  eb 87 d6 a5 eb 98 cf af       U) Tn        
00000168    aa 2f 51 f9 f5 93 4b 97  39 73 f2 fa e2 77 34 ca     /Q   K 9s   w4 
00000169    bd 33 c0 83 f9 f0 60 19  f6 17 60 06 03 00 28 25     3    `   `   (%
00000170    89 22 52 95 52 e0 ff d6  1e 66 06 00 38 18 06 e0     "R R    f  8   
00000171    86 0c 03 81 77 fe 24 a8  05 15 93 ca 3d 3c dc e9        w $     =<  
00000172    a5 a1 2c 18 01 a0 60 17  30 20 d0 81 00 38 7f e0      ,   ` 0    8  
00000173    60 1c 40 3a 89 43 fb 4b  c7 ca af c4 a2 fc fb 5f    ` @: C K       _
00000174    c1 e2 b4 fe 25 6c 08 00  c0 08 83 00 e4 0c 00 a1        %l          
00000175    7a b1 20 10 41 09 55 c9  fa 5c 5d f0 3f db 39 6a    z   A U  \] ? 9j
00000176    b9 c6 98 95 ed 52 97 5d  cb 2d b9 c2 29 3b 3f 33         R ] -  );?3
00000177    65 8e ea e7 d5 8f ae 9f  5c 3e d5 29 d5 0f a9 9f    e       \> )    
00000178    5a 3e b2 7d aa 53 a9 1f  50 3e b0 7d 58 fb 5c a7    Z> } S  P> }X \ 
00000179    4e 3e 98 7d 50 fa 91 f6  c9 3e 94 7d 22 3d 42 3d    N> }P    > }"=B=
00000180    38 fb 64 9f 45 3e 86 7d  2c fa 49 f6 e9 2e 82 7c    8 d E> }, I  . |
00000181    fc fa 39 f4 48 f6 f9 1e  7a 7c ec fa 11 f4 03 ee      9 H   z|      
00000182    11 fd a0 fb 49 f6 e3 ed  e7 d9 e5 fb 51 f6 b3 ee        I       Q   
00000183    07 dc 8f b3 4b f6 d3 ed  c7 dc cf ba 1f 66 97 ed        K        f  
00000184    e7 db cf ba 9f 76 3e cb  2f dc 0f b8 1f 77 3e ee         v> /    w> 
00000185    7d 96 63 b8 1f 70 3e ee  7d d8 fb 2c bb db cf b7    } c  p> }  ,    
00000186    1f 76 3f 10 10 c1 80 69  08 02 47 3d 68 43 2f dc     v?    i  G=hC/ 
00000187    9e 1f 77 da b5 2f 03 51  16 bd 78 4a 00 e1 24 21      w  / Q  xJ  $!
00000188    dc 1f 17 5f a7 6b d1 2f  e9 c9 af ab f7 9b 7b db       _ k /      { 
00000189    8f b6 7b 7f 73 d3 5f 20  08 01 0c 18 06 60 60 1b      { s _      `` 
00000190    87 e1 06 02 00 90 3e 56  ac 10 3d 07 ea c1 80 0f          >V  =     
00000191    51 7e 25 8f c7 ca 54 7e  aa bb 2f d4 5b 14 a8 b2    Q~%   T~  / [   
00000192    f6 63 9b 01 80 0e 12 55  03 00 dc 24 2b f0 fb d4     c     U   $+   
00000193    48 08 32 d1 f5 12 bf 73  e2 50 43 93 41 0a af 15    H 2    s PC A   
00000194    b4 ab e8 ae 3d 58 bc 18  00 e0 42 b6 7c 4b 2e 57        =X    B |K.W
00000195    4b 94 51 2f c5 d7 7e 3e  8a da b7 5b f5 5e 19 ab    K Q/  ~>   [ ^  
00000196    04 10 60 1c 87 e2 42 a2  e5 63 e1 2b 55 2b 57 3d      `   B  c +U+W=
00000197    47 ff bc f6 d9 37 19 43  c3 2f 6b 12 c1 80 11 06    G    7 C /k     
00000198    01 a4 03 7f e1 20 18 01  38 ab 6f af c7 e2 5f ec            8 o   _ 
00000199    50 a9 62 ef 56 7e f6 e0  60 07 c2 10 06 83 00 28    P b V~  `      (
00000200    0c 03 44 2e 06 01 b6 fc  10 fe a5 57 ed 2f 1e aa      D.       W /  
00000201    53 f9 a5 d8 06 35 a3 c9  40 c0 11 80 78 fc 48 f4    S    5  @   x H 
00000202    a3 f9 73 54 59 9d 21 29  06 01 a8 21 04 20 41 08      sTY !)   !  A 
00000203    03 f1 f9 72 af 50 86 07  80 e7 da b7 fe ef ba bb       r P          
00000204    11 eb 7f 00 f5 62 47 fe  01 d2 2a f8 06 2b 83 cb         bG   *  +  
00000205    7d 22 9d a0 a5 aa db 7d  29 72 81 24 ba 97 8f 6c    }"     })r $   l
00000206    12 55 5d ca 23 59 18 73  db 01 80 63 06 01 a9 5f     U] #Y s   c   _
00000207    c7 ea d5 02 18 91 ce 61  78 90 aa 7e 23 fc be a9           ax  ~#   
00000208    4f cd aa 06 01 c8 bd 51  7e 97 d2 eb de 79 57 8b    O      Q~    yW 
00000209    ad a9 01 40 aa 65 21 7c  06 01 9c 18 01 40 60 1c       @ e!|     @` 
00000210    a8 07 ab 57 e1 22 97 cb  20 1a be fd 02 1a a4 65       W "         e
00000211    3e 5e 24 17 89 57 04 b1  f1 78 fa ac 5e ab e5 d4    >^$  W   x  ^   
00000212    b6 39 20 48 08 00 c0 08  ff c5 d4 7b 14 e4 df 52     9 H       {   R
00000213    f5 1b 52 53 d5 d0 18 07  1b fb 7d 15 aa b6 72 c6      RS      }   r 
00000214    5c f7 60 60 04 41 80 6d  06 00 3c 18 07 0f f8 4a    \ `` A m  <    J
00000215    f0 07 f8 4a 08 7f e4 12  84 bf 2b 96 d5 a2 b5 53       J      +    S
00000216    e3 ea 04 74 91 51 50 30  02 01 0f 6d f7 95 0f c7       t QP0   m    
00000217    9d 51 62 99 56 53 e6 43  2a 12 ed 1f 17 f8 49 1f     Qb VS C*     I 
00000218    58 5c 25 f9 a1 fc 9e 53  11 d8 f8 81 28 18 06 20    X\%    S    (   
00000219    60 1b cb 95 2b 12 87 e5  f0 03 c7 c5 f7 68 21 d5    `   +        h! 
00000220    70 7f e6 01 44 3f f7 e5  f4 65 46 5f 69 84 60 86    p   D?   eF_i ` 
00000221    0c 00 aa a5 7d 08 18 a8  0b 7e df f9 15 31 59 02        }    ~   1Y 
00000222    1a b5 33 55 97 c9 18 f2  8e a1 8f 7a 61 78 30 0a      3U       zax0 
00000223    a0 c0 38 02 04 85 c2 58  30 03 55 5f d5 72 51 20      8    X0 U_ rQ 
00000224    7e a1 56 24 ff 95 e6 11  b3 2b 06 01 b4 21 83 00    ~ V$     +   !  
00000225    da 10 cb c2 08 fc 03 20  07 7a cf 49 07 a3 de fa             z I    
00000226    47 4e 80 68 97 44 9b b2  17 b7 bd 57 15 95 9e 9d    GN h D     W    
00000227    08 40 c0 37 83 00 e0 0c  00 74 ff c0 3a 4e c5 0a     @ 7     t  :N  
00000228    d5 2b 9e 1f 8f 7c 9b d7  27 b2 b3 ed 56 4c a6 a8     +   |  '   VL  
00000229    18 01 81 2f 5a 12 8b 80  be 8b 28 7e 0c 03 97 38       /Z     (~   8
00000230    ae da c4 57 cf 23 ae 7b  21 f6 4a 5f ef 4d ff e7       W # {! J_ M  
00000231    a2 ff b2 41 9b b0 30 02  e0 c0 35 09 60 c0 38 6d       A  0   5 ` 8m
00000232    04 10 80 0c 00 70 06 09  4a b9 47 ea d5 d1 f0 ff         p  J G     
00000233    f5 37 95 d9 31 7f db 1c  b4 0c 03 79 70 07 04 2f     7  1      yp  /
00000234    2a 56 a8 03 68 07 09 7e  c2 f2 ea 3c 56 ae ca dc    *V  h  ~   <V   
00000235    bb 95 52 79 d7 af 04 30  0c 56 10 3f 9f ba 05 fd      Ry   0 V ?    
00000236    3f 7c 87 1d 46 25 00 7a  b5 7f 51 f1 f4 8b ff c3    ?|  F% z  Q     
00000237    f9 62 0b 3c f5 b2 78 4b  2e ff 3c ad 57 ea d3 f1     b <  xK. < W   
00000238    ec 07 e1 02 18 30 0c a0  c0 37 89 76 89 05 e0 c0         0   7 v    
00000239    34 fc bc 7c b5 00 cf 49  bb cb a3 cf 75 27 c8 db    4  |   I    u'  
00000240    42 15 80 a0 55 85 f4 7f  b3 40 fd 32 ce 7a 5b af    B   U    @ 2 z[ 
00000241    1f 5c 3e ca 7d 90 fb 4c  b7 5b 3e b2 7d 88 fb 01     \> }  L [> }   
00000242    f6 99 5e ac 7d 52 3d 76  3d 6e 3d aa 57 a9 1f 4f      ^ }R=v=n= W  O
00000243    3e af 1e ab 1e d7 29 d3  63 d2 a3 d4 a3 d4 0f b6    >     ) c       
00000244    49 74 73 e8 a7 d3 4f a5  c7 b7 49 77 43 ee a7 de    Its   O   IwC   
00000245    8f bd 9f 65 97 ee e7 de  0f be 9f 7d 94 eb 24 c3       e       }  $ 
00000246    de 4f bd 1f 67 08 02 41  71 79 95 02 e0 60 06 61     O  g  Aqy   ` a
00000247    7b 55 cc f7 f1 d2 d3 d5  ee df 1f 0f a5 03 01 46    {U             F
00000248    f4 ca 0c 01 18 05 b5 02  84 02 a4 c1 08 02 de fa                    
00000249    7d f4 fa 9d 06 00 8c 1a  07 b6 40 41 08 00 18 0c    }         @A    
00000250    00 ac 94 18 07 21 2c 79  7e 06 8b a4 aa c7 ff 4f         !,y~      O
00000251    15 46 13 dc 73 31 75 05  0f c7 85 d1 4f d6 85 76     F  s1u     O  v
00000252    de 48 9d f2 bf f5 55 37  fc 73 de 3c 5d f7 b8 89     H    U7 s <]   
00000253    2a a0 06 00 60 96 01 e3  fc 05 11 77 ff e5 42 29    *   `      w  B)
00000254    7f e4 e4 4d 35 bf d4 51  ed 60 c0 0a 03 00 c4 0c       M5  Q `      
00000255    00 b0 96 0c 03 94 00 cf  89 05 f2 7f d4 10 be 5f                   _
00000256    c5 25 f3 b9 ff 46 b6 6e  1d 8a 06 01 94 18 07 18     %   F n        
00000257    25 80 78 20 89 52 fb 21  7d 55 55 2a 96 7b d2 fa    % x  R !}UU* {  
00000258    64 cf a6 45 6e 9e 50 08  40 c0 09 0f e0 43 9e 08    d  En P @    C  
00000259    6a b3 d3 d0 7f cb 2c fc  d6 b8 33 a4 1f 84 19 98    j     ,   3     
00000260    5d ba 9e 30 bb 9f 5c 5e  10 c1 04 10 6a af 8f fe    ]  0  \^    j   
00000261    ad 58 91 f6 cb 84 8d 54  5c 5c 9a c6 6c 5f f6 d2     X     T\\  l_  
00000262    59 f2 f0 60 04 bd 41 80  10 08 41 04 4a cb 7e 3d    Y  `  A   A J ~=
00000263    1f 17 ab 9a a5 50 41 2e  a3 b1 f0 97 37 33 fe 50         PA.    73 P
00000264    5e a9 75 5f 96 65 75 b1  70 93 f1 2b 4b 87 ff fc    ^ u_ eu p  +K   
00000265    2e 1e 62 a6 7d 3f 35 47  f0 af 0d d5 83 00 1e 25    . b }?5G       %
00000266    83 00 dc 5f 3e 5c 01 9e  fa bd ba 24 df e5 12 2c       _>\     $   ,
00000267    f0 1a f9 7c fe 2a f5 2e  db 85 fb 33 dc ec 38 58       | * .   3  8X
00000268    ae 64 f6 7f c5 d1 55 d9  18 b3 31 46 42 83 36 5f     d    U   1FB 6_
00000269    9f ff 54 c4 cf 7c 20 30  03 00 c0 2a f8 18 01 90      T  | 0   *    
00000270    60 04 0b 81 40 0c 00 97  8b c1 04 b8 7e 5e a8 03    `   @       ~^  
00000271    4b d4 f8 7d 02 08 42 fe  02 15 12 55 8f 3e 24 ab    K  }  B    U >$ 
00000272    12 c7 d6 7d 52 ac aa 95  4c 9f fa 9b fb 1c fc 0c       }R   L       
00000273    00 88 06 c0 60 1c 41 80  63 06 01 a8 18 07 20 0e        ` A c       
00000274    53 40 38 21 a8 56 ab 07  d4 bb 07 f6 cb e5 58 c9    S@8! V        X 
00000275    7f 87 7b 59 37 20 01 80  c0 0d 02 05 08 05 e0 1a      {Y7           
00000276    10 c2 0a a0 83 44 ab 60  fc b8 7d f9 ff 79 5b 79         D `  }  y[y
00000277    85 d9 fb ba dd ff a5 ee  5c 75 28 30 1d 02 48 94            \u(0  H 
00000278    24 97 50 81 82 41 70 40  2f 9f 2e 55 ec 9c 97 bc    $ P  Ap@/ .U    
00000279    f6 7a 6c 6f 96 d6 7e f3  aa 10 04 a1 fd 08 70 4a     zlo  ~       pJ
00000280    12 a0 f8 bd 5c be 1f 89  7f 96 5f 8f 6c 56 d3 3b        \     _ lV ;
00000281    64 b2 98 b4 12 40 f8 43  12 a8 40 b4 0e 58 cb 11    d    @ C  @  X  
00000282    6f 79 74 ef 79 b8 30 01  d4 4a 08 63 ed fa aa 5c    oyt y 0  J c   \
00000283    5e a6 cf c5 eb 34 e9 57  c7 85 d2 61 0a ff c1 80    ^    4 W   a    
00000284    71 03 c0 c0 39 04 00 60  1c bd fb 44 80 60 1c 44    q   9  `   D ` D
00000285    81 f0 96 dd c9 6c c5 68  3e e2 b5 4a bc 25 c9 fd         l h>  J %  
00000286    d9 64 7b 39 7f 94 7e a9  dd 9f b3 f1 6f ba 8c 48     d{9  ~     o  H
00000287    2e f1 70 f9 57 bd d8 a2  49 7f df a3 7b d1 c1 04    . p W   I   {   
00000288    10 00 34 7f 54 8f 84 82  f2 f9 5a aa a2 91 83 80      4 T     Z     
00000289    30 0c 20 c0 34 83 00 da  0c 00 9c 00 e0 60 03 b3    0   4        `  
00000290    d1 5f d5 51 2c 7e a8 20  8f a1 7e 97 5b 55 0f 8b     _ Q,~    ~ [U  
00000291    ac f8 14 ff e9 01 20 20  0f e8 42 aa cb 84 bb b3              B     
00000292    df bf 55 41 50 b5 22 46  06 01 a1 50 42 08 20 c0      UAP "F   PB   
00000293    39 89 60 1f 24 fd 8a 82  12 a9 fe 35 15 5b 9a 9a    9 ` $      5 [  
00000294    d7 b0 02 00 92 ad 50 93  3e ac 7d e1 f8 fb d2 e6          P > }     
00000295    c6 6c cf 1f b5 00 d0 40  04 00 40 2e 56 08 23 e0     l     @  @.V # 
00000296    3f 42 1c bd 2e fa b0 51  74 47 b9 9a 33 7b c0 30    ?B  .  QtG  3{ 0
00000297    03 3e 12 81 80 69 54 3f  1f 82 08 30 02 01 00 ba     >   iT?   0    
00000298    42 f8 3e 55 f1 22 17 89  40 7e c9 2a bb 9e b2 4c    B >U "  @~ *   L
00000299    b7 d3 77 b7 49 9f 87 e0  c0 0b 84 2a 08 34 20 17      w I      * 4  
00000300    7c bc bb 44 9f 0f a5 e2  bc 85 ff 92 37 96 e4 fa    |  D        7   
00000301    e7 57 73 2d b2 4e af 2f  13 fa cb 2b 99 2a aa a8     Ws- N /   + *  
00000302    7f 04 b5 5e d5 7e fc 57  ef fc 77 f9 ff 76 5b bc       ^ ~ W  w  v[ 
00000303    b9 63 31 ea a5 e2 5f ff  fa ac bb f4 48 b5 a8 5d     c1   _     H  ]
00000304    6c f3 b2 44 a0 60 1a 21  75 b4 bc 20 79 5c a2 27    l  D ` !u   y\ '
00000305    bf 98 85 cf 67 08 20 c0  1f 03 00 dc ab 68 40 00        g        h@ 
00000306    d0 0c 08 72 6f 15 81 f0  84 5e 54 3f 54 0d 2b ec       ro    ^T?T + 
00000307    38 96 0c 00 f0 30 0d 01  02 ff c5 e0 c0 07 0f fe    8    0          
00000308    a1 ba 25 60 93 10 97 5b  46 48 c2 40 30 0c 00 83      %`   [FH @0   
00000309    e0 60 7d c1 0b e9 55 5f  54 2a ae 0c 94 c1 80 70     `}   U_T*     p
00000310    06 01 ac 4b 08 20 18 10  55 04 09 40 38 7e 24 cb       K    U  @8~$ 
00000311    e8 25 8f 7e 5e 08 4a 6c  1e fd 53 56 55 1b 15 63     % ~^ Jl  SVU  c
00000312    4f 55 12 00 34 18 06 f5  5f c5 62 4a a2 f5 5c e9    OU  4   _ bJ  \ 
00000313    7f a5 b1 7b c9 3f 46 b7  6a c1 80 6c 06 00 4c 7f       { ?F j  l  L 
00000314    e9 7c 3e 12 c2 12 bd ec  b3 7d 62 6b 2b df 39 f2     |>      }bk+ 9 
00000315    ec a0 74 ba ca 05 fd 7d  46 72 ff 00 f0 82 a8 48      t    }Fr     H
00000316    be 82 49 7d 54 c2 a2 fa  08 7e 89 21 7a 9c 44 79      I}T    ~ !z Dy
00000317    c0 48 1f 04 31 28 7d 22  b0 41 12 40 3f 58 08 76     H  1(}" A @?X v
00000318    7e 22 f7 fd 4a c3 36 aa  25 03 00 1e 5f eb 02 08    ~"  J 6 %   _   
00000319    07 09 4a ed 80 a4 f5 1f  7e d0 2a 25 7c 02 90 e8      J     ~ *%|   
00000320    30 0e 1d c0 3d 0b d5 01  0d 45 54 bb 34 18 0e ef    0   =    ET 4   
00000321    45 73 3d f1 e0 19 dc a8  05 6f b9 2f 04 11 2d 4e    Es=      o /  -N
00000322    82 89 11 73 9f 01 80 12  fd 06 02 38 20 5f b7 4b       s       8 _ K
00000323    b6 d4 b5 f0 60 c0 4e 03  00 25 7e a6 84 2a 92 04        ` N  %~  *  
00000324    31 2e c1 73 71 7a a5 43  e5 42 57 8b bd 3d ef 78    1. sqz C BW  = x
00000325    79 3d 1a 93 c6 93 01 80  6f 00 f1 f0 43 12 bc a8    y=      o   C   
00000326    b8 7c 07 94 2b 53 6e 7b  bc cc e3 ae 01 80 0e 06     |  +Sn{        
00000327    01 b1 50 92 10 4b 84 af  17 2b 83 d9 f9 b6 c6 22      P  K   +     "
00000328    91 77 6d 3e d6 7d cc fb  91 f6 79 7e d0 7d 98 fb     wm> }    y~ }  
00000329    79 f6 d3 ec f2 fd 92 3d  86 3d ac fb 44 7b 3c b7    y      = =  D{< 
00000330    5f 3e b9 1e c9 1e c5 1e  d1 2d d6 63 d5 a3 d7 e3    _>       - c    
00000331    d7 0f b5 4a f5 38 f4 f3  eb 31 ea c7 da e5 36 ff       J 8   1    6 
00000332    02 7e 10 fc 31 f6 39 8f  06 7e 14 fc 41 f8 aa f6     ~  1 9  ~  A   
00000333    29 85 c0 aa 12 55 97 5d  f3 db 47 c0 c0 33 09 65    )    U ]  G  3 e
00000334    c6 63 4f c7 1f 63 3d 30  f5 ff 03 00 b6 24 c3 2b     cO  c=0     $ +
00000335    40 c0 6c 03 40 f5 20 7e  40 fb 20 f3 cb 3a 5d 4d    @ l @  ~@    :]M
00000336    27 7a b4 06 01 58 bc 9d  68 18 01 d5 70 18 06 a0    'z   X  h   p   
00000337    60 18 41 80 6f 00 f0 83  0b 84 b9 01 01 57 8b 95    ` A o        W  
00000338    17 cf 5b 27 ec 92 8f a7  e5 93 27 bf ef d0 54 92      ['      '   T 
00000339    c3 89 40 1c 25 89 37 15  2a af 7a 06 01 c4 18 01      @ % 7 * z     
00000340    90 60 1b c1 80 0f 04 0f  83 00 d6 3e a2 59 75 08     `         > Yu 
00000341    52 02 1d 12 bf ff 17 c1  2b fc bf 1d 59 6f bd 71    R       +   Yo q
00000342    4c 5b a6 17 c4 a0 50 29  f8 91 ff c1 f4 08 3e a2    L[    P)      > 
00000343    40 f7 9b eb fb 2c 45 6c  c7 cc fc 0f e4 57 25 56    @    ,El     W%V
00000344    07 a8 f7 17 3c f8 20 60  12 41 80 11 06 00 3c 18        <  ` A    < 
00000345    01 00 0f 08 03 f0 60 05  a4 12 07 fe 97 ff 12 8b          `         
00000346    94 d8 5c ac 4a fb 2a ff  ff e9 7d fe ea a9 99 27      \ J *   }    '
00000347    97 d7 ce c0 0d 06 00 54  18 06 e5 40 a1 08 25 f0           T   @  % 
00000348    10 01 06 4f 45 6a fb 85  f3 cb 37 ef cb dc 94 0e       OEj    7     
00000349    ba 0c 20 03 00 d0 a8 10  40 38 03 81 80 ec aa c1            @8      
00000350    80 68 00 d1 f2 b1 2e 00  77 be aa 09 0a c1 0a 8e     h    . w       
00000351    a0 21 fe dc 12 e5 8d 51  e5 db 37 36 57 de 03 00     !     Q  76W   
00000352    e0 01 8a c1 00 20 ab 55  f1 28 bc 48 f2 bb f2 f1           U ( H    
00000353    29 51 7b 63 cb 5a d6 bf  8b b8 e0 10 07 c0 84 a8    )Q{c Z          
00000354    b8 10 04 9b d2 ed 83 e1  f8 f7 15 ab 1d db c9 b6                    
00000355    c7 56 ab 57 e0 86 3f 57  b0 79 b5 4f a4 b9 53 a2     V W  ?W y O  S 
00000356    7b df c1 80 17 06 00 50  10 02 08 20 09 20 c0 38    {      P       8
00000357    04 08 5e 0c 03 84 12 95  fd 52 b1 ea a0 3d e1 f5      ^      R   =  
00000358    99 93 fe 97 73 f2 cf cf  f3 7d 5a 87 6a 81 80 69        s    }Z j  i
00000359    08 60 83 f5 74 b9 5e 0f  82 17 b9 02 00 95 e8 3e     `  t ^        >
00000360    1f 48 9e 7a 41 ed 4f f3  f2 a0 c0 1c 03 00 c7 01     H zA O         
00000361    80 19 06 01 aa 0f 82 1d  06 04 40 18 06 e5 01 00              @     
00000362    03 d4 81 a5 5f b7 04 81  26 a6 f7 ed 54 ad 72 e5        _   &   T r 
00000363    7f a6 17 41 80 57 06 00  99 57 42 18 42 9a 3e fa       A W   WB B > 
00000364    30 3d fa 22 50 2d 41 49  48 10 0b c4 90 60 23 c1    0= "P-AIH    `# 
00000365    80 70 50 24 f4 48 08 32  51 20 4b ff 6a ab 39 f1     pP$ H 2Q K j 9 
00000366    f5 b5 6b d8 9a ba 9e 09  10 0f 01 f1 f7 55 97 01      k          U  
00000367    b5 4d ef 98 fc 2a 7b ee  80 30 03 cb c0 33 ea 4b     M   *{  0   3 K
00000368    a8 1c 57 e1 e2 b5 4c 2b  54 06 3e 91 bf 56 1f 74      W   L+T >  V t
00000369    5e 07 47 fa 5d 2f c7 ad  49 2d a9 31 d7 62 48 07    ^ G ]/  I- 1 bH 
00000370    0f c2 10 fe 97 28 b7 7f  93 d9 6d db 72 49 17 b4         (    m rI  
00000371    d3 80 30 03 c0 c0 16 83  00 b4 08 0a 81 0d 54 04      0           T 
00000372    00 81 40 3d 52 af fc 18  0f 8b f5 62 58 95 26 df      @=R      bX & 
00000373    4a 3e 93 22 ba 22 cd d8  d5 ae 32 00 d0 60 1c 42    J> " "    2  ` B
00000374    10 07 2a 9f 56 3f 2e f2  b5 70 14 5b b5 4c 8c d8      * V?.  p [ L  
00000375    aa 5b 90 63 54 a8 7c ac  4b e8 fb ca 6c 45 9a bb     [ cT | K   lE  
00000376    df 22 01 81 04 10 60 20  0f 81 00 03 42 10 fc ba     "    `     B   
00000377    00 76 d0 84 5e ac b9 5f  b5 52 b6 d5 6d 99 cb 97     v  ^  _ R  m   
00000378    c4 29 80 c0 11 83 00 df  42 18 95 42 07 fe 0a 31     )      B  B   1
00000379    29 a4 ea 8c 28 83 00 e4  0c 00 e8 30 0c e3 f1 22    )   (      0   "
00000380    09 4a ef 84 be cf ab 57  3d 3c a6 c5 bd b5 47 91     J     W=<    G 
00000381    d7 b6 84 00 60 04 01 80  6d 06 00 50 03 4b 84 90        `   m  P K  
00000382    50 7f 3d 73 fc bd 55 4d  b0 84 10 0f 1f 04 11 ff    P =s  UM        
00000383    d5 ca aa 5f ad be 96 81  9c b7 95 f3 6a 82 10 fd       _        j   
00000384    52 ac ff ba 48 f2 e1 25  56 aa 57 19 fc 36 e0 01    R   H  %V W  6  
00000385    e0 1a 25 83 00 38 3e 2f  1f 2b 2e 81 0f c0 87 7f      %  8>/ +.     
00000386    d5 77 ca d4 7e 4b ec f8  ee 72 72 4a e2 ab f8 af     w  ~K   rrJ    
00000387    de c6 64 b5 34 7b 80 30  0a 20 a0 53 61 78 21 c9      d 4{ 0   Sax! 
00000388    9e e8 fa 2a fb 5e 03 0a  ea a2 c5 64 0b 40 c0 09       * ^     d @  
00000389    74 4b 83 ef c5 2a 95 f6  f9 7c 35 50 01 fe f5 99    tK   *   |5P    
00000390    64 b1 ab 1e f7 60 60 03  81 80 63 00 e0 60 1c 76    d    ``   c  ` v
00000391    7b df 1f 17 f9 b2 ef d9  6a 59 79 ae 61 82 5a aa    {       jYy a Z 
00000392    72 ec 7c 5e 24 09 02 4c  12 d5 09 21 0e 55 5f de    r |^$  L   ! U_ 
00000393    04 22 e9 6f 5a ba 10 73  1e 82 0c 00 e0 30 01 c0     " oZ  s     0  
00000394    80 10 c1 80 0e 00 e0 60  1c 22 aa 5f f1 2c b8 7b           ` " _ , {
00000395    29 7d 53 27 6c e2 a9 c9  78 b5 23 48 06 01 b4 7c    )}S'l   x #H   |
00000396    25 00 67 a0 fe 0f bf 9f  aa 32 5c 93 fd ab be e8    % g      2\     
00000397    18 09 70 84 25 ab cf 40  42 96 6d ca d2 49 b5 cf      p %  @B m  I  
00000398    5d 3c c8 3f b7 d0 e1 05  2e 99 95 52 af a6 c1 52    ]< ?    .  R   R
00000399    10 30 0d a0 c0 07 00 65  1f a9 f6 f8 be fa 45 65     0     e      Ee
00000400    de e0 fb cc 8f 54 66 82  1d 6a de 45 10 d3 18 f8         Tf  j E    
00000401    4b a5 ea d4 28 46 ec 00  0f 06 00 78 b8 7c aa ab    K   (F     x |  
00000402    f0 07 d4 d0 bd 54 ed 5b  7d 45 cf 6d 1e fe f2 bd         T [}E m    
00000403    98 21 00 60 43 00 d5 63  c1 21 55 57 fc d6 e7 a4     ! `C  c !UW    
00000404    8b db 4c 35 7c 4b 06 00  43 c2 4f c4 8f 04 31 f5      L5|K  C O   1 
00000405    2f f7 ad 57 2f e6 2a 93  27 6b 5b b6 64 36 d8 0c    /  W/ * 'k[ d6  
00000406    07 42 a1 f0 f8 be 03 00  20 24 84 21 24 10 66 41     B       $ !$ fA
00000407    f7 be 5f 8c 81 d9 2e fb  79 2e 7a 7c da 80 30 11      _   . y.z|  0 
00000408    82 4d 9c f7 e5 e7 7e aa  e4 d6 5f 96 25 03 00 2b     M    ~   _ %  +
00000409    e6 c4 b5 77 ec a8 f4 be  96 2d 7f db 36 2c e7 a5       w     -  6,  
00000410    17 83 00 3b f0 0e 08 54  49 08 62 4f af f9 83 e2       ;   TI bO    
00000411    ee c4 6a eb 68 9c de 01  b7 07 84 2e a0 80 0c 02      j h      .    
00000412    e8 fc 4b 1f fc 03 81 80  6c 2f f8 95 ff ee 40 87      K     l/    @ 
00000413    cf 7b ec 48 af 93 2b 0a  71 2b d8 82 18 07 41 ff     { H  + q+    A 
00000414    a7 73 f3 be fc ef ac a5  71 eb 4a c2 10 92 25 4b     s      q J   %K
00000415    04 be ed e2 be 0f 2a 4b  38 88 de 21 7f c4 a0 60          *K8  !   `
00000416    03 81 80 70 12 04 9f 09  10 b8 03 3c ab 34 7c a7       p       < 4| 
00000417    fd b3 f2 fd 5d ff 6a bd  a8 24 3c 77 b3 ef 27 df        ] j  $<w  ' 
00000418    cb d5 2b 12 be a0 be 65  2c a4 50 47 d9 27 0b 87      +    e, PG '  
00000419    ea 87 ea a1 79 77 e2 b5  5f 6a bb bb 1f 74 3e fe        yw  _j   t> 
00000420    01 2f a7 d9 66 3b 91 f6  f8 f7 a3 ee f1 ec d3 1d     /  f;          
00000421    b2 3d a4 fb a4 7b 89 f6  69 7e cd 1e c7 1e dd 1e     =   {  i~      
00000422    d5 1e cf 2f d8 23 d7 4f  b3 47 b1 9f 69 96 f1 27       / # O G  i  '
00000423    e2 cf c7 57 c8 d7 b1 4c  f8 e3 f2 07 e5 0f ca 4d       W   L       M
00000424    b7 1c c3 32 f9 23 f2 53  69 a7 10 60 03 c1 80 24       2 # Si  `   $
00000425    00 d0 41 cb 66 00 51 c0  30 01 e0 c0 15 00 70 42      A f Q 0     pB
00000426    ca 5e 3b 7b 42 b1 25 52  a4 ef 94 54 3e fa b2 27     ^;{B %R   T>  '
00000427    ff 80 60 30 02 c0 1a 01  f6 28 2e fe 3d 34 18 00      `0     (. =4  
00000428    f0 60 0b 9a f8 34 0f 22  22 50 40 12 84 89 85 ca     `   4 ""P@     
00000429    bc e0 d2 f1 f9 70 fa 29  2f fc 55 e5 ed 3c e9 f0         p )/ U  <  
00000430    86 af fc f1 c8 b8 24 2a  f3 67 5e 3a 01 c0 c0 0b          $* g^:    
00000431    00 60 07 48 a1 54 af 0d  56 5c 3e 56 5c ac 7c 5f     ` H T  V\>V\ |_
00000432    ef 7c 7c 5e ab e5 c5 fe  b3 9f 55 f9 9f 95 35 71     ||^      U   5q
00000433    80 06 83 00 aa 0c 00 a0  07 7f 07 b4 bd 71 e1 82                 q  
00000434    d0 84 0c 02 b8 30 03 40  1b c8 01 fe 55 56 2f 1d         0 @    UV/ 
00000435    0c 5d a8 90 25 de a8 7c  48 21 8f a7 25 73 e6 81     ]  %  |H!  %s  
00000436    80 6e 08 40 c0 51 03 00  b9 4b b4 0f 80 62 a5 73     n @ Q   K   b s
00000437    e2 42 b6 f4 10 15 ec 02  22 53 4a e2 fe d3 3f e0     B      "SJ   ? 
00000438    c0 2e c1 f0 06 83 00 2a  aa f9 42 b2 f8 3f 2e 56     .     *  B  ?.V
00000439    10 3f 3b 7b 2c fc f2 fe  2f f3 71 34 a6 13 c1 80     ?;{,   / q4    
00000440    56 06 00 84 18 00 e0 60  1b 01 80 11 12 c1 04 18    V      `        
00000441    01 0c 2f 04 10 81 28 43  2e 85 c5 f3 e0 87 e9 34      /   (C.      4
00000442    7e 5d 19 9f be ef 93 cc  33 3c 0c 00 88 41 f6 09    ~]      3<   A  
00000443    03 e2 e1 f7 e8 43 aa ff  44 8f 04 35 73 e5 ca b9         C  D  5s   
00000444    94 be 17 17 4f 51 e8 8f  20 f6 8e a6 e1 b4 30 60        OQ        0`
00000445    03 84 90 60 3c c2 10 f9  5f e7 e0 41 04 0c f1 7e       `<   _  A   ~
00000446    f1 4f a4 df 2b a5 36 33  1f 56 0c 03 90 94 07 87     O  + 63 V      
00000447    f0 bc bb 33 b6 59 79 8b  50 28 30 77 90 8e 0c 01       3 Yy P(0w    
00000448    b8 30 0c ca 81 80 64 06  00 62 82 00 06 e1 78 06     0    d  b    x 
00000449    83 00 1f ef 04 20 87 3f  da 24 84 0b 30 21 09 29           ? $  0! )
00000450    ed 90 0f 2b ad 81 f5 4a  5b 0c ec 01 80 67 06 00       +   J[    g  
00000451    48 20 83 00 1e 24 c1 f8  96 ab d0 b8 03 ef ec f8    H    $          
00000452    fa e9 74 2f 6e fa 33 e4  94 67 5c 0c 00 f8 30 0c      t/n 3  g\   0 
00000453    e2 58 30 03 a0 c0 36 89  40 c0 39 0f bb ec a3 e0     X0   6 @ 9     
00000454    86 3e 60 7e 10 95 e2 9f  25 f5 c2 f5 4b 7f ce 35     >`~    %   K  5
00000455    04 10 84 a8 18 06 c0 60  03 bf ef 59 4b e0 ec 7d           `   YK  }
00000456    f5 87 fe 2d 93 f0 74 fa  50 80 25 a8 be 1f c6 af       -  t P %     
00000457    3f 06 2f 28 08 40 c0 11  03 00 2a 10 82 0c 04 18    ? /( @    *     
00000458    24 01 f1 27 d7 47 df 12  e8 f6 0f aa ea a6 62 95    $  ' G        b 
00000459    bf 76 c3 01 97 e7 91 44  80 60 19 41 80 eb fa af     v     D ` A    
00000460    82 00 07 c1 f5 b9 68 f9  58 b8 68 4a 85 ea e1 75          h X hJ   u
00000461    2f ff 55 5b 55 e3 5f 26  74 12 cb 95 fa 82 8e db    / U[U _&t       
00000462    f9 cb 3f b6 cc e6 10 46  17 2b f5 e1 77 a4 8d f0      ?    F +  w   
00000463    e3 c2 8f 96 83 00 d2 0c  03 39 70 20 5f 89 5e 8a             9p _ ^ 
00000464    be af de 52 5f f5 73 62  a9 9a d3 37 0e 86 4c 7b       R_ sb   7  L{
00000465    40 30 0e 21 04 18 0f 21  2c 20 84 31 e5 aa 82 1d    @0 !   !,  1    
00000466    12 55 4f 4b d1 e6 ab 12  26 5b 93 72 00 4b 78 20     UOK    &[ r Kx 
00000467    0f bc ad 5d fa b9 aa 95  28 d0 27 47 11 f4 60 c0       ]    ( 'G  ` 
00000468    30 00 78 fc 20 00 70 fa  0f 95 d2 ea 3f 12 e7 95    0 x   p     ?   
00000469    ff dc f8 1e 92 db 25 63  49 de bf e0 60 19 84 b1          %cI   `   
00000470    26 40 86 24 04 38 ac bb  f6 f2 4c 9a b7 fd 9b 29    &@ $ 8    L    )
00000471    1b f0 30 0b 61 06 04 12  e8 5f 99 6c ec 4f 1d 04      0 a    _ l O  
00000472    0c 00 98 30 01 c0 1a 0c  00 7c 08 25 c3 f5 5a 24       0     | %  Z$
00000473    2b f7 8b e8 96 da a2 f1  fc b8 a8 10 aa de f5 fc    +               
00000474    be e5 6e fc c4 78 30 0c  df d1 21 c9 40 19 07 d4      n  x0   ! @   
00000475    10 60 21 4a 07 a7 ec f6  b6 7b 28 7d 85 c0 c0 39     `!J     {(}   9
00000476    4f 09 02 4d 1f 89 42 59  74 57 47 77 f9 6c dc ba    O  M  BYtWGw l  
00000477    22 c4 3d c7 bf 23 1b 4a  2f 3a 06 00 b0 21 7c 18    " =  # J/:   !| 
00000478    0a 40 60 06 2c 9b a1 01  8b 97 28 fc bd f8 de ed     @` ,     (     
00000479    e7 4e a3 83 00 4a 0c 02  c0 30 03 60 1c 0c 00 e0     N   J   0 `    
00000480    20 00 70 92 01 f4 48 1f  83 00 1e 08 1e f0 97 0b      p   H         
00000481    cb 95 c9 ff cd bf 82 57  f2 f8 7f 75 88 f4 40 60           W   u  @`
00000482    17 d4 ab 12 04 81 f0 95  14 97 d1 27 47 93 bf b1               'G   
00000483    47 fd b8 95 f3 e0 1d 55  42 fb 55 dc f4 55 7a 9c    G      UB U  Uz 
00000484    e3 cc 41 80 65 06 01 a8  21 03 00 3a 01 df 08 60      A e   !  :   `
00000485    1f 55 ea b0 38 10 04 a9  6a 91 f9 7e 31 d1 19 53     U  8   j  ~1  S
00000486    6a eb 13 23 75 d3 20 c0  35 83 00 b4 0c 03 90 20    j  #u   5       
00000487    a9 8a bf 44 82 e5 7b 7e  a2 82 8f fa dd 56 ad b4       D  {~     V  
00000488    4e a0 06 01 7c 18 06 31  24 18 06 30 60 1c 0b c1    N   |  1$  0`   
00000489    04 7f aa 84 af 69 70 06  09 0c 09 42 44 bc 8b 01         ip    BD   
00000490    f2 f1 1c be b3 fe 35 1d  7e ab ca c4 95 6a 0b a5          5 ~    j  
00000491    57 72 fe e3 5e f7 9b b6  b1 3c 64 e8 18 01 10 60    Wr  ^    <d    `
00000492    04 07 c0 80 3e f5 2f 51  42 0d 8a 4b fe b2 65 49        > /QB  K  eI
00000493    bc d6 c6 df 52 24 ab 82  5a bb f6 8b bd aa 34 ad        R$  Z     4 
00000494    2b df 72 a9 51 7a b1 f0  f4 b9 52 b5 79 b0 bd 44    + r Qz    R y  D
00000495    bf cb bb 2a bc de 11 df  2a dd f1 77 a5 56 af ff       *    *  w V  
00000496    c9 e9 db 7f 5a 8e ba 08  43 e2 f0 43 57 2e 8f 15        Z   C  CW.  
00000497    49 55 c5 5f b8 af 6c 8a  a5 be bd f5 dd 48 e9 b2    IU _  l      H  
00000498    e1 f2 af 55 4a e2 a9 e1  f9 76 a8 2f f7 a4 b2 d9       UJ    v /    
00000499    95 a6 e6 20 30 6a 10 41  80 71 f0 40 1f c2 e5 0a        0j A q @    
00000500    77 db 68 29 b5 a4 ab 98  a9 85 db 2d 94 89 f3 e0    w h)       -    
00000501    c0 08 04 20 60 1c 81 80  6e 06 01 c4 10 01 80 14        `   n       
00000502    2f fa 95 42 48 fe 7b ff  b3 c3 f5 62 40 f9 54 bf    /  BH {    b@ T 
00000503    12 ef 8b fb 66 2a f7 f9  4c 93 17 03 00 2e 0c 00        f*  L    .  
00000504    9a a0 40 54 08 02 50 91  04 8f 2a f4 8a bf 20 28      @T  P   *    (
00000505    fd 57 b6 f7 f6 88 a7 de  81 80 23 f8 30 0d 60 c0     W        # 0 ` 
00000506    07 03 00 e0 01 f0 18 01  41 20 b8 18 07 11 24 49            A     $I
00000507    12 a9 78 41 54 24 c1 e0  21 2a 2e fe e7 a8 1f f0      xAT$  !*.     
00000508    28 ff ff e3 64 04 00 82  0c 03 68 20 03 00 22 3e    (   d     h   ">
00000509    12 0b c1 80 72 f8 21 51  26 4c 2e f7 aa b5 73 f9        r !Q&L.   s 
00000510    2a be 7b b8 aa f2 bd 7c  18 06 90 0d a0 1e 10 cb    * {    |        
00000511    bc 3e ba a9 57 87 52 6f  be 3d bd fa bf c2 c8 49     >  W Ro =     I
00000512    30 01 82 52 a8 25 2b 05  0f bd 47 bc 56 a6 4b ef    0  R %+   G V K 
00000513    f9 a5 04 af 36 1f 83 00  48 10 95 8e 84 90 87 55        6   H      U
00000514    aa b0 0e 4f 7a 78 7e a6  ad 6d b3 08 1c 81 80 6a       Ozx~  m     j
00000515    06 00 50 bc 18 01 31 26  aa da 5e 25 5c aa 87 f6      P   1&  ^%\   
00000516    65 ca c2 b9 1a b2 a2 79  6c 06 01 8c 18 01 71 2d    e      yl     q-
00000517    5c 04 20 60 1c 84 a9 f1  e7 f2 17 e9 72 b6 39 07    \  `        r 9 
00000518    be 83 15 50 60 18 01 80  68 04 01 f0 30 0e 4a 84       P`   h   0 J 
00000519    a1 25 58 e9 58 40 2e aa  78 ad 56 7e fe 76 cb 49     %X X@. x V~ v I
00000520    9a 42 02 a0 86 3f 12 d4  55 34 75 44 7b 23 43 2a     B   ?  U4uD{#C*
00000521    00 0d f1 70 20 00 75 a0  c0 08 84 22 e5 20 a1 55       p  u    "   U
00000522    68 f8 ba 62 9f 50 36 9e  71 a2 67 c3 9f 85 3e f2    h  b P6 q g   > 
00000523    08 20 81 ef 7a 04 30 81  f5 22 55 cb 64 bb 92 b3        z 0  "U d   
00000524    ea cf 86 11 a7 d8 af af  ae 3e 8c 03 80 3c b8 03             >   <  
00000525    4b 87 aa b6 65 50 ad 57  f9 f2 ef ac ff 05 1e fd    K   eP W        
00000526    1f 15 1f 0d 1e c9 33 de  e3 de 23 e0 e3 df e3 d9          3   #     
00000527    66 7b a4 7b 8c 7b e4 7b  c4 7b 2c c7 6d 8f 6a 3e    f{ { { { {, m j>
00000528    eb 1e e2 7d 9e 5f c9 d7  cb 1f 98 af 9a af 61 99       } _        a 
00000529    f3 07 e6 8f ce 9f a0 af  60 99 7c 08 30 01 e0 c0            ` | 0   
00000530    2f 83 00 dc 0c 00 8d 80  84 3f 52 a8 14 c5 fd 72    /        ?R    r
00000531    58 30 01 c0 c0 21 84 05  79 e0 60 1c 44 8f 89 15    X0   !  y ` D   
00000532    92 f2 e1 da a1 6d 15 0c  e9 00 25 8f e5 ca 95 b7         m    %     
00000533    23 e6 27 ef 9a 7b c6 40  30 18 06 d0 60 03 81 80    # '  { @0   `   
00000534    72 f6 80 70 41 1f 0f d3  4f c5 28 fc 6c 5c 03 40    r  pA   O ( l\ @
00000535    38 48 04 12 ea 3e 12 7d  be 4d 72 ba 93 c3 f5 65    8H   > } Mr    e
00000536    d6 2b 1f 2b f9 77 e3 5f  55 fb ea 85 cf c0 c0 07     + + w _U       
00000537    03 00 9a 0c 03 40 20 67  8b cb fc 08 23 e8 22 04         @ g    # " 
00000538    30 37 89 8b fd d4 27 da  fe 01 91 b5 57 08 e4 60    07    '     W  `
00000539    43 a2 2c 3e f5 60 60 1b  c1 80 38 9f bf 56 0c 00    C ,> ``   8  V  
00000540    70 07 d9 58 2e 12 4a 3e  e8 e0 60 1c 81 80 1c 06    p  X. J>  `     
00000541    01 b0 18 01 0d a1 0c 03  0b c7 93 80 7c ba 01 f2                |   
00000542    e4 3e dc 21 4d 04 00 60  11 81 80 13 00 fc f0 06     > !M  `        
00000543    c2 f0 60 03 cb d7 56 0a  31 29 11 7a ae c0 27 e7      `   V 1) z  ' 
00000544    13 17 04 31 20 20 97 01  f1 20 21 97 09 25 df 11       1      !  %  
00000545    4b 8b fc 5f ea 82 d1 9b  5c a2 49 73 7d 83 59 3b    K  _    \ Is} Y;
00000546    4b be d1 c7 cf 97 00 68  43 08 57 44 a1 24 7e 3e    K      hC WD $~>
00000547    b0 14 ca fe eb 3f 89 65  e1 00 18 07 21 f0 20 fb         ? e    !   
00000548    b4 7b 15 d6 25 ba 07 7c  98 7b 2b cf 81 04 18 02     {  %  | {+     
00000549    00 60 05 81 80 70 da 3f  08 5e f1 7d e8 21 89 58     `   p ? ^ } ! X
00000550    3e 2f 2c e5 47 1e e0 0c  03 80 30 07 62 5e 7e 83    >/, G     0 b^~ 
00000551    00 20 01 ea 84 b9 97 04  80 83 14 2b 4d 21 7c d5               +M!| 
00000552    56 ac a6 06 4b 60 c0 07  03 00 e3 02 10 97 3f f9    V   K`        ? 
00000553    9e a0 70 be 37 2a aa da  03 73 63 f1 f2 9f 56 9e      p 7*   sc   V 
00000554    f8 20 60 1a c2 08 07 03  00 d6 0c 00 a4 54 3e cf      `          T> 
00000555    97 97 97 8f 81 41 28 1b  54 3f 5b 18 b9 2e ea c7         A( T?[  .  
00000556    f2 3f 29 77 ea a5 73 3e  aa df 7e 2d 5d 3e 0c 00     ?)w  s>  ~-]>  
00000557    88 96 5f e1 25 50 30 01  e0 84 5c 10 80 3c bc 7f      _ %P0   \  <  
00000558    47 d4 be 4b 47 e2 56 6f  bf ea 0a 1b ff 7b ff be    G  KG Vo     {  
00000559    b6 a8 8d 63 a3 c1 80 65  06 00 a0 4b 1f 7c b8 10       c   e   K |  
00000560    61 7c 08 7e 08 63 f2 e1  28 bf f5 55 95 50 fc be    a| ~ c  (  U P  
00000561    e2 aa 5e a8 ba 5d f7 ef  07 b2 71 98 64 d5 50 96      ^  ]    q d P 
00000562    25 02 02 b8 aa d9 e2 e1  27 55 97 56 e5 5a a9 63    %       'U V Z c
00000563    4c 59 97 50 50 5f fa 81  a9 2d 1b eb de e2 0c 01    LY PP_   -      
00000564    e0 20 82 00 41 12 15 82  00 20 aa 55 02 18 07 7e        A      U   ~
00000565    7c 03 82 1d e0 1f 12 8b  ef 44 a1 eb 57 e3 ef ee    |        D  W   
00000566    46 4b bf 21 81 00 0e 55  e1 e5 b3 ae 8a 06 00 44    FK !   U       D
00000567    18 01 a0 60 1b 41 00 4b  8a e0 07 97 04 09 2b 65       ` A K      +e
00000568    d7 e5 f5 5a 61 e4 51 30  0a d8 44 9b 00 34 03 55       Za Q0  D  4 U
00000569    2a 1e 02 84 0f 73 b6 73  f5 14 34 b0 0c 03 58 94    *    s s  4   X 
00000570    10 02 18 93 f1 f8 90 3f  03 79 55 c9 eb 8a 59 b3           ? yU   Y 
00000571    57 33 69 01 80 61 2e 08  57 3d e8 24 ad 26 aa c4    W3i  a. W= $ &  
00000572    bf a5 0e 78 88 41 08 20  1c 10 bd 41 0c 7e a9 52       x A     A ~ R
00000573    6d 1d a0 36 be a9 50 94  3e b4 4a 92 0f d5 76 2a    m  6  P > J   v*
00000574    db 7d fd bf b6 58 65 55  56 17 97 81 dd 57 6b 3f     }   XeUV    Wk?
00000575    7a fd 1f fc 18 00 e0 60  1c 20 fc 0f 0f 47 9f 8a    z      `     G  
00000576    e7 98 b1 ec 62 50 fa 09  5b e9 6d bf 6f 6f 2d 86        bP  [ m oo- 
00000577    b3 01 80 71 06 01 b4 4b  06 01 bc 0f 51 2e 2a 54       q   K    Q.*T
00000578    24 17 0f c0 d1 74 ef a5  6e df de db 89 1c f6 00    $    t  n       
00000579    87 80 a0 b2 79 5f ac fd  9d bb 2e 2d c7 47 83 00        y_    .- G  
00000580    d6 5c ad 5c 9e cd 55 8d  92 a8 80 78 30 0d 0a 81     \ \  U    x0   
00000581    80 6c f5 08 5f 06 00 58  7e 0c 00 70 96 5c af d4     l  _  X~  p \  
00000582    10 f4 b9 58 42 57 3f cc  2e 2f 55 3c a7 5b 83 a6       XBW? ./U< [  
00000583    a1 d6 b1 2c 10 01 0a d9  7d be 7a a8 40 12 fe 10       ,    } z @   
00000584    15 cf fe 2a 52 b7 95 c5  7e b2 ac fd 31 2c 10 80       *R   ~   1,  
00000585    30 03 ef c1 03 f4 49 12  a7 bd ff 5a dc 9f 9d f5    0     I    Z    
00000586    22 7d 61 70 20 0f 21 7d  93 2f bd 27 80 a9 7f d4    "}ap  !} / '    
00000587    0b 67 c1 80 6b 08 5f 08  00 84 01 cd 82 11 72 a1     g  k _       r 
00000588    ed 03 23 fc 94 be 81 1b  91 34 73 60 20 03 00 22      #      4s`   "
00000589    0c 00 70 07 fe 97 17 ab  82 55 bf 55 9a d7 c7 9f      p      U U    
00000590    9f cd 2c c7 ad 03 00 d0  0c 01 39 78 41 1f d1 24      ,       9xA  $
00000591    48 cf cf dd c6 5c 9b 41  04 7c 0c 07 81 70 92 3f    H    \ A |   p ?
00000592    d9 e9 ec 96 ce 4b 1d 64  01 c2 4c 00 f0 0e f8 28         K d  L    (
00000593    2c aa fd 40 e9 76 c1 ec  44 cc 4e e7 bb 03 00 7a    ,  @ v  D N    z
00000594    0c 00 e0 90 08 22 58 43  06 01 bc 10 44 88 24 97         "XC    D $ 
00000595    00 72 b5 42 40 42 2e 57  27 d5 02 85 5c 52 ac 7f     r B@B.W'   \R  
00000596    ff 5e 50 42 f2 89 ed d9  aa f4 c1 d8 30 0d c0 c0     ^PB        0   
00000597    19 03 00 20 0c 03 88 93  42 18 95 41 80 90 12 fe            B  A    
00000598    5c 08 5f f8 20 17 0f 95  78 bf db a0 a3 2e 50 a3    \ _     x    .P 
00000599    f4 0b cc b7 9b 6e 38 e0  18 02 f0 60 04 47 e0 c0         n8    ` G  
00000600    09 03 00 1c 24 89 03 f9  9f f8 06 ab 12 d1 55 45        $         UE
00000601    de 83 00 be f8 c3 40 30  0e 00 82 ad 58 30 0e 03          @0    X0  
00000602    f1 2c 7c 3f b3 33 c3 ef  fa e6 2a 92 1a 92 00 e1     ,|? 3    *     
00000603    2b d4 03 95 42 f2 e6 f6  fe 58 c9 f7 aa 84 20 60    +   B    X     `
00000604    0f 41 80 11 08 41 08 bc  b8 18 0f 50 82 10 c2 0d     A   A     P    
00000605    fc d1 26 45 5e 55 e0 42  b6 fb c0 85 27 f8 c2 b1      &E^U B    '   
00000606    d8 d1 72 03 00 42 07 87  ea ad 96 aa 8a e5 b3 1b      r  B          
00000607    9e bb 84 23 df fd f5 a5  fe f6 a9 bf 9f ee 28 96       #          ( 
00000608    5e e1 92 f0 0c 1f 97 89  73 ea fe 10 04 91 23 2d    ^       s     #-
00000609    f2 a0 43 8a 00 fe 5c d5  52 f5 45 5e c9 16 3e dd      C   \ R E^  > 
00000610    02 18 fd 5d fa b1 f8 97  ff db 9b 55 ff 6b e4 8b       ]       U k  
00000611    c7 d2 28 f1 71 72 07 3d  14 18 02 f2 e2 e5 42 48      ( qr =      BH
00000612    30 0c e5 c5 f9 00 36 00  79 75 57 d9 b4 b9 5a ab    0     6 yuW   Z 
00000613    55 62 35 36 90 32 83 00  ad f5 63 f0 60 1a 04 9f    Ub56 2    c `   
00000614    51 ee 09 4a 87 c3 e2 e5  a4 1e 45 77 13 5f fe bc    Q  J      Ew _  
00000615    98 03 81 80 1c 00 e5 40  83 41 80 6c 82 50 1e 00           @ A l P  
00000616    d5 73 3e ac 7a ae 2a fa  7f 8f b7 3f f8 c7 f1 52     s> z *    ?   R
00000617    47 97 03 00 d6 0c 00 d8  42 06 01 bc 20 c1 ff 80    G       B       
00000618    3e 97 04 3b ef 2a b2 17  42 f5 5f 1e 35 ff f6 59    >  ; *  B _ 5  Y
00000619    ce 1b 72 82 5f a0 97 55  f6 cb ec 7c 70 21 7e 8f      r _  U   |p!~ 
00000620    bd ee 4b bc 73 e3 a1 70  1d 56 ac 7a a4 7b 50 49      K s  p V z {PI
00000621    24 9e 84 51 be 85 fb f1  ec 96 5b 64 9c ff a1 73    $  Q      [d   s
00000622    13 2a 6e d2 56 20 0d 06  00 80 bc 18 01 0b ff 2b     *n V          +
00000623    12 c0 38 be 4b 6c b3 f5  55 cf ed 9b ef c6 d3 4f      8 Kl  U      O
00000624    be 2c 14 20 18 aa 01 ef  84 11 2b c2 5c 96 28 a3     ,        + \ ( 
00000625    ff 6a 98 8d 97 b1 2a d2  f8 a7 d2 d8 45 33 f0 39     j    *     E3 9
00000626    72 d9 1e f8 b1 28 10 4b  c7 c3 ef ce 17 0f ae c0    r    ( K        
00000627    50 5f 49 ef 78 a2 d3 6e  23 e0 60 09 81 80 69 08    P_I x  n# `   i 
00000628    60 1c 3e 00 d0 84 10 bd  ff 09 7e 1e f8 7c 0a 19    ` >       ~  |  
00000629    07 f1 55 6a d8 5e a1 b9  d9 d7 c5 8f fd 41 40 3d      Uj ^       A@=
00000630    03 70 75 7e d5 df 6c 3e  30 08 01 0c 4a 12 c4 80     pu~  l>0   J   
00000631    85 44 aa 3e cf 89 03 ef  c2 e8 5f e5 3e f5 f4 c9     D >      _ >   
00000632    9a c1 a6 81 26 97 fa 41  e4 67 bc 3f 2a 25 78 7d        &  A g ?*%x}
00000633    6f d4 e4 41 0f be 08 49  06 00 48 18 06 e0 0e 06    o  A   I  H     
00000634    07 e0 18 00 e0 0f 1f 63  34 4a fd f0 11 f4 be 19           c4J      
00000635    c7 80 48 52 9c 6e 57 0a  89 20 c0 08 03 00 22 01      HR nW       " 
00000636    bc 08 60 80 01 c3 f4 ea  c7 ca 94 81 1c 9f 19 b9      `             
00000637    40 0d 57 1b f4 a3 28 da  10 3d 5a f8 bd f1 70 33    @ W   (  =Z   p3
00000638    87 81 99 80 92 0c 01 88  30 03 62 5f 4b 87 c3 f0            0 b_K   
00000639    81 57 12 07 f7 00 81 78  f5 cc 80 c0 32 03 00 3a     W     x    2  :
00000640    10 01 80 18 f8 95 3c 01  a5 e2 5d 1f ed 03 df 55          <   ]    U
00000641    25 9e 8c d5 7e a2 e6 b5  42 4f 8b ea 89 2c 7c 9a    %   ~   BO   ,| 
00000642    b1 2a aa 9b b2 bb c2 47  c0 c7 c1 d8 5d b3 d2 dd     *     G    ]   
00000643    6d 7c 74 10 f3 2c cf cb  a4 0c 93 1d f6 3d e6 3f    m|t  ,       = ?
00000644    fd 42 b8 63 fc fb 2c c4  e7 67 e7 e8 98 66 67 43     B c  ,  g   fgC
00000645    49 49 4e c1 33 3a 7a 9a  aa c5 f9 ac d5 d7 57 d7    IIN 3:z       W 
00000646    d9 57 af 4d 42 b0 ad 12  02 18 96 01 ca c0 f8 40     W MB          @
00000647    12 c7 e1 08 bc 0c 8f 95  d2 ea 8f 1b 49 4f da 57                IO W
00000648    db af 4d a8 81 06 00 38  18 04 90 60 1b 82 16 78      M    8   `   x
00000649    03 4b 84 a0 60 1c 95 46  fe 0c 0c 38 43 02 2a a5     K  `  F   8C * 
00000650    2e fd 02 a3 e2 15 00 40  06 01 44 18 01 90 41 cf    .      @  D   A 
00000651    7c 49 a0 80 3f a0 a4 04  18 3c f6 a6 12 66 89 48    |I  ?    <   f H
00000652    7e a8 8a e6 f2 bd 82 aa  55 4c 4d bd ec 18 05 60    ~       ULM    `
00000653    60 06 c2 0d 06 01 bc 48  06 01 a0 18 01 6f 02 1c    `      H     o  
00000654    12 6a aa 3f a2 55 9f 83  e0 83 eb 2f 84 9d 97 01     j ? U     /    
00000655    08 4b 80 6f f1 a5 72 e8  34 3f aa 05 c5 e2 49 78     K o  r 4?    Ix
00000656    94 10 07 c1 06 41 29 5a  ba aa 09 6a 2e fa 5b cb         A)Z   j. [ 
00000657    65 e6 c8 fa a0 60 04 c1  80 61 06 01 c8 18 01 95    e    `   a      
00000658    76 00 67 fd f2 e5 76 52  f0 84 3f f0 f4 4a 56 89    v g   vR  ?  JV 
00000659    b5 5f 4e aa c4 e7 dc 04  a0 60 12 c1 80 14 04 11     _N      `      
00000660    f0 21 03 00 e2 01 df 06  00 38 be e6 09 23 e5 50     !       8   # P
00000661    4b 1f 4a b8 1c 90 be 45  f5 55 23 58 56 25 8f e8    K J    E U#XV%  
00000662    94 3e 12 ae 7d 50 fb 47  96 f0 be 4a c6 ad 0c 55     >  }P G   J   U
00000663    82 08 40 05 0a a2 f9 04  7b eb 8b 5a de 56 73 f9      @     {  Z Vs 
00000664    8f 79 b8 06 83 00 2e a8  7e 01 c3 e5 41 00 78 07     y    . ~   A x 
00000665    fd 3a af fc a2 5e b3 6c  e4 ba 69 53 e0 1c 5e 01     :   ^ l  iS  ^ 
00000666    c5 e2 40 30 01 c1 0e fa  84 3f f6 ab f7 f7 3d f0      @0     ?    = 
00000667    35 2b 1a 65 73 c0 c0 33  89 00 82 24 17 04 21 20    5+ es  3   $  ! 
00000668    18 07 10 0e 2f 85 c2 30  1e 12 61 71 7f e3 13 ea        /  0  aq    
00000669    95 d5 4a f8 44 be 10 c1  0e d1 20 bf 15 cf 97 2a      J D          *
00000670    9e aa ee ff 6f ae fb f1  ec 65 ca be a9 b5 7b df        o    e    { 
00000671    f1 2b 3a 6f 28 18 06 ef  d1 fd 9f 06 02 4a 84 25     +:o(        J %
00000672    71 50 fb d5 5c 1e e5 1e  4c be 9e 9a 5c a1 89 5b    qP  \   L   \  [
00000673    3c f5 10 0d 00 c1 2c 10  24 08 6a bf 4b d5 7a c5    <     , $ j K z 
00000674    32 98 52 57 f2 e0 0f 9e  80 85 ff 8f f3 47 57 2b    2 RW         GW+
00000675    92 04 b0 60 1b d5 09 20  c0 07 fd 57 e6 0f ff 27       `       W   '
00000676    aa cd c3 ab 41 08 4b 81  00 18 07 0b 3f 75 55 aa        A K     ?uU 
00000677    2f e7 66 59 31 ec 32 7f  3a dc 35 b4 3f f8 94 07    / fY1 2 : 5 ?   
00000678    fd 44 a0 38 5c 5f ef 41  2f cb 5d ef 1e f6 1f fe     D 8\_ A/ ]     
00000679    aa 57 0b ac 56 5d 67 7d  3e 2e 52 54 3f f1 7f 87     W  V]g}>.RT?   
00000680    82 5e 25 7a 40 30 0d 42  51 72 b8 10 bf 3b 61 78     ^%z@0 BQr   ;ax
00000681    ea c6 ef b3 5e a4 0c 00  9f 8b 80 3c 10 e1 7c 96        ^      <  | 
00000682    cc c8 48 c1 bb e3 fb 43  c8 a8 20 c1 f1 72 b8 a3      H    C     r  
00000683    f9 8a ef 60 ee 19 7a f8  43 04 0a a8 0e 97 77 cb       `  z C     w 
00000684    b9 bc 7e 10 41 80 69 12  a0 f4 18 00 e0 60 1c 84      ~ A i      `  
00000685    b9 63 61 08 be c1 f9 75  02 90 be 55 15 31 a4 e0     ca    u   U 1  
00000686    42 f8 07 02 0d 12 bd aa  c2 11 79 75 57 e2 e6 ec    B         yuW   
00000687    b0 99 5b e0 c0 07 89 65  e3 e1 2c 03 c7 83 cf 17      [    e  ,     
00000688    5e cb 32 a2 5c f2 e2 a1  24 10 ac fa 98 c6 6e 5d    ^ 2 \   $     n]
00000689    d6 2c 33 ae a8 48 00 f8  3f fd 08 5e 56 08 62 52     ,3  H  ?  ^V bR
00000690    91 ed 2e 94 79 9e 8a 6d  49 86 1e ea 10 01 80 42      . y  mI      B
00000691    06 01 ac 10 3f 80 1e 25  04 01 27 39 ff 7d 57 c4        ?  %  '9 }W 
00000692    a2 f4 3e b8 0d 28 1a 18  30 03 60 c0 21 83 00 22      >  (  0 ` !  "
00000693    0c 03 30 fc 18 08 d1 f8  fb e0 c0 07 00 6a b9 41      0          j A
00000694    41 15 fe 7c 7c 25 dc 03  4a 8b f9 8a d6 f7 e4 d3    A  ||%  J       
00000695    11 40 c0 32 83 00 1c a8  03 c4 82 f1 fa af 60 41     @ 2          `A
00000696    12 a5 ba d0 f7 de 6f d5  7e 77 cf c7 12 8b d5 97          o ~w      
00000697    09 63 e2 f2 eb 61 79 71  7a a5 6a ac 6e 7f df 96     c   ayqz j n   
00000698    0c d0 55 00 70 30 01 e0  c0 37 89 0a ea b0 84 5c      U p0   7     \
00000699    3e 12 e7 2a b5 51 45 f5  cd 9f 96 37 13 3e c4 18    >  * QE    7 >  
00000700    01 40 60 1c 47 e2 59 72  91 ef e2 b2 eb 8a cb b7     @` G Yr        
00000701    7c 9a c9 58 a8 5c f2 18  ad 52 d5 e3 20 c0 7c 0f    |  X \   R    | 
00000702    bd 73 c0 74 bd 37 68 a9  14 10 41 80 65 12 01 80     s t 7h   A e   
00000703    0e 00 ea 25 8f fa ac 79  3f 0b a5 1d fd 5d f6 a8       %   y?    ]  
00000704    91 2b 2d 3a 5c 18 07 00  60 16 41 80 6b 06 01 c1     +-:\   ` A k   
00000705    4a aa 3f 80 18 10 be d8  42 00 ed 2e d8 b0 fc 7f    J ?     B  .    
00000706    a0 86 5e 9e 6c 54 0d 18  1b 7c 08 0a 6d d3 12 14      ^ lT   |  m   
00000707    49 c6 bb 5e f1 41 ea 98  cf f5 80 52 44 1c 78 af    I  ^ A     RD x 
00000708    e8 92 3e 12 95 d0 80 5c  3e f8 93 83 a8 25 ff e5      >    \>    %  
00000709    ca ad 02 f4 b9 57 ff 91  68 ea c1 20 18 07 10 41         W  h      A
00000710    08 16 50 84 01 a0 1e 24  63 40 84 10 c7 ea a2 09      P    $c@      
00000711    fc 19 45 09 00 c0 24 83  00 bc 24 64 a0 83 e0 87      E   $   $d    
00000712    e8 c0 41 12 07 ca f5 2f  fd 15 2b 46 a4 fb 6d 00      A    /  +F  m 
00000713    c0 31 f4 2e 92 81 0e b7  f9 a9 29 e7 b6 83 00 c4     1 .      )     
00000714    0c 00 c8 30 0a 40 83 f2  ff ab f0 94 3f 2e 2f b7       0 @      ?./ 
00000715    f7 c0 c0 08 0f f3 53 7c  10 ef 84 95 71 32 9b ec          S|    q2  
00000716    6b ce b7 08 20 c0 07 89  60 80 5f 4b 82 08 42 2f    k       ` _K  B/
00000717    08 45 f2 f7 c3 f5 57 c5  66 db c4 a0 70 1e ec 49     E    W f   p  I
00000718    e4 08 20 c0 09 0f 95 af  44 81 27 e5 d5 17 8b fc            D '     
00000719    ae 0c de 81 04 18 05 40  60 04 81 80 72 12 ed 12           @`   r   
00000720    c1 80 11 08 7f 1f 2b cd  f1 7c 08 45 f2 73 b9 2f          +  | E s /
00000721    fa 32 62 12 00 fa ae 2b  c5 18 62 64 7e a2 f7 d2     2b    +  bd~   
00000722    ee 99 7d 98 95 6c 6a d8  91 f4 d0 bd 5e ca 3d 57      }  lj     ^ =W
00000723    be 99 ea a6 cb e8 b4 b2  ec 73 88 42 06 02 ec 10             s B    
00000724    7d 21 7a 90 40 2f 5e c1  24 20 75 2f 80 f8 34 00    }!z @/^ $ u/  4 
00000725    cc a0 c0 58 83 00 2c 08  3d 06 00 50 18 06 4f 7d       X  , =  P  O}
00000726    6f 7e 89 01 0e 21 2e fa  a5 63 26 2f 8f f2 f5 52    o~   !.  c&/   R
00000727    b4 b2 1f 9b 54 a9 3b 9f  1a 10 01 80 60 06 01 9c        T ;     `   
00000728    03 b0 14 00 c0 37 00 7c  90 44 56 5e 0a 1f 20 1f         7 | DV^    
00000729    80 4b 58 42 06 01 10 18  06 a5 5c 08 63 e0 0c 00     KXB      \ c   
00000730    f8 bd 55 47 ca d0 aa b8  32 74 12 41 80 39 06 01      UG    2t A 9  
00000731    98 4a e8 95 40 38 03 e0  16 2f a3 ef a0 1f c9 06     J  @8   /      
00000732    70 c0 c0 53 03 00 c3 34  18 11 31 2f 8b 7c 7d 50    p  S   4  1/ |}P
00000733    62 b3 ac d5 58 95 f6 ef  80 9c 16 cb 4f 17 c6 a5    b   X       O   
00000734    20 7a 60 40 06 01 08 10  3f c0 82 08 01 0c 48 4e     z`@    ?     HN
00000735    25 2b f0 ff c8 2a 87 b2  03 00 ca 0c 03 80 42 06    %+   *        B 
00000736    01 a4 03 fc 24 80 61 7d  2f 1f 78 7d ef 62 bf 5f        $ a}/ x} b _
00000737    7d 57 fb b2 c7 ca 89 60  c0 07 03 00 e2 10 38 24    }W     `      8$
00000738    82 00 07 8f b1 ba ac 48  2f 91 0e 1e 9c f0 ff fe           H/       
00000739    11 bf 7d 53 3d 9f ca 84  ad 52 ab 35 33 e5 6f cb      }S=    R 53 o 
00000740    f3 3f a4 3d fc 78 3b a3  ae ce 57 c0 c5 51 4d ff     ? = x;   W  QM 
00000741    77 90 69 22 a8 18 07 21  25 53 33 fa bb a1 cb 81    w i"   !%S3     
00000742    80 70 9d 1e 8f b7 6e f1  ec 73 0f 7f 9e fc fd fe     p    n  s      
00000743    ec 60 91 40 18 01 d0 60  07 8b 81 80 6e 04 05 61     ` @   `    n  a
00000744    0c 20 fd 5f 87 bf aa 84  ac 62 7b 97 e3 57 ff 00       _     b{  W  
00000745    78 07 7e 5d b7 d6 eb 6e  4c 06 00 70 18 05 c0 60    x ~]   nL  p   `
00000746    04 01 80 0e 06 00 40 b8  21 04 1f fe 7c b8 bf fe          @ !   |   
00000747    83 c9 bc b7 67 c8 9a 47  f1 5d 54 07 1b 3f 26 5d        g  G ]T  ?&]
00000748    7d 28 ec d4 e8 a9 69 aa  18 26 67 4f 53 57 5a c1    }(    i  &gOSWZ 
00000749    33 3a ca fb 0b 26 09 99  d8 da 5a 5b 30 4d 4e da    3:   &    Z[0MN 
00000750    e2 de e5 82 69 dd d6 07  df 7d 5f 8a db d7 5d 38        i    }_   ]8
00000751    f8 78 b9 d3 f8 18 98 f0  33 5b 9e 2d 81 5d 74 3e     x      3[ - ]t>
00000752    00 d0 0d 08 5d 08 41 04  21 89 49 8b 84 b5 7e 88        ] A ! I   ~ 
00000753    37 06 58 27 f0 97 a8 6b  8a 52 8f 81 80 5c 06 01    7 X'   k R   \  
00000754    a8 7c d8 96 5c 10 e2 c1  0c 48 55 fa 04 55 42 e5     |  \    HU  UB 
00000755    63 39 45 40 c0 2a ee 50  86 0c 00 87 80 a8 42 12    c9E@ * P      B 
00000756    ef a8 10 1f c5 70 1a 57  dc 3c 45 ea 82 f0 80 24         p W <E    $
00000757    17 f1 58 f8 bb e9 7c e2  8f a2 50 30 0a e0 c0 09      X   |   P0    
00000758    2b aa 94 97 84 30 85 ff  34 0a 11 2a ff d3 78 d9    +    0  4  *  x 
00000759    75 56 30 6c 04 00 60 1c  00 3a 09 70 7e 01 c0 82    uV0l  `  : p~   
00000760    24 17 dd b7 20 fe 4b 95  8a ab f9 4f 62 63 2f 5a    $     K    Obc/Z
00000761    97 83 00 2e 08 1f 6b df  2e a0 a8 fd 85 94 fb d4       .  k .       
00000762    be 08 2a fe 10 e0 43 cc  be 1f 58 5d 67 60 04 9f      *   C   X]g`  
00000763    00 60 30 0d 3a 25 09 74  03 f3 ff 55 8a 15 02 9e     `0 :% t   U    
00000764    54 a7 2f 87 c0 1e 01 e1  0b a2 50 41 12 47 c0 50    T /       PA G P
00000765    7f fc 41 b5 d5 9e 06 01  84 18 06 b9 32 84 38 5e      A         2 8^
00000766    9d 59 77 b5 0c 3c c1 7c  ab f3 2b ae bc 0c 00 98     Yw  < |  +     
00000767    92 24 17 82 18 07 02 0f  fc af ba 3d 9b aa 66 5d     $         =  f]
00000768    cb d4 da e7 c1 83 00 c6  0c 03 78 94 24 51 27 32              x $Q'2
00000769    0f 67 f1 54 02 e6 31 c4  a0 82 a8 48 2f f1 7e 4b     g T  1    H/ ~K
00000770    ff 5d f4 cd ec 6a e0 b7  1c 21 ab 12 c4 9b 55 97     ]   j   !    U 
00000771    ab b4 ce 8f bd 25 72 b7  84 bf c2 ef 97 78 75 ea         %r      xu 
00000772    de 19 ab 04 00 40 54 10  3d 9d fa bb 75 5f d7 7d         @T =   u_ }
00000773    2f 07 e0 c0 37 80 78 07  fe fe fe f5 bf ab aa 7e    /   7 x        ~
00000774    38 75 f1 fc 4c 45 f9 b5  df 00 4d e1 fc 40 ca fc    8u  LE    M  @  
00000775    fb 03 e6 e4 bb ba be 3f  78 c1 35 8b 67 59 cb 97           ?x 5 gY  
00000776    67 2a f3 8f b9 b7 3e c1  34 f9 d0 80 0c 03 28 30    g*    > 4     (0
00000777    0d 40 19 90 21 03 00 dc  01 aa a4 6b c0 a1 fa b5     @  !      k    
00000778    40 40 bd 4b 9b 81 80 10  07 01 f0 01 80 72 12 a0    @@ K         r  
00000779    f0 18 07 00 84 a8 10 58  04 32 eb 04 81 fa 39 94           X 2    9 
00000780    ba 8c ad 4f d9 9f 62 be  9f 6e 47 cc cb 61 a7 a1       O  b  nG  a  
00000781    84 00 60 11 41 80 1c 12  66 09 34 21 80 75 67 c3      ` A   f 4! ug 
00000782    f1 f0 f7 c0 44 7c 3d 71  a7 be 0c 01 58 30 0d 52        D|=q    X0 R
00000783    d8 0c 04 a0 93 ad f0 7c  5c 3f 2d 1e 17 ba c0 fd           |\?-     
00000784    79 f6 8a 10 07 cc 6c 48  f9 48 25 ab 5e 54 ae 79    y     lH H% ^T y
00000785    f8 96 0c 01 b0 30 03 a2  4c b4 4a a1 0c 21 6b 54         0  L J  !kT
00000786    4a 12 47 be 88 95 a9 72  e8 41 06 00 f4 21 fa 5a    J G    r A   ! Z
00000787    08 00 c0 38 2a 12 d6 2e  12 4b 80 f5 a0 4f ca 60       8*  . K   O `
00000788    ca 80 21 03 00 dc 0c 03  88 07 70 21 02 08 41 12      !       p!  A 
00000789    a2 ca 84 b5 73 c8 34 f4  28 f8 18 05 10 60 18 4b        s 4 (    ` K
00000790    bc a0 18 0f 31 ea 72 f1  f1 77 fe 84 f3 50 28 0b        1 r  w   P( 
00000791    fc 23 09 75 19 d9 30 3f  ea d9 74 44 71 f1 01 0c     # u  0?  tDq   
00000792    18 01 80 60 1a c1 02 d8  24 83 00 1c 10 e6 08 b0       `    $       
00000793    ba ab 54 8b ea b5 d2 cd  5e cc ab 59 8f 73 08 00      T     ^  Y s  
00000794    c0 23 03 00 d0 3e e0 97  02 10 41 80 5d 50 f2 a1     #   >    A ]P  
00000795    55 45 cb 63 e0 60 13 c1  80 14 04 1b 3c 0c 03 78    UE c `      <  x
00000796    30 03 0a bb 9a 5f 20 f8  7f 22 7e cf 00 5b 46 89    0    _   "~  [F 
00000797    71 85 4a c6 52 a0 a2 02  f0 66 d0 f8 1c 08 45 06    q J R    f    E 
00000798    00 6c 18 2f 60 60 1b 01  84 fa 00 c0 69 a3 50 01     l /``      i P 
00000799    c0 84 70 60 06 c1 82 f6  06 01 ac 18 4f 90 0c 06      p`        O   
00000800    9a 38 04 05 f0 10 c7 e0  a7 56 04 5f 06 07 8b 81     8       V _    
00000801    4c a8 08 39 a2 08 38 10  90 0c 00 e0 30 5e c0 c0    L  9  8     0^  
00000802    35 83 09 f2 01 80 d3 47  21 03 81 09 20 c0 0e 03    5      G!       
00000803    05 ec 0c 03 50 30 9f 20  18 0d 34 70 08 0b e0 21        P0    4p   !
00000804    8f c1 4e 5e 04 5f 06 07  8b 81 4c a8 08 39 a2 10      N^ _    L  9  
00000805    38 10 94 0c 00 e0 30 5e  c0 c0 35 03 09 f2 08 00    8     0^  5     
00000806    d3 47 21 83 81 09 60 c0  0e 83 05 ea 0c 03 50 30     G!   `       P0
00000807    9f 20 80 0d 34 70 08 0b  e0 21 8f c1 4e ac 08 be        4p   !  N   
00000808    0c 0f 17 02 99 50 10 73  44 40 70 21 30 18 01 d0         P sD@p!0   
00000809    60 bd 41 80 6a 06 13 e4  10 01 a6 8f 45 07 02 13    ` A j       E   
00000810    01 80 1d 06 0b d4 18 06  90 61 3e 41 00 1a 68 f0             a>A  h 
00000811    10 17 c0 43 1f 82 9d 58  11 7c 18 1e 2e 05 32 a0       C   X |  . 2 
00000812    20 e6 88 c0 e0 42 68 30  03 c0 c1 7a 83 00 d2 0c         Bh0   z    
00000813    27 c8 20 03 4d 1e 8e 0e  04 27 03 00 3c 0c 17 a8    '   M    '  <   
00000814    30 0d 20 c2 7c 82 00 34  d1 e0 20 2f 80 86 3f 05    0   |  4   /  ? 
00000815    39 78 11 7c 18 1e 2e 05  32 a0 20 e6 89 20 e0 42    9x |  . 2      B
00000816    88 30 03 f5 b0 41 06 01  a0 18 4f 90 40 06 9a 41     0   A    O @  A
00000817    28 1c 08 52 06 00 80 18  2f 40 60 1a 01 84 f8 04    (  R    /@`     
00000818    00 69 a4 00 40 5f 01 42  3f 60 7e 5e 04 7e 2d 84     i  @_ B?`~^ ~- 
00000819    03 c5 c0 a6 54 04 1c d1  2c 1c 08 53 06 00 80 18        T   ,  S    
00000820    2f 40 60 19 c1 84 f8 04  00 69 a4 13 01 c0 85 40    /@`      i     @
00000821    60 08 41 82 f4 06 01 9c  18 4f 80 40 06 9a 40 04    ` A      O @  @ 
00000822    05 ec 14 23 f0 60 be cb  c0 8f c5 b0 80 84 5c 0a       # `        \ 
00000823    65 40 41 cd 12 e0 38 00  a6 0c 01 04 68 10 41 80    e@A   8     h A 
00000824    67 06 13 e0 10 01 a6 90  4a 07 02 14 81 80 1f 06    g       J       
00000825    0b d0 18 06 80 61 3e 01  00 1a 69 00 10 17 c0 43         a>   i    C
00000826    1f b2 3f 2f 02 3f 16 c2  02 10 f8 14 ca 80 83 9a      ?/ ?          
00000827    25 03 81 0a 40 c0 0f 83  05 e8 0c 03 40 30 9f 00    %   @       @0  
00000828    80 0d 34 82 50 38 10 a4  0c 00 f8 30 5e 80 c0 34      4 P8     0^  4
00000829    03 09 f0 08 00 d3 48 00  80 be 02 18 fc 14 e5 e0          H         
00000830    45 f0 80 84 3e 05 32 a0  20 e6 89 40 e0 42 90 30    E   > 2    @ B 0
00000831    04 00 c1 7a 03 00 d0 0c  27 c0 20 03 4d 20 94 0e       z    '   M   
00000832    04 29 03 00 40 0c 17 a0  30 0d 00 c2 7c 02 00 34     )  @   0   |  4
00000833    d2 00 20 2f 80 86 3f 05  39 78 11 7c 20 21 0f 81       /  ? 9x | !  
00000834    4c a8 08 39 a2 50 38 10  a4 0c 01 00 30 5e 80 c0    L  9 P8     0^  
00000835    34 03 09 f0 08 00 d3 48  25 03 81 0a 40 c0 10 03    4      H%   @   
00000836    05 e8 0c 03 40 30 9f 00  80 0d 34 80 08 0b e0 21        @0    4    !
00000837    8f c1 4e 5e 04 5f 08 08  45 c0 a6 54 04 1c d1 2c      N^ _  E  T   ,
00000838    1c 08 53 06 00 80 18 2f  40 60 19 c1 84 f8 04 00      S    /@`      
00000839    69 a4 13 41 c0 85 60 60  08 41 82 f4 06 01 9c 18    i  A  `` A      
00000840    4f 80 40 06 9a 44 04 05  f0 14 23 f6 07 e5 e0 47    O @  D    #    G
00000841    e2 d8 40 42 1f 02 99 50  10 73 44 90 70 21 44 18      @B   P sD p!D 
00000842    01 f0 60 bd 01 80 68 06  13 e0 10 01 a6 90 4a 07      `   h       J 
00000843    02 14 81 80 20 06 0b d0  18 06 80 61 3e 01 00 1a               a>   
00000844    69 00 10 17 c0 43 1f 82  9d 58 11 7c 18 1e 2e 05    i    C   X |  . 
00000845    32 a0 20 e6 89 40 e0 42  90 30 04 00 c1 7a 03 00    2    @ B 0   z  
00000846    d0 0c 27 c0 20 03 4d 20  92 0e 04 28 83 00 3e 0c      '   M    (  > 
00000847    17 a0 30 0d 00 c2 7c 02  00 34 d2 00 20 2f 80 86      0   |  4   /  
00000848    3f 64 bd 58 11 7c 18 1e  2e 05 32 a0 20 e6 89 20    ?d X |  . 2     
00000849    e0 42 88 30 03 e0 c1 7a  03 00 d0 0c 27 c0 20 03     B 0   z    '   
00000850    4d 20 90 0e 04 28 03 00  3e 0c 17 a0 30 0d 00 c2    M    (  >   0   
00000851    7c 82 00 34 d2 00 20 2f  80 86 3f 05 3a b0 22 f8    |  4   /  ? : " 
00000852    30 3c 5c 0a 65 40 41 cd  11 c1 c0 84 f0 60 07 81    0<\ e@A      `  
00000853    82 f5 06 01 a0 18 4f 90  40 06 9a 3d 18 1c 08 4e          O @  =   N
00000854    06 00 78 18 2f 50 60 1a  41 84 f9 04 00 69 a3 c0      x /P` A    i  
00000855    40 5f 01 0c 7e 0a 75 60  45 f0 60 78 b8 14 ca 80    @_  ~ u`E `x    
00000856    83 9a 22 83 81 09 80 c0  0e 83 05 ea 0c 03 48 30      "           H0
00000857    9f 20 80 0d 34 7a 20 38  10 98 0c 00 e8 30 5e a0        4z 8     0^ 
00000858    c0 35 03 09 f2 08 00 d3  47 80 80 be 02 19 78 29     5      G     x)
00000859    d5 81 17 c1 81 e2 e0 53  2a 02 0e 68 86 0e 04 25           S*  h   %
00000860    03 00 3a 0c 17 a8 30 0d  40 c2 7c 82 00 34 d1 e8      :   0 @ |  4  
00000861    60 e0 42 50 30 03 80 c1  7a 83 00 d4 0c 27 c8 20    ` BP0   z    '  
00000862    03 4d 1c 02 02 f8 08 65  e0 a7 56 04 5f 06 07 8b     M     e  V _   
00000863    81 4c a8 08 39 a2 10 38  10 92 0c 00 e0 30 5e c0     L  9  8     0^ 
00000864    c0 35 03 09 f2 01 80 d3  47 20 83 81 09 00 c0 0e     5      G       
00000865    03 05 ec 0c 03 58 30 9f  20 18 0d 34 70 08 0b e0         X0    4p   
00000866    21 8f c1 4e ac 08 be 0c  0f 17 02 99 50 10 73 44    !  N        P sD
00000867    00 70 21 18 18 01 b0 60  bd 81 80 6b 06 13 e8 03     p!    `   k    
00000868    01 a6 8e 3e 07 02 11 41  80 1a 06 0b d8 18 06 c0       >   A        
00000869    61 3e 80 30 1a 68 d0 10  17 c0 43 2f 05 3a b0 22    a> 0 h    C/ : "
00000870    f8 30 3c 5c 0a 65 40 41  d3 01 01 01 01 ff ff 98     0<\ e@A        
00000871    08 08 08 0f ff fc c0 40  40 40 7f ff e6 02 02 02           @@@      
00000872    03 ff ff 30 10 10 10 1f  ff f9 80 80 80 80 ff ff       0            
00000873    cc 04 04 04 07 ff fe 60  20 20 20 3f ff f3 01 01           `   ?    
00000874    01 01 ff ff 98 08 08 08  0f ff fc c0 40 40 40 7f                @@@ 
00000875    ff e6 02 02 02 03 ff ff  30 10 10 10 1f ff f9 80            0       
00000876    80 80 80 ff ff cc 04 04  04 07 ff fe 60 20 20 20                `   
00000877    3f ff f3 01 01 01 01 ff  ff 98 08 08 08 0f ff fc    ?               
00000878    c0 40 40 40 7f ff e6 02  02 02 03 ff fe              @@@           
```

### **第 3 个 TAG 的长度**

```xml
00000878                                            00 00 34                   4
00000879    bd
```

### **第 4 个 FLV TAG（AudioTag）**

```xml
00000879       08 00 01 3a 00 00 1a  00 00 00 00 2e ff fb 70        :       .  p
00000880    c4 0a 00 19 55 95 33 38  fc 00 01 92 31 e9 f7 86        U 38    1   
00000881    30 00 28 45 ce 2c 0d 31  13 50 96 4c c4 92 1d 30    0 (E , 1 P L   0
00000882    e9 cc c8 94 e3 8b 27 50  45 d8 f2 a7 03 7f 9e 18          'PE       
00000883    54 7b 22 f4 13 8a 56 d9  25 71 6a d8 4e d8 b4 e5    T{"   V %qj N   
00000884    cb ef d4 94 4a 1f 79 62  ee 70 bb 8e 57 b5 84 82        J yb p  W   
00000885    37 5b 0c 2d bb 2d 0a 24  ef c9 70 ad fb cf 7f ba    7[ - - $  p     
00000886    f4 9f 52 be fb 69 4d 56  9c b6 1a 9a 96 6b 1e 77      R  iMV     k w
00000887    77 2d d4 ca 37 2a a4 f9  89 65 9c ad 5d 58 6a 29    w-  7*   e  ]Xj)
00000888    34 fd 2b a8 ee b8 17 b9  57 9b ee 76 f2 ef 5d f8    4 +     W  v  ] 
00000889    0e 57 aa 4a 96 2c 7f 7b  7d c3 61 cd 91 df 96 56     W J , {} a    V
00000890    8a d3 f3 0c 31 de 7f df  fd 7e ff f3 fe 5f ff cb        1    ~   _  
00000891    7f f8 63 6e ff ef de 58  e5 bd 3f d3 14 12 19 7d      cn   X  ?    }
00000892    8c e5 94 a9 ff ff ff f0  ec 65 82 00 01 24 93 2a             e   $ *
00000893    f7 a3 46 d5 c8 67 7b 69  98 e2 d1 e8 b9 86 25 76      F  g{i      %v
00000894    04 14 10 a7 20 24 04 98  dd 2a f7 22 2a b3 6d 3d         $   * "* m=
00000895    91 0d d7 e1 dd 3d 8f 80  ab f6 be 64 f1 11 83 ff         =     d    
00000896    e7 7e ff f7 2c a5 05 91  3a 1c 07 da cd 1e 4c ca     ~  ,   :     L 
00000897    e5 e4 4c 60 d6 18 49 b9  6d 5d 8e 5b 9a ae 4f 7d      L`  I m] [  O}
00000898    77 14 ed 08 d7 ce 91 11  c6 51 f8 3d 00 2a af 00    w        Q = *  
00000899    00 12 89 21 da 14                                      !           
```

#### **AudioTagHeader**

```xml
00000879                                         2e                         .
```

#### **AUDIODATA**

```xml
00000879                                            ff fb 70                   p
00000880    c4 0a 00 19 55 95 33 38  fc 00 01 92 31 e9 f7 86        U 38    1   
00000881    30 00 28 45 ce 2c 0d 31  13 50 96 4c c4 92 1d 30    0 (E , 1 P L   0
00000882    e9 cc c8 94 e3 8b 27 50  45 d8 f2 a7 03 7f 9e 18          'PE       
00000883    54 7b 22 f4 13 8a 56 d9  25 71 6a d8 4e d8 b4 e5    T{"   V %qj N   
00000884    cb ef d4 94 4a 1f 79 62  ee 70 bb 8e 57 b5 84 82        J yb p  W   
00000885    37 5b 0c 2d bb 2d 0a 24  ef c9 70 ad fb cf 7f ba    7[ - - $  p     
00000886    f4 9f 52 be fb 69 4d 56  9c b6 1a 9a 96 6b 1e 77      R  iMV     k w
00000887    77 2d d4 ca 37 2a a4 f9  89 65 9c ad 5d 58 6a 29    w-  7*   e  ]Xj)
00000888    34 fd 2b a8 ee b8 17 b9  57 9b ee 76 f2 ef 5d f8    4 +     W  v  ] 
00000889    0e 57 aa 4a 96 2c 7f 7b  7d c3 61 cd 91 df 96 56     W J , {} a    V
00000890    8a d3 f3 0c 31 de 7f df  fd 7e ff f3 fe 5f ff cb        1    ~   _  
00000891    7f f8 63 6e ff ef de 58  e5 bd 3f d3 14 12 19 7d      cn   X  ?    }
00000892    8c e5 94 a9 ff ff ff f0  ec 65 82 00 01 24 93 2a             e   $ *
00000893    f7 a3 46 d5 c8 67 7b 69  98 e2 d1 e8 b9 86 25 76      F  g{i      %v
00000894    04 14 10 a7 20 24 04 98  dd 2a f7 22 2a b3 6d 3d         $   * "* m=
00000895    91 0d d7 e1 dd 3d 8f 80  ab f6 be 64 f1 11 83 ff         =     d    
00000896    e7 7e ff f7 2c a5 05 91  3a 1c 07 da cd 1e 4c ca     ~  ,   :     L 
00000897    e5 e4 4c 60 d6 18 49 b9  6d 5d 8e 5b 9a ae 4f 7d      L`  I m] [  O}
00000898    77 14 ed 08 d7 ce 91 11  c6 51 f8 3d 00 2a af 00    w        Q = *  
00000899    00 12 89 21 da 14                                      !           
```

### **第 4 个 TAG 的长度**

```xml
00000899                      00 00  01 45                               E      
```

### **第 5 个 FLV TAG（VideoTag）**

```xml
00000899                                   09 00 06 f3 00 00                    
00000900    28 00 00 00 00 22 00 00  84 06 b1 93 de f7 bd 3d    (    "         =
00000901    ef 7b d3 de f7 bd 3d ef  7b d3 de f7 bd 3d ef 7b     {    = {    = {
00000902    d3 de f7 bd 3d ef 7b d3  de f7 bd 3d ef 7b d3 de        = {    = {  
00000903    f7 bd 3d ef 7b d3 de f7  bd 3d ef 7b d3 de f7 bd      = {    = {    
00000904    3d ef 7b d3 de f7 bd 3d  ef 7b d3 de f7 bd 3d ef    = {    = {    = 
00000905    7b d9 de f6 77 bd 9d ef  67 7b d9 de f6 77 bd 9d    {   w   g{   w  
00000906    ef 67 7b d9 de f6 77 bd  9d ef 67 7b d9 de f6 77     g{   w   g{   w
00000907    bd 9d ef 67 7b d2 6f 7b  d2 6f 78 d0 53 bd e3 44       g{ o{ ox S  D
00000908    9b de 80 07 fa 9b 9d e1  7c 43 de ac 6d 29 5c 5c            |C  m)\\
00000909    d2 f9 de d4 26 84 b9 d4  40 ad c8 42 1a d1 a0 d3        &   @  B    
00000910    42 8b 75 35 da d1 b4 82  0a 1c e9 3e 06 a8 1e d5    B u5       >    
00000911    1d 19 e7 4e 9d 3b 4f b0  db dd 24 79 d3 a7 54 ce       N ;O   $y  T 
00000912    9d ff 85 5c 15 b3 06 a1  ff 25 94 91 b3 05 66 71       \     %    fq
00000913    ae 8f 7a 69 27 52 30 34  c6 2a 39 17 59 25 22 57      zi'R04 *9 Y%"W
00000914    21 1b d4 55 37 23 82 ef  f2 f8 3b fe b6 7e 4b 31    !  U7#    ;  ~K1
00000915    dd 9c e9 d6 73 a7 5d af  fd 6c 6f d0 fe 4a f5 ba        s ]  lo  J  
00000916    8e 71 6e da 02 2e 4f b3  9d 3a c3 13 dd 87 ff 85     qn  .O  :      
00000917    4c 34 8d c9 85 7d db 99  96 43 75 02 dc ba 6b 96    L4   }   Cu   k 
00000918    9a 58 a3 4e 89 71 bb 84  3c 68 3b ee 1a 87 82 8f     X N q  <h;     
00000919    cb d9 12 a3 c3 4b ea 8d  d3 32 07 b2 bd 38 df 63         K   2   8 c
00000920    0f 1c 6f 6a 73 b1 7c 3d  b8 71 ff e3 8f b1 8e f0      ojs |= q      
00000921    35 a8 90 53 a1 db 45 89  73 79 34 f0 8f 9b 99 cd    5  S  E sy4     
00000922    a8 f8 57 34 b1 cc 88 05  05 2c d7 20 10 2b 68 9f      W4     ,   +h 
00000923    cd 23 40 0b e4 f4 1e 48  51 b4 64 d4 5b a9 b1 3e     #@    HQ d [  >
00000924    4e e9 6c 4f 4f 4d 61 8d  6d d3 49 84 5f 57 98 0b    N lOOMa m I _W  
00000925    91 b1 6a 2b 2f 48 10 27  9c 3f 2a fa c2 1a 15 25      j+/H ' ?*    %
00000926    da 5b eb 25 93 91 be a0  a6 34 09 a3 10 a5 9c ed     [ %     4      
00000927    3e 86 90 a9 a4 be 4b d2  a7 2d 0a 50 ce 77 48 c4    >     K  - P wH 
00000928    3d 70 fd 04 7d d4 ca 90  76 c8 9c f6 5a c9 56 6a    =p  }   v   Z Vj
00000929    23 0c 43 77 6f 71 8f ad  b9 40 6b ac 22 12 71 13    # Cwoq   @k " q 
00000930    f8 cb 50 23 16 82 09 3f  ca 12 96 47 68 70 88 9c      P#   ?   Ghp  
00000931    39 10 61 69 19 90 a7 f6  bc 56 1a 15 6f 6e e5 e6    9 ai     V  on  
00000932    9a eb 19 b6 e1 80 e3 03  46 78 e5 d1 ca 56 86 c6            Fx   V  
00000933    8d 08 fc 51 2e d6 d3 61  5f 4b 39 4f 2e 56 03 e6       Q.  a_K9O.V  
00000934    2f ce f4 35 e9 22 49 1f  8d b2 1c 49 68 e2 65 a7    /  5 "I    Ih e 
00000935    99 c8 d2 4c d4 f4 c0 8f  d8 77 71 69 59 6b b0 70       L     wqiYk p
00000936    57 87 83 ea ee 6d 6b 6a  ed 8e 78 ce 12 8e 10 69    W    mkj  x    i
00000937    57 36 dc 3c 9e d5 1c ca  cd d6 13 44 a6 d8 1b d2    W6 <       D    
00000938    da ec c8 83 2c 64 52 31  23 55 77 b9 35 b4 56 67        ,dR1#Uw 5 Vg
00000939    20 d6 e5 dc cd a9 87 10  ff 5d 4a d2 c4 8c 44 78             ]J   Dx
00000940    44 21 7b 4b 1c db 2b 66  e7 b2 06 af e7 11 58 ca    D!{K  +f      X 
00000941    24 c8 61 00 dc da 7e 16  93 a1 af 92 25 84 67 1a    $ a   ~     % g 
00000942    4d fb db bb 76 fd d5 97  f9 ff ca e4 9f ce 29 ed    M   v         ) 
00000943    4d 0c 5d c9 c2 51 17 42  57 ac 6d 59 31 5b ab 69    M ]  Q BW mY1[ i
00000944    9e ca 2a c6 a3 6e 78 5a  5a 1b 6a 0c b6 ca c7 1c      *  nxZZ j     
00000945    32 63 a9 9d ad 13 76 b5  22 80 e5 9a 4b a2 8d 1c    2c    v "   K   
00000946    ee a7 01 2a 73 63 75 19  a9 ac cf 6c 62 db 98 89       *scu    lb   
00000947    bd eb 38 70 67 99 4c 91  18 9a 6b 65 1a 8a 42 2f      8pg L   ke  B/
00000948    47 88 75 79 34 c2 98 e4  71 b8 46 1e b7 83 34 5a    G uy4   q F   4Z
00000949    1e d5 e3 d7 99 75 15 69  9c 2a 7f 22 5d 62 8d b9         u i * "]b  
00000950    89 75 e2 4d aa 61 86 90  44 da 36 c9 12 dd a7 50     u M a  D 6    P
00000951    16 da 79 3c ae 46 9c 80  06 5e a3 cd c3 25 22 c1      y< F   ^   %" 
00000952    8f 88 a0 a6 cb ad 21 3e  90 ac 15 ee 4d 9a 56 1f          !>    M V 
00000953    33 34 eb b7 7b 77 eb 39  95 38 90 cc ca 65 9e 68    34  {w 9 8   e h
00000954    61 02 f5 3b 98 dd 17 dc  a8 4e 46 5f 3a e2 24 18    a  ;     NF_: $ 
00000955    cc 0d 65 1c 53 a5 7c 43  16 16 08 bc 51 35 74 51      e S |C    Q5tQ
00000956    86 ae 73 b0 3d 3a 6a f5  b1 b6 c7 a3 48 d8 b4 73      s =:j     H  s
00000957    b4 fb 90 87 a1 f8 69 39  d3 c2 1f d0 d9 7e 50 a2          i9     ~P 
00000958    0c 86 ef 5c 28 45 89 c7  3c 92 11 d9 e4 fe 47 c0       \(E  <     G 
00000959    46 eb c4 5f 22 09 53 6e  b7 88 11 36 fe ac ba 11    F  _" Sn   6    
00000960    55 fe 56 9a 99 9b 4b 6a  31 72 32 db 1e 33 d3 91    U V   Kj1r2  3  
00000961    82 f5 0b e6 75 66 43 a8  cf 34 8c 72 c5 5d 6b 49        ufC  4 r ]kI
00000962    93 f6 a4 d5 c3 b4 37 2c  8e 99 c1 bd 46 ce 05 43          7,    F  C
00000963    9b 31 33 25 a9 67 2e 65  79 50 84 d3 55 76 da 3e     13% g.eyP  Uv >
00000964    23 e0 22 2d 51 54 26 86  c1 a2 f3 51 c2 67 b4 ce    # "-QT&    Q g  
00000965    20 e3 ca 68 81 96 d9 d2  d7 8c 3f 1f a6 4a 47 01       h      ?  JG 
00000966    e5 33 76 dd bd ec 73 e5  c4 e4 97 95 2d 5f a8 61     3v   s     -_ a
00000967    82 d7 33 b6 d9 51 8e 6a  e1 b5 72 5e 75 2b 75 fd      3  Q j  r^u+u 
00000968    60 86 c9 18 65 78 ca 77  8c fc b4 12 03 c0 19 a3    `   ex w        
00000969    6e 15 f1 cd cd de 8c 92  45 ae 0e 08 46 17 43 53    n       E   F CS
00000970    a9 6d 78 78 b6 54 5c e5  70 c7 a4 68 b0 15 ba 34     mxx T\ p  h   4
00000971    9d 26 ed e1 82 de 4b 80  9a 23 ee 50 11 90 b1 90     &    K  # P    
00000972    e6 c7 a6 eb c3 b0 46 b7  2e 38 6d 2a cb a0 36 23          F .8m*  6#
00000973    fb c0 44 a8 e3 c3 a4 bd  73 49 f5 90 ff a1 60 7e      D     sI    `~
00000974    b3 d0 06 fb 1a a9 9b 94  d8 51 6d 8e 70 96 b9 b4             Qm p   
00000975    43 68 35 da 81 1d 4d 3b  09 5b 08 8a 7f a8 dd ca    Ch5   M; [      
00000976    d2 3e fc ec 22 49 c9 ac  82 35 a8 6e ed 61 a7 4c     >  "I   5 n a L
00000977    ad 9c 66 68 d9 1f 4f 08  f1 c8 6f b9 b1 83 81 e5      fh  O   o     
00000978    ea 3c 29 32 4e 40 0a de  76 0d 11 e1 df 03 aa 8e     <)2N@  v       
00000979    bf 6d 09 4e 08 2c a3 a8  3b 1a 2b c3 47 8e 8c 6b     m N ,  ; + G  k
00000980    ca b3 cd dd 21 45 a3 21  a2 51 1d 49 4e a7 2c e6        !E ! Q IN , 
00000981    51 c1 be 3f 70 a1 bb 0c  a7 af d7 b3 eb ce a4 cf    Q  ?p           
00000982    f6 ae 02 8e 8a 7e c6 61  47 01 10 fd 9d 69 07 25         ~ aG    i %
00000983    48 df 46 28 d0 b9 4f a6  4a aa 33 6a 7c 10 04 34    H F(  O J 3j|  4
00000984    07 03 f1 05 8c b8 db d9  c1 17 bc d2 20 db 92 cd                    
00000985    36 2b c3 96 51 47 23 73  39 58 4d 76 f4 b3 61 55    6+  QG#s9XMv  aU
00000986    15 b3 d6 b7 80 34 d0 15  5f f2 4b ab 6a 55 d1 8a         4  _ K jU  
00000987    0a a6 d2 16 e4 81 82 9d  db 27 1e 0a c4 b0 f1 1f             '      
00000988    af d4 c5 35 0b 73 9c 93  8e 92 8b 1b 24 e7 89 c4       5 s      $   
00000989    6a 88 a5 cb 44 dc 8c e4  4d a2 e8 a5 7e 22 16 5d    j   D   M   ~" ]
00000990    f2 e5 b2 a1 87 0f a5 e6  2c 52 54 76 76 0e 2c 01            ,RTvv , 
00000991    57 13 82 ec 8a bb e3 4a  5a e9 c3 7d da 76 dd 0f    W      JZ  } v  
00000992    75 67 41 1e 90 fb 3a 3c  16 a4 66 20 41 8e 49 dc    ugA   :<  f A I 
00000993    67 67 07 1d e9 a3 dc 8f  0b 3f 42 50 e9 96 2f 17    gg       ?BP  / 
00000994    86 4b 33 83 21 cf 03 8d  2a b2 3d 16 1d 87 17 b0     K3 !   * =     
00000995    c3 29 6d 3a 70 51 e4 3c  6b 43 70 f8 43 98 fc 78     )m:pQ <kCp C  x
00000996    59 ff b5 b2 be 8b 6f 39  c4 e8 92 0e 0f 34 29 43    Y     o9     4)C
00000997    5c cf 43 d9 44 18 96 3b  9c 69 b2 94 86 53 e7 51    \ C D  ; i   S Q
00000998    87 0b 10 ce c6 20 6b b9  a3 4b a4 af 04 1e aa e5          k  K      
00000999    4e 37 cf 24 7f 04 0b 5c  24 57 bd 0d 43 45 a2 47    N7 $   \$W  CE G
00001000    7b d0 96 d1 68 91 27 bd  0d 43 45 a2 4d ef 43 50    {   h '  CE M CP
00001001    d1 68 93 7b d0 96 d1 68  93 7b d1 8c 42 9f bd 13     h {   h {  B   
00001002    05 0f 78 56 80 07 05 4d  ef 20 1a 05 4d ef 42 5b      xV   M    M B[
00001003    45 a8 22 08 2c 0a 9b de  84 b6 8b 50 44 10 58 15    E " ,      PD X 
00001004    37 bd 09 6d 16 a0 01 c3  45 7b c6 82 4d ef 43 50    7  m    E{  M CP
00001005    d1 68 87 bd ef 44 03 44  9b de 35 12 6f 78 d4 49     h   D D  5 ox I
00001006    bd e8 40 78 93 7b d0 80  f1 26 f7 8d 44 9b de 84      @x {   &  D   
00001007    07 a7 bd ef 7a 7b de f7  a7 bd ef 7a 7b de f7 a7        z{     z{   
00001008    bd ef 7a 7b de f7 a7 bd  ef 7a 7b de f7 a7 bd ef      z{     z{     
00001009    7a 7b de f7 a7 bd ef 7a  7b de f7 a7 bd ef 7a 7b    z{     z{     z{
00001010    de f7 a7 bd ef 7a 7b de  f7 a7 bd ef 7a 7b de f7         z{     z{  
00001011    a7 bd ef 7a 7b de f7 80                                z{           
```

#### **VideoTagHeader**

```xml
00000900                   22                                        "
```

#### **VIDEODATA**

```xml
00000900                      00 00  84 06 b1 93 de f7 bd 3d                   =
00000901    ef 7b d3 de f7 bd 3d ef  7b d3 de f7 bd 3d ef 7b     {    = {    = {
00000902    d3 de f7 bd 3d ef 7b d3  de f7 bd 3d ef 7b d3 de        = {    = {  
00000903    f7 bd 3d ef 7b d3 de f7  bd 3d ef 7b d3 de f7 bd      = {    = {    
00000904    3d ef 7b d3 de f7 bd 3d  ef 7b d3 de f7 bd 3d ef    = {    = {    = 
00000905    7b d9 de f6 77 bd 9d ef  67 7b d9 de f6 77 bd 9d    {   w   g{   w  
00000906    ef 67 7b d9 de f6 77 bd  9d ef 67 7b d9 de f6 77     g{   w   g{   w
00000907    bd 9d ef 67 7b d2 6f 7b  d2 6f 78 d0 53 bd e3 44       g{ o{ ox S  D
00000908    9b de 80 07 fa 9b 9d e1  7c 43 de ac 6d 29 5c 5c            |C  m)\\
00000909    d2 f9 de d4 26 84 b9 d4  40 ad c8 42 1a d1 a0 d3        &   @  B    
00000910    42 8b 75 35 da d1 b4 82  0a 1c e9 3e 06 a8 1e d5    B u5       >    
00000911    1d 19 e7 4e 9d 3b 4f b0  db dd 24 79 d3 a7 54 ce       N ;O   $y  T 
00000912    9d ff 85 5c 15 b3 06 a1  ff 25 94 91 b3 05 66 71       \     %    fq
00000913    ae 8f 7a 69 27 52 30 34  c6 2a 39 17 59 25 22 57      zi'R04 *9 Y%"W
00000914    21 1b d4 55 37 23 82 ef  f2 f8 3b fe b6 7e 4b 31    !  U7#    ;  ~K1
00000915    dd 9c e9 d6 73 a7 5d af  fd 6c 6f d0 fe 4a f5 ba        s ]  lo  J  
00000916    8e 71 6e da 02 2e 4f b3  9d 3a c3 13 dd 87 ff 85     qn  .O  :      
00000917    4c 34 8d c9 85 7d db 99  96 43 75 02 dc ba 6b 96    L4   }   Cu   k 
00000918    9a 58 a3 4e 89 71 bb 84  3c 68 3b ee 1a 87 82 8f     X N q  <h;     
00000919    cb d9 12 a3 c3 4b ea 8d  d3 32 07 b2 bd 38 df 63         K   2   8 c
00000920    0f 1c 6f 6a 73 b1 7c 3d  b8 71 ff e3 8f b1 8e f0      ojs |= q      
00000921    35 a8 90 53 a1 db 45 89  73 79 34 f0 8f 9b 99 cd    5  S  E sy4     
00000922    a8 f8 57 34 b1 cc 88 05  05 2c d7 20 10 2b 68 9f      W4     ,   +h 
00000923    cd 23 40 0b e4 f4 1e 48  51 b4 64 d4 5b a9 b1 3e     #@    HQ d [  >
00000924    4e e9 6c 4f 4f 4d 61 8d  6d d3 49 84 5f 57 98 0b    N lOOMa m I _W  
00000925    91 b1 6a 2b 2f 48 10 27  9c 3f 2a fa c2 1a 15 25      j+/H ' ?*    %
00000926    da 5b eb 25 93 91 be a0  a6 34 09 a3 10 a5 9c ed     [ %     4      
00000927    3e 86 90 a9 a4 be 4b d2  a7 2d 0a 50 ce 77 48 c4    >     K  - P wH 
00000928    3d 70 fd 04 7d d4 ca 90  76 c8 9c f6 5a c9 56 6a    =p  }   v   Z Vj
00000929    23 0c 43 77 6f 71 8f ad  b9 40 6b ac 22 12 71 13    # Cwoq   @k " q 
00000930    f8 cb 50 23 16 82 09 3f  ca 12 96 47 68 70 88 9c      P#   ?   Ghp  
00000931    39 10 61 69 19 90 a7 f6  bc 56 1a 15 6f 6e e5 e6    9 ai     V  on  
00000932    9a eb 19 b6 e1 80 e3 03  46 78 e5 d1 ca 56 86 c6            Fx   V  
00000933    8d 08 fc 51 2e d6 d3 61  5f 4b 39 4f 2e 56 03 e6       Q.  a_K9O.V  
00000934    2f ce f4 35 e9 22 49 1f  8d b2 1c 49 68 e2 65 a7    /  5 "I    Ih e 
00000935    99 c8 d2 4c d4 f4 c0 8f  d8 77 71 69 59 6b b0 70       L     wqiYk p
00000936    57 87 83 ea ee 6d 6b 6a  ed 8e 78 ce 12 8e 10 69    W    mkj  x    i
00000937    57 36 dc 3c 9e d5 1c ca  cd d6 13 44 a6 d8 1b d2    W6 <       D    
00000938    da ec c8 83 2c 64 52 31  23 55 77 b9 35 b4 56 67        ,dR1#Uw 5 Vg
00000939    20 d6 e5 dc cd a9 87 10  ff 5d 4a d2 c4 8c 44 78             ]J   Dx
00000940    44 21 7b 4b 1c db 2b 66  e7 b2 06 af e7 11 58 ca    D!{K  +f      X 
00000941    24 c8 61 00 dc da 7e 16  93 a1 af 92 25 84 67 1a    $ a   ~     % g 
00000942    4d fb db bb 76 fd d5 97  f9 ff ca e4 9f ce 29 ed    M   v         ) 
00000943    4d 0c 5d c9 c2 51 17 42  57 ac 6d 59 31 5b ab 69    M ]  Q BW mY1[ i
00000944    9e ca 2a c6 a3 6e 78 5a  5a 1b 6a 0c b6 ca c7 1c      *  nxZZ j     
00000945    32 63 a9 9d ad 13 76 b5  22 80 e5 9a 4b a2 8d 1c    2c    v "   K   
00000946    ee a7 01 2a 73 63 75 19  a9 ac cf 6c 62 db 98 89       *scu    lb   
00000947    bd eb 38 70 67 99 4c 91  18 9a 6b 65 1a 8a 42 2f      8pg L   ke  B/
00000948    47 88 75 79 34 c2 98 e4  71 b8 46 1e b7 83 34 5a    G uy4   q F   4Z
00000949    1e d5 e3 d7 99 75 15 69  9c 2a 7f 22 5d 62 8d b9         u i * "]b  
00000950    89 75 e2 4d aa 61 86 90  44 da 36 c9 12 dd a7 50     u M a  D 6    P
00000951    16 da 79 3c ae 46 9c 80  06 5e a3 cd c3 25 22 c1      y< F   ^   %" 
00000952    8f 88 a0 a6 cb ad 21 3e  90 ac 15 ee 4d 9a 56 1f          !>    M V 
00000953    33 34 eb b7 7b 77 eb 39  95 38 90 cc ca 65 9e 68    34  {w 9 8   e h
00000954    61 02 f5 3b 98 dd 17 dc  a8 4e 46 5f 3a e2 24 18    a  ;     NF_: $ 
00000955    cc 0d 65 1c 53 a5 7c 43  16 16 08 bc 51 35 74 51      e S |C    Q5tQ
00000956    86 ae 73 b0 3d 3a 6a f5  b1 b6 c7 a3 48 d8 b4 73      s =:j     H  s
00000957    b4 fb 90 87 a1 f8 69 39  d3 c2 1f d0 d9 7e 50 a2          i9     ~P 
00000958    0c 86 ef 5c 28 45 89 c7  3c 92 11 d9 e4 fe 47 c0       \(E  <     G 
00000959    46 eb c4 5f 22 09 53 6e  b7 88 11 36 fe ac ba 11    F  _" Sn   6    
00000960    55 fe 56 9a 99 9b 4b 6a  31 72 32 db 1e 33 d3 91    U V   Kj1r2  3  
00000961    82 f5 0b e6 75 66 43 a8  cf 34 8c 72 c5 5d 6b 49        ufC  4 r ]kI
00000962    93 f6 a4 d5 c3 b4 37 2c  8e 99 c1 bd 46 ce 05 43          7,    F  C
00000963    9b 31 33 25 a9 67 2e 65  79 50 84 d3 55 76 da 3e     13% g.eyP  Uv >
00000964    23 e0 22 2d 51 54 26 86  c1 a2 f3 51 c2 67 b4 ce    # "-QT&    Q g  
00000965    20 e3 ca 68 81 96 d9 d2  d7 8c 3f 1f a6 4a 47 01       h      ?  JG 
00000966    e5 33 76 dd bd ec 73 e5  c4 e4 97 95 2d 5f a8 61     3v   s     -_ a
00000967    82 d7 33 b6 d9 51 8e 6a  e1 b5 72 5e 75 2b 75 fd      3  Q j  r^u+u 
00000968    60 86 c9 18 65 78 ca 77  8c fc b4 12 03 c0 19 a3    `   ex w        
00000969    6e 15 f1 cd cd de 8c 92  45 ae 0e 08 46 17 43 53    n       E   F CS
00000970    a9 6d 78 78 b6 54 5c e5  70 c7 a4 68 b0 15 ba 34     mxx T\ p  h   4
00000971    9d 26 ed e1 82 de 4b 80  9a 23 ee 50 11 90 b1 90     &    K  # P    
00000972    e6 c7 a6 eb c3 b0 46 b7  2e 38 6d 2a cb a0 36 23          F .8m*  6#
00000973    fb c0 44 a8 e3 c3 a4 bd  73 49 f5 90 ff a1 60 7e      D     sI    `~
00000974    b3 d0 06 fb 1a a9 9b 94  d8 51 6d 8e 70 96 b9 b4             Qm p   
00000975    43 68 35 da 81 1d 4d 3b  09 5b 08 8a 7f a8 dd ca    Ch5   M; [      
00000976    d2 3e fc ec 22 49 c9 ac  82 35 a8 6e ed 61 a7 4c     >  "I   5 n a L
00000977    ad 9c 66 68 d9 1f 4f 08  f1 c8 6f b9 b1 83 81 e5      fh  O   o     
00000978    ea 3c 29 32 4e 40 0a de  76 0d 11 e1 df 03 aa 8e     <)2N@  v       
00000979    bf 6d 09 4e 08 2c a3 a8  3b 1a 2b c3 47 8e 8c 6b     m N ,  ; + G  k
00000980    ca b3 cd dd 21 45 a3 21  a2 51 1d 49 4e a7 2c e6        !E ! Q IN , 
00000981    51 c1 be 3f 70 a1 bb 0c  a7 af d7 b3 eb ce a4 cf    Q  ?p           
00000982    f6 ae 02 8e 8a 7e c6 61  47 01 10 fd 9d 69 07 25         ~ aG    i %
00000983    48 df 46 28 d0 b9 4f a6  4a aa 33 6a 7c 10 04 34    H F(  O J 3j|  4
00000984    07 03 f1 05 8c b8 db d9  c1 17 bc d2 20 db 92 cd                    
00000985    36 2b c3 96 51 47 23 73  39 58 4d 76 f4 b3 61 55    6+  QG#s9XMv  aU
00000986    15 b3 d6 b7 80 34 d0 15  5f f2 4b ab 6a 55 d1 8a         4  _ K jU  
00000987    0a a6 d2 16 e4 81 82 9d  db 27 1e 0a c4 b0 f1 1f             '      
00000988    af d4 c5 35 0b 73 9c 93  8e 92 8b 1b 24 e7 89 c4       5 s      $   
00000989    6a 88 a5 cb 44 dc 8c e4  4d a2 e8 a5 7e 22 16 5d    j   D   M   ~" ]
00000990    f2 e5 b2 a1 87 0f a5 e6  2c 52 54 76 76 0e 2c 01            ,RTvv , 
00000991    57 13 82 ec 8a bb e3 4a  5a e9 c3 7d da 76 dd 0f    W      JZ  } v  
00000992    75 67 41 1e 90 fb 3a 3c  16 a4 66 20 41 8e 49 dc    ugA   :<  f A I 
00000993    67 67 07 1d e9 a3 dc 8f  0b 3f 42 50 e9 96 2f 17    gg       ?BP  / 
00000994    86 4b 33 83 21 cf 03 8d  2a b2 3d 16 1d 87 17 b0     K3 !   * =     
00000995    c3 29 6d 3a 70 51 e4 3c  6b 43 70 f8 43 98 fc 78     )m:pQ <kCp C  x
00000996    59 ff b5 b2 be 8b 6f 39  c4 e8 92 0e 0f 34 29 43    Y     o9     4)C
00000997    5c cf 43 d9 44 18 96 3b  9c 69 b2 94 86 53 e7 51    \ C D  ; i   S Q
00000998    87 0b 10 ce c6 20 6b b9  a3 4b a4 af 04 1e aa e5          k  K      
00000999    4e 37 cf 24 7f 04 0b 5c  24 57 bd 0d 43 45 a2 47    N7 $   \$W  CE G
00001000    7b d0 96 d1 68 91 27 bd  0d 43 45 a2 4d ef 43 50    {   h '  CE M CP
00001001    d1 68 93 7b d0 96 d1 68  93 7b d1 8c 42 9f bd 13     h {   h {  B   
00001002    05 0f 78 56 80 07 05 4d  ef 20 1a 05 4d ef 42 5b      xV   M    M B[
00001003    45 a8 22 08 2c 0a 9b de  84 b6 8b 50 44 10 58 15    E " ,      PD X 
00001004    37 bd 09 6d 16 a0 01 c3  45 7b c6 82 4d ef 43 50    7  m    E{  M CP
00001005    d1 68 87 bd ef 44 03 44  9b de 35 12 6f 78 d4 49     h   D D  5 ox I
00001006    bd e8 40 78 93 7b d0 80  f1 26 f7 8d 44 9b de 84      @x {   &  D   
00001007    07 a7 bd ef 7a 7b de f7  a7 bd ef 7a 7b de f7 a7        z{     z{   
00001008    bd ef 7a 7b de f7 a7 bd  ef 7a 7b de f7 a7 bd ef      z{     z{     
00001009    7a 7b de f7 a7 bd ef 7a  7b de f7 a7 bd ef 7a 7b    z{     z{     z{
00001010    de f7 a7 bd ef 7a 7b de  f7 a7 bd ef 7a 7b de f7         z{     z{  
00001011    a7 bd ef 7a 7b de f7 80                                z{           
```

### **第 5 个 TAG 的长度**

```xml
00001011                             00 00 06 fe
```

### **第 6 个 FLV TAG（AudioTag）**

```xml
00001011                                         08 00 01 06                    
00001012    00 00 34 00 00 00 00 2e  ff fb 60 c4 04 00 0c dd      4    .  `     
00001013    39 4b 41 98 ab c9 c4 b0  69 68 91 95 b9 c2 60 2c    9KA     ih    `,
00001014    e8 40 81 96 03 27 04 ec  62 60 86 98 10 08 06 ed     @   '  b`      
00001015    cc f7 46 c1 4f e7 c2 29  ec 75 95 f7 e1 51 e8 46      F O  ) u   Q F
00001016    17 ab 32 92 ea 99 5d b5  72 bb ab c9 57 3a 23 a1      2   ] r   W:# 
00001017    5f d1 09 d5 ee 44 9c 8a  8c a4 5a 32 a3 ba 8b b6    _    D    Z2    
00001018    4e 23 b3 ec 90 d4 c8 3a  e2 2d 70 29 d6 d3 1d 46    N#     : -p)   F
00001019    e7 48 1d 03 fc fc 87 b7  ae 98 65 f5 f0 04 00 24     H        e    $
00001020    82 45 d0 42 9c 67 92 c9  b9 0e 0c 86 ad c0 03 50     E B g         P
00001021    11 41 94 38 e2 c5 8b 85  2e e3 b9 38 0b c1 c3 49     A 8    .  8   I
00001022    e6 80 3f 0e c5 ce 19 23  55 aa 8d a3 66 33 2a ba      ?    #U   f3* 
00001023    23 25 89 71 15 b3 3b f5  46 a2 ba da a9 41 16 2b    #% q  ; F    A +
00001024    bb b2 a1 8d 3b a1 d1 75  0f 62 6f b1 86 89 0a a9        ;  u bo     
00001025    18 c7 8a 86 d5 59 55 4c  4b d2 8c 76 41 32 95 4e         YULK  vA2 N
00001026    68 81 0e ac 56 8a 1c 0c  82 a6 7a 54 bc 90 94 40    h   V     zT   @
00001027    00 10 4a 40 3a bf 04 e2  39 44 62 40 ec d1 53 2a      J@:   9Db@  S*
00001028    59 c0 6f 80 82 11 5a 65  b0 13 b1 7e d2             Y o   Ze   ~    
```

#### **AudioTagHeader**

```xml
00001012                         2e                                    .
```

#### **AUDIODATA**

```xml
00001012                             ff fb 60 c4 04 00 0c dd              `     
00001013    39 4b 41 98 ab c9 c4 b0  69 68 91 95 b9 c2 60 2c    9KA     ih    `,
00001014    e8 40 81 96 03 27 04 ec  62 60 86 98 10 08 06 ed     @   '  b`      
00001015    cc f7 46 c1 4f e7 c2 29  ec 75 95 f7 e1 51 e8 46      F O  ) u   Q F
00001016    17 ab 32 92 ea 99 5d b5  72 bb ab c9 57 3a 23 a1      2   ] r   W:# 
00001017    5f d1 09 d5 ee 44 9c 8a  8c a4 5a 32 a3 ba 8b b6    _    D    Z2    
00001018    4e 23 b3 ec 90 d4 c8 3a  e2 2d 70 29 d6 d3 1d 46    N#     : -p)   F
00001019    e7 48 1d 03 fc fc 87 b7  ae 98 65 f5 f0 04 00 24     H        e    $
00001020    82 45 d0 42 9c 67 92 c9  b9 0e 0c 86 ad c0 03 50     E B g         P
00001021    11 41 94 38 e2 c5 8b 85  2e e3 b9 38 0b c1 c3 49     A 8    .  8   I
00001022    e6 80 3f 0e c5 ce 19 23  55 aa 8d a3 66 33 2a ba      ?    #U   f3* 
00001023    23 25 89 71 15 b3 3b f5  46 a2 ba da a9 41 16 2b    #% q  ; F    A +
00001024    bb b2 a1 8d 3b a1 d1 75  0f 62 6f b1 86 89 0a a9        ;  u bo     
00001025    18 c7 8a 86 d5 59 55 4c  4b d2 8c 76 41 32 95 4e         YULK  vA2 N
00001026    68 81 0e ac 56 8a 1c 0c  82 a6 7a 54 bc 90 94 40    h   V     zT   @
00001027    00 10 4a 40 3a bf 04 e2  39 44 62 40 ec d1 53 2a      J@:   9Db@  S*
00001028    59 c0 6f 80 82 11 5a 65  b0 13 b1 7e d2             Y o   Ze   ~    
```

### **第 6 个 TAG 的长度**

```xml
00001028                                            00 00 01
00001029    11
```

### **第 7 个 FLV TAG（AudioTag）**

```xml
00001029       08 00 00 d1 00 00 4e  00 00 00 00 2e ff fb 50           N    .  P
00001030    c4 10 00 0d 89 77 4b a3  18 ad c1 ae ac a9 74 63         wK       tc
00001031    0d b9 c6 6f ee 71 fb 29  2f 7e 75 f5 2c cc ed fe       o q )/~u ,   
00001032    71 e2 45 ce 25 51 37 3b  8d 43 0e 33 a5 4c 22 54    q E %Q7; C 3 L"T
00001033    56 4e f7 b3 9e 8d aa 31  25 d5 aa 2c 83 44 65 2d    VN     1%  , De-
00001034    e6 b3 0e ef ea 18 c4 ba  11 86 3b ed fb b9 b1 f5              ;     
00001035    32 a1 0a 30 8c 3d 90 68  e5 58 6d a4 b1 85 29 68    2  0 = h Xm   )h
00001036    10 40 00 94 89 2e a2 33  29 15 45 26 52 ad 30 c6     @   . 3) E&R 0 
00001037    2c 89 62 b9 c8 93 1e 0e  04 f6 eb 06 cb e1 8e 68    , b            h
00001038    d6 fb 8f 6d ff c7 66 9d  4c ba cc a0 2a 7c 7b 08       m  f L   *|{ 
00001039    48 c5 99 11 1d c8 ee d8  c6 77 8b 93 2f 25 f2 5e    H        w  /% ^
00001040    3f 9f bb b2 95 a4 0a f4  43 9f c6 70 bd 22 f3 dc    ?       C  p "  
00001041    fa c4 c3 17 8d 86 8a 7e  5e 73 f6 83 60 6a 60 fe           ~^s  `j` 
00001042    58 89 ad 45 5f d3 b7 fb  fe ad 96 c6 08             X  E_           
```

#### **AudioTagHeader**

```xml
00001029                                         2e                         .
```

#### **AUDIODATA**

```xml
00001029                                            ff fb 50                   P
00001030    c4 10 00 0d 89 77 4b a3  18 ad c1 ae ac a9 74 63         wK       tc
00001031    0d b9 c6 6f ee 71 fb 29  2f 7e 75 f5 2c cc ed fe       o q )/~u ,   
00001032    71 e2 45 ce 25 51 37 3b  8d 43 0e 33 a5 4c 22 54    q E %Q7; C 3 L"T
00001033    56 4e f7 b3 9e 8d aa 31  25 d5 aa 2c 83 44 65 2d    VN     1%  , De-
00001034    e6 b3 0e ef ea 18 c4 ba  11 86 3b ed fb b9 b1 f5              ;     
00001035    32 a1 0a 30 8c 3d 90 68  e5 58 6d a4 b1 85 29 68    2  0 = h Xm   )h
00001036    10 40 00 94 89 2e a2 33  29 15 45 26 52 ad 30 c6     @   . 3) E&R 0 
00001037    2c 89 62 b9 c8 93 1e 0e  04 f6 eb 06 cb e1 8e 68    , b            h
00001038    d6 fb 8f 6d ff c7 66 9d  4c ba cc a0 2a 7c 7b 08       m  f L   *|{ 
00001039    48 c5 99 11 1d c8 ee d8  c6 77 8b 93 2f 25 f2 5e    H        w  /% ^
00001040    3f 9f bb b2 95 a4 0a f4  43 9f c6 70 bd 22 f3 dc    ?       C  p "  
00001041    fa c4 c3 17 8d 86 8a 7e  5e 73 f6 83 60 6a 60 fe           ~^s  `j` 
00001042    58 89 ad 45 5f d3 b7 fb  fe ad 96 c6 08
```

### **第 7 个 TAG 的长度**

```xml
00001042                                            00 00 00
00001043    dc
```

### **第 8 个 FLV TAG（VideoTag）**

```xml
00001043       09 00 10 39 00 00 50  00 00 00 00 22 00 00 84        9  P    "   
00001044    0a b1 3f ff ff ff c2 ff  28 83 e1 6b 42 61 46 81      ?     (  kBaF 
00001045    fc 0f dd 0b e1 cb 83 87  85 f0 e5 c1 c3 c2 c7 d2                    
00001046    4b 81 fb 83 87 85 8d 03  f9 43 06 97 2c 78 56 ce    K        C  ,xV 
00001047    8d 1a 5c b1 ff 63 76 17  e1 e3 a1 53 6c 35 9b 09      \  cv    Sl5  
00001048    26 37 b2 f1 b8 fe b1 26  86 d4 9a d4 ab 07 e9 cc    &7     &        
00001049    05 65 29 5b 2b af db 9e  e4 fa 7b b7 8d c5 06 ec     e)[+     {     
00001050    ef 94 ce f8 2b 1a f5 dc  5e 2f bd 14 de 94 4e e4        +   ^/    N 
00001051    02 34 e0 51 6f 5b 7b ab  e6 22 72 fb 0b 58 5b a2     4 Qo[{  "r  X[ 
00001052    a9 61 4b 57 22 7b 75 85  ce 8d ad ef 68 de ac 70     aKW"{u     h  p
00001053    65 e9 f7 9d dd 9b 2b 73  8f 8e 73 a7 69 f4 fb 1f    e     +s  s i   
00001054    63 de fb 39 d3 a8 bc 79  dc 7f 0a 17 0f b8 e3 c5    c  9   y        
00001055    1b bc 7a a6 f7 b1 bc 28  be 66 19 7a fa b9 4a f1      z    ( f z  J 
00001056    e8 ef 79 95 6a fe de 5a  d7 36 2f 21 80 a5 f6 f7      y j  Z 6/!    
00001057    19 99 2c 60 75 8d 44 d2  1c 69 1c bb 4a 25 e9 a2      ,`u D  i  J%  
00001058    cd a4 cd f3 ad ea d3 76  af 5c b6 58 d6 2d 64 b1           v \ X -d 
00001059    23 c2 cf b7 67 24 c4 12  48 c6 c2 ce ec 27 46 98    #   g$  H    'F 
00001060    ed 41 4b 3f d9 d6 1d 9d  99 79 36 72 f6 72 ec 66     AK?     y6r r f
00001061    b8 29 79 33 1b 47 8b 37  cb 25 6d 68 71 0e a3 95     )y3 G 7 %mhq   
00001062    23 5a 83 4e 2d 32 71 be  76 f2 99 6f 89 24 fd d6    #Z N-2q v  o $  
00001063    b1 8d 59 2b d9 1b 4c 36  14 7a d2 cb 7a dc ec b3      Y+  L6 z  z   
00001064    bc 82 6c 41 4a 71 be ee  3e f7 90 a6 b3 75 7b 71      lAJq  >    u{q
00001065    62 4d c9 64 a9 e5 5e 2f  21 b6 db b1 6b 16 ab 5f    bM d  ^/!   k  _
00001066    de 16 b4 4c 3f 89 1a b1  c1 7e 1e a7 e1 63 b0 33       L?    ~   c 3
00001067    d7 7e 3e 2f 75 bd d8 e3  75 af 53 73 be b7 79 c1     ~>/u   u Ss  y 
00001068    b6 86 d2 bc 28 bb bd 88  d8 9d 60 aa 61 84 42 1b        (     ` a B 
00001069    7f 43 d3 51 86 4a a6 71  06 10 c0 1f ab 2f 18 24     C Q J q     / $
00001070    0b 38 74 69 b3 7b 25 ba  9f 6d 4e 92 53 c3 6c c4     8ti {%  mN S l 
00001071    c9 54 1f 8d f7 bc ec 69  be 3e 9f 1f d3 f1 97 34     T     i >     4
00001072    69 cf 14 fe 3e eb 1d fc  28 5d 3f ba b3 39 8d fe    i   >   (]?  9  
00001073    d7 e6 dd c9 6c f4 df e4  dc 8b bf 7e 4e 15 36 83        l      ~N 6 
00001074    db ce 95 5d bc 17 79 4f  72 6e a7 f6 77 83 92 ce       ]  yOrn  w   
00001075    b0 fc 42 cf 26 8c ef 0d  c4 7e b5 f6 69 4d c9 ad      B &    ~  iM  
00001076    36 6d 25 a9 23 12 6e ad  35 a7 6a ca 64 cb 98 8e    6m% # n 5 j d   
00001077    5a 51 2a db 98 6a d8 55  39 96 27 8c 63 ba 9d 25    ZQ*  j U9 ' c  %
00001078    91 32 f5 da fb eb bf f8  db 89 97 2b bc 9d 5f 58     2         +  _X
00001079    da 44 b6 dc 89 8c 88 5f  2d c6 fb 72 47 4f 4a 8d     D     _-  rGOJ 
00001080    bc 88 91 4c c6 57 d9 c3  5d f6 35 74 a6 6c 59 9a       L W  ] 5t lY 
00001081    bc 9b 6b b9 85 93 3b 16  cb 99 17 cd ec 62 c7 05      k   ;      b  
00001082    3f f9 2e 8e e1 67 6b 4d  59 b8 bf cf 60 eb 85 36    ? .  gkMY   `  6
00001083    f6 35 56 86 52 ee 44 fb  b2 de 5e 66 b9 a4 c8 ac     5V R D   ^f    
00001084    41 c5 f3 bb d8 72 d1 cb  43 68 df b7 72 d6 26 1a    A    r  Ch  r & 
00001085    ee a1 34 23 f6 a7 bb 89  e1 45 3f db 79 da 8e 5d      4#     E? y  ]
00001086    ca db 58 45 79 71 f7 3b  94 07 e3 56 2f d9 8c 2e      XEyq ;   V/  .
00001087    9d 03 c2 9f 63 72 55 28  11 37 de d6 24 8e 61 2e        crU( 7  $ a.
00001088    d6 6e 16 cb 08 fa b5 5e  59 2e 25 b0 72 6f 01 17     n     ^Y.% ro  
00001089    37 39 b2 09 ec c4 cd f6  65 5f b5 9b 98 d3 71 3b    79      e_    q;
00001090    b7 bd d9 dc de 95 9c 0a  78 8e cd f2 5b c2 c9 c5            x   [   
00001091    91 70 e6 22 ec 62 b6 c4  91 b8 c5 47 4e 34 87 32     p " b     GN4 2
00001092    24 4f 7b 56 db 34 cf 6a  2d 85 7a b4 cb 71 05 c6    $O{V 4 j- z  q  
00001093    8f f2 56 d1 d6 d0 4e bf  72 cb e6 e9 4a 0d 91 3b      V   N r   J  ;
00001094    82 8b 63 b7 24 29 5b 76  1d 4f 47 b1 04 45 da 14      c $)[v OG  E  
00001095    07 9e 97 58 97 3f 31 33  55 ab 30 e6 eb 59 ba d6       X ?13U 0  Y  
00001096    2e b7 79 6a f8 69 92 8c  ea 0d 28 81 98 51 e8 5a    . yj i    (  Q Z
00001097    dd 44 dc bb 37 86 7d 66  88 a3 fb d9 5b df 45 f3     D  7 }f    [ E 
00001098    6b 17 b7 4e ec 66 15 f7  6d bb 4a 23 b6 56 72 a4    k  N f  m J# Vr 
00001099    75 52 e1 4f 23 e4 79 ca  5f 1f af d7 a7 d7 1e 95    uR O# y _       
00001100    fa f8 9b af af a4 e5 7c  3f 39 c3 e1 4f 9d 26 99           |?9  O & 
00001101    49 99 c3 5f c9 5d dc 37  78 60 4f f6 be de 93 75    I  _ ] 7x`O    u
00001102    3d 1a 05 17 92 bf e6 76  c8 9f ed f3 1a 95 26 4e    =      v      &N
00001103    f3 b5 be 4a f4 c8 51 46  d6 69 9e c8 e9 c6 ee 25       J  QF i     %
00001104    28 92 6d af 0a 7d cd 31  2a 5e b1 0a 6e 22 49 34    ( m  } 1*^  n"I4
00001105    ce 36 53 b1 0b 13 2e 4b  9d db 0f 4d 6d 64 94 35     6S   .K   Mmd 5
00001106    b6 5c 89 dc da 2c cc dd  66 4c b4 a2 3d 0f 6b 75     \   ,  fL  = ku
00001107    ac 70 53 f8 cb 7b 6a c9  18 c9 17 ce ec 77 22 64     pS  {j      w"d
00001108    41 cb 17 32 c2 11 0d a9  6a d5 07 2e 31 a6 f9 bd    A  2    j  .1   
00001109    d5 12 33 b2 2f 59 9c a9  7b 76 cb 9a fa b2 2c c5      3 /Y  {v    , 
00001110    b6 ea 76 aa dc dd cb 0f  ed 5e f2 27 ba be 59 b6      v      ^ '  Y 
00001111    94 d9 4c 85 3f 54 d9 93  31 1a f3 1a c4 f3 b7 64      L ?T  1      d
00001112    30 de 34 92 b4 86 23 63  76 f5 aa eb be 11 2e e1    0 4   #cv     . 
00001113    ad d1 16 ce 2d 2c 9c ad  66 32 98 ed 5e cc 67 66        -,  f2  ^ gf
00001114    0d b7 25 5d 06 d7 a1 b2  16 c9 ce c5 b7 31 34 30      %]         140
00001115    14 7e b3 16 45 b8 8a 5b  6d dd b8 6d 78 b7 10 5c     ~  E  [m  mx  \
00001116    99 3b 91 9b 64 27 e2 e8  b1 99 61 bd d8 9a 2d 2f     ;  d'    a   -/
00001117    1b b2 e4 5b 5c 92 6e 62  ed f3 ab 74 f8 d7 c9 f6       [\ nb   t    
00001118    2d f2 34 f0 a6 56 b3 6d  b0 c7 53 e2 4b db 08 38    - 4  V m  S K  8
00001119    8e 94 5b 67 39 d5 e6 da  78 29 ed e7 5b db cb 18      [g9   x)  [   
00001120    c8 ca 5a c6 72 61 eb 97  8b 23 c6 39 13 cb 9a 6b      Z ra   # 9   k
00001121    7b ce 63 e2 4c 4b 6e b3  18 b7 b5 ab 3a c3 9b a5    { c LKn     :   
00001122    76 48 9b 24 9e 96 1a bc  e4 b1 89 11 6c ec dd 3a    vH $        l  :
00001123    05 3d fd 5f 18 fe 2c a6  6f 51 52 87 45 32 eb 73     = _  , oQR E2 s
00001124    2a 8b cd 9a 49 3b b1 26  5a d5 bb 16 77 91 64 67    *   I; &Z   w dg
00001125    36 d4 36 ca 99 b9 9b 6e  c8 e9 18 5b 71 16 94 43    6 6    n   [q  C
00001126    1a c8 70 b7 64 e8 16 0c  82 9f 3b 13 31 53 26 6a      p d     ; 1S&j
00001127    3e 48 c4 15 73 36 4d 5f  07 bf de 2d a6 bb d4 97    >H  s6M_   -    
00001128    44 1e 59 75 bb d8 4f d4  b5 66 39 19 cd a3 1b ce    D Yu  O  f9     
00001129    e4 ad b7 91 0e 6d 40 6c  29 fa cc 69 2c e8 44 dd         m@l)  i, D 
00001130    2c e5 c9 a6 6d ba de 24  13 f6 2f 9c 85 52 64 db    ,   m  $  /  Rd 
00001131    6f 22 f6 be 44 35 02 eb  59 6b 7b af a5 ab da c5    o"  D5  Yk{     
00001132    cb 99 27 6e f6 3f 47 da  fb 1f af ab d7 9e 57 3d      'n ?G       W=
00001133    1c cf a1 99 f8 e5 ee a5  75 db 33 2b fa f6 9e 67            u 3+   g
00001134    c9 c2 9f eb ac c7 28 e6  a7 e6 a0 9c 29 6a 49 6b          (     )jIk
00001135    36 d7 37 c4 f5 67 c5 1a  de e2 f3 64 9b 81 cc 3b    6 7  g     d   ;
00001136    21 44 dc 9b c5 f2 e2 66  66 4f 9f 65 16 5e ee 66    !D     ffO e ^ f
00001137    5d 88 0e 85 3d 79 71 a1  ce 60 c6 14 ce de d2 c9    ]   =yq  `      
00001138    23 08 38 da 83 b8 57 dd  a9 33 6d 96 a2 9d 63 5a    # 8   W  3m   cZ
00001139    8f 28 a5 8d dc 97 18 d9  13 ac e9 1a 4f 36 94 6a     (          O6 j
00001140    ed 58 69 25 9d 61 ae 67  2c 99 13 6a 7b 1e 14 fc     Xi% a g,  j{   
00001141    fa cc c9 3a d4 bd bb 6f  6d a9 f4 f7 2e a9 57 54       :   om   . WT
00001142    f5 68 59 1a f1 5d 79 64  e3 49 e6 de cb ce 6c da     hY  ]yd I    l 
00001143    22 6e 56 9c c4 b9 33 6d  bc 98 8b 43 6b dc af d9    "nV   3m   Ck   
00001144    75 9a 37 91 96 18 ec 36  db 12 b6 8d 2c 67 8b 36    u 7    6    ,g 6
00001145    70 29 eb dc 46 b7 53 2f  a8 d9 8c c7 f3 52 52 ba    p)  F S/     RR 
00001146    8a ef dd 63 13 2b 12 66  cd 41 da b6 49 6d bc 77       c + f A  Im w
00001147    c7 9c ca 3f 9e ac fa 45  3b d8 d7 f5 46 2f 9a 9b       ?   E;   F/  
00001148    b2 ba 64 88 d7 95 2d c6  7b 1a bc 8c 47 b6 dc 95      d   - {   G   
00001149    6a b2 4e e4 d7 05 1e b6  75 b6 56 d9 b2 9a b6 41    j N     u V    A
00001150    ce e6 ae a7 85 33 a4 ff  c4 b7 b2 e3 aa 7b 12 d9         3       {  
00001151    27 71 0d a4 ac 49 2e 84  81 49 ea 1e d8 85 c3 9b    'q   I.  I      
00001152    6c 0f 9a 5f 1d 73 2e ae  8a ef 1b 8c c4 16 12 f7    l  _ s.         
00001153    58 2a b3 89 6c 5e b2 ee  6c eb 7a 96 55 e5 70 53    X*  l^  l z U pS
00001154    c3 f8 99 17 2a 59 25 5d  cd 6c 6e 43 09 f9 98 d7        *Y%] lnC    
00001155    66 6a 74 49 5d d4 59 96  54 0b e1 dd 93 26 4c 97    fjtI] Y T    &L 
00001156    91 7d ce 27 93 25 60 c6  ce c4 77 7a c6 6c 47 bb     } ' %`   wz lG 
00001157    23 c2 9f bd 8d 71 7b ea  7b 64 c3 79 b2 5b bc 8d    #    q{ {d y [  
00001158    69 54 8f 47 11 b4 57 64  ee de ea ce 9a d2 70 ea    iT G  Wd      p 
00001159    b9 6e 73 18 6d 01 a0 a7  ad bd 28 84 55 69 b1 3e     ns m     ( Ui >
00001160    b3 25 92 b7 72 65 b3 5f  39 13 49 73 67 63 6b de     %  re _9 Isgck 
00001161    4b 7b 3b 48 03 6c 62 e2  5a 2c 45 2f 5a 4d 17 ce    K{;H lb Z,E/ZM  
00001162    dc f3 76 1b 5e 56 26 71  1a 45 f7 2b 1c 78 51 f5      v ^V&q E + xQ 
00001163    1d d9 ce 4c e9 65 e0 9e  66 37 c2 8d 4a d4 30 36       L e  f7  J 06
00001164    ce 2e 41 c4 bb 52 d4 51  06 f7 13 b9 7b b2 30 e0     .A  R Q    { 0 
00001165    a9 b7 36 02 2c ad 57 1e  e3 f9 2e 63 3d 96 35 43      6 , W   .c= 5C
00001166    99 72 b8 47 e3 f1 e7 b1  e7 31 46 ff 1e 0a b9 e8     r G     1F     
00001167    ed 75 8f ad f1 a5 ed c7  05 35 3f 19 93 2d b4 9a     u       5?  -  
00001168    c4 e8 60 b6 ec ce 49 b3  35 c9 0b 1c 96 44 6f 0a      `   I 5    Do 
00001169    7d be 44 9c ef c4 e8 33  19 8e 8d 4b d9 c2 2b db    } D    3   K  + 
00001170    75 8b c9 23 6f 4b 58 e9  20 53 ea 2c cc ed 8e 40    u  #oKX  S ,   @
00001171    8b 99 9d 99 88 ec 7c bf  bc 92 43 7e 53 1a a9 50          |   C~S  P
00001172    b1 b0 e6 27 ea 33 a8 71  15 3c 14 f9 85 88 3b c9       ' 3 q <    ; 
00001173    99 cd ed 89 77 99 5d c6  62 1e 37 58 d8 75 0c d2        w ] b 7X u  
00001174    b9 a9 ad 4b c6 ec ec 79  48 6b 67 52 fb 92 7f 9a       K   yHkgR    
00001175    75 0e e2 f7 a3 9c fc 89  e8 d4 71 38 5b 6f 1b cd    u         q8[o  
00001176    6f 36 f5 c2 3c 43 96 33  2b 56 54 f2 eb b1 28 6b    o6  <C 3+VT   (k
00001177    2a d3 62 d1 9c 95 d8 cb  4d e5 5a ae db 26 6c 6f    * b     M Z  &lo
00001178    5b 92 f7 b8 de cb a8 1b  5d 35 7e 87 cc 26 bb 89    [       ]5~  &  
00001179    d0 eb 0e 0a 7d 96 cd 48  8e 6b 1a 53 9a 9d c9 6e        }  H k S   n
00001180    a4 87 b1 14 c9 99 50 ee  a7 72 4d dc 61 8c c9 ca          P  rM a   
00001181    92 c4 15 fd ab dd 0a ec  cc 5a 54 d3 bb dd be 86             ZT     
00001182    42 93 3d db bc c9 32 d6  f2 16 4a f5 eb 1d ab 35    B =   2   J    5
00001183    27 7f 67 4d df 09 b6 32  1d 4d ec ec 25 0a 7d 8b    ' gM   2 M  % } 
00001184    e2 d2 5e 2f ab c4 53 21  cd d2 8a 82 4b 89 a9 8e      ^/  S!    K   
00001185    55 bb 91 64 47 77 5c 8b  91 8d 0e ab 11 6c 33 b6    U  dGw\      l3 
00001186    08 75 9e b2 9a ce da 60  29 ec d2 36 d1 4d d2 8d     u     `)  6 M  
00001187    6b 56 37 cb 73 82 b9 7f  ce ca d4 e4 5e 30 82 e4    kV7 s       ^0  
00001188    d5 e3 f9 6b 3a d5 5b 8c  48 9a dc 72 34 d1 9c 8d       k: [ H  r4   
00001189    59 2e 24 78 53 e7 fe 04  6b 1b 5a 0e 6d 59 dc 53    Y.$xS   k Z mY S
00001190    c4 41 b2 d3 32 4e 3b de  d9 b7 ed fd bc 2c 92 24     A  2N;      , $
00001191    ed c5 de 9e 49 36 69 56  d4 58 65 16 30 8a ad 96        I6iV Xe 0   
00001192    a1 d8 bf f7 ae 5f 73 71  86 24 4e 1e 73 d0 d0 53         _sq $N s  S
00001193    c3 82 dd cd ec 6a 9a e5  4e 61 8d 63 8c 5c d4 cc         j  Na c \  
00001194    46 d0 f4 9e f3 81 bc 88  e1 5a cf 1c 65 5b 98 c4    F        Z  e[  
00001195    66 dc ec 58 ce 6d d6 ac  dd 29 d4 5a 4c 14 fc f2    f  X m   ) ZL   
00001196    04 2c ec 89 ab 1c c2 2f  ea d8 81 1a db ba 9f 2d     ,     /       -
00001197    b4 f4 dc c4 97 78 df 4a  19 a4 fb 5a ad de 6c c3         x J   Z  l 
00001198    ad a7 99 99 26 2d 16 aa  25 d6 74 c3 10 39 4b 19        &-  % t  9K 
00001199    32 14 fa bc 41 c4 5d ac  5e f7 13 69 b2 da cc 0d    2   A ] ^  i    
00001200    f1 d1 b6 a7 52 31 68 e6  35 c9 69 17 1a 05 6a 36        R1h 5 i   j6
00001201    f9 72 76 bf 23 51 ed 44  ad ee f1 98 cd cd e4 ec     rv #Q D        
00001202    e6 c9 91 87 85 1f 8f 61  66 ad ec e2 67 e3 ed 67           af   g  g
00001203    99 65 cb 2f 1e 88 37 95  cd a5 6b ef 62 3e 57 e3     e /  7   k b>W 
00001204    ce c8 fd 75 dc 29 39 66  ee 5c c4 18 b2 ee f4 d9       u )9f \      
00001205    c5 b2 43 d1 e8 e2 33 36  68 da 5e 20 34 14 f9 b3      C   36h ^ 4   
00001206    8c e8 73 76 0d 0e b1 6b  7a da 4a 86 3e 34 53 ca      sv   kz J >4S 
00001207    36 b2 90 44 db 74 2b 49  cc 5e 70 64 14 ff f2 b3    6  D t+I ^pd    
00001208    9a c3 7e 47 47 2e 1d 75  7d ad f5 0d a2 8f 6a e9      ~GG. u}     j 
00001209    0e 40 f0 aa e4 c2 08 9a  ed c4 00 46 75 d7 25 8d     @         Fu % 
00001210    75 01 61 a1 1f b2 70 51  ba 9b 22 4d b9 ab 71 a1    u a   pQ  "M  q 
00001211    9d b6 b5 4b 77 94 8d a6  03 58 d6 f4 6f f7 2f 2f       Kw    X  o //
00001212    7b 3a 8c 2e 11 e9 8a 0b  43 cc 89 a4 7d 5d 84 bc    {: .    C   }]  
00001213    bb b8 fe 66 6c 65 3d 66  4e 23 4d a6 10 23 59 6a       fle=fN#M  #Yj
00001214    51 c6 72 5c 8f 43 4b 6e  4b 26 ef 24 ce 48 83 7e    Q r\ CKnK& $ H ~
00001215    74 29 f2 60 73 24 a5 39  da bd cd 8e c5 b9 db 33    t) `s$ 9       3
00001216    2e 9a a8 26 64 c4 d1 99  de eb 58 f4 91 8e 23 ad    .  &d     X   # 
00001217    de 6e 6b 79 2c c3 45 bd  41 39 12 76 b7 ce 29 da     nky, E A9 v  ) 
00001218    66 ce 4c 1c 67 46 21 4f  9a ed a8 1a 90 a7 64 43    f L gF!O      dC
00001219    30 8d 26 37 6e db b6 d7  96 67 58 e6 64 99 ac 74    0 &7n    gX d  t
00001220    a6 e3 da c4 c9 c5 5b da  81 ad 4d 1b dc 5f 13 98          [   M  _  
00001221    0b 3e 60 e6 3f 58 93 9a  59 45 0d ec 46 ba dc 19     >` ?X  YE  F   
00001222    35 b3 a8 60 7a b7 d2 1f  0a 7a 24 45 9d da ce 5b    5  `z    z$E   [
00001223    95 77 a4 68 3f 40 e4 36  56 f1 8b 65 d9 8c 4b 72     w h?@ 6V  e  Kr
00001224    ac 6e 07 6e 2d 8c e2 12  ad af 59 9b 75 1e 5b ca     n n-     Y u [ 
00001225    ba f2 37 1e 14 fa 9e 08  53 25 47 c6 33 65 dd 86      7     S%G 3e  
00001226    ad de b0 bc c4 0b 77 5b  6b 93 1e 91 ab 6f 26 58          w[k    o&X
00001227    1d 3d 3d a5 8f 4b 2b 2d  f5 99 92 6b 12 dd b6 33     ==  K+-   k   3
00001228    6d 3e 57 ad eb 6c b7 17  91 c1 4f d4 c9 a9 0d b6    m>W  l    O     
00001229    6c d6 3b 4a 73 36 59 cc  d5 9e bc 66 6e 72 c1 be    l ;Js6Y    fnr  
00001230    f7 25 91 7d a6 83 6d bb  b5 7d 63 62 ee ed e8 7a     % }  m  }cb   z
00001231    d3 2c 48 c4 e4 73 59 33  1b ab e4 4b 57 a4 a3 1e     ,H  sY3   KW   
00001232    83 56 d8 c2 6b 1b 1c de  f4 d4 e4 a8 ba cc 6e 59     V  k         nY
00001233    5a 6f 89 17 27 1b 74 b5  6c ea 4e 67 9d 69 57 6f    Zo  ' t l Ng iWo
00001234    58 ef 2a 73 a1 4f 6c 38  9d 19 25 0e f2 31 67 59    X *s Ol8  %  1gY
00001235    d9 dd 7a 4b 5a d6 5b 49  b3 26 6d 88 a5 94 ef 77      zKZ [I &m    w
00001236    72 b5 12 6d cb 27 2a 2b  37 2d 94 d1 42 e7 57 b2    r  m '*+7-  B W 
00001237    cd 24 0a 7e ad 65 4c cc  88 24 99 89 27 78 79 6d     $ ~ eL  $  'xym
00001238    cd 8b cc ba b7 38 da d2  ea 8a ee 45 e6 73 8b 33         8     E s 3
00001239    62 0d 72 c8 71 76 96 89  91 6b 98 5f 7a bf 26 2d    b r qv   k _z &-
00001240    8f b8 ca f2 76 5c ba 9f  f3 36 b8 29 b8 ee 24 95        v\   6 )  $ 
00001241    78 fc a4 b5 0a 39 6b bc  a3 cd b3 42 ab ca dc 41    x    9k    B   A
00001242    5f 64 5e a1 15 09 3d 65  ae a2 90 36 87 a4 9d 6f    _d^   =e   6   o
00001243    8b 54 03 6c c7 65 9b 36  5e ea d6 71 3c 78 cf d7     T l e 6^  q<x  
00001244    da f2 ab 67 67 10 d3 5b  d9 88 e3 7b 8b bc e2 97       gg  [   {    
00001245    be 1b 67 74 69 cb 1b 76  ed 59 e1 fe 4e 5c 69 99      gti  v Y  N\i 
00001246    a5 16 6c 4b 30 90 29 79  95 8b b2 f2 6f 50 7c 53      lK0 )y    oP|S
00001247    e9 73 14 8e 9a 9b db 12  cc de 6f 25 a6 f5 e5 54     s        o%   T
00001248    69 da c3 c2 9e cd 89 ea  2e 6e 6e 4b ad 2f 63 24    i       .nnK /c$
00001249    d0 a5 99 75 6b 9d 91 b9  54 f7 21 2d 85 99 99 53       uk   T !-   S
00001250    33 3b 2b 5c d6 e9 32 1a  bd d2 ab 56 63 30 d1 65    3;+\  2    Vc0 e
00001251    61 71 ce a4 e1 89 db 39  7b 33 25 93 1b 84 41 49    aq     9{3%   AI
00001252    b5 79 8b 41 c4 45 ca b1  2a 39 8d 6b 2d db 33 59     y A E  *9 k- 3Y
00001253    e7 ae e4 e6 c7 77 07 2b  dd 7a d6 6d b1 be c8 be         w + z m    
00001254    c1 bc 68 fe d8 b8 72 d6  66 b1 5b c9 25 3a 14 5a      h   r f [ %: Z
00001255    c8 30 41 97 62 6a ed c4  ed 62 f2 d6 6e c6 19 7b     0A bj   b  n  {
00001256    5b 76 0d a5 ac 56 b0 c2  39 b7 7e 43 d8 59 ad 21    [v   V  9 ~C Y !
00001257    c6 fb 3c b1 90 a7 a4 c4  32 21 5a 3d a4 9a 92 3a      <     2!Z=   :
00001258    20 da 53 bc e1 17 22 36  fa b5 dd c3 2d 45 d7 b1      S   "6    -E  
00001259    aa ed f3 79 6d d6 8a e7  96 a4 c1 4f 63 bc cd d9       ym      Oc   
00001260    96 5c 21 8c c9 bc 91 8b  18 42 59 9a c1 ee 58 c3     \!      BY   X 
00001261    72 a1 ea 98 91 a9 1c 54  9a db d9 56 96 34 4f 9a    r      T   V 4O 
00001262    85 79 73 37 1a bb 31 73  8c 23 b7 89 a2 f5 ae ad     ys7  1s #      
00001263    51 76 b8 29 f1 1f 37 52  d8 c2 67 dc 5a f5 6b 97    Qv )  7R  g Z k 
00001264    6c bb bc 4d 35 0c da f2  a5 90 6d d2 d8 be cc dc    l  M5     m     
00001265    78 6e 8a cc 67 8e 9c bd  4c c6 4e a4 e4 eb 77 65    xn  g   L N   we
00001266    67 cf 5a e2 3d dc bb 36  ae dc d9 8b cb 16 7a 78    g Z =  6      zx
00001267    48 81 b9 11 25 b7 88 7f  e7 6b 17 72 f3 79 12 54    H   %    k r y T
00001268    fd bc a9 b6 bc 71 b2 49  50 d6 bb a3 30 a5 f1 7e         q IP   0  ~
00001269    da 98 38 46 e4 9c ee 8d  b6 f7 10 75 31 b4 85 9d      8F       u1   
00001270    b8 8b 4f 0c b9 dc d7 85  3e 36 dd 45 c9 3a 99 9c      O     >6 E :  
00001271    e7 58 b3 09 2d 98 da 26  2d a9 b9 88 fd 5f 3b 0a     X  -  &-    _; 
00001272    b9 b3 6a ec c4 55 fb 68  77 2f 26 ac 8e 53 f9 6e      j  U hw/&  S n
00001273    d9 ad 35 9c ea 48 79 17  17 9b bc 23 0a 7c 86 a2      5  Hy    # |  
00001274    6a d2 c3 6c 99 13 1a 93  f9 32 fa d6 6f 07 15 f2    j  l     2  o   
00001275    5f 58 b5 a1 cf 5c dd 49  bc 13 2d 6f 25 d6 ad 3c    _X   \ I  -o%  <
00001276    14 f8 bc ec 5b a2 69 d6  e5 de 07 ae 6a b5 66 c6        [ i     j f 
00001277    3b 05 d7 3b 94 35 d7 f4  13 b8 20 5d 85 8f 0a 7f    ;  ; 5     ]    
00001278    96 df 44 f5 c8 a9 66 27  c6 a5 4f bd 8f 4f b1 24      D   f'  O  O $
00001279    58 25 93 97 95 2b 14 84  b1 a2 b4 c7 f7 bc 63 49    X%   +        cI
00001280    93 e4 92 ed ce f2 71 bb  24 d9 12 43 fb 3b cb 9b          q $  C ;  
00001281    32 cf b5 08 93 a2 dd cb  69 0b 19 78 1a 3c 29 f1    2       i  x <) 
00001282    25 62 4c 9c 6a 75 1d 7d  e5 9a ce d6 ac 43 75 bd    %bL ju }     Cu 
00001283    62 69 82 ae ec 46 c6 ba  a1 85 32 f6 95 dd 7e ad    bi   F    2   ~ 
00001284    d6 1a b9 70 8e 4c dd ab  56 e6 9d 49 ca fd 7e bd       p L  V  I  ~ 
00001285    d2 6f d7 dc 62 42 f5 fa  f0 ad 54 cb 77 1e 71 2f     o  bB    T w q/
00001286    39 67 4b 5d 65 cb 0a fa  55 b8 89 f0 d8 df ee e4    9gK]e   U       
00001287    43 46 f2 96 f5 fa fe 32  de b7 73 a8 64 9f 19 85    CF     2  s d   
00001288    9d 98 55 2c 0e 39 da e4  45 17 7b 6d 8d c4 53 0e      U, 9  E {m  S 
00001289    1d 99 b7 22 59 77 64 c1  90 cf de a6 ce 34 5b 49       "Ywd      4[I
00001290    0b 37 31 87 56 72 20 1b  51 63 0b 2c bb f3 96 a3     71 Vr  Qc ,    
00001291    30 14 ba e9 76 44 d6 cb  22 27 31 52 65 5f 2d e9    0   vD  "'1Re_- 
00001292    12 e8 5a 67 49 71 35 5b  57 c9 32 4d 22 62 5c b2      ZgIq5[W 2M"b\ 
00001293    ea fc db db b8 b6 db 8e  19 f6 96 95 ea d1 f9 6e                   n
00001294    75 fb 8f 8f eb c4 2e 43  f6 1c 76 64 7a 4c ed 8f    u     .C  vdzL  
00001295    c7 c4 7f 81 58 c6 5e 61  06 33 db 64 58 73 9d a9        X ^a 3 dXs  
00001296    eb 8f 57 2d 94 60 15 f8  53 c6 e5 37 1c cb 18 13      W- `  S  7    
00001297    ed b1 b9 70 80 29 e2 0a  64 56 c1 92 7a b4 5e e9       p )  dV  z ^ 
00001298    13 6d cd 0d 2e 8c 36 a0  81 a6 a3 72 2d 91 6b 0e     m  . 6    r- k 
00001299    6d 27 19 f3 61 45 a5 35  78 4b db 99 b5 0d 2a b8    m'  aE 5xK    * 
00001300    f8 9e 58 83 e3 67 0d ec  e3 99 67 12 c9 85 0f fe      X  g    g     
00001301    39 ea 51 71 a2 c4 10 59  c2 cf 42 ae 40 c2 a5 e1    9 Qq   Y  B @   
00001302    09 62 08 2c c5 e5 a8 68  b4 5e 5a 86 8b 71 79 6a     b ,   h ^Z  qyj
00001303    1a 2d ff ff f8                                       -
```

#### **VideoTagHeader**

```xml
00001043                                         22                         "   
```

#### **VIDEODATA**

```xml
00001043                                            00 00 84
00001044    0a b1 3f ff ff ff c2 ff  28 83 e1 6b 42 61 46 81      ?     (  kBaF 
00001045    fc 0f dd 0b e1 cb 83 87  85 f0 e5 c1 c3 c2 c7 d2                    
00001046    4b 81 fb 83 87 85 8d 03  f9 43 06 97 2c 78 56 ce    K        C  ,xV 
00001047    8d 1a 5c b1 ff 63 76 17  e1 e3 a1 53 6c 35 9b 09      \  cv    Sl5  
00001048    26 37 b2 f1 b8 fe b1 26  86 d4 9a d4 ab 07 e9 cc    &7     &        
00001049    05 65 29 5b 2b af db 9e  e4 fa 7b b7 8d c5 06 ec     e)[+     {     
00001050    ef 94 ce f8 2b 1a f5 dc  5e 2f bd 14 de 94 4e e4        +   ^/    N 
00001051    02 34 e0 51 6f 5b 7b ab  e6 22 72 fb 0b 58 5b a2     4 Qo[{  "r  X[ 
00001052    a9 61 4b 57 22 7b 75 85  ce 8d ad ef 68 de ac 70     aKW"{u     h  p
00001053    65 e9 f7 9d dd 9b 2b 73  8f 8e 73 a7 69 f4 fb 1f    e     +s  s i   
00001054    63 de fb 39 d3 a8 bc 79  dc 7f 0a 17 0f b8 e3 c5    c  9   y        
00001055    1b bc 7a a6 f7 b1 bc 28  be 66 19 7a fa b9 4a f1      z    ( f z  J 
00001056    e8 ef 79 95 6a fe de 5a  d7 36 2f 21 80 a5 f6 f7      y j  Z 6/!    
00001057    19 99 2c 60 75 8d 44 d2  1c 69 1c bb 4a 25 e9 a2      ,`u D  i  J%  
00001058    cd a4 cd f3 ad ea d3 76  af 5c b6 58 d6 2d 64 b1           v \ X -d 
00001059    23 c2 cf b7 67 24 c4 12  48 c6 c2 ce ec 27 46 98    #   g$  H    'F 
00001060    ed 41 4b 3f d9 d6 1d 9d  99 79 36 72 f6 72 ec 66     AK?     y6r r f
00001061    b8 29 79 33 1b 47 8b 37  cb 25 6d 68 71 0e a3 95     )y3 G 7 %mhq   
00001062    23 5a 83 4e 2d 32 71 be  76 f2 99 6f 89 24 fd d6    #Z N-2q v  o $  
00001063    b1 8d 59 2b d9 1b 4c 36  14 7a d2 cb 7a dc ec b3      Y+  L6 z  z   
00001064    bc 82 6c 41 4a 71 be ee  3e f7 90 a6 b3 75 7b 71      lAJq  >    u{q
00001065    62 4d c9 64 a9 e5 5e 2f  21 b6 db b1 6b 16 ab 5f    bM d  ^/!   k  _
00001066    de 16 b4 4c 3f 89 1a b1  c1 7e 1e a7 e1 63 b0 33       L?    ~   c 3
00001067    d7 7e 3e 2f 75 bd d8 e3  75 af 53 73 be b7 79 c1     ~>/u   u Ss  y 
00001068    b6 86 d2 bc 28 bb bd 88  d8 9d 60 aa 61 84 42 1b        (     ` a B 
00001069    7f 43 d3 51 86 4a a6 71  06 10 c0 1f ab 2f 18 24     C Q J q     / $
00001070    0b 38 74 69 b3 7b 25 ba  9f 6d 4e 92 53 c3 6c c4     8ti {%  mN S l 
00001071    c9 54 1f 8d f7 bc ec 69  be 3e 9f 1f d3 f1 97 34     T     i >     4
00001072    69 cf 14 fe 3e eb 1d fc  28 5d 3f ba b3 39 8d fe    i   >   (]?  9  
00001073    d7 e6 dd c9 6c f4 df e4  dc 8b bf 7e 4e 15 36 83        l      ~N 6 
00001074    db ce 95 5d bc 17 79 4f  72 6e a7 f6 77 83 92 ce       ]  yOrn  w   
00001075    b0 fc 42 cf 26 8c ef 0d  c4 7e b5 f6 69 4d c9 ad      B &    ~  iM  
00001076    36 6d 25 a9 23 12 6e ad  35 a7 6a ca 64 cb 98 8e    6m% # n 5 j d   
00001077    5a 51 2a db 98 6a d8 55  39 96 27 8c 63 ba 9d 25    ZQ*  j U9 ' c  %
00001078    91 32 f5 da fb eb bf f8  db 89 97 2b bc 9d 5f 58     2         +  _X
00001079    da 44 b6 dc 89 8c 88 5f  2d c6 fb 72 47 4f 4a 8d     D     _-  rGOJ 
00001080    bc 88 91 4c c6 57 d9 c3  5d f6 35 74 a6 6c 59 9a       L W  ] 5t lY 
00001081    bc 9b 6b b9 85 93 3b 16  cb 99 17 cd ec 62 c7 05      k   ;      b  
00001082    3f f9 2e 8e e1 67 6b 4d  59 b8 bf cf 60 eb 85 36    ? .  gkMY   `  6
00001083    f6 35 56 86 52 ee 44 fb  b2 de 5e 66 b9 a4 c8 ac     5V R D   ^f    
00001084    41 c5 f3 bb d8 72 d1 cb  43 68 df b7 72 d6 26 1a    A    r  Ch  r & 
00001085    ee a1 34 23 f6 a7 bb 89  e1 45 3f db 79 da 8e 5d      4#     E? y  ]
00001086    ca db 58 45 79 71 f7 3b  94 07 e3 56 2f d9 8c 2e      XEyq ;   V/  .
00001087    9d 03 c2 9f 63 72 55 28  11 37 de d6 24 8e 61 2e        crU( 7  $ a.
00001088    d6 6e 16 cb 08 fa b5 5e  59 2e 25 b0 72 6f 01 17     n     ^Y.% ro  
00001089    37 39 b2 09 ec c4 cd f6  65 5f b5 9b 98 d3 71 3b    79      e_    q;
00001090    b7 bd d9 dc de 95 9c 0a  78 8e cd f2 5b c2 c9 c5            x   [   
00001091    91 70 e6 22 ec 62 b6 c4  91 b8 c5 47 4e 34 87 32     p " b     GN4 2
00001092    24 4f 7b 56 db 34 cf 6a  2d 85 7a b4 cb 71 05 c6    $O{V 4 j- z  q  
00001093    8f f2 56 d1 d6 d0 4e bf  72 cb e6 e9 4a 0d 91 3b      V   N r   J  ;
00001094    82 8b 63 b7 24 29 5b 76  1d 4f 47 b1 04 45 da 14      c $)[v OG  E  
00001095    07 9e 97 58 97 3f 31 33  55 ab 30 e6 eb 59 ba d6       X ?13U 0  Y  
00001096    2e b7 79 6a f8 69 92 8c  ea 0d 28 81 98 51 e8 5a    . yj i    (  Q Z
00001097    dd 44 dc bb 37 86 7d 66  88 a3 fb d9 5b df 45 f3     D  7 }f    [ E 
00001098    6b 17 b7 4e ec 66 15 f7  6d bb 4a 23 b6 56 72 a4    k  N f  m J# Vr 
00001099    75 52 e1 4f 23 e4 79 ca  5f 1f af d7 a7 d7 1e 95    uR O# y _       
00001100    fa f8 9b af af a4 e5 7c  3f 39 c3 e1 4f 9d 26 99           |?9  O & 
00001101    49 99 c3 5f c9 5d dc 37  78 60 4f f6 be de 93 75    I  _ ] 7x`O    u
00001102    3d 1a 05 17 92 bf e6 76  c8 9f ed f3 1a 95 26 4e    =      v      &N
00001103    f3 b5 be 4a f4 c8 51 46  d6 69 9e c8 e9 c6 ee 25       J  QF i     %
00001104    28 92 6d af 0a 7d cd 31  2a 5e b1 0a 6e 22 49 34    ( m  } 1*^  n"I4
00001105    ce 36 53 b1 0b 13 2e 4b  9d db 0f 4d 6d 64 94 35     6S   .K   Mmd 5
00001106    b6 5c 89 dc da 2c cc dd  66 4c b4 a2 3d 0f 6b 75     \   ,  fL  = ku
00001107    ac 70 53 f8 cb 7b 6a c9  18 c9 17 ce ec 77 22 64     pS  {j      w"d
00001108    41 cb 17 32 c2 11 0d a9  6a d5 07 2e 31 a6 f9 bd    A  2    j  .1   
00001109    d5 12 33 b2 2f 59 9c a9  7b 76 cb 9a fa b2 2c c5      3 /Y  {v    , 
00001110    b6 ea 76 aa dc dd cb 0f  ed 5e f2 27 ba be 59 b6      v      ^ '  Y 
00001111    94 d9 4c 85 3f 54 d9 93  31 1a f3 1a c4 f3 b7 64      L ?T  1      d
00001112    30 de 34 92 b4 86 23 63  76 f5 aa eb be 11 2e e1    0 4   #cv     . 
00001113    ad d1 16 ce 2d 2c 9c ad  66 32 98 ed 5e cc 67 66        -,  f2  ^ gf
00001114    0d b7 25 5d 06 d7 a1 b2  16 c9 ce c5 b7 31 34 30      %]         140
00001115    14 7e b3 16 45 b8 8a 5b  6d dd b8 6d 78 b7 10 5c     ~  E  [m  mx  \
00001116    99 3b 91 9b 64 27 e2 e8  b1 99 61 bd d8 9a 2d 2f     ;  d'    a   -/
00001117    1b b2 e4 5b 5c 92 6e 62  ed f3 ab 74 f8 d7 c9 f6       [\ nb   t    
00001118    2d f2 34 f0 a6 56 b3 6d  b0 c7 53 e2 4b db 08 38    - 4  V m  S K  8
00001119    8e 94 5b 67 39 d5 e6 da  78 29 ed e7 5b db cb 18      [g9   x)  [   
00001120    c8 ca 5a c6 72 61 eb 97  8b 23 c6 39 13 cb 9a 6b      Z ra   # 9   k
00001121    7b ce 63 e2 4c 4b 6e b3  18 b7 b5 ab 3a c3 9b a5    { c LKn     :   
00001122    76 48 9b 24 9e 96 1a bc  e4 b1 89 11 6c ec dd 3a    vH $        l  :
00001123    05 3d fd 5f 18 fe 2c a6  6f 51 52 87 45 32 eb 73     = _  , oQR E2 s
00001124    2a 8b cd 9a 49 3b b1 26  5a d5 bb 16 77 91 64 67    *   I; &Z   w dg
00001125    36 d4 36 ca 99 b9 9b 6e  c8 e9 18 5b 71 16 94 43    6 6    n   [q  C
00001126    1a c8 70 b7 64 e8 16 0c  82 9f 3b 13 31 53 26 6a      p d     ; 1S&j
00001127    3e 48 c4 15 73 36 4d 5f  07 bf de 2d a6 bb d4 97    >H  s6M_   -    
00001128    44 1e 59 75 bb d8 4f d4  b5 66 39 19 cd a3 1b ce    D Yu  O  f9     
00001129    e4 ad b7 91 0e 6d 40 6c  29 fa cc 69 2c e8 44 dd         m@l)  i, D 
00001130    2c e5 c9 a6 6d ba de 24  13 f6 2f 9c 85 52 64 db    ,   m  $  /  Rd 
00001131    6f 22 f6 be 44 35 02 eb  59 6b 7b af a5 ab da c5    o"  D5  Yk{     
00001132    cb 99 27 6e f6 3f 47 da  fb 1f af ab d7 9e 57 3d      'n ?G       W=
00001133    1c cf a1 99 f8 e5 ee a5  75 db 33 2b fa f6 9e 67            u 3+   g
00001134    c9 c2 9f eb ac c7 28 e6  a7 e6 a0 9c 29 6a 49 6b          (     )jIk
00001135    36 d7 37 c4 f5 67 c5 1a  de e2 f3 64 9b 81 cc 3b    6 7  g     d   ;
00001136    21 44 dc 9b c5 f2 e2 66  66 4f 9f 65 16 5e ee 66    !D     ffO e ^ f
00001137    5d 88 0e 85 3d 79 71 a1  ce 60 c6 14 ce de d2 c9    ]   =yq  `      
00001138    23 08 38 da 83 b8 57 dd  a9 33 6d 96 a2 9d 63 5a    # 8   W  3m   cZ
00001139    8f 28 a5 8d dc 97 18 d9  13 ac e9 1a 4f 36 94 6a     (          O6 j
00001140    ed 58 69 25 9d 61 ae 67  2c 99 13 6a 7b 1e 14 fc     Xi% a g,  j{   
00001141    fa cc c9 3a d4 bd bb 6f  6d a9 f4 f7 2e a9 57 54       :   om   . WT
00001142    f5 68 59 1a f1 5d 79 64  e3 49 e6 de cb ce 6c da     hY  ]yd I    l 
00001143    22 6e 56 9c c4 b9 33 6d  bc 98 8b 43 6b dc af d9    "nV   3m   Ck   
00001144    75 9a 37 91 96 18 ec 36  db 12 b6 8d 2c 67 8b 36    u 7    6    ,g 6
00001145    70 29 eb dc 46 b7 53 2f  a8 d9 8c c7 f3 52 52 ba    p)  F S/     RR 
00001146    8a ef dd 63 13 2b 12 66  cd 41 da b6 49 6d bc 77       c + f A  Im w
00001147    c7 9c ca 3f 9e ac fa 45  3b d8 d7 f5 46 2f 9a 9b       ?   E;   F/  
00001148    b2 ba 64 88 d7 95 2d c6  7b 1a bc 8c 47 b6 dc 95      d   - {   G   
00001149    6a b2 4e e4 d7 05 1e b6  75 b6 56 d9 b2 9a b6 41    j N     u V    A
00001150    ce e6 ae a7 85 33 a4 ff  c4 b7 b2 e3 aa 7b 12 d9         3       {  
00001151    27 71 0d a4 ac 49 2e 84  81 49 ea 1e d8 85 c3 9b    'q   I.  I      
00001152    6c 0f 9a 5f 1d 73 2e ae  8a ef 1b 8c c4 16 12 f7    l  _ s.         
00001153    58 2a b3 89 6c 5e b2 ee  6c eb 7a 96 55 e5 70 53    X*  l^  l z U pS
00001154    c3 f8 99 17 2a 59 25 5d  cd 6c 6e 43 09 f9 98 d7        *Y%] lnC    
00001155    66 6a 74 49 5d d4 59 96  54 0b e1 dd 93 26 4c 97    fjtI] Y T    &L 
00001156    91 7d ce 27 93 25 60 c6  ce c4 77 7a c6 6c 47 bb     } ' %`   wz lG 
00001157    23 c2 9f bd 8d 71 7b ea  7b 64 c3 79 b2 5b bc 8d    #    q{ {d y [  
00001158    69 54 8f 47 11 b4 57 64  ee de ea ce 9a d2 70 ea    iT G  Wd      p 
00001159    b9 6e 73 18 6d 01 a0 a7  ad bd 28 84 55 69 b1 3e     ns m     ( Ui >
00001160    b3 25 92 b7 72 65 b3 5f  39 13 49 73 67 63 6b de     %  re _9 Isgck 
00001161    4b 7b 3b 48 03 6c 62 e2  5a 2c 45 2f 5a 4d 17 ce    K{;H lb Z,E/ZM  
00001162    dc f3 76 1b 5e 56 26 71  1a 45 f7 2b 1c 78 51 f5      v ^V&q E + xQ 
00001163    1d d9 ce 4c e9 65 e0 9e  66 37 c2 8d 4a d4 30 36       L e  f7  J 06
00001164    ce 2e 41 c4 bb 52 d4 51  06 f7 13 b9 7b b2 30 e0     .A  R Q    { 0 
00001165    a9 b7 36 02 2c ad 57 1e  e3 f9 2e 63 3d 96 35 43      6 , W   .c= 5C
00001166    99 72 b8 47 e3 f1 e7 b1  e7 31 46 ff 1e 0a b9 e8     r G     1F     
00001167    ed 75 8f ad f1 a5 ed c7  05 35 3f 19 93 2d b4 9a     u       5?  -  
00001168    c4 e8 60 b6 ec ce 49 b3  35 c9 0b 1c 96 44 6f 0a      `   I 5    Do 
00001169    7d be 44 9c ef c4 e8 33  19 8e 8d 4b d9 c2 2b db    } D    3   K  + 
00001170    75 8b c9 23 6f 4b 58 e9  20 53 ea 2c cc ed 8e 40    u  #oKX  S ,   @
00001171    8b 99 9d 99 88 ec 7c bf  bc 92 43 7e 53 1a a9 50          |   C~S  P
00001172    b1 b0 e6 27 ea 33 a8 71  15 3c 14 f9 85 88 3b c9       ' 3 q <    ; 
00001173    99 cd ed 89 77 99 5d c6  62 1e 37 58 d8 75 0c d2        w ] b 7X u  
00001174    b9 a9 ad 4b c6 ec ec 79  48 6b 67 52 fb 92 7f 9a       K   yHkgR    
00001175    75 0e e2 f7 a3 9c fc 89  e8 d4 71 38 5b 6f 1b cd    u         q8[o  
00001176    6f 36 f5 c2 3c 43 96 33  2b 56 54 f2 eb b1 28 6b    o6  <C 3+VT   (k
00001177    2a d3 62 d1 9c 95 d8 cb  4d e5 5a ae db 26 6c 6f    * b     M Z  &lo
00001178    5b 92 f7 b8 de cb a8 1b  5d 35 7e 87 cc 26 bb 89    [       ]5~  &  
00001179    d0 eb 0e 0a 7d 96 cd 48  8e 6b 1a 53 9a 9d c9 6e        }  H k S   n
00001180    a4 87 b1 14 c9 99 50 ee  a7 72 4d dc 61 8c c9 ca          P  rM a   
00001181    92 c4 15 fd ab dd 0a ec  cc 5a 54 d3 bb dd be 86             ZT     
00001182    42 93 3d db bc c9 32 d6  f2 16 4a f5 eb 1d ab 35    B =   2   J    5
00001183    27 7f 67 4d df 09 b6 32  1d 4d ec ec 25 0a 7d 8b    ' gM   2 M  % } 
00001184    e2 d2 5e 2f ab c4 53 21  cd d2 8a 82 4b 89 a9 8e      ^/  S!    K   
00001185    55 bb 91 64 47 77 5c 8b  91 8d 0e ab 11 6c 33 b6    U  dGw\      l3 
00001186    08 75 9e b2 9a ce da 60  29 ec d2 36 d1 4d d2 8d     u     `)  6 M  
00001187    6b 56 37 cb 73 82 b9 7f  ce ca d4 e4 5e 30 82 e4    kV7 s       ^0  
00001188    d5 e3 f9 6b 3a d5 5b 8c  48 9a dc 72 34 d1 9c 8d       k: [ H  r4   
00001189    59 2e 24 78 53 e7 fe 04  6b 1b 5a 0e 6d 59 dc 53    Y.$xS   k Z mY S
00001190    c4 41 b2 d3 32 4e 3b de  d9 b7 ed fd bc 2c 92 24     A  2N;      , $
00001191    ed c5 de 9e 49 36 69 56  d4 58 65 16 30 8a ad 96        I6iV Xe 0   
00001192    a1 d8 bf f7 ae 5f 73 71  86 24 4e 1e 73 d0 d0 53         _sq $N s  S
00001193    c3 82 dd cd ec 6a 9a e5  4e 61 8d 63 8c 5c d4 cc         j  Na c \  
00001194    46 d0 f4 9e f3 81 bc 88  e1 5a cf 1c 65 5b 98 c4    F        Z  e[  
00001195    66 dc ec 58 ce 6d d6 ac  dd 29 d4 5a 4c 14 fc f2    f  X m   ) ZL   
00001196    04 2c ec 89 ab 1c c2 2f  ea d8 81 1a db ba 9f 2d     ,     /       -
00001197    b4 f4 dc c4 97 78 df 4a  19 a4 fb 5a ad de 6c c3         x J   Z  l 
00001198    ad a7 99 99 26 2d 16 aa  25 d6 74 c3 10 39 4b 19        &-  % t  9K 
00001199    32 14 fa bc 41 c4 5d ac  5e f7 13 69 b2 da cc 0d    2   A ] ^  i    
00001200    f1 d1 b6 a7 52 31 68 e6  35 c9 69 17 1a 05 6a 36        R1h 5 i   j6
00001201    f9 72 76 bf 23 51 ed 44  ad ee f1 98 cd cd e4 ec     rv #Q D        
00001202    e6 c9 91 87 85 1f 8f 61  66 ad ec e2 67 e3 ed 67           af   g  g
00001203    99 65 cb 2f 1e 88 37 95  cd a5 6b ef 62 3e 57 e3     e /  7   k b>W 
00001204    ce c8 fd 75 dc 29 39 66  ee 5c c4 18 b2 ee f4 d9       u )9f \      
00001205    c5 b2 43 d1 e8 e2 33 36  68 da 5e 20 34 14 f9 b3      C   36h ^ 4   
00001206    8c e8 73 76 0d 0e b1 6b  7a da 4a 86 3e 34 53 ca      sv   kz J >4S 
00001207    36 b2 90 44 db 74 2b 49  cc 5e 70 64 14 ff f2 b3    6  D t+I ^pd    
00001208    9a c3 7e 47 47 2e 1d 75  7d ad f5 0d a2 8f 6a e9      ~GG. u}     j 
00001209    0e 40 f0 aa e4 c2 08 9a  ed c4 00 46 75 d7 25 8d     @         Fu % 
00001210    75 01 61 a1 1f b2 70 51  ba 9b 22 4d b9 ab 71 a1    u a   pQ  "M  q 
00001211    9d b6 b5 4b 77 94 8d a6  03 58 d6 f4 6f f7 2f 2f       Kw    X  o //
00001212    7b 3a 8c 2e 11 e9 8a 0b  43 cc 89 a4 7d 5d 84 bc    {: .    C   }]  
00001213    bb b8 fe 66 6c 65 3d 66  4e 23 4d a6 10 23 59 6a       fle=fN#M  #Yj
00001214    51 c6 72 5c 8f 43 4b 6e  4b 26 ef 24 ce 48 83 7e    Q r\ CKnK& $ H ~
00001215    74 29 f2 60 73 24 a5 39  da bd cd 8e c5 b9 db 33    t) `s$ 9       3
00001216    2e 9a a8 26 64 c4 d1 99  de eb 58 f4 91 8e 23 ad    .  &d     X   # 
00001217    de 6e 6b 79 2c c3 45 bd  41 39 12 76 b7 ce 29 da     nky, E A9 v  ) 
00001218    66 ce 4c 1c 67 46 21 4f  9a ed a8 1a 90 a7 64 43    f L gF!O      dC
00001219    30 8d 26 37 6e db b6 d7  96 67 58 e6 64 99 ac 74    0 &7n    gX d  t
00001220    a6 e3 da c4 c9 c5 5b da  81 ad 4d 1b dc 5f 13 98          [   M  _  
00001221    0b 3e 60 e6 3f 58 93 9a  59 45 0d ec 46 ba dc 19     >` ?X  YE  F   
00001222    35 b3 a8 60 7a b7 d2 1f  0a 7a 24 45 9d da ce 5b    5  `z    z$E   [
00001223    95 77 a4 68 3f 40 e4 36  56 f1 8b 65 d9 8c 4b 72     w h?@ 6V  e  Kr
00001224    ac 6e 07 6e 2d 8c e2 12  ad af 59 9b 75 1e 5b ca     n n-     Y u [ 
00001225    ba f2 37 1e 14 fa 9e 08  53 25 47 c6 33 65 dd 86      7     S%G 3e  
00001226    ad de b0 bc c4 0b 77 5b  6b 93 1e 91 ab 6f 26 58          w[k    o&X
00001227    1d 3d 3d a5 8f 4b 2b 2d  f5 99 92 6b 12 dd b6 33     ==  K+-   k   3
00001228    6d 3e 57 ad eb 6c b7 17  91 c1 4f d4 c9 a9 0d b6    m>W  l    O     
00001229    6c d6 3b 4a 73 36 59 cc  d5 9e bc 66 6e 72 c1 be    l ;Js6Y    fnr  
00001230    f7 25 91 7d a6 83 6d bb  b5 7d 63 62 ee ed e8 7a     % }  m  }cb   z
00001231    d3 2c 48 c4 e4 73 59 33  1b ab e4 4b 57 a4 a3 1e     ,H  sY3   KW   
00001232    83 56 d8 c2 6b 1b 1c de  f4 d4 e4 a8 ba cc 6e 59     V  k         nY
00001233    5a 6f 89 17 27 1b 74 b5  6c ea 4e 67 9d 69 57 6f    Zo  ' t l Ng iWo
00001234    58 ef 2a 73 a1 4f 6c 38  9d 19 25 0e f2 31 67 59    X *s Ol8  %  1gY
00001235    d9 dd 7a 4b 5a d6 5b 49  b3 26 6d 88 a5 94 ef 77      zKZ [I &m    w
00001236    72 b5 12 6d cb 27 2a 2b  37 2d 94 d1 42 e7 57 b2    r  m '*+7-  B W 
00001237    cd 24 0a 7e ad 65 4c cc  88 24 99 89 27 78 79 6d     $ ~ eL  $  'xym
00001238    cd 8b cc ba b7 38 da d2  ea 8a ee 45 e6 73 8b 33         8     E s 3
00001239    62 0d 72 c8 71 76 96 89  91 6b 98 5f 7a bf 26 2d    b r qv   k _z &-
00001240    8f b8 ca f2 76 5c ba 9f  f3 36 b8 29 b8 ee 24 95        v\   6 )  $ 
00001241    78 fc a4 b5 0a 39 6b bc  a3 cd b3 42 ab ca dc 41    x    9k    B   A
00001242    5f 64 5e a1 15 09 3d 65  ae a2 90 36 87 a4 9d 6f    _d^   =e   6   o
00001243    8b 54 03 6c c7 65 9b 36  5e ea d6 71 3c 78 cf d7     T l e 6^  q<x  
00001244    da f2 ab 67 67 10 d3 5b  d9 88 e3 7b 8b bc e2 97       gg  [   {    
00001245    be 1b 67 74 69 cb 1b 76  ed 59 e1 fe 4e 5c 69 99      gti  v Y  N\i 
00001246    a5 16 6c 4b 30 90 29 79  95 8b b2 f2 6f 50 7c 53      lK0 )y    oP|S
00001247    e9 73 14 8e 9a 9b db 12  cc de 6f 25 a6 f5 e5 54     s        o%   T
00001248    69 da c3 c2 9e cd 89 ea  2e 6e 6e 4b ad 2f 63 24    i       .nnK /c$
00001249    d0 a5 99 75 6b 9d 91 b9  54 f7 21 2d 85 99 99 53       uk   T !-   S
00001250    33 3b 2b 5c d6 e9 32 1a  bd d2 ab 56 63 30 d1 65    3;+\  2    Vc0 e
00001251    61 71 ce a4 e1 89 db 39  7b 33 25 93 1b 84 41 49    aq     9{3%   AI
00001252    b5 79 8b 41 c4 45 ca b1  2a 39 8d 6b 2d db 33 59     y A E  *9 k- 3Y
00001253    e7 ae e4 e6 c7 77 07 2b  dd 7a d6 6d b1 be c8 be         w + z m    
00001254    c1 bc 68 fe d8 b8 72 d6  66 b1 5b c9 25 3a 14 5a      h   r f [ %: Z
00001255    c8 30 41 97 62 6a ed c4  ed 62 f2 d6 6e c6 19 7b     0A bj   b  n  {
00001256    5b 76 0d a5 ac 56 b0 c2  39 b7 7e 43 d8 59 ad 21    [v   V  9 ~C Y !
00001257    c6 fb 3c b1 90 a7 a4 c4  32 21 5a 3d a4 9a 92 3a      <     2!Z=   :
00001258    20 da 53 bc e1 17 22 36  fa b5 dd c3 2d 45 d7 b1      S   "6    -E  
00001259    aa ed f3 79 6d d6 8a e7  96 a4 c1 4f 63 bc cd d9       ym      Oc   
00001260    96 5c 21 8c c9 bc 91 8b  18 42 59 9a c1 ee 58 c3     \!      BY   X 
00001261    72 a1 ea 98 91 a9 1c 54  9a db d9 56 96 34 4f 9a    r      T   V 4O 
00001262    85 79 73 37 1a bb 31 73  8c 23 b7 89 a2 f5 ae ad     ys7  1s #      
00001263    51 76 b8 29 f1 1f 37 52  d8 c2 67 dc 5a f5 6b 97    Qv )  7R  g Z k 
00001264    6c bb bc 4d 35 0c da f2  a5 90 6d d2 d8 be cc dc    l  M5     m     
00001265    78 6e 8a cc 67 8e 9c bd  4c c6 4e a4 e4 eb 77 65    xn  g   L N   we
00001266    67 cf 5a e2 3d dc bb 36  ae dc d9 8b cb 16 7a 78    g Z =  6      zx
00001267    48 81 b9 11 25 b7 88 7f  e7 6b 17 72 f3 79 12 54    H   %    k r y T
00001268    fd bc a9 b6 bc 71 b2 49  50 d6 bb a3 30 a5 f1 7e         q IP   0  ~
00001269    da 98 38 46 e4 9c ee 8d  b6 f7 10 75 31 b4 85 9d      8F       u1   
00001270    b8 8b 4f 0c b9 dc d7 85  3e 36 dd 45 c9 3a 99 9c      O     >6 E :  
00001271    e7 58 b3 09 2d 98 da 26  2d a9 b9 88 fd 5f 3b 0a     X  -  &-    _; 
00001272    b9 b3 6a ec c4 55 fb 68  77 2f 26 ac 8e 53 f9 6e      j  U hw/&  S n
00001273    d9 ad 35 9c ea 48 79 17  17 9b bc 23 0a 7c 86 a2      5  Hy    # |  
00001274    6a d2 c3 6c 99 13 1a 93  f9 32 fa d6 6f 07 15 f2    j  l     2  o   
00001275    5f 58 b5 a1 cf 5c dd 49  bc 13 2d 6f 25 d6 ad 3c    _X   \ I  -o%  <
00001276    14 f8 bc ec 5b a2 69 d6  e5 de 07 ae 6a b5 66 c6        [ i     j f 
00001277    3b 05 d7 3b 94 35 d7 f4  13 b8 20 5d 85 8f 0a 7f    ;  ; 5     ]    
00001278    96 df 44 f5 c8 a9 66 27  c6 a5 4f bd 8f 4f b1 24      D   f'  O  O $
00001279    58 25 93 97 95 2b 14 84  b1 a2 b4 c7 f7 bc 63 49    X%   +        cI
00001280    93 e4 92 ed ce f2 71 bb  24 d9 12 43 fb 3b cb 9b          q $  C ;  
00001281    32 cf b5 08 93 a2 dd cb  69 0b 19 78 1a 3c 29 f1    2       i  x <) 
00001282    25 62 4c 9c 6a 75 1d 7d  e5 9a ce d6 ac 43 75 bd    %bL ju }     Cu 
00001283    62 69 82 ae ec 46 c6 ba  a1 85 32 f6 95 dd 7e ad    bi   F    2   ~ 
00001284    d6 1a b9 70 8e 4c dd ab  56 e6 9d 49 ca fd 7e bd       p L  V  I  ~ 
00001285    d2 6f d7 dc 62 42 f5 fa  f0 ad 54 cb 77 1e 71 2f     o  bB    T w q/
00001286    39 67 4b 5d 65 cb 0a fa  55 b8 89 f0 d8 df ee e4    9gK]e   U       
00001287    43 46 f2 96 f5 fa fe 32  de b7 73 a8 64 9f 19 85    CF     2  s d   
00001288    9d 98 55 2c 0e 39 da e4  45 17 7b 6d 8d c4 53 0e      U, 9  E {m  S 
00001289    1d 99 b7 22 59 77 64 c1  90 cf de a6 ce 34 5b 49       "Ywd      4[I
00001290    0b 37 31 87 56 72 20 1b  51 63 0b 2c bb f3 96 a3     71 Vr  Qc ,    
00001291    30 14 ba e9 76 44 d6 cb  22 27 31 52 65 5f 2d e9    0   vD  "'1Re_- 
00001292    12 e8 5a 67 49 71 35 5b  57 c9 32 4d 22 62 5c b2      ZgIq5[W 2M"b\ 
00001293    ea fc db db b8 b6 db 8e  19 f6 96 95 ea d1 f9 6e                   n
00001294    75 fb 8f 8f eb c4 2e 43  f6 1c 76 64 7a 4c ed 8f    u     .C  vdzL  
00001295    c7 c4 7f 81 58 c6 5e 61  06 33 db 64 58 73 9d a9        X ^a 3 dXs  
00001296    eb 8f 57 2d 94 60 15 f8  53 c6 e5 37 1c cb 18 13      W- `  S  7    
00001297    ed b1 b9 70 80 29 e2 0a  64 56 c1 92 7a b4 5e e9       p )  dV  z ^ 
00001298    13 6d cd 0d 2e 8c 36 a0  81 a6 a3 72 2d 91 6b 0e     m  . 6    r- k 
00001299    6d 27 19 f3 61 45 a5 35  78 4b db 99 b5 0d 2a b8    m'  aE 5xK    * 
00001300    f8 9e 58 83 e3 67 0d ec  e3 99 67 12 c9 85 0f fe      X  g    g     
00001301    39 ea 51 71 a2 c4 10 59  c2 cf 42 ae 40 c2 a5 e1    9 Qq   Y  B @   
00001302    09 62 08 2c c5 e5 a8 68  b4 5e 5a 86 8b 71 79 6a     b ,   h ^Z  qyj
00001303    1a 2d ff ff f8                                       -
```

### **第 8 个 TAG 的长度**

```xml
00001303                   00 00 10  44                                 D
```

### **第 9 个 FLV TAG（AudioTag）**

```xml
00001303                                08 00 01 06 00 00 68                   h
00001304    00 00 00 00 2e ff fb 60  c4 01 80 0c c1 6f 51 a1        .  `     oQ 
00001305    8c ad c9 b3 af 69 b4 63  15 b9 00 04 94 65 5e 19         i c     e^ 
00001306    85 86 30 ce 08 3b 1a ca  30 64 f6 0a aa 50 d3 26      0  ;  0d   P &
00001307    06 af b5 a8 0e f1 70 b9  67 ea d2 20 33 16 3e e7          p g   3 > 
00001308    fe 74 46 29 d1 93 54 1e  e9 a1 59 64 4a 32 9b 6f     tF)  T   YdJ2 o
00001309    3b b9 1f d5 64 b3 d1 9d  9d f6 7a 09 4c d4 a3 33    ;   d     z L  3
00001310    d8 ce 82 f7 13 04 46 aa  bb 25 d5 11 5b 49 95 d9          F  %  [I  
00001311    04 4e 30 62 13 e1 2b 3f  99 d8 ce 3f 7a 76 32 40     N0b  +?   ?zv2@
00001312    00 02 5a 44 cd 74 b0 c4  59 ce 5a d6 7e 92 36 9c      ZD t  Y Z ~ 6 
00001313    e5 1e ee e2 2e 62 8d f6  81 85 bc a8 9c ef 77 6d        .b        wm
00001314    bf 3b ea 1e 8e 73 e8 ab  e6 a7 6c ae 66 1b d6 46     ;   s    l f  F
00001315    45 71 cd d0 e7 39 f9 1c  ad 55 d9 ca 57 72 37 21    Eq   9   U  Wr7!
00001316    ab 39 aa 39 e7 7e ae 51  19 15 ba bb 3a b3 9c 1d     9 9 ~ Q    :   
00001317    50 6a f6 57 73 35 6e 77  56 44 64 41 11 cc c7 21    Pj Ws5nwVDdA   !
00001318    14 62 18 82 be 6d 2a 1c  bf 36 ed 17 80 00 12 12     b   m*  6      
00001319    41 cc 02 89 95 cd d9 ce  25 a6 84 54 e7 4b 9b 3c    A       %  T K <
00001320    d0 fa 39 ea 1b 4b 22 91  ad 07                        9  K"         
```

#### **AudioTagHeader**

```xml
00001304                2e                                          .
```

#### **AUDIODATA**

```xml
00001304                   ff fb 60  c4 01 80 0c c1 6f 51 a1           `     oQ 
00001305    8c ad c9 b3 af 69 b4 63  15 b9 00 04 94 65 5e 19         i c     e^ 
00001306    85 86 30 ce 08 3b 1a ca  30 64 f6 0a aa 50 d3 26      0  ;  0d   P &
00001307    06 af b5 a8 0e f1 70 b9  67 ea d2 20 33 16 3e e7          p g   3 > 
00001308    fe 74 46 29 d1 93 54 1e  e9 a1 59 64 4a 32 9b 6f     tF)  T   YdJ2 o
00001309    3b b9 1f d5 64 b3 d1 9d  9d f6 7a 09 4c d4 a3 33    ;   d     z L  3
00001310    d8 ce 82 f7 13 04 46 aa  bb 25 d5 11 5b 49 95 d9          F  %  [I  
00001311    04 4e 30 62 13 e1 2b 3f  99 d8 ce 3f 7a 76 32 40     N0b  +?   ?zv2@
00001312    00 02 5a 44 cd 74 b0 c4  59 ce 5a d6 7e 92 36 9c      ZD t  Y Z ~ 6 
00001313    e5 1e ee e2 2e 62 8d f6  81 85 bc a8 9c ef 77 6d        .b        wm
00001314    bf 3b ea 1e 8e 73 e8 ab  e6 a7 6c ae 66 1b d6 46     ;   s    l f  F
00001315    45 71 cd d0 e7 39 f9 1c  ad 55 d9 ca 57 72 37 21    Eq   9   U  Wr7!
00001316    ab 39 aa 39 e7 7e ae 51  19 15 ba bb 3a b3 9c 1d     9 9 ~ Q    :   
00001317    50 6a f6 57 73 35 6e 77  56 44 64 41 11 cc c7 21    Pj Ws5nwVDdA   !
00001318    14 62 18 82 be 6d 2a 1c  bf 36 ed 17 80 00 12 12     b   m*  6      
00001319    41 cc 02 89 95 cd d9 ce  25 a6 84 54 e7 4b 9b 3c    A       %  T K <
00001320    d0 fa 39 ea 1b 4b 22 91  ad 07                        9  K"         
```

### **第 9 个 TAG 的长度**

```xml
00001320                                   00 00 01 11
```

### **第 10 个 FLV TAG（VideoTag）**

```xml
00001320                                               09 00    
00001321    02 67 00 00 78 00 00 00  00 22 00 00 84 0e b1 3f     g  x    "     ?
00001322    ff ff c2 f8 2d 41 69 f0  b1 f6 09 9b 10 45 60 b4        -Ai      E` 
00001323    e1 7c 39 70 70 2a ff ff  1b 70 38 bd 8e 87 d4 f8     |9pp*   p8     
00001324    2c 85 7b cb d2 de 15 6c  59 ff f0 ab b9 c7 0e 8f    , {    lY       
00001325    e1 fc 4d b8 e7 6b 43 34  2d af e5 2a b0 40 a5 39      M  kC4-  * @ 9
00001326    ca e9 ce ae cf 74 87 c2  d6 e3 90 fd 27 23 ce d3         t      '#  
00001327    fa 1e e8 ff c6 dc 77 8f  e3 bd ab dc 7d 97 86 bf          w     }   
00001328    fe 17 e1 e3 ba 9f 3a c2  3a 36 b3 5f 72 b2 a1 17          : :6 _r   
00001329    c0 4c 43 eb 30 67 96 d6  08 a6 bc 8b ff a1 f0 fb     LC 0g          
00001330    b5 78 7e f0 2d 8a 7e d5  82 9d b6 f6 bf 98 6c 65     x~ - ~       le
00001331    ea dc 8f cc ce 64 88 12  33 6d eb b3 56 ad 6c 66         d  3m  V lf
00001332    5e 26 ee 90 2d 73 bd 6e  66 a4 a6 53 f7 71 84 f2    ^&  -s nf  S q  
00001333    d6 9f 3a cc 0e 65 6a c4  2c b9 a9 5a dc ac c9 7b      :  ej ,  Z   {
00001334    45 f7 57 82 e4 fc ce d9  15 cc 5b a5 72 1c ac 99    E W       [ r   
00001335    cb 7b 1e d4 ad 06 fc a2  f5 3b 53 79 e5 98 9f ba     {       ;Sy    
00001336    95 ca 76 5c 8d 17 27 e7  4a 85 2c b7 97 2d 77 e3      v\  ' J ,  -w 
00001337    4e be 3d dc 4d dc b5 69  13 99 61 7b 72 e0 e2 47    N = M  i  a{r  G
00001338    21 f6 42 04 92 bd 3c a9  34 46 dd 6e ba 07 6c 4a    ! B   < 4F n  lJ
00001339    fa e2 bd 6b d3 6d a2 f4  fc 9e ed ad 96 c4 77 72       k m        wr
00001340    47 eb 72 4c 86 9b bb bc  d6 37 57 b3 bb 75 7c 86    G rL     7W  u| 
00001341    5a 8b ba 25 ef 32 5f c8  fd 8d 6e 71 db 88 ac d0    Z  % 2_   nq    
00001342    ff 47 36 b9 9e 24 98 c1  b9 57 d1 4a 7f ba b4 a9     G6  $   W J    
00001343    28 b6 4f c4 f3 17 88 7b  35 d7 80 af 97 95 0b ae    ( O    {5       
00001344    b3 11 6a 73 28 7d de f7  a0 88 8b 25 3d bd 5b a0      js(}     %= [ 
00001345    5a 0b 14 e2 15 db 85 bc  74 34 bf 5d 1d b1 ca ff    Z       t4 ]    
00001346    b5 ab b5 cd 72 ed c8 b4  67 ae 5f 92 64 6c b7 71        r   g _ dl q
00001347    0d a2 a4 b5 04 99 ba 34  67 e3 0f 49 ee 4a c2 f5           4g  I J  
00001348    7d de ea 77 a2 e3 e5 1a  4d ec c5 5b 1b dd 85 52    }  w    M  [   R
00001349    13 29 cd 22 95 f6 36 33  a9 79 43 1a 9f 2a cf 61     ) "  63 yC  * a
00001350    2c 63 4c a4 ca 10 bc 8a  59 5d a7 9e 95 e7 a3 c4    ,cL     Y]      
00001351    b9 e3 ce c4 7c 79 f8 e3  d1 de b7 2f 75 ba 59 08        |y     /u Y 
00001352    7e 22 ff 91 06 06 b7 6f  53 76 da c3 f2 a4 96 90    ~"     oSv      
00001353    ff f7 6c 9e 6a 21 ab fe  a7 2a 84 53 1e de 57 bb      l j!   * S  W 
00001354    77 ba f6 79 6e 70 8a 47  31 da b9 4a 1d 86 ff ff    w  ynp G1  J    
00001355    53 5e 2c 9d 01 4c bc 43  a7 1a ee 2e 47 5f 89 96    S^,  L C   .G_  
00001356    8b 32 62 13 fb 7f 62 e2  ef fe df cc 95 fe a7 33     2b   b        3
00001357    4d a2 a9 ac ee a4 3b e2  f9 5c c7 23 bf ff ed 7b    M     ;  \ #   {
00001358    fe ce cd 0f 68 e4 da 7d  d9 a6 b9 29 99 84 91 df        h  }   )    
00001359    8e 7b 49 38 17 71 31 64  16 7f ed fc 77 ff ff f0     {I8 q1d    w   
```

#### **VideoTagHeader**

```xml
00001321                                22
```

#### **VIDEODATA**

```xml
00001321                                   00 00 84 0e b1 3f                   ?
00001322    ff ff c2 f8 2d 41 69 f0  b1 f6 09 9b 10 45 60 b4        -Ai      E` 
00001323    e1 7c 39 70 70 2a ff ff  1b 70 38 bd 8e 87 d4 f8     |9pp*   p8     
00001324    2c 85 7b cb d2 de 15 6c  59 ff f0 ab b9 c7 0e 8f    , {    lY       
00001325    e1 fc 4d b8 e7 6b 43 34  2d af e5 2a b0 40 a5 39      M  kC4-  * @ 9
00001326    ca e9 ce ae cf 74 87 c2  d6 e3 90 fd 27 23 ce d3         t      '#  
00001327    fa 1e e8 ff c6 dc 77 8f  e3 bd ab dc 7d 97 86 bf          w     }   
00001328    fe 17 e1 e3 ba 9f 3a c2  3a 36 b3 5f 72 b2 a1 17          : :6 _r   
00001329    c0 4c 43 eb 30 67 96 d6  08 a6 bc 8b ff a1 f0 fb     LC 0g          
00001330    b5 78 7e f0 2d 8a 7e d5  82 9d b6 f6 bf 98 6c 65     x~ - ~       le
00001331    ea dc 8f cc ce 64 88 12  33 6d eb b3 56 ad 6c 66         d  3m  V lf
00001332    5e 26 ee 90 2d 73 bd 6e  66 a4 a6 53 f7 71 84 f2    ^&  -s nf  S q  
00001333    d6 9f 3a cc 0e 65 6a c4  2c b9 a9 5a dc ac c9 7b      :  ej ,  Z   {
00001334    45 f7 57 82 e4 fc ce d9  15 cc 5b a5 72 1c ac 99    E W       [ r   
00001335    cb 7b 1e d4 ad 06 fc a2  f5 3b 53 79 e5 98 9f ba     {       ;Sy    
00001336    95 ca 76 5c 8d 17 27 e7  4a 85 2c b7 97 2d 77 e3      v\  ' J ,  -w 
00001337    4e be 3d dc 4d dc b5 69  13 99 61 7b 72 e0 e2 47    N = M  i  a{r  G
00001338    21 f6 42 04 92 bd 3c a9  34 46 dd 6e ba 07 6c 4a    ! B   < 4F n  lJ
00001339    fa e2 bd 6b d3 6d a2 f4  fc 9e ed ad 96 c4 77 72       k m        wr
00001340    47 eb 72 4c 86 9b bb bc  d6 37 57 b3 bb 75 7c 86    G rL     7W  u| 
00001341    5a 8b ba 25 ef 32 5f c8  fd 8d 6e 71 db 88 ac d0    Z  % 2_   nq    
00001342    ff 47 36 b9 9e 24 98 c1  b9 57 d1 4a 7f ba b4 a9     G6  $   W J    
00001343    28 b6 4f c4 f3 17 88 7b  35 d7 80 af 97 95 0b ae    ( O    {5       
00001344    b3 11 6a 73 28 7d de f7  a0 88 8b 25 3d bd 5b a0      js(}     %= [ 
00001345    5a 0b 14 e2 15 db 85 bc  74 34 bf 5d 1d b1 ca ff    Z       t4 ]    
00001346    b5 ab b5 cd 72 ed c8 b4  67 ae 5f 92 64 6c b7 71        r   g _ dl q
00001347    0d a2 a4 b5 04 99 ba 34  67 e3 0f 49 ee 4a c2 f5           4g  I J  
00001348    7d de ea 77 a2 e3 e5 1a  4d ec c5 5b 1b dd 85 52    }  w    M  [   R
00001349    13 29 cd 22 95 f6 36 33  a9 79 43 1a 9f 2a cf 61     ) "  63 yC  * a
00001350    2c 63 4c a4 ca 10 bc 8a  59 5d a7 9e 95 e7 a3 c4    ,cL     Y]      
00001351    b9 e3 ce c4 7c 79 f8 e3  d1 de b7 2f 75 ba 59 08        |y     /u Y 
00001352    7e 22 ff 91 06 06 b7 6f  53 76 da c3 f2 a4 96 90    ~"     oSv      
00001353    ff f7 6c 9e 6a 21 ab fe  a7 2a 84 53 1e de 57 bb      l j!   * S  W 
00001354    77 ba f6 79 6e 70 8a 47  31 da b9 4a 1d 86 ff ff    w  ynp G1  J    
00001355    53 5e 2c 9d 01 4c bc 43  a7 1a ee 2e 47 5f 89 96    S^,  L C   .G_  
00001356    8b 32 62 13 fb 7f 62 e2  ef fe df cc 95 fe a7 33     2b   b        3
00001357    4d a2 a9 ac ee a4 3b e2  f9 5c c7 23 bf ff ed 7b    M     ;  \ #   {
00001358    fe ce cd 0f 68 e4 da 7d  d9 a6 b9 29 99 84 91 df        h  }   )    
00001359    8e 7b 49 38 17 71 31 64  16 7f ed fc 77 ff ff f0     {I8 q1d    w   
```

### **第 10 个 TAG 的长度**

```xml
00001360    00 00 02 72                                            r
```

### **第 11 个 FLV TAG（AudioTag）**

```xml
00001360                08 00 01 06  00 00 83 00 00 00 00 2e                   .
00001361    ff fb 60 c4 10 00 0e 0d  69 4d a4 98 ad c9 bd 33      `     iM     3
00001362    a9 68 33 0d b9 d6 b7 6a  cd ed ad b7 b3 dc d2 08     h3    j        
...
（中间部分省略）
...
00001375    3c 9d 7a 9c b3 a8 f1 01  73 2b 02 c2 63 61 a8 31    < z     s+  ca 1
00001376    60 c5 6e f2 1c a4 a1 7e  39 e4 bb 6c 7c f5 44 2a    ` n    ~9  l| D*
00001377    51 2b a2 b2 8e                                      Q+
```

### **第 11 个 TAG 的长度**

```xml
00001377                   00 00 01  11
```

### **第 12 个 FLV TAG（AudioTag）**

```xml
00001377                                08 00 00 b7 00 00 9d
00001378    00 00 00 00 2e ff fb 40  c4 18 00 0c 45 63 55 43        .  @    EcUC
00001379    0c ad d9 9b b2 e9 a8 31  95 b8 74 39 d9 c2 dd f1           1  t9    
...
（中间部分省略）
...
00001387    36 d5 42 f4 b8 2d 56 32  ab 43 51 48 ce dc fe 1e    6 B  -V2 CQH    
00001388    50 49 49 1c ce 86 38 b2  c5 dd 05 cb 9e ce 42 a8    PII   8       B 
00001389    e6 15 90 77 c2 93 40 37  1d 6d 6f                      w  @7 mo     
```

### **第 12 个 TAG 的长度**

```xml
00001389                                      00 00 00 c2
```

### **第 13 个 FLV TAG（VideoTag）**

```xml
00001389                                                  09
00001390    00 01 19 00 00 a0 00 00  00 00 22 00 00 84 12 b1              "     
00001391    3f ff ff ff ff ff ff e8  7c 3e ef e1 7c e4 3f ff    ?       |>  | ? 
...
（中间部分省略）
...
00001406    f2 bf ed 77 66 2c 9e 3d  27 c9 66 9c 05 10 81 45       wf, =' f    E
00001407    98 6d 2f 14 58 cd 21 83  b2 b7 dd ab 8b ff ff ff     m/ X !         
00001408    ff ff e0
```

### **第 13 个 TAG 的长度**

```xml
00001408             00 00 01 24                                      $         
```

### **第 14 个 FLV TAG（AudioTag）**

```xml
00001408                         08  00 00 d1 00 00 b7 00 00
00001409    00 00 2e ff fb 50 c4 04  00 0c 1d 5f 55 a1 8c ad      .  P     _U   
00001410    c1 6f a5 ea 74 31 95 79  25 06 2c 31 cb 58 85 a3     o  t1 y% ,1 X  
...
（中间部分省略）
...
00001420    75 47 b2 27 75 22 0f 08  13 e6 bf 09 a9 6f e7 a3    uG 'u"       o  
00001421    c3 07 02 93 55 93 6f 38  46 9a 7f 6c 8d bb 86 51        U o8F  l   Q
00001422    1d 43 f3                                             C              
```

### **第 14 个 TAG 的长度**

```xml
00001422             00 00 00 dc
```

### **第 15 个 FLV TAG（VideoTag）**

```xml
00001422                         09  00 00 84 00 00 c8 00 00                    
00001423    00 00 22 00 00 84 1a b1  3f ff ff e1 7c 16 bd 04      "     ?   |   
00001424    ef ff ff ff f8 be 1e f1  de 76 1f c5 e7 3f f0 be             v   ?  
...
（中间部分省略）
...
00001429    6b bf ff fb 7c 72 1f db  03 f9 48 24 7f fd ae 4c    k   |r    H$   L
00001430    63 4c ff ff fb 3c 3f 60  c1 9e ba 63 da fa fa db    cL   <?`   c    
00001431    ff ff ff ff ff c0
```

### **第 15 个 TAG 的长度**

```xml
00001431                      00 00  00 8f
```

### **第 16 个 FLV TAG（AudioTag）**

```xml
00001431                                   08 00 00 d1 00 00                    
00001432    d1 00 00 00 00 2e ff fb  50 c4 03 00 0b f9 31 4d         .  P     1M
00001433    27 b0 6b c1 54 26 29 a4  63 09 78 89 f1 4d 6b 44    ' k T&) c x  MkD
...
（中间部分省略）
...
00001443    f9 3e 25 58 5a 48 bc da  a5 8a 8c a3 13 5e 97 11     >%XZH       ^  
00001444    55 ec d6 f3 6e 51 d2 75  10 a9 ac 01 00 91 83 f4    U   nQ u        
00001445    eb 1b e0 6e 32 2e                                      n2.          
```

### **第 16 个 TAG 的长度**

```xml
00001445                      00 00  00 dc
```

### **第 17 个 FLV TAG（AudioTag）**

```xml
00001445                                   08 00 00 b7 00 00
00001446    eb 00 00 00 00 2e ff fb  40 c4 06 00 0a 81 1b 4d         .  @      M
00001447    21 98 6b 81 45 23 29 f4  31 89 70 a4 df c4 8d 29    ! k E#) 1 p    )
...
（中间部分省略）
...
00001455    97 b7 29 dd 8e ee cb 6f  fe e4 ad 41 3e ad 2b b6      )    o   A> + 
00001456    11 eb fa 82 a5 4f b0 d9  11 00 76 39 2a 93 08 3b         O    v9*  ;
00001457    d6 ea 5b fa 55 f5 12 00  02 83 ea ac                  [ U           
```

### **第 17 个 TAG 的长度**

```xml
00001457                                         00 00 00 c2
```

### **第 18 个 FLV TAG（VideoTag）**

```xml
00001458    09 00 00 8d 00 00 f0 00  00 00 00 22 00 00 84 1e               "    
00001459    b1 3f ff fc 2f 82 d4 16  81 63 a1 60 2d 41 69 ff     ?  /    c `-Ai 
00001460    ff e2 8d c7 3c 2f c3 c7  76 3e 44 89 79 a9 e6 b6        </  v>D y   
...
（中间部分省略）
...
00001465    ff ff ff 6b 6b 50 c2 6c  5f 53 e6 27 58 6d 61 0f       kkP l_S 'Xma 
00001466    ff d8 ee d4 a8 f5 8c 46  45 d7 da 22 98 87 84 3f           FE  "   ?
00001467    ff ff ff ff ff ff ff fe
```

### **第 18 个 TAG 的长度**

```xml
00001467                             00 00 00 98
```

### **第 19 个 FLV TAG（AudioTag）**

```xml
00001467                                         08 00 00 b7                    
00001468    00 01 05 00 00 00 00 2e  ff fb 40 c4 03 81 0a 49           .  @    I
00001469    17 4d 21 98 4b 81 47 a3  a9 a4 33 0d 70 2d 22 86     M! K G   3 p-" 
...
（中间部分省略）
...
00001477    b2 79 fd 72 a7 bc c8 a4  25 ff cf 99 e4 65 cc 39     y r    %    e 9
00001478    93 73 8a dc 73 11 d2 34  43 a0 8b a3 98 30 d1 ed     s  s  4C    0  
00001479    b8 e1 cc 6e fd 69 19 d4  8b b4 2a ba 45 00             n i    * E 
```

### **第 19 个 TAG 的长度**

```xml
00001479                                               00 00
00001480    00 c2
```

### **第 20 个 FLV TAG（VideoTag）**

```xml
00001480          09 00 02 e5 00 01  18 00 00 00 00 22 00 00                 "  
00001481    84 22 b1 3f ff ff 0b e0  b5 05 a0 5f 05 a8 2d 30     " ?       _  -0
00001482    be 0b 5e 82 70 54 e8 58  77 80 9e 0b 4f ff e1 7e      ^ pT Xw   O  ~
...
（中间部分省略）
...
00001525    99 ed c5 d0 ec 2b 39 c4  c8 fb 87 c4 dd d2 23 c8         +9       # 
00001526    5c d8 cf 1f cf 51 bc 29  86 3f ff fb 7d 93 05 3f    \    Q ) ?  }  ?
00001527    ff fe
```

### **第 20 个 TAG 的长度**

```xml
00001527          00 00 02 f0
```

### **（中间部分省略）**

### **最后的 FLV TAG（VideoTag）**

```xml
00018757                09 00 0c 8c  00 15 90 00 00 00 00 22                   "
00018758    00 00 86 96 b1 3f ff ff  ff ff ff ff ff fe cf 0c         ?          
00018759    85 0c f2 ed b6 fc 38 c3  97 88 f7 4f a1 f1 f7 fc          8    O    
...
（中间部分省略）
...
00018956    52 72 71 4a 7c 41 1f 3f  cf 6c 6e dc 15 cf c9 45    RrqJ|A ? ln    E
00018957    49 70 85 7b 2f 9b b5 ba  c1 2d f5 be d5 0d ae ec    Ip {/    -      
00018958    f7 a3 9a eb bf ff ff ff  ff ff 80
```

### **最后的 TAG 的长度**

```xml
00018958                                      00 00 0c 97
```

### **（暂不清楚这个 d0 的意义）**

```xml
00018958                                                  d0
```

---

# **FFmpeg 封装 FLV**

<table><tr><td>参数</td><td>类型</td><td>说明</td></tr><tr><td rowspan="8">flvflags</td></tr><tr><td>flag</td><td>设置生成 FLV 时使用的 flag</td></tr><td>aac_seq_header_detect</td><td>添加 AAC 音频的 Sequence Header</td><tr></tr><td>no_sequence_end</td><td>生成 FLV 结束时不写入 Sequence End</td><tr><td>no_metadata</td><td>生成 FLV 时不写入 metadata</td></tr><tr><td>no_duration_filesize</td><td>用于直播时不在 metadata 中写入 duration 与 filesize</td></tr><tr><td>add_keyframe_index</td><td>生成 FLV 时自动写入关键帧索引信息到 metadata 头</td></tr></table>

## **MP4 转封装 FLV**

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c copy -f flv output.flv
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1526737808493_41.png)

### **生成带关键索引的 FLV**

将 FLV 文件中的关键帧建立索引，并将索引写入 Metadata 头中。

1、输入命令：

```shell
ffmpeg -i sample.mp4 -c copy -f flv -flvflags add_keyframe_index output_index.flv
```

2、输出结果：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1526737803664_35.png)

3、使用 FLV analyzer 对比察看：

（1）原文件：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1526737804563_37.png)

（2）带关键索引的文件：

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1526737804895_39.png)

![](http://otkw6sse5.bkt.clouddn.com/FLV-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1526737788918_24.png)

---