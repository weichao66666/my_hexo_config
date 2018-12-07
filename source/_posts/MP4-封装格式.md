---
layout: article
title: MP4 封装格式
date: 2018-05-16 23:57:15
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》
* [Mp4文件格式解析](https://www.cnblogs.com/CoderTian/p/8277965.html "https://www.cnblogs.com/CoderTian/p/8277965.html")
* [ffmpeg demux mp4](https://blog.csdn.net/wenmang1977/article/details/7736238 "https://blog.csdn.net/wenmang1977/article/details/7736238")

---

# **MP4 的 box 树**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/MP4-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F1.png)

MP4 常用参考标准排列方式：

一级|二级|三级|四级|五级|六级|必须性|说明
--|--
ftyp||||||是|文件类型
pdin||||||否|下载进度信息
moov||||||是|音视频数据的 metadata 信息<br>如果希望在视频点播中 MP4 被快速打开，则需要将 moov 存放在 mdat 前面<br>如果放在后面，则需要将 MP4 下载完才可以播放
|mvhd|||||是|电影文件头
|trak|||||是|流的 track
||tkhd||||是|流的 track 头
||tref||||否|track 参考容器
||edts||||否|edit list 容器
|||elst|||否|edit list 元素信息
||mdia||||是|track 里面的 media 信息
|||mdhd|||是|media 信息头
|||hdlr|||是|media 信息的句柄
|||minf|||是|media 信息容器
||||vmhd||否|视频 media 头（只存在于视频的 track）
|||||smhd|否|音频 media 头（只存在于音频的 track）
|||||hmhd|否|提示 media 头（只存在于提示的 track）
|||||nmhd|否|空 media 头（其他的 track）
||||dinf||是|数据信息容器
|||||dref|是|数据参考容器，track 中 media 的参考信息
||||stbl||是|采样表容器，容器做时间与数据所在位置的描述
|||||stsd|是|采样描述（codec 类型与初始化信息）
|||||stts|是|（decoding）采样时间
|||||ctts|否|（composition）采样时间
|||||stsc|是|chunk 采样，数据片段信息
|||||stsz|否|采样大小
|||||stz2|否|采样大小详细描述
|||||stco|是|chunk 偏移信息，数据偏移信息
|||||co64|否|64 位 chunk 偏移信息
|||||stss|否|同步采样表
|||||stsh|否|采样同步表
|||||padb|否|采样 padding
|||||stdp|否|采样退化优先描述
|||||sdtp|否|独立于可支配采样描述
|||||sbgp|否|采样组
|||||sgpd|否|采样组描述
|||||subs|否|子采样信息
|mvex|||||否|视频扩展容器
||mehd||||否|视频扩展容器头
||trex||||是|track 扩展信息
|ipmc|||||否|IPMP 控制容器
moof||||||否|视频分片
|mfhd|||||是|视频分片头
|traf|||||否|track 分片
||tfhd||||是|track 分片头
||trun||||否|track 分片 run 信息
||sdtp||||否|独立和可支配的采样
||sbgp||||否|采样组
||subs||||否|子采样信息
mfra||||||否|视频分片访问控制信息
|tfra|||||否|track 分片访问控制信息
|mfro|||||是|拼分片访问控制偏移量
mdat||||||否|media 数据容器
free||||||否|空闲区域
skip||||||否|空闲区域
|udta|||||否|用户数据
||cprt||||否|copyright 信息
meta||||||否|元数据
|hdlr|||||是|定义元数据的句柄
|dinf|||||否|数据信息容器
||dref||||否|元数据的源参考信息
|ipmc|||||否|IPMP 控制容器
|iloc|||||否|所在位置信息容器
|ipro|||||否|样本保护容器
||sinf||||否|计划信息保护容器
|||frma|||否|原格式容器
|||imif|||否|IPMP 信息容器
|||schm|||否|计划类型容器
|||schi|||否|计划信息容器
|iinf|||||否|容器所在项目信息
|xml|||||否|XML 容器
|bxml|||||否|binary XML 容器
|pitm|||||否|主要参考容器
|fiin|||||否|文件发送信息
||paen||||否|partition 入口
|||fpar|||否|文件片段容器
|||fecr|||否|FEC reservoir
||segr||||否|文件发送 session 组信息
||gitn||||否|组 id 转名称信息
||tsel||||否|track 选择信息
meco||||||否|追加的 metadata 信息
|mere|||||否|metabox 关系

一个 box 是由 box header 和 box 里面包含的数据组成。

## **box header**

box header 中包含了 box 的大小（size）和类型（type）等信息。
其中，size 指明了整个 box 所占用的大小（包括 header 部分）：
（1）size = 0 表示该 box 是最后一个 box；
（2）如果 box 很大（例如存放具体视频数据的 mdat box），超过了 uint32 的最大数值，size 就被设置为 1，并用接下来的 uint64 的 largesize 来存放大小。
type = uuid 表示该 box 中的数据是用户自定义扩展类型。
box 中的字节序为网络字节序，也就是大端字节序（Big-Endian）。

box 根据 header 部分包含的信息的不同可以分为 Box 和 Full Box：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/MP4-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F2.png)

Box 定义：

{% codeblock lang:c %}
aligned(8) class Box (unsigned int(32) boxtype, optional unsigned int(8)[16] extended_type) 
{
    unsigned int(32) size;
    unsigned int(32) type = boxtype;
    if (size==1) 
    {
        unsigned int(64) largesize;
    } 
    else if (size==0) 
    {
        // box extends to end of file
    }
    if (boxtype==‘uuid’) 
    {
        unsigned int(8)[16] usertype = extended_type;
    }
}
{% endcodeblock %}

Full Box 定义：

{% codeblock lang:c %}
aligned(8) class FullBox(unsigned int(32) boxtype, unsigned int(8) v, bit(24) f) extends Box(boxtype) 
{
    unsigned int(8) version = v;
    bit(24) flags = f;
}
{% endcodeblock %}

---

# **box 介绍**

## **ftyp（File Type）**

文件层 box，有且只有在文件最开始有一个，用于表明文件类型。

定义：

{% codeblock lang:c %}
aligned(8) class FileTypeBox extends Box(‘ftyp’) 
{
    unsigned int(32) major_brand;
    unsigned int(32) minor_version;
    unsigned int(32) compatible_brands[]; // to end of the box
}
{% endcodeblock %}

## **free（Free Space）**

free box 中的内容是无关紧要的，可以被忽略。该 box 被删除后，不会对播放产生任何影响，它的 type 域可以是 free 或 skip。

定义：

{% codeblock lang:c %}
aligned(8) class FreeSpaceBox extends Box(free_type) 
{
    unsigned int(8) data[];
}
{% endcodeblock %}

## **mdat（Media Data）**

文件层 box，可以有多个，也可以没有（当媒体数据全部为外部文件引用时），用于存储媒体数据。

数据直接跟在 box type 字段后面，它的结构是由 metadata 来描述的，metadata 通过文件中的绝对偏移来引用媒体数据。

定义：

{% codeblock lang:c %}
aligned(8) class MediaDataBox extends Box(‘mdat’) 
{
    bit(8) data[];
}
{% endcodeblock %}

## **moov（Movie）**

文件层 box，有且只有一个，用于存放媒体信息，至少包含以下三种中的一个：
（1）mvhd：存放未压缩过的电影信息；
（2）cmov：存放压缩过的电影信息；
（3）rmra：存放参考电影信息。

定义：

{% codeblock lang:c %}
aligned(8) class MovieBox extends Box(‘moov’)
{
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|moov

### **mvhd（Movie Header）**

用于存放文件的总体信息，如时长和创建时间等。它是独立于媒体的，并且与整个播放相关。

定义：

{% codeblock lang:c %}
aligned(8) class MovieHeaderBox extends FullBox(‘mvhd’, version, 0) 
{
    if (version==1) 
    {
        unsigned int(64) creation_time;
        unsigned int(64) modification_time;
        unsigned int(32) timescale;
        unsigned int(64) duration;
    } 
    else 
    { // version==0
        unsigned int(32) creation_time;
        unsigned int(32) modification_time;
        unsigned int(32) timescale;
        unsigned int(32) duration;
    }
    template int(32) rate = 0x00010000; // typically 1.0
    template int(16) volume = 0x0100; // typically, full volume
    const bit(16) reserved = 0;
    const unsigned int(32)[2] reserved = 0;
    // Unity matrix
    template int(32)[9] matrix = { 0x00010000,0,0,0,0x00010000,0,0,0,0x40000000 };
    bit(32)[6] pre_defined = 0;
    unsigned int(32) next_track_ID;
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|mvhd
version|1|版本
flags|3|扩展的标志，这里是 0
creation_time|4|起始时间，基准时间是 1904-1-1 0:00 AM
modification_time|4|修订时间，基准时间是 1904-1-1 0:00 AM
timescale|4|时间计算单位，就像系统时间单位换为 60 秒一样
duration|4|通过这个值可以得到影片的播放长度时间值
rate|4|播放速度，1.0 为正常播放速度（16.16 的浮点表示）
volume|2|播放音量，1.0 为最大音量（8.8 的浮点表示）
reserved|10|保留，这里是 0
matrix|36|该矩阵定义了两个坐标空间的映射关系
预览时间|4|开始预览的时间
预览 duration|4|以 time scale 为单位，预览的 duration
poster time|4|poster 的时间值
selection time|4|当前选择时间的开始时间值
selection duration|4|当前选择时间的计算后的时间值
当前时间|4|当前时间
next_track_ID|4|下一个待添加 track 的 ID 值，0 不是一个有效的 ID 值

### **iods（Initial Object Descriptor）**

### **trak（Track）**

trak box 是一个 container box，其子 box 包含了该 track 的媒体数据引用和描述（流媒体协议的打包信息 hint track 除外，其可以引用或复制对应的媒体采样数据）。一个 MP4 文件中的媒体有一个或多个 track，这些 track 之间彼此独立，有自己的时间和空间信息。trak box 必须包含一个 tkhd box 和一个 mdia box。

定义：

{% codeblock lang:c %}
aligned(8) class TrackBox extends Box(‘trak’) 
{
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|trak

#### **tkhd（Track Header）**

包含了该 track 的特性和总体信息，如时长，宽高等。

定义：

{% codeblock lang:c %}
aligned(8) class TrackHeaderBox extends FullBox(‘tkhd’, version, flags)
{
    if (version==1) 
    {
        unsigned int(64) creation_time;
        unsigned int(64) modification_time;
        unsigned int(32) track_ID;
        const unsigned int(32) reserved = 0;
        unsigned int(64) duration;
    } 
    else 
    { // version==0
        unsigned int(32) creation_time;
        unsigned int(32) modification_time;
        unsigned int(32) track_ID;
        const unsigned int(32) reserved = 0;
        unsigned int(32) duration;
    }
    const unsigned int(32)[2] reserved = 0;
    template int(16) layer = 0;
    template int(16) alternate_group = 0;
    template int(16) volume = {if track_is_audio 0x0100 else 0};
    const unsigned int(16) reserved = 0;
    // unity matrix
    template int(32)[9] matrix= { 0x00010000,0,0,0,0x00010000,0,0,0,0x40000000 };
    unsigned int(32) width;
    unsigned int(32) height;
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|tkhd
version|1|版本
flags|3|有效的标志：<br>0x0001：track 生效<br>0x0002：track 被用在 Movie 中<br>0x0003：track 被用在 Movie 预览中<br>0x0004：track 被用在 Movie 的 poster 中
creation_time|4|起始时间，基准时间是 1904-1-1 0:00 AM
modification_time|4|修订时间，基准时间是 1904-1-1 0:00 AM
track_ID|4|唯一标志该 track 的一个非零值
reserved|4|这里为 0
duration|4|track 的 duration，在电影的时间戳中。与 track 的 edts list 进行的时间戳会建立关联，然后进行时间戳计算，得到对应的 track 的播放时间坐标
reserved|8|这里为 0
layer|2|视频层，默认为 0，值小的在上层
alternate_group|2|track 分组信息，默认为 0，表示该 track 未与其他 track 有群组关系
volume|2|播放此 track 的音量。1.0 为正常音量
reserved|2|这里为 0
matrix|36|该矩阵定义了两个坐标空间的映射关系
width|4|如果该 track 是 video track，那么此值为图像的宽度（16.16 浮点表示）
height|4|如果该 track 是 video track，那么此值为图像的高度（16.16 浮点表示）

#### **mdia（Media）**

包含了该 track 的媒体信息，比如媒体类型和 sample 信息。其必须包含：
（1）一个媒体头 mdhd；
（2）一个句柄参考 hdlr；
（3）一个媒体信息 minf 和 udta。

定义：

{% codeblock lang:c %}
aligned(8) class MediaBox extends Box(‘mdia’) 
{
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|mdia

##### **mdhd（Media Header）**

包含了该 track 的总体信息，mdhd 和 tkhd 内容大致都是一样的。tkhd 通常是对指定的 track 设定相关属性和内容，而 mdhd 是针对于独立的 media 来设置的，一般情况下二者相同。

定义：

{% codeblock lang:c %}
aligned(8) class MediaHeaderBox extends FullBox(‘mdhd’, version, 0) 
{
    if (version==1) 
    {
        unsigned int(64) creation_time;
        unsigned int(64) modification_time;
        unsigned int(32) timescale;
        unsigned int(64) duration;
    } 
    else 
    { 
        // version==0
        unsigned int(32) creation_time;
        unsigned int(32) modification_time;
        unsigned int(32) timescale;
        unsigned int(32) duration;
    }
    bit(1) pad = 0;
    unsigned int(5)[3] language; // ISO-639-2/T language code
    unsigned int(16) pre_defined = 0;
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|mdhd
version|1|版本
flags|3|扩展的标志，这里是 0
creation_time|4|起始时间，基准时间是 1904-1-1 0:00 AM
modification_time|4|修订时间，基准时间是 1904-1-1 0:00 AM
timescale|4|时间计算单位，就像系统时间单位换为 60 秒一样
duration|4|通过这个值可以得到影片的播放长度时间值
language|2|媒体的语言码
质量|2|媒体的回放质量

##### **hdlr（Handler Reference）**

解释了媒体的播放过程信息，该 box 也可以被包含在 meta box（meta）中。

定义：

{% codeblock lang:c %}
aligned(8) class HandlerBox extends FullBox(‘hdlr’, version = 0, 0) 
{
    unsigned int(32) pre_defined = 0;
    unsigned int(32) handler_type;
    const unsigned int(32)[3] reserved = 0;
    string name;
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|hdlr
version|1|版本
flags|3|扩展的标志，这里是 0
handler_type|4|有效的 handle 类型：<br>（1）mhlr：media handlers<br>（2）dhlr：data handlers
component_type|4|media handler 或 data handler 的类型。如果 component type 是 mhlr，那么这个字段定义的就是数据的类型，例如，vide 是 video 数据，soun 是 sound 数据；如果 component type 是 dhlr，那么这个字段定义的就是数据引用的类型，例如，alis 是文件的别名
reserved|12|保留字段，默认为 0
name|可变|这个 component 的名字，也就是生成此 media 的 media handler。该字段的长度可以为 0

##### **minf（Media Information）**

minf box 包含了所有描述该 track 中的媒体信息的对象，信息存储在其子 box 中。

定义：

{% codeblock lang:c %}
aligned(8) class MediaInformationBox extends Box(‘minf’) 
{
}
{% endcodeblock %}

###### **vmhd（Video Media Information Header）**

用在视频 track 中，包含当前 track 的视频描述信息（如视频编码等信息）。

定义：

{% codeblock lang:c %}
aligned(8) class VideoMediaHeaderBox extends FullBox(‘vmhd’, version = 0, 1) 
{
    template unsigned int(16) graphicsmode = 0; // copy, see below
    template unsigned int(16)[3] opcolor = {0, 0, 0};
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|vmhd
version|1|版本
flags|3|扩展的标志，这里是 1
graphicsmode|2|传输模式，传输模式指定的布尔值
opcolor|6|颜色值，RGB 颜色值

###### **smhd（Sound Media Information Header）**

用在音频 track 中，包含当前 track 的音频描述信息（如编码格式等信息）。

定义：

{% codeblock lang:c %}
aligned(8) class SoundMediaHeaderBox extends FullBox(‘smhd’, version = 0, 0) 
{
    template int(16) balance = 0;
    const unsigned int(16) reserved = 0;
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|smhd
version|1|版本
flags|3|扩展的标志，这里是 0
balance|2|音频的均衡是用来控制计算机的两个扬声器的声音混合效果，一般是 0
reserved|2|保留字段，默认为 0

###### **dinf（Data Information）**

用于解释如何定位媒体信息，是一个 container box。dinf box 一般包含一个 dref box，即 data reference box。

定义：

{% codeblock lang:c %}
aligned(8) class DataInformationBox extends Box(‘dinf’) 
{
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|dinf
version|1|版本
flags|3|扩展的标志，这里是 0
条目数目|4|data reference 的数目

####### **dref（Data Reference）**

用于设置当前 Box 描述信息的 data_entry，dref box 下会包含若干个 url 或 urn，这些 box 组成一个表，用于定位 track 数据。简单的说，track 可以被分成若干段，每一段都可以根据 url 或 urn 指向的地址来获取数据，sample 描述中会用这些片段的序号将这些片段组成一个完整的 track。一般情况下，当数据被完全包含在文件中时，url 或 urn 中的定位字符串是空的。

定义：

{% codeblock lang:c %}
aligned(8) class DataEntryUrlBox (bit(24) flags) extends FullBox(‘url ’, version = 0, flags)
{
    string location;
}
aligned(8) class DataEntryUrnBox (bit(24) flags) extends FullBox(‘urn ’, version = 0, flags) 
{
    string name;
    string location;
}
aligned(8) class DataReferenceBox extends FullBox(‘dref’, version = 0, 0) 
{
    unsigned int(32) entry_count;
    for (i=1; i <= entry_count; i++) 
    {
        DataEntryBox(entry_version, entry_flags) data_entry;
    }
}
{% endcodeblock %}

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|url/alis/rsrc
version|1|版本
flags|3|扩展的标志，这里是 1
数据|可变|data reference 的信息

###### **stbl（Sample Table）**

包含转化媒体时间到实际的 sample 的信息，例如，视频数据是否需要被解压缩、解压缩算法是什么等信息。

定义：

{% codeblock lang:c %}
aligned(8) class SampleTableBox extends Box(‘stbl’) 
{
}
{% endcodeblock %}

####### **stsd**

box header 和 version 字段后会有一个 entry count 字段，根据 entry 的个数，每个 entry 会有 type 信息，如 vide、sund 等，根据 type 不同 sample description 会提供不同的信息，例如对于 video track，会有 VisualSampleEntry 类型信息，对于 audio track 会有 AudioSampleEntry 类型信息。视频的编码类型、宽高、长度，音频的声道、采样等信息都会出现在这个 box 中。

定义：

{% codeblock lang:c %}
aligned(8) abstract class SampleEntry (unsigned int(32) format) extends Box(format)
{
    const unsigned int(8)[6] reserved = 0;
    unsigned int(16) data_reference_index;
}

aligned(8) class SampleDescriptionBox (unsigned int(32) handler_type) extends FullBox('stsd', version, 0)
{
    int i ;
    unsigned int(32) entry_count;
    for (i = 1 ; i <= entry_count ; i++)
    {
        SampleEntry(); // an instance of a class derived from SampleEntry
    }
}
{% endcodeblock %}

######## **esds（Element Stream Descriptors）**

存放 Element Stream Descriptors。

参数：

字段|长度/字节|说明
--|--
size|4|字节数
type|4|esds
version|1|版本
flags|3|扩展的标志，这里是 0
数据|可变|数据

####### **stts**

stts box 存储了 sample 的 duration，描述了 sample 时序的映射方法，我们通过它可以找到任何时间的 sample。stts box 可以包含一个压缩的表来映射时间和 sample 序号，用其他的表来提供每个 sample 的长度和指针。表中每个条目提供了在同一个时间偏移量里面连续的 sample 序号，以及 samples 的偏移量。递增这些偏移量，就可以建立一个完整的 time to sample 表（时间戳到 sample 序号的映射表）。

定义：

{% codeblock lang:c %}
aligned(8) class TimeToSampleBox extends FullBox(’stts’, version = 0, 0) 
{
    unsigned int(32) entry_count;
    int i;
    for (i=0; i < entry_count; i++) 
    {
        unsigned int(32) sample_count;
        unsigned int(32) sample_delta;
    }
}
{% endcodeblock %}

####### **stsz**

stsz 定义了每个 sample 的大小，包含了媒体中全部 sample 的数目和一张给出每个 sample 大小的表。这个 box 相对来说体积是比较大的。

定义：

{% codeblock lang:c %}
aligned(8) class SampleSizeBox extends FullBox(‘stsz’, version = 0, 0) 
{
    unsigned int(32) sample_size;
    unsigned int(32) sample_count;
    if (sample_size==0) 
    {
        for (i=1; i <= sample_count; i++) 
        {
            unsigned int(32) entry_size;
        }
    }
}
{% endcodeblock %}

####### **stsc**

用 chunk 组织 sample 可以方便优化数据获取，一个 thunk 包含一个或多个 sample。stsc 中用一个表描述了 sample 与 chunk 的映射关系，查看这张表就可以找到包含指定 sample 的 thunk，从而找到这个 sample。

定义：

{% codeblock lang:c %}
aligned(8) class SampleToChunkBox extends FullBox(‘stsc’, version = 0, 0) 
{
    unsigned int(32) entry_count;
    for (i=1; i <= entry_count; i++) 
    {
        unsigned int(32) first_chunk;
        unsigned int(32) samples_per_chunk;
        unsigned int(32) sample_description_index;
    }
}
{% endcodeblock %}

####### **stco**

stco 定义了每个 thunk 在媒体流中的位置，sample 的偏移可以根据其他 box 推算出来。位置有两种可能，32 位的和 64 位的，后者对非常大的电影很有用。在一个表中只会有一种可能，这个位置是在整个文件中的，而不是在任何 box 中的，这样做就可以直接在文件中找到媒体数据，而不用解释 box。需要注意的是一旦前面的 box 有了任何改变，这张表都要重新建立，因为位置信息已经改变了。

定义：

{% codeblock lang:c %}
aligned(8) class ChunkOffsetBox extends FullBox(‘stco’, version = 0, 0) 
{
    unsigned int(32) entry_count;
    for (i=1; i <= entry_count; i++) 
    {
        unsigned int(32) chunk_offset;
    }
}
aligned(8) class ChunkLargeOffsetBox extends FullBox(‘co64’, version = 0, 0) 
{
    unsigned int(32) entry_count;
    for (i=1; i <= entry_count; i++) 
    {
        unsigned int(64) chunk_offset;
    }
}
{% endcodeblock %}

####### **stss**

stss 确定 media 中的关键帧。对于压缩媒体数据，关键帧是一系列压缩序列的开始帧，其解压缩时不依赖以前的帧，而后续帧的解压缩将依赖于这个关键帧。stss 可以非常紧凑的标记媒体内的随机存取点，它包含一个 sample 序号表，表内的每一项严格按照 sample 的序号排列，说明了媒体中的哪一个 sample 是关键帧。如果此表不存在，说明每一个 sample 都是一个关键帧，是一个随机存取点。

定义：

{% codeblock lang:c %}
aligned(8) class SyncSampleBox extends FullBox(‘stss’, version = 0, 0) 
{
    unsigned int(32) entry_count;
    int i;
    for (i=0; i < entry_count; i++) 
    {
        unsigned int(32) sample_number;
    }
}
{% endcodeblock %}

####### **sdtp（Sample Degradation Priority）**

### **udta（User Data）**

#### **meta（Metadata）**

##### **ilst（未确定用途）**

---

# **将 MP4 文件按字节转十六进制**

{% codeblock lang:java %}
public class Changer {
	public static byte[] subBytes(byte[] src, int begin, int count) {
		byte[] bs = new byte[count];
		System.arraycopy(src, begin, bs, 0, count);
		return bs;
	}

	public static String converHexStr(byte[] bytes) {
		StringBuffer stringBuffer = new StringBuffer();
		for (int i = 0, size = bytes.length, halfSize = size >> 1; i < size; i++) {
			if (i == halfSize) {
				stringBuffer.append(" ");// 从中间区分出来
			}
			if ((bytes[i] & 0xff) < 0x10) {
				stringBuffer.append("0");// 补0
			}
			stringBuffer.append(Long.toString(bytes[i] & 0xff, 16) + " ");// 转十六进制
		}
		return stringBuffer.toString().trim();
	}

	public static String convertUtf8Str(byte[] bytes) {
		try {
			byte[] newBytes = new byte[bytes.length];
			for (int i = 0, size = bytes.length; i < size; i++) {
				if (bytes[i] < 32 || bytes[i] > 126) {
					newBytes[i] = 0x20;// 不识别的使用空格代替
				} else {
					newBytes[i] = bytes[i];
				}
			}
			return new String(newBytes, "utf-8");
		} catch (UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
	}

	public static void main(String[] args) {
		File inFile = new File("d:/sample.mp4");
		File outFile = new File("d:/sample.txt");

		DecimalFormat format = new DecimalFormat("00000000");
		int line = 0;
		// int count = 0;

		InputStream in = null;
		PrintWriter out = null;
		byte[] bytes = new byte[bufferSize];
		try {
			in = new FileInputStream(inFile);
			out = new PrintWriter(outFile);
			int len;
			while ((len = in.read(bytes)) != -1) {
				// count++;
				// System.out.println("count: " + count);
				// if (len < bufferSize) {
				// System.out.println("查询到最后一次");
				// }
				int i = 0;
				for (; i < len; i += lineSize) {
					line++;
					byte[] subBytes = subBytes(bytes, i, lineSize);
					out.write(
							format.format(line) + "    " + converHexStr(subBytes) + "    " + convertUtf8Str(subBytes));
					out.write("\r\n");
				}
			}
			out.flush();
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			if (in != null) {
				try {
					in.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
}
{% endcodeblock %}

---

# **实际 MP4 文件的 box 树**

## **ftyp**

{% codeblock lang:shell %}
00000001    00 00 00 1c 66 74 79 70  6d 70 34 32 00 00 00 00        ftypmp42    
00000002    6d 70 34 32 69 73 6f 6d  61 76 63 31                mp42isomavc1
{% endcodeblock %}

## **free**

{% codeblock lang:shell %}
00000002                                         00 00 00 84    
00000003    66 72 65 65 00 00 00 00  00 00 00 00 00 00 00 00    free            
00000004    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00000005    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00000006    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00000007    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00000008    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00000009    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00000010    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
{% endcodeblock %}

## **mdat**

{% codeblock lang:shell %}
00000011    00 05 cb e8 6d 64 61 74  00 00 02 09 06 05 ff ff        mdat        
00000012    05 dc 45 e9 bd e6 d9 48  b7 96 2c d8 20 d9 23 ee      E    H  ,   # 
00000013    ef 78 32 36 34 20 2d 20  63 6f 72 65 20 37 39 20     x264 - core 79 
.
（内容超多，会导致编辑器渲染时间过长，此处省略）
.
00023751    37 e5 dd ef 14 00 00 00  00 00 ff 13 e0 34 f6 58    7            4 X
00023752    00 00 00 00 00 fe ed d2  ff 74 00 00 1e dc 77 00             t    w 
00023753    00 00 00 00 00 00 03 80
{% endcodeblock %}

## **moov**

{% codeblock lang:shell %}
00023753                             00 00 0d 83 6d 6f 6f 76                moov
00023754    00 00 00 6c 6d 76 68 64  00 00 00 00 c7 ca ee a7       lmvhd        
00023755    c7 ca ee a8 00 01 5f 90  00 07 a5 80 00 01 00 00          _         
00023756    01 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023757    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023758    00 00 00 00 00 00 00 00  00 00 00 00 40 00 00 00                @   
00023759    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023760    00 00 00 00 00 00 00 00  00 00 00 03 00 00 00 18                    
00023761    69 6f 64 73 00 00 00 00  10 80 80 80 07 00 4f ff    iods          O 
00023762    ff 0f 7f ff 00 00 06 0a  74 72 61 6b 00 00 00 5c            trak   \
00023763    74 6b 68 64 00 00 00 01  c7 ca ee a7 c7 ca ee a8    tkhd            
00023764    00 00 00 01 00 00 00 00  00 07 99 50 00 00 00 00               P    
00023765    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023766    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023767    00 00 00 00 00 00 00 00  00 00 00 00 40 00 00 00                @   
00023768    02 30 00 00 01 40 00 00  00 00 05 a6 6d 64 69 61     0   @      mdia
00023769    00 00 00 20 6d 64 68 64  00 00 00 00 c7 ca ee a7        mdhd        
00023770    c7 ca ee a8 00 01 5f 90  00 07 99 50 55 c4 00 00          _    PU   
00023771    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023772    76 69 64 65 00 00 00 00  00 00 00 00 00 00 00 00    vide            
00023773    00 00 00 05 5d 6d 69 6e  66 00 00 00 14 76 6d 68        ]minf    vmh
00023774    64 00 00 00 01 00 00 00  00 00 00 00 00 00 00 00    d               
00023775    24 64 69 6e 66 00 00 00  1c 64 72 65 66 00 00 00    $dinf    dref   
00023776    00 00 00 00 01 00 00 00  0c 75 72 6c 20 00 00 00             url    
00023777    01 00 00 05 1d 73 74 62  6c 00 00 00 ab 73 74 73         stbl    sts
00023778    64 00 00 00 00 00 00 00  01 00 00 00 9b 61 76 63    d            avc
00023779    31 00 00 00 00 00 00 00  01 00 00 00 00 00 00 00    1               
00023780    00 00 00 00 00 00 00 00  00 02 30 01 40 00 48 00              0 @ H 
00023781    00 00 48 00 00 00 00 00  00 00 01 0e 4a 56 54 2f      H         JVT/
00023782    41 56 43 20 43 6f 64 69  6e 67 00 00 00 00 00 00    AVC Coding      
00023783    00 00 00 00 00 00 00 00  00 00 00 00 18 ff ff 00                    
00023784    00 00 33 61 76 63 43 01  42 c0 1e ff e1 00 1b 67      3avcC B      g
00023785    42 c0 1e 9e 21 81 18 53  4d 40 40 40 50 00 00 03    B   !  SM@@@P   
00023786    00 10 00 00 03 03 c8 f1  62 ee 01 00 05 68 ce 06            b    h  
00023787    cb 20 00 00 00 12 63 6f  6c 72 6e 63 6c 63 00 01          colrnclc  
00023788    00 01 00 01 00 00 00 18  73 74 74 73 00 00 00 00            stts    
00023789    00 00 00 01 00 00 00 a6  00 00 0b b8 00 00 02 ac                    
00023790    73 74 73 7a 00 00 00 00  00 00 00 00 00 00 00 a6    stsz            
00023791    00 00 56 27 00 00 0b 20  00 00 05 bc 00 00 05 e2      V'            
00023792    00 00 05 c1 00 00 04 37  00 00 04 07 00 00 03 b6           7        
00023793    00 00 06 45 00 00 03 73  00 00 05 12 00 00 03 26       E   s       &
00023794    00 00 02 e9 00 00 03 7b  00 00 03 4a 00 00 03 6b           {   J   k
00023795    00 00 02 b6 00 00 03 4c  00 00 02 7a 00 00 02 c7           L   z    
00023796    00 00 02 2e 00 00 03 16  00 00 02 26 00 00 02 7f       .       &    
00023797    00 00 01 ec 00 00 01 ea  00 00 01 f5 00 00 01 eb                    
00023798    00 00 01 fa 00 00 01 e7  00 00 01 fc 00 00 01 dd                    
00023799    00 00 01 c6 00 00 01 ae  00 00 01 c8 00 00 01 b9                    
00023800    00 00 01 90 00 00 01 93  00 00 01 8c 00 00 01 da                    
00023801    00 00 01 c2 00 00 05 d0  00 00 07 b8 00 00 06 7a                   z
00023802    00 00 09 a9 00 00 0a 2c  00 00 0a 7c 00 00 0c b3           ,   |    
00023803    00 00 09 8c 00 00 09 52  00 00 0c 04 00 00 0d c1           R        
00023804    00 00 0f 74 00 00 10 48  00 00 11 06 00 00 10 61       t   H       a
00023805    00 00 0c 63 00 00 0c 31  00 00 0b 42 00 00 0c 0d       c   1   B    
00023806    00 00 0f 32 00 00 0a 7b  00 00 0d 0f 00 00 0a e0       2   {        
00023807    00 00 0a 0e 00 00 0b 6b  00 00 08 74 00 00 0c 36           k   t   6
00023808    00 00 09 e6 00 00 06 8d  00 00 04 f8 00 00 07 8a                    
00023809    00 00 07 c1 00 00 09 f3  00 00 07 c7 00 00 0a cb                    
00023810    00 00 0a d2 00 00 0b 74  00 00 0c 28 00 00 0a 9a           t   (    
00023811    00 00 0c 60 00 00 0d 6d  00 00 0c 3e 00 00 0f fc       `   m   >    
00023812    00 00 0e 82 00 00 0b 79  00 00 0d e4 00 00 0d 24           y       $
00023813    00 00 0a 17 00 00 11 aa  00 00 12 65 00 00 0d 7b               e   {
00023814    00 00 12 a0 00 00 13 d8  00 00 11 49 00 00 0e 59               I   Y
00023815    00 00 10 15 00 00 16 81  00 00 09 b4 00 00 06 eb                    
00023816    00 00 05 ef 00 00 05 8a  00 00 03 d7 00 00 04 0d                    
00023817    00 00 03 bb 00 00 04 6b  00 00 03 40 00 00 03 30           k   @   0
00023818    00 00 02 de 00 00 03 ae  00 00 05 cf 00 00 04 6c                   l
00023819    00 00 05 69 00 00 05 00  00 00 06 a1 00 00 03 35       i           5
00023820    00 00 04 1a 00 00 03 fa  00 00 06 3d 00 00 05 d6               =    
00023821    00 00 04 68 00 00 02 d6  00 00 04 b5 00 00 02 d9       h            
00023822    00 00 02 7f 00 00 02 4d  00 00 02 7d 00 00 03 8c           M   }    
00023823    00 00 02 06 00 00 02 01  00 00 07 7f 00 00 05 ef                    
00023824    00 00 05 b8 00 00 04 0a  00 00 02 99 00 00 03 1d                    
00023825    00 00 07 c5 00 00 05 ac  00 00 04 78 00 00 08 71               x   q
00023826    00 00 08 99 00 00 08 e9  00 00 08 99 00 00 05 73                   s
00023827    00 00 07 c7 00 00 08 3d  00 00 0b 59 00 00 0a 36           =   Y   6
00023828    00 00 06 ba 00 00 05 f9  00 00 07 2e 00 00 06 eb               .    
00023829    00 00 04 c6 00 00 04 ba  00 00 05 66 00 00 04 31               f   1
00023830    00 00 06 8a 00 00 06 cf  00 00 06 fe 00 00 04 97                    
00023831    00 00 02 43 00 00 03 e2  00 00 04 06 00 00 02 e6       C            
00023832    00 00 02 6b 00 00 02 75  00 00 00 28 73 74 73 63       k   u   (stsc
00023833    00 00 00 00 00 00 00 02  00 00 00 01 00 00 00 04                    
00023834    00 00 00 01 00 00 00 2a  00 00 00 02 00 00 00 01           *        
00023835    00 00 00 b8 73 74 63 6f  00 00 00 00 00 00 00 2a        stco       *
00023836    00 00 00 a8 00 00 73 e6  00 00 8b f4 00 00 a4 08          s         
00023837    00 00 b7 64 00 00 c8 a5  00 00 d7 d8 00 00 e4 a5       d            
00023838    00 00 ec 5f 00 00 f8 5f  00 01 04 41 00 01 1f c3       _   _   A    
00023839    00 01 51 85 00 01 84 7e  00 01 cc a6 00 02 03 c0      Q    ~        
00023840    00 02 3c 52 00 02 66 75  00 02 8a 1c 00 02 b4 a6      <R  fu        
00023841    00 02 e7 66 00 03 23 01  00 03 5d ac 00 03 9e 97       f  #   ]     
00023842    00 03 ea 64 00 04 26 0a  00 04 3e 69 00 04 4c ff       d  &   >i  L 
00023843    00 04 63 d9 00 04 7e 43  00 04 98 9b 00 04 ad a9      c   ~C        
00023844    00 04 be f7 00 04 d7 94  00 04 ed 6c 00 05 0e 3b               l   ;
00023845    00 05 2d c9 00 05 59 f0  00 05 7b 82 00 05 95 2b      -   Y   {    +
00023846    00 05 b3 da 00 05 c6 67  00 00 00 14 73 74 73 73           g    stss
00023847    00 00 00 00 00 00 00 01  00 00 00 01 00 00 00 b2                    
00023848    73 64 74 70 00 00 00 00  04 44 44 44 44 44 44 44    sdtp     DDDDDDD
00023849    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023850    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023851    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023852    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023853    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023854    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023855    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023856    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023857    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023858    44 44 44 44 44 44 44 44  44 44 44 44 44 44 00 00    DDDDDDDDDDDDDD  
00023859    06 7e 74 72 61 6b 00 00  00 5c 74 6b 68 64 00 00     ~trak   \tkhd  
00023860    00 03 c7 ca ee a7 c7 ca  ee a8 00 00 00 02 00 00                    
00023861    00 00 00 07 a5 80 00 00  00 00 00 00 00 00 00 00                    
00023862    00 00 01 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023863    00 00 00 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023864    00 00 00 00 00 00 40 00  00 00 00 00 00 00 00 00          @         
00023865    00 00 00 00 06 04 6d 64  69 61 00 00 00 20 6d 64          mdia    md
00023866    68 64 00 00 00 00 c7 ca  ee a7 c7 ca ee a8 00 00    hd              
00023867    bb 80 00 04 14 00 15 c7  00 00 00 00 00 21 68 64                 !hd
00023868    6c 72 00 00 00 00 00 00  00 00 73 6f 75 6e 00 00    lr        soun  
00023869    00 00 00 00 00 00 00 00  00 00 00 00 00 05 bb 6d                   m
00023870    69 6e 66 00 00 00 10 73  6d 68 64 00 00 00 00 00    inf    smhd     
00023871    00 00 00 00 00 00 24 64  69 6e 66 00 00 00 1c 64          $dinf    d
00023872    72 65 66 00 00 00 00 00  00 00 01 00 00 00 0c 75    ref            u
00023873    72 6c 20 00 00 00 01 00  00 05 7f 73 74 62 6c 00    rl         stbl 
00023874    00 00 67 73 74 73 64 00  00 00 00 00 00 00 01 00      gstsd         
00023875    00 00 57 6d 70 34 61 00  00 00 00 00 00 00 01 00      Wmp4a         
00023876    00 00 00 00 00 00 00 00  01 00 10 00 00 00 00 bb                    
00023877    80 00 00 00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02 00 00  00 18 73 74 74 73 00 00              stts  
00023881    00 00 00 00 00 01 00 00  01 05 00 00 04 00 00 00                    
00023882    04 28 73 74 73 7a 00 00  00 00 00 00 00 00 00 00     (stsz          
00023883    01 05 00 00 00 f7 00 00  00 db 00 00 00 e1 00 00                    
00023884    00 e5 00 00 00 e9 00 00  00 e8 00 00 00 f0 00 00                    
00023885    00 f1 00 00 00 ef 00 00  00 d8 00 00 00 e6 00 00                    
00023886    00 e7 00 00 00 e9 00 00  00 eb 00 00 00 ea 00 00                    
00023887    00 e8 00 00 00 e1 00 00  00 e7 00 00 00 d7 00 00                    
00023888    00 da 00 00 00 d9 00 00  00 db 00 00 00 e9 00 00                    
00023889    00 ee 00 00 00 e5 00 00  00 e1 00 00 00 e6 00 00                    
00023890    00 e5 00 00 00 d8 00 00  00 dd 00 00 00 dd 00 00                    
00023891    00 d5 00 00 00 ea 00 00  00 dd 00 00 00 d0 00 00                    
00023892    00 d6 00 00 00 e9 00 00  00 bc 00 00 00 ab 00 00                    
00023893    00 b3 00 00 00 b5 00 00  00 bc 00 00 00 ce 00 00                    
00023894    00 b4 00 00 00 b6 00 00  00 b3 00 00 00 b6 00 00                    
00023895    00 b7 00 00 00 bf 00 00  00 b7 00 00 00 cd 00 00                    
00023896    00 c1 00 00 00 ba 00 00  00 a7 00 00 00 b4 00 00                    
00023897    00 b1 00 00 00 be 00 00  00 d0 00 00 00 ba 00 00                    
00023898    00 bc 00 00 00 c4 00 00  00 c6 00 00 00 cb 00 00                    
00023899    00 c4 00 00 00 c3 00 00  00 c8 00 00 00 d2 00 00                    
00023900    00 d2 00 00 00 d6 00 00  00 f5 00 00 00 fa 00 00                    
00023901    00 f6 00 00 01 02 00 00  00 fc 00 00 00 fc 00 00                    
00023902    00 ee 00 00 00 e6 00 00  00 ea 00 00 00 ea 00 00                    
00023903    00 e8 00 00 00 de 00 00  00 df 00 00 00 e7 00 00                    
00023904    00 f6 00 00 00 ff 00 00  01 03 00 00 00 f6 00 00                    
00023905    01 08 00 00 01 03 00 00  00 fd 00 00 01 05 00 00                    
00023906    01 02 00 00 01 00 00 00  01 14 00 00 01 18 00 00                    
00023907    00 fd 00 00 00 fb 00 00  01 11 00 00 01 05 00 00                    
00023908    01 05 00 00 01 0a 00 00  01 01 00 00 00 f3 00 00                    
00023909    00 f7 00 00 00 f7 00 00  01 01 00 00 01 02 00 00                    
00023910    00 f8 00 00 00 f8 00 00  00 ef 00 00 00 ed 00 00                    
00023911    00 e3 00 00 00 ec 00 00  00 e2 00 00 00 e8 00 00                    
00023912    00 dc 00 00 00 e0 00 00  00 f3 00 00 00 df 00 00                    
00023913    00 e1 00 00 00 cf 00 00  00 ce 00 00 00 d8 00 00                    
00023914    00 ce 00 00 00 c7 00 00  00 cd 00 00 00 b7 00 00                    
00023915    00 af 00 00 00 c8 00 00  00 d7 00 00 00 e5 00 00                    
00023916    00 e4 00 00 00 c6 00 00  00 d1 00 00 00 d5 00 00                    
00023917    00 e5 00 00 00 d8 00 00  00 c8 00 00 00 be 00 00                    
00023918    00 bf 00 00 00 cb 00 00  00 d2 00 00 00 c8 00 00                    
00023919    00 ca 00 00 00 b1 00 00  00 a3 00 00 00 c7 00 00                    
00023920    00 dc 00 00 00 d9 00 00  00 dd 00 00 00 d1 00 00                    
00023921    00 d2 00 00 00 c2 00 00  00 bc 00 00 00 b1 00 00                    
00023922    00 9b 00 00 00 89 00 00  00 a2 00 00 00 9f 00 00                    
00023923    00 b5 00 00 00 a6 00 00  00 b2 00 00 00 b5 00 00                    
00023924    00 ae 00 00 00 b4 00 00  00 b0 00 00 00 c6 00 00                    
00023925    00 c3 00 00 00 d5 00 00  00 e4 00 00 00 f6 00 00                    
00023926    00 d6 00 00 00 db 00 00  00 cc 00 00 00 e7 00 00                    
00023927    00 f9 00 00 00 cb 00 00  00 d8 00 00 00 d6 00 00                    
00023928    00 e4 00 00 00 f1 00 00  00 e4 00 00 00 e6 00 00                    
00023929    00 df 00 00 00 ee 00 00  00 d7 00 00 00 c7 00 00                    
00023930    00 e7 00 00 00 f9 00 00  00 ed 00 00 00 cf 00 00                    
00023931    00 f1 00 00 00 e6 00 00  00 dc 00 00 00 e4 00 00                    
00023932    00 ef 00 00 00 e5 00 00  00 f1 00 00 00 e3 00 00                    
00023933    00 ec 00 00 00 ec 00 00  00 f3 00 00 00 f5 00 00                    
00023934    00 fd 00 00 01 0b 00 00  01 10 00 00 01 11 00 00                    
00023935    01 03 00 00 01 01 00 00  00 fb 00 00 00 fa 00 00                    
00023936    00 e7 00 00 00 e5 00 00  00 f0 00 00 00 d2 00 00                    
00023937    00 e5 00 00 00 f3 00 00  00 f1 00 00 00 f2 00 00                    
00023938    00 ff 00 00 00 f7 00 00  00 ee 00 00 00 d5 00 00                    
00023939    00 d9 00 00 00 ea 00 00  00 e3 00 00 00 df 00 00                    
00023940    00 f7 00 00 00 ff 00 00  00 f8 00 00 00 fa 00 00                    
00023941    00 fd 00 00 00 f7 00 00  00 f9 00 00 00 fb 00 00                    
00023942    00 f8 00 00 00 f6 00 00  00 f0 00 00 00 fe 00 00                    
00023943    01 02 00 00 00 e9 00 00  00 ec 00 00 00 ec 00 00                    
00023944    00 e7 00 00 00 ea 00 00  00 de 00 00 00 e2 00 00                    
00023945    00 c9 00 00 00 d4 00 00  00 d4 00 00 00 c7 00 00                    
00023946    00 c9 00 00 00 c8 00 00  00 c1 00 00 00 c0 00 00                    
00023947    00 bd 00 00 00 de 00 00  00 cb 00 00 00 cd 00 00                    
00023948    00 d4 00 00 00 6d 00 00  00 28 73 74 73 63 00 00         m   (stsc  
00023949    00 00 00 00 00 02 00 00  00 01 00 00 00 07 00 00                    
00023950    00 01 00 00 00 26 00 00  00 02 00 00 00 01 00 00         &          
00023951    00 a8 73 74 63 6f 00 00  00 00 00 00 00 26 00 00      stco       &  
00023952    6d 8d 00 00 85 9b 00 00  9d e4 00 00 b1 21 00 00    m            !  
00023953    c2 a7 00 00 d2 8e 00 00  df 8e 00 00 f3 54 00 00                 T  
00023954    fe e8 00 01 1a 05 00 01  4a c7 00 01 7e 28 00 01            J   ~(  
00023955    c5 a1 00 01 fc 89 00 02  35 5c 00 02 83 6a 00 02            5\   j  
00023956    ae 62 00 02 e1 ae 00 03  1d 6d 00 03 58 04 00 03     b       m  X   
00023957    99 4d 00 03 e4 b1 00 04  21 99 00 04 39 67 00 04     M      !   9g  
00023958    5d c6 00 04 78 18 00 04  92 6a 00 04 a7 67 00 04    ]   x    j   g  
00023959    b8 7e 00 04 d0 6c 00 04  e7 0c 00 05 07 c6 00 05     ~   l          
00023960    53 5c 00 05 74 bc 00 05  8e 99 00 05 ae 19 00 05    S\  t           
00023961    c0 eb 00 05 cb 47 00 00  00 16 75 64 74 61 00 00         G    udta  
00023962    00 0e 6e 61 6d 65 53 74  65 72 65 6f 00 00 00 6f      nameStereo   o
00023963    75 64 74 61 00 00 00 67  6d 65 74 61 00 00 00 00    udta   gmeta    
00023964    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023965    6d 64 69 72 00 00 00 00  00 00 00 00 00 00 00 00    mdir            
00023966    00 00 00 00 3a 69 6c 73  74 00 00 00 32 a9 74 6f        :ilst   2 to
00023967    6f 00 00 00 2a 64 61 74  61 00 00 00 01 00 00 00    o   *data       
00023968    00 48 61 6e 64 42 72 61  6b 65 20 30 2e 39 2e 34     HandBrake 0.9.4
00023969    20 32 30 30 39 31 31 32  33 30 30                    2009112300
{% endcodeblock %}

### **mvhd**

{% codeblock lang:shell %}
00023754    00 00 00 6c 6d 76 68 64  00 00 00 00 c7 ca ee a7       lmvhd        
00023755    c7 ca ee a8 00 01 5f 90  00 07 a5 80 00 01 00 00          _         
00023756    01 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023757    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023758    00 00 00 00 00 00 00 00  00 00 00 00 40 00 00 00                @   
00023759    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023760    00 00 00 00 00 00 00 00  00 00 00 03
{% endcodeblock %}

### **iods**

{% codeblock lang:shell %}
00023760                                         00 00 00 18                    
00023761    69 6f 64 73 00 00 00 00  10 80 80 80 07 00 4f ff    iods          O 
00023762    ff 0f 7f ff
{% endcodeblock %}

### **trak**

{% codeblock lang:shell %}
00023762                00 00 06 0a  74 72 61 6b 00 00 00 5c            trak   \
00023763    74 6b 68 64 00 00 00 01  c7 ca ee a7 c7 ca ee a8    tkhd            
00023764    00 00 00 01 00 00 00 00  00 07 99 50 00 00 00 00               P    
00023765    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023766    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023767    00 00 00 00 00 00 00 00  00 00 00 00 40 00 00 00                @   
00023768    02 30 00 00 01 40 00 00  00 00 05 a6 6d 64 69 61     0   @      mdia
00023769    00 00 00 20 6d 64 68 64  00 00 00 00 c7 ca ee a7        mdhd        
00023770    c7 ca ee a8 00 01 5f 90  00 07 99 50 55 c4 00 00          _    PU   
00023771    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023772    76 69 64 65 00 00 00 00  00 00 00 00 00 00 00 00    vide            
00023773    00 00 00 05 5d 6d 69 6e  66 00 00 00 14 76 6d 68        ]minf    vmh
00023774    64 00 00 00 01 00 00 00  00 00 00 00 00 00 00 00    d               
00023775    24 64 69 6e 66 00 00 00  1c 64 72 65 66 00 00 00    $dinf    dref   
00023776    00 00 00 00 01 00 00 00  0c 75 72 6c 20 00 00 00             url    
00023777    01 00 00 05 1d 73 74 62  6c 00 00 00 ab 73 74 73         stbl    sts
00023778    64 00 00 00 00 00 00 00  01 00 00 00 9b 61 76 63    d            avc
00023779    31 00 00 00 00 00 00 00  01 00 00 00 00 00 00 00    1               
00023780    00 00 00 00 00 00 00 00  00 02 30 01 40 00 48 00              0 @ H 
00023781    00 00 48 00 00 00 00 00  00 00 01 0e 4a 56 54 2f      H         JVT/
00023782    41 56 43 20 43 6f 64 69  6e 67 00 00 00 00 00 00    AVC Coding      
00023783    00 00 00 00 00 00 00 00  00 00 00 00 18 ff ff 00                    
00023784    00 00 33 61 76 63 43 01  42 c0 1e ff e1 00 1b 67      3avcC B      g
00023785    42 c0 1e 9e 21 81 18 53  4d 40 40 40 50 00 00 03    B   !  SM@@@P   
00023786    00 10 00 00 03 03 c8 f1  62 ee 01 00 05 68 ce 06            b    h  
00023787    cb 20 00 00 00 12 63 6f  6c 72 6e 63 6c 63 00 01          colrnclc  
00023788    00 01 00 01 00 00 00 18  73 74 74 73 00 00 00 00            stts    
00023789    00 00 00 01 00 00 00 a6  00 00 0b b8 00 00 02 ac                    
00023790    73 74 73 7a 00 00 00 00  00 00 00 00 00 00 00 a6    stsz            
00023791    00 00 56 27 00 00 0b 20  00 00 05 bc 00 00 05 e2      V'            
00023792    00 00 05 c1 00 00 04 37  00 00 04 07 00 00 03 b6           7        
00023793    00 00 06 45 00 00 03 73  00 00 05 12 00 00 03 26       E   s       &
00023794    00 00 02 e9 00 00 03 7b  00 00 03 4a 00 00 03 6b           {   J   k
00023795    00 00 02 b6 00 00 03 4c  00 00 02 7a 00 00 02 c7           L   z    
00023796    00 00 02 2e 00 00 03 16  00 00 02 26 00 00 02 7f       .       &    
00023797    00 00 01 ec 00 00 01 ea  00 00 01 f5 00 00 01 eb                    
00023798    00 00 01 fa 00 00 01 e7  00 00 01 fc 00 00 01 dd                    
00023799    00 00 01 c6 00 00 01 ae  00 00 01 c8 00 00 01 b9                    
00023800    00 00 01 90 00 00 01 93  00 00 01 8c 00 00 01 da                    
00023801    00 00 01 c2 00 00 05 d0  00 00 07 b8 00 00 06 7a                   z
00023802    00 00 09 a9 00 00 0a 2c  00 00 0a 7c 00 00 0c b3           ,   |    
00023803    00 00 09 8c 00 00 09 52  00 00 0c 04 00 00 0d c1           R        
00023804    00 00 0f 74 00 00 10 48  00 00 11 06 00 00 10 61       t   H       a
00023805    00 00 0c 63 00 00 0c 31  00 00 0b 42 00 00 0c 0d       c   1   B    
00023806    00 00 0f 32 00 00 0a 7b  00 00 0d 0f 00 00 0a e0       2   {        
00023807    00 00 0a 0e 00 00 0b 6b  00 00 08 74 00 00 0c 36           k   t   6
00023808    00 00 09 e6 00 00 06 8d  00 00 04 f8 00 00 07 8a                    
00023809    00 00 07 c1 00 00 09 f3  00 00 07 c7 00 00 0a cb                    
00023810    00 00 0a d2 00 00 0b 74  00 00 0c 28 00 00 0a 9a           t   (    
00023811    00 00 0c 60 00 00 0d 6d  00 00 0c 3e 00 00 0f fc       `   m   >    
00023812    00 00 0e 82 00 00 0b 79  00 00 0d e4 00 00 0d 24           y       $
00023813    00 00 0a 17 00 00 11 aa  00 00 12 65 00 00 0d 7b               e   {
00023814    00 00 12 a0 00 00 13 d8  00 00 11 49 00 00 0e 59               I   Y
00023815    00 00 10 15 00 00 16 81  00 00 09 b4 00 00 06 eb                    
00023816    00 00 05 ef 00 00 05 8a  00 00 03 d7 00 00 04 0d                    
00023817    00 00 03 bb 00 00 04 6b  00 00 03 40 00 00 03 30           k   @   0
00023818    00 00 02 de 00 00 03 ae  00 00 05 cf 00 00 04 6c                   l
00023819    00 00 05 69 00 00 05 00  00 00 06 a1 00 00 03 35       i           5
00023820    00 00 04 1a 00 00 03 fa  00 00 06 3d 00 00 05 d6               =    
00023821    00 00 04 68 00 00 02 d6  00 00 04 b5 00 00 02 d9       h            
00023822    00 00 02 7f 00 00 02 4d  00 00 02 7d 00 00 03 8c           M   }    
00023823    00 00 02 06 00 00 02 01  00 00 07 7f 00 00 05 ef                    
00023824    00 00 05 b8 00 00 04 0a  00 00 02 99 00 00 03 1d                    
00023825    00 00 07 c5 00 00 05 ac  00 00 04 78 00 00 08 71               x   q
00023826    00 00 08 99 00 00 08 e9  00 00 08 99 00 00 05 73                   s
00023827    00 00 07 c7 00 00 08 3d  00 00 0b 59 00 00 0a 36           =   Y   6
00023828    00 00 06 ba 00 00 05 f9  00 00 07 2e 00 00 06 eb               .    
00023829    00 00 04 c6 00 00 04 ba  00 00 05 66 00 00 04 31               f   1
00023830    00 00 06 8a 00 00 06 cf  00 00 06 fe 00 00 04 97                    
00023831    00 00 02 43 00 00 03 e2  00 00 04 06 00 00 02 e6       C            
00023832    00 00 02 6b 00 00 02 75  00 00 00 28 73 74 73 63       k   u   (stsc
00023833    00 00 00 00 00 00 00 02  00 00 00 01 00 00 00 04                    
00023834    00 00 00 01 00 00 00 2a  00 00 00 02 00 00 00 01           *        
00023835    00 00 00 b8 73 74 63 6f  00 00 00 00 00 00 00 2a        stco       *
00023836    00 00 00 a8 00 00 73 e6  00 00 8b f4 00 00 a4 08          s         
00023837    00 00 b7 64 00 00 c8 a5  00 00 d7 d8 00 00 e4 a5       d            
00023838    00 00 ec 5f 00 00 f8 5f  00 01 04 41 00 01 1f c3       _   _   A    
00023839    00 01 51 85 00 01 84 7e  00 01 cc a6 00 02 03 c0      Q    ~        
00023840    00 02 3c 52 00 02 66 75  00 02 8a 1c 00 02 b4 a6      <R  fu        
00023841    00 02 e7 66 00 03 23 01  00 03 5d ac 00 03 9e 97       f  #   ]     
00023842    00 03 ea 64 00 04 26 0a  00 04 3e 69 00 04 4c ff       d  &   >i  L 
00023843    00 04 63 d9 00 04 7e 43  00 04 98 9b 00 04 ad a9      c   ~C        
00023844    00 04 be f7 00 04 d7 94  00 04 ed 6c 00 05 0e 3b               l   ;
00023845    00 05 2d c9 00 05 59 f0  00 05 7b 82 00 05 95 2b      -   Y   {    +
00023846    00 05 b3 da 00 05 c6 67  00 00 00 14 73 74 73 73           g    stss
00023847    00 00 00 00 00 00 00 01  00 00 00 01 00 00 00 b2                    
00023848    73 64 74 70 00 00 00 00  04 44 44 44 44 44 44 44    sdtp     DDDDDDD
00023849    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023850    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023851    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023852    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023853    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023854    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023855    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023856    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023857    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023858    44 44 44 44 44 44 44 44  44 44 44 44 44 44          DDDDDDDDDDDDDD
{% endcodeblock %}

#### **tkhd**

{% codeblock lang:shell %}
00023762                                         00 00 00 5c                   \
00023763    74 6b 68 64 00 00 00 01  c7 ca ee a7 c7 ca ee a8    tkhd            
00023764    00 00 00 01 00 00 00 00  00 07 99 50 00 00 00 00               P    
00023765    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023766    00 00 00 00 00 00 00 00  00 00 00 00 00 01 00 00                    
00023767    00 00 00 00 00 00 00 00  00 00 00 00 40 00 00 00                @   
00023768    02 30 00 00 01 40 00 00                              0   @
{% endcodeblock %}

#### **mdia**

{% codeblock lang:shell %}
00023768                             00 00 05 a6 6d 64 69 61                mdia
00023769    00 00 00 20 6d 64 68 64  00 00 00 00 c7 ca ee a7        mdhd        
00023770    c7 ca ee a8 00 01 5f 90  00 07 99 50 55 c4 00 00          _    PU   
00023771    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023772    76 69 64 65 00 00 00 00  00 00 00 00 00 00 00 00    vide            
00023773    00 00 00 05 5d 6d 69 6e  66 00 00 00 14 76 6d 68        ]minf    vmh
00023774    64 00 00 00 01 00 00 00  00 00 00 00 00 00 00 00    d               
00023775    24 64 69 6e 66 00 00 00  1c 64 72 65 66 00 00 00    $dinf    dref   
00023776    00 00 00 00 01 00 00 00  0c 75 72 6c 20 00 00 00             url    
00023777    01 00 00 05 1d 73 74 62  6c 00 00 00 ab 73 74 73         stbl    sts
00023778    64 00 00 00 00 00 00 00  01 00 00 00 9b 61 76 63    d            avc
00023779    31 00 00 00 00 00 00 00  01 00 00 00 00 00 00 00    1               
00023780    00 00 00 00 00 00 00 00  00 02 30 01 40 00 48 00              0 @ H 
00023781    00 00 48 00 00 00 00 00  00 00 01 0e 4a 56 54 2f      H         JVT/
00023782    41 56 43 20 43 6f 64 69  6e 67 00 00 00 00 00 00    AVC Coding      
00023783    00 00 00 00 00 00 00 00  00 00 00 00 18 ff ff 00                    
00023784    00 00 33 61 76 63 43 01  42 c0 1e ff e1 00 1b 67      3avcC B      g
00023785    42 c0 1e 9e 21 81 18 53  4d 40 40 40 50 00 00 03    B   !  SM@@@P   
00023786    00 10 00 00 03 03 c8 f1  62 ee 01 00 05 68 ce 06            b    h  
00023787    cb 20 00 00 00 12 63 6f  6c 72 6e 63 6c 63 00 01          colrnclc  
00023788    00 01 00 01 00 00 00 18  73 74 74 73 00 00 00 00            stts    
00023789    00 00 00 01 00 00 00 a6  00 00 0b b8 00 00 02 ac                    
00023790    73 74 73 7a 00 00 00 00  00 00 00 00 00 00 00 a6    stsz            
00023791    00 00 56 27 00 00 0b 20  00 00 05 bc 00 00 05 e2      V'            
00023792    00 00 05 c1 00 00 04 37  00 00 04 07 00 00 03 b6           7        
00023793    00 00 06 45 00 00 03 73  00 00 05 12 00 00 03 26       E   s       &
00023794    00 00 02 e9 00 00 03 7b  00 00 03 4a 00 00 03 6b           {   J   k
00023795    00 00 02 b6 00 00 03 4c  00 00 02 7a 00 00 02 c7           L   z    
00023796    00 00 02 2e 00 00 03 16  00 00 02 26 00 00 02 7f       .       &    
00023797    00 00 01 ec 00 00 01 ea  00 00 01 f5 00 00 01 eb                    
00023798    00 00 01 fa 00 00 01 e7  00 00 01 fc 00 00 01 dd                    
00023799    00 00 01 c6 00 00 01 ae  00 00 01 c8 00 00 01 b9                    
00023800    00 00 01 90 00 00 01 93  00 00 01 8c 00 00 01 da                    
00023801    00 00 01 c2 00 00 05 d0  00 00 07 b8 00 00 06 7a                   z
00023802    00 00 09 a9 00 00 0a 2c  00 00 0a 7c 00 00 0c b3           ,   |    
00023803    00 00 09 8c 00 00 09 52  00 00 0c 04 00 00 0d c1           R        
00023804    00 00 0f 74 00 00 10 48  00 00 11 06 00 00 10 61       t   H       a
00023805    00 00 0c 63 00 00 0c 31  00 00 0b 42 00 00 0c 0d       c   1   B    
00023806    00 00 0f 32 00 00 0a 7b  00 00 0d 0f 00 00 0a e0       2   {        
00023807    00 00 0a 0e 00 00 0b 6b  00 00 08 74 00 00 0c 36           k   t   6
00023808    00 00 09 e6 00 00 06 8d  00 00 04 f8 00 00 07 8a                    
00023809    00 00 07 c1 00 00 09 f3  00 00 07 c7 00 00 0a cb                    
00023810    00 00 0a d2 00 00 0b 74  00 00 0c 28 00 00 0a 9a           t   (    
00023811    00 00 0c 60 00 00 0d 6d  00 00 0c 3e 00 00 0f fc       `   m   >    
00023812    00 00 0e 82 00 00 0b 79  00 00 0d e4 00 00 0d 24           y       $
00023813    00 00 0a 17 00 00 11 aa  00 00 12 65 00 00 0d 7b               e   {
00023814    00 00 12 a0 00 00 13 d8  00 00 11 49 00 00 0e 59               I   Y
00023815    00 00 10 15 00 00 16 81  00 00 09 b4 00 00 06 eb                    
00023816    00 00 05 ef 00 00 05 8a  00 00 03 d7 00 00 04 0d                    
00023817    00 00 03 bb 00 00 04 6b  00 00 03 40 00 00 03 30           k   @   0
00023818    00 00 02 de 00 00 03 ae  00 00 05 cf 00 00 04 6c                   l
00023819    00 00 05 69 00 00 05 00  00 00 06 a1 00 00 03 35       i           5
00023820    00 00 04 1a 00 00 03 fa  00 00 06 3d 00 00 05 d6               =    
00023821    00 00 04 68 00 00 02 d6  00 00 04 b5 00 00 02 d9       h            
00023822    00 00 02 7f 00 00 02 4d  00 00 02 7d 00 00 03 8c           M   }    
00023823    00 00 02 06 00 00 02 01  00 00 07 7f 00 00 05 ef                    
00023824    00 00 05 b8 00 00 04 0a  00 00 02 99 00 00 03 1d                    
00023825    00 00 07 c5 00 00 05 ac  00 00 04 78 00 00 08 71               x   q
00023826    00 00 08 99 00 00 08 e9  00 00 08 99 00 00 05 73                   s
00023827    00 00 07 c7 00 00 08 3d  00 00 0b 59 00 00 0a 36           =   Y   6
00023828    00 00 06 ba 00 00 05 f9  00 00 07 2e 00 00 06 eb               .    
00023829    00 00 04 c6 00 00 04 ba  00 00 05 66 00 00 04 31               f   1
00023830    00 00 06 8a 00 00 06 cf  00 00 06 fe 00 00 04 97                    
00023831    00 00 02 43 00 00 03 e2  00 00 04 06 00 00 02 e6       C            
00023832    00 00 02 6b 00 00 02 75  00 00 00 28 73 74 73 63       k   u   (stsc
00023833    00 00 00 00 00 00 00 02  00 00 00 01 00 00 00 04                    
00023834    00 00 00 01 00 00 00 2a  00 00 00 02 00 00 00 01           *        
00023835    00 00 00 b8 73 74 63 6f  00 00 00 00 00 00 00 2a        stco       *
00023836    00 00 00 a8 00 00 73 e6  00 00 8b f4 00 00 a4 08          s         
00023837    00 00 b7 64 00 00 c8 a5  00 00 d7 d8 00 00 e4 a5       d            
00023838    00 00 ec 5f 00 00 f8 5f  00 01 04 41 00 01 1f c3       _   _   A    
00023839    00 01 51 85 00 01 84 7e  00 01 cc a6 00 02 03 c0      Q    ~        
00023840    00 02 3c 52 00 02 66 75  00 02 8a 1c 00 02 b4 a6      <R  fu        
00023841    00 02 e7 66 00 03 23 01  00 03 5d ac 00 03 9e 97       f  #   ]     
00023842    00 03 ea 64 00 04 26 0a  00 04 3e 69 00 04 4c ff       d  &   >i  L 
00023843    00 04 63 d9 00 04 7e 43  00 04 98 9b 00 04 ad a9      c   ~C        
00023844    00 04 be f7 00 04 d7 94  00 04 ed 6c 00 05 0e 3b               l   ;
00023845    00 05 2d c9 00 05 59 f0  00 05 7b 82 00 05 95 2b      -   Y   {    +
00023846    00 05 b3 da 00 05 c6 67  00 00 00 14 73 74 73 73           g    stss
00023847    00 00 00 00 00 00 00 01  00 00 00 01 00 00 00 b2
{% endcodeblock %}

##### **mdhd**

{% codeblock lang:shell %}
00023769    00 00 00 20 6d 64 68 64  00 00 00 00 c7 ca ee a7        mdhd        
00023770    c7 ca ee a8 00 01 5f 90  00 07 99 50 55 c4 00 00          _    PU   
{% endcodeblock %}

##### **hdlr**

{% codeblock lang:shell %}
00023771    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023772    76 69 64 65 00 00 00 00  00 00 00 00 00 00 00 00    vide            
00
{% endcodeblock %}

##### **minf**

{% codeblock lang:shell %}
00023773       00 00 05 5d 6d 69 6e  66 00 00 00 14 76 6d 68        ]minf    vmh
00023774    64 00 00 00 01 00 00 00  00 00 00 00 00 00 00 00    d               
00023775    24 64 69 6e 66 00 00 00  1c 64 72 65 66 00 00 00    $dinf    dref   
00023776    00 00 00 00 01 00 00 00  0c 75 72 6c 20 00 00 00             url    
00023777    01 00 00 05 1d 73 74 62  6c 00 00 00 ab 73 74 73         stbl    sts
00023778    64 00 00 00 00 00 00 00  01 00 00 00 9b 61 76 63    d            avc
00023779    31 00 00 00 00 00 00 00  01 00 00 00 00 00 00 00    1               
00023780    00 00 00 00 00 00 00 00  00 02 30 01 40 00 48 00              0 @ H 
00023781    00 00 48 00 00 00 00 00  00 00 01 0e 4a 56 54 2f      H         JVT/
00023782    41 56 43 20 43 6f 64 69  6e 67 00 00 00 00 00 00    AVC Coding      
00023783    00 00 00 00 00 00 00 00  00 00 00 00 18 ff ff 00                    
00023784    00 00 33 61 76 63 43 01  42 c0 1e ff e1 00 1b 67      3avcC B      g
00023785    42 c0 1e 9e 21 81 18 53  4d 40 40 40 50 00 00 03    B   !  SM@@@P   
00023786    00 10 00 00 03 03 c8 f1  62 ee 01 00 05 68 ce 06            b    h  
00023787    cb 20 00 00 00 12 63 6f  6c 72 6e 63 6c 63 00 01          colrnclc  
00023788    00 01 00 01 00 00 00 18  73 74 74 73 00 00 00 00            stts    
00023789    00 00 00 01 00 00 00 a6  00 00 0b b8 00 00 02 ac                    
00023790    73 74 73 7a 00 00 00 00  00 00 00 00 00 00 00 a6    stsz            
00023791    00 00 56 27 00 00 0b 20  00 00 05 bc 00 00 05 e2      V'            
00023792    00 00 05 c1 00 00 04 37  00 00 04 07 00 00 03 b6           7        
00023793    00 00 06 45 00 00 03 73  00 00 05 12 00 00 03 26       E   s       &
00023794    00 00 02 e9 00 00 03 7b  00 00 03 4a 00 00 03 6b           {   J   k
00023795    00 00 02 b6 00 00 03 4c  00 00 02 7a 00 00 02 c7           L   z    
00023796    00 00 02 2e 00 00 03 16  00 00 02 26 00 00 02 7f       .       &    
00023797    00 00 01 ec 00 00 01 ea  00 00 01 f5 00 00 01 eb                    
00023798    00 00 01 fa 00 00 01 e7  00 00 01 fc 00 00 01 dd                    
00023799    00 00 01 c6 00 00 01 ae  00 00 01 c8 00 00 01 b9                    
00023800    00 00 01 90 00 00 01 93  00 00 01 8c 00 00 01 da                    
00023801    00 00 01 c2 00 00 05 d0  00 00 07 b8 00 00 06 7a                   z
00023802    00 00 09 a9 00 00 0a 2c  00 00 0a 7c 00 00 0c b3           ,   |    
00023803    00 00 09 8c 00 00 09 52  00 00 0c 04 00 00 0d c1           R        
00023804    00 00 0f 74 00 00 10 48  00 00 11 06 00 00 10 61       t   H       a
00023805    00 00 0c 63 00 00 0c 31  00 00 0b 42 00 00 0c 0d       c   1   B    
00023806    00 00 0f 32 00 00 0a 7b  00 00 0d 0f 00 00 0a e0       2   {        
00023807    00 00 0a 0e 00 00 0b 6b  00 00 08 74 00 00 0c 36           k   t   6
00023808    00 00 09 e6 00 00 06 8d  00 00 04 f8 00 00 07 8a                    
00023809    00 00 07 c1 00 00 09 f3  00 00 07 c7 00 00 0a cb                    
00023810    00 00 0a d2 00 00 0b 74  00 00 0c 28 00 00 0a 9a           t   (    
00023811    00 00 0c 60 00 00 0d 6d  00 00 0c 3e 00 00 0f fc       `   m   >    
00023812    00 00 0e 82 00 00 0b 79  00 00 0d e4 00 00 0d 24           y       $
00023813    00 00 0a 17 00 00 11 aa  00 00 12 65 00 00 0d 7b               e   {
00023814    00 00 12 a0 00 00 13 d8  00 00 11 49 00 00 0e 59               I   Y
00023815    00 00 10 15 00 00 16 81  00 00 09 b4 00 00 06 eb                    
00023816    00 00 05 ef 00 00 05 8a  00 00 03 d7 00 00 04 0d                    
00023817    00 00 03 bb 00 00 04 6b  00 00 03 40 00 00 03 30           k   @   0
00023818    00 00 02 de 00 00 03 ae  00 00 05 cf 00 00 04 6c                   l
00023819    00 00 05 69 00 00 05 00  00 00 06 a1 00 00 03 35       i           5
00023820    00 00 04 1a 00 00 03 fa  00 00 06 3d 00 00 05 d6               =    
00023821    00 00 04 68 00 00 02 d6  00 00 04 b5 00 00 02 d9       h            
00023822    00 00 02 7f 00 00 02 4d  00 00 02 7d 00 00 03 8c           M   }    
00023823    00 00 02 06 00 00 02 01  00 00 07 7f 00 00 05 ef                    
00023824    00 00 05 b8 00 00 04 0a  00 00 02 99 00 00 03 1d                    
00023825    00 00 07 c5 00 00 05 ac  00 00 04 78 00 00 08 71               x   q
00023826    00 00 08 99 00 00 08 e9  00 00 08 99 00 00 05 73                   s
00023827    00 00 07 c7 00 00 08 3d  00 00 0b 59 00 00 0a 36           =   Y   6
00023828    00 00 06 ba 00 00 05 f9  00 00 07 2e 00 00 06 eb               .    
00023829    00 00 04 c6 00 00 04 ba  00 00 05 66 00 00 04 31               f   1
00023830    00 00 06 8a 00 00 06 cf  00 00 06 fe 00 00 04 97                    
00023831    00 00 02 43 00 00 03 e2  00 00 04 06 00 00 02 e6       C            
00023832    00 00 02 6b 00 00 02 75  00 00 00 28 73 74 73 63       k   u   (stsc
00023833    00 00 00 00 00 00 00 02  00 00 00 01 00 00 00 04                    
00023834    00 00 00 01 00 00 00 2a  00 00 00 02 00 00 00 01           *        
00023835    00 00 00 b8 73 74 63 6f  00 00 00 00 00 00 00 2a        stco       *
00023836    00 00 00 a8 00 00 73 e6  00 00 8b f4 00 00 a4 08          s         
00023837    00 00 b7 64 00 00 c8 a5  00 00 d7 d8 00 00 e4 a5       d            
00023838    00 00 ec 5f 00 00 f8 5f  00 01 04 41 00 01 1f c3       _   _   A    
00023839    00 01 51 85 00 01 84 7e  00 01 cc a6 00 02 03 c0      Q    ~        
00023840    00 02 3c 52 00 02 66 75  00 02 8a 1c 00 02 b4 a6      <R  fu        
00023841    00 02 e7 66 00 03 23 01  00 03 5d ac 00 03 9e 97       f  #   ]     
00023842    00 03 ea 64 00 04 26 0a  00 04 3e 69 00 04 4c ff       d  &   >i  L 
00023843    00 04 63 d9 00 04 7e 43  00 04 98 9b 00 04 ad a9      c   ~C        
00023844    00 04 be f7 00 04 d7 94  00 04 ed 6c 00 05 0e 3b               l   ;
00023845    00 05 2d c9 00 05 59 f0  00 05 7b 82 00 05 95 2b      -   Y   {    +
00023846    00 05 b3 da 00 05 c6 67  00 00 00 14 73 74 73 73           g    stss
00023847    00 00 00 00 00 00 00 01  00 00 00 01 00 00 00 b2                    
{% endcodeblock %}

###### **vmhd**

{% codeblock lang:shell %}
00023773                                00 00 00 14 76 6d 68        ]minf    vmh
00023774    64 00 00 00 01 00 00 00  00 00 00 00 00             d
{% endcodeblock %}

###### **dinf**

{% codeblock lang:shell %}
00023774                                            00 00 00                   
00023775    24 64 69 6e 66 00 00 00  1c 64 72 65 66 00 00 00    $dinf    dref   
00023776    00 00 00 00 01 00 00 00  0c 75 72 6c 20 00 00 00             url    
00023777    01
{% endcodeblock %}

####### **dref**

{% codeblock lang:shell %}
00023775                   00 00 00  1c 64 72 65 66 00 00 00             dref   
00023776    00 00 00 00 01 00 00 00  0c 75 72 6c 20 00 00 00             url    
00023777    01
{% endcodeblock %}

###### **stbl**

{% codeblock lang:shell %}
00023777       00 00 05 1d 73 74 62  6c 00 00 00 ab 73 74 73         stbl    sts
00023778    64 00 00 00 00 00 00 00  01 00 00 00 9b 61 76 63    d            avc
00023779    31 00 00 00 00 00 00 00  01 00 00 00 00 00 00 00    1               
00023780    00 00 00 00 00 00 00 00  00 02 30 01 40 00 48 00              0 @ H 
00023781    00 00 48 00 00 00 00 00  00 00 01 0e 4a 56 54 2f      H         JVT/
00023782    41 56 43 20 43 6f 64 69  6e 67 00 00 00 00 00 00    AVC Coding      
00023783    00 00 00 00 00 00 00 00  00 00 00 00 18 ff ff 00                    
00023784    00 00 33 61 76 63 43 01  42 c0 1e ff e1 00 1b 67      3avcC B      g
00023785    42 c0 1e 9e 21 81 18 53  4d 40 40 40 50 00 00 03    B   !  SM@@@P   
00023786    00 10 00 00 03 03 c8 f1  62 ee 01 00 05 68 ce 06            b    h  
00023787    cb 20 00 00 00 12 63 6f  6c 72 6e 63 6c 63 00 01          colrnclc  
00023788    00 01 00 01 00 00 00 18  73 74 74 73 00 00 00 00            stts    
00023789    00 00 00 01 00 00 00 a6  00 00 0b b8 00 00 02 ac                    
00023790    73 74 73 7a 00 00 00 00  00 00 00 00 00 00 00 a6    stsz            
00023791    00 00 56 27 00 00 0b 20  00 00 05 bc 00 00 05 e2      V'            
00023792    00 00 05 c1 00 00 04 37  00 00 04 07 00 00 03 b6           7        
00023793    00 00 06 45 00 00 03 73  00 00 05 12 00 00 03 26       E   s       &
00023794    00 00 02 e9 00 00 03 7b  00 00 03 4a 00 00 03 6b           {   J   k
00023795    00 00 02 b6 00 00 03 4c  00 00 02 7a 00 00 02 c7           L   z    
00023796    00 00 02 2e 00 00 03 16  00 00 02 26 00 00 02 7f       .       &    
00023797    00 00 01 ec 00 00 01 ea  00 00 01 f5 00 00 01 eb                    
00023798    00 00 01 fa 00 00 01 e7  00 00 01 fc 00 00 01 dd                    
00023799    00 00 01 c6 00 00 01 ae  00 00 01 c8 00 00 01 b9                    
00023800    00 00 01 90 00 00 01 93  00 00 01 8c 00 00 01 da                    
00023801    00 00 01 c2 00 00 05 d0  00 00 07 b8 00 00 06 7a                   z
00023802    00 00 09 a9 00 00 0a 2c  00 00 0a 7c 00 00 0c b3           ,   |    
00023803    00 00 09 8c 00 00 09 52  00 00 0c 04 00 00 0d c1           R        
00023804    00 00 0f 74 00 00 10 48  00 00 11 06 00 00 10 61       t   H       a
00023805    00 00 0c 63 00 00 0c 31  00 00 0b 42 00 00 0c 0d       c   1   B    
00023806    00 00 0f 32 00 00 0a 7b  00 00 0d 0f 00 00 0a e0       2   {        
00023807    00 00 0a 0e 00 00 0b 6b  00 00 08 74 00 00 0c 36           k   t   6
00023808    00 00 09 e6 00 00 06 8d  00 00 04 f8 00 00 07 8a                    
00023809    00 00 07 c1 00 00 09 f3  00 00 07 c7 00 00 0a cb                    
00023810    00 00 0a d2 00 00 0b 74  00 00 0c 28 00 00 0a 9a           t   (    
00023811    00 00 0c 60 00 00 0d 6d  00 00 0c 3e 00 00 0f fc       `   m   >    
00023812    00 00 0e 82 00 00 0b 79  00 00 0d e4 00 00 0d 24           y       $
00023813    00 00 0a 17 00 00 11 aa  00 00 12 65 00 00 0d 7b               e   {
00023814    00 00 12 a0 00 00 13 d8  00 00 11 49 00 00 0e 59               I   Y
00023815    00 00 10 15 00 00 16 81  00 00 09 b4 00 00 06 eb                    
00023816    00 00 05 ef 00 00 05 8a  00 00 03 d7 00 00 04 0d                    
00023817    00 00 03 bb 00 00 04 6b  00 00 03 40 00 00 03 30           k   @   0
00023818    00 00 02 de 00 00 03 ae  00 00 05 cf 00 00 04 6c                   l
00023819    00 00 05 69 00 00 05 00  00 00 06 a1 00 00 03 35       i           5
00023820    00 00 04 1a 00 00 03 fa  00 00 06 3d 00 00 05 d6               =    
00023821    00 00 04 68 00 00 02 d6  00 00 04 b5 00 00 02 d9       h            
00023822    00 00 02 7f 00 00 02 4d  00 00 02 7d 00 00 03 8c           M   }    
00023823    00 00 02 06 00 00 02 01  00 00 07 7f 00 00 05 ef                    
00023824    00 00 05 b8 00 00 04 0a  00 00 02 99 00 00 03 1d                    
00023825    00 00 07 c5 00 00 05 ac  00 00 04 78 00 00 08 71               x   q
00023826    00 00 08 99 00 00 08 e9  00 00 08 99 00 00 05 73                   s
00023827    00 00 07 c7 00 00 08 3d  00 00 0b 59 00 00 0a 36           =   Y   6
00023828    00 00 06 ba 00 00 05 f9  00 00 07 2e 00 00 06 eb               .    
00023829    00 00 04 c6 00 00 04 ba  00 00 05 66 00 00 04 31               f   1
00023830    00 00 06 8a 00 00 06 cf  00 00 06 fe 00 00 04 97                    
00023831    00 00 02 43 00 00 03 e2  00 00 04 06 00 00 02 e6       C            
00023832    00 00 02 6b 00 00 02 75  00 00 00 28 73 74 73 63       k   u   (stsc
00023833    00 00 00 00 00 00 00 02  00 00 00 01 00 00 00 04                    
00023834    00 00 00 01 00 00 00 2a  00 00 00 02 00 00 00 01           *        
00023835    00 00 00 b8 73 74 63 6f  00 00 00 00 00 00 00 2a        stco       *
00023836    00 00 00 a8 00 00 73 e6  00 00 8b f4 00 00 a4 08          s         
00023837    00 00 b7 64 00 00 c8 a5  00 00 d7 d8 00 00 e4 a5       d            
00023838    00 00 ec 5f 00 00 f8 5f  00 01 04 41 00 01 1f c3       _   _   A    
00023839    00 01 51 85 00 01 84 7e  00 01 cc a6 00 02 03 c0      Q    ~        
00023840    00 02 3c 52 00 02 66 75  00 02 8a 1c 00 02 b4 a6      <R  fu        
00023841    00 02 e7 66 00 03 23 01  00 03 5d ac 00 03 9e 97       f  #   ]     
00023842    00 03 ea 64 00 04 26 0a  00 04 3e 69 00 04 4c ff       d  &   >i  L 
00023843    00 04 63 d9 00 04 7e 43  00 04 98 9b 00 04 ad a9      c   ~C        
00023844    00 04 be f7 00 04 d7 94  00 04 ed 6c 00 05 0e 3b               l   ;
00023845    00 05 2d c9 00 05 59 f0  00 05 7b 82 00 05 95 2b      -   Y   {    +
00023846    00 05 b3 da 00 05 c6 67  00 00 00 14 73 74 73 73           g    stss
00023847    00 00 00 00 00 00 00 01  00 00 00 01 00 00 00 b2                    
00023848    73 64 74 70 00 00 00 00  04 44 44 44 44 44 44 44    sdtp     DDDDDDD
00023849    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023850    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023851    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023852    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023853    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023854    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023855    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023856    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023857    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023858    44 44 44 44 44 44 44 44  44 44 44 44 44 44 00 00    DDDDDDDDDDDDDD  
00023859    06 7e 74 72 61 6b 00 00  00 5c 74 6b 68 64 00 00     ~trak   \tkhd  
00023860    00 03 c7 ca ee a7 c7 ca  ee a8 00 00 00 02 00 00                    
00023861    00 00 00 07 a5 80 00 00  00 00 00 00 00 00 00 00                    
00023862    00 00 01 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023863    00 00 00 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023864    00 00 00 00 00 00 40 00  00 00 00 00 00 00 00 00          @         
00023865    00 00 00 00 06 04 6d 64  69 61 00 00 00 20 6d 64          mdia    md
00023866    68 64 00 00 00 00 c7 ca  ee a7 c7 ca ee a8 00 00    hd              
00023867    bb 80 00 04 14 00 15 c7  00 00 00 00 00 21 68 64                 !hd
00023868    6c 72 00 00 00 00 00 00  00 00 73 6f 75 6e 00 00    lr        soun  
00023869    00 00 00 00 00 00 00 00  00 00 00 00 00 05 bb 6d                   m
00023870    69 6e 66 00 00 00 10 73  6d 68 64 00 00 00 00 00    inf    smhd     
00023871    00 00 00 00 00 00 24 64  69 6e 66 00 00 00 1c 64          $dinf    d
00023872    72 65 66 00 00 00 00 00  00 00 01 00 00 00 0c 75    ref            u
00023873    72 6c 20 00 00 00 01 00  00 05 7f 73 74 62 6c 00    rl         stbl 
00023874    00 00 67 73 74 73 64 00  00 00 00 00 00 00 01 00      gstsd         
00023875    00 00 57 6d 70 34 61 00  00 00 00 00 00 00 01 00      Wmp4a         
00023876    00 00 00 00 00 00 00 00  01 00 10 00 00 00 00 bb                    
00023877    80 00 00 00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02 00 00  00 18 73 74 74 73 00 00              stts  
00023881    00 00 00 00 00 01 00 00  01 05 00 00 04 00 00 00                    
00023882    04 28 73 74 73 7a 00 00  00 00 00 00 00 00 00 00     (stsz          
00023883    01 05 00 00 00 f7 00 00  00 db 00 00 00 e1 00 00                    
00023884    00 e5 00 00 00 e9 00 00  00 e8 00 00 00 f0 00 00                    
00023885    00 f1 00 00 00 ef 00 00  00 d8 00 00 00 e6 00 00                    
00023886    00 e7 00 00 00 e9 00 00  00 eb 00 00 00 ea 00 00                    
00023887    00 e8 00 00 00 e1 00 00  00 e7 00 00 00 d7 00 00                    
00023888    00 da 00 00 00 d9 00 00  00 db 00 00 00 e9 00 00                    
00023889    00 ee 00 00 00 e5 00 00  00 e1 00 00 00 e6 00 00                    
00023890    00 e5 00 00 00 d8 00 00  00 dd 00 00 00 dd 00 00                    
00023891    00 d5 00 00 00 ea 00 00  00 dd 00 00 00 d0 00 00                    
00023892    00 d6 00 00 00 e9 00 00  00 bc 00 00 00 ab 00 00                    
00023893    00 b3 00 00 00 b5 00 00  00 bc 00 00 00 ce 00 00                    
00023894    00 b4 00 00 00 b6 00 00  00 b3 00 00 00 b6 00 00                    
00023895    00 b7 00 00 00 bf 00 00  00 b7 00 00 00 cd 00 00                    
00023896    00 c1 00 00 00 ba 00 00  00 a7 00 00 00 b4 00 00                    
00023897    00 b1 00 00 00 be 00 00  00 d0 00 00 00 ba 00 00                    
00023898    00 bc 00 00 00 c4 00 00  00 c6 00 00 00 cb 00 00                    
00023899    00 c4 00 00 00 c3 00 00  00 c8 00 00 00 d2 00 00                    
00023900    00 d2 00 00 00 d6 00 00  00 f5 00 00 00 fa 00 00                    
00023901    00 f6 00 00 01 02 00 00  00 fc 00 00 00 fc 00 00                    
00023902    00 ee 00 00 00 e6 00 00  00 ea 00 00 00 ea 00 00                    
00023903    00 e8 00 00 00 de 00 00  00 df 00 00 00 e7 00 00                    
00023904    00 f6 00 00 00 ff 00 00  01 03 00 00 00 f6 00 00                    
00023905    01 08 00 00 01 03 00 00  00 fd 00 00 01 05 00 00                    
00023906    01 02 00 00 01 00 00 00  01 14 00 00 01 18 00 00                    
00023907    00 fd 00 00 00 fb 00 00  01 11 00 00 01 05 00 00                    
00023908    01 05 00 00 01 0a 00 00  01 01 00 00 00 f3 00 00                    
00023909    00 f7 00 00 00 f7 00 00  01 01 00 00 01 02 00 00                    
00023910    00 f8 00 00 00 f8 00 00  00 ef 00 00 00 ed 00 00                    
00023911    00 e3 00 00 00 ec 00 00  00 e2 00 00 00 e8 00 00                    
00023912    00 dc 00 00 00 e0 00 00  00 f3 00 00 00 df 00 00                    
00023913    00 e1 00 00 00 cf 00 00  00 ce 00 00 00 d8 00 00                    
00023914    00 ce 00 00 00 c7 00 00  00 cd 00 00 00 b7 00 00                    
00023915    00 af 00 00 00 c8 00 00  00 d7 00 00 00 e5 00 00                    
00023916    00 e4 00 00 00 c6 00 00  00 d1 00 00 00 d5 00 00                    
00023917    00 e5 00 00 00 d8 00 00  00 c8 00 00 00 be 00 00                    
00023918    00 bf 00 00 00 cb 00 00  00 d2 00 00 00 c8 00 00                    
00023919    00 ca 00 00 00 b1 00 00  00 a3 00 00 00 c7 00 00                    
00023920    00 dc 00 00 00 d9 00 00  00 dd 00 00 00 d1 00 00                    
00023921    00 d2 00 00 00 c2 00 00  00 bc 00 00 00 b1 00 00                    
00023922    00 9b 00 00 00 89 00 00  00 a2 00 00 00 9f 00 00                    
00023923    00 b5 00 00 00 a6 00 00  00 b2 00 00 00 b5 00 00                    
00023924    00 ae 00 00 00 b4 00 00  00 b0 00 00 00 c6 00 00                    
00023925    00 c3 00 00 00 d5 00 00  00 e4 00 00 00 f6 00 00                    
00023926    00 d6 00 00 00 db 00 00  00 cc 00 00 00 e7 00 00                    
00023927    00 f9 00 00 00 cb 00 00  00 d8 00 00 00 d6 00 00                    
00023928    00 e4 00 00 00 f1 00 00  00 e4 00 00 00 e6 00 00                    
00023929    00 df 00 00 00 ee 00 00  00 d7 00 00 00 c7 00 00                    
00023930    00 e7 00 00 00 f9 00 00  00 ed 00 00 00 cf 00 00                    
00023931    00 f1 00 00 00 e6 00 00  00 dc 00 00 00 e4 00 00                    
00023932    00 ef 00 00 00 e5 00 00  00 f1 00 00 00 e3 00 00                    
00023933    00 ec 00 00 00 ec 00 00  00 f3 00 00 00 f5 00 00                    
00023934    00 fd 00 00 01 0b 00 00  01 10 00 00 01 11 00 00                    
00023935    01 03 00 00 01 01 00 00  00 fb 00 00 00 fa 00 00                    
00023936    00 e7 00 00 00 e5 00 00  00 f0 00 00 00 d2 00 00                    
00023937    00 e5 00 00 00 f3 00 00  00 f1 00 00 00 f2 00 00                    
00023938    00 ff 00 00 00 f7 00 00  00 ee 00 00 00 d5 00 00                    
00023939    00 d9 00 00 00 ea 00 00  00 e3 00 00 00 df 00 00                    
00023940    00 f7 00 00 00 ff 00 00  00 f8 00 00 00 fa 00 00                    
00023941    00 fd 00 00 00 f7 00 00  00 f9 00 00 00 fb 00 00                    
00023942    00 f8 00 00 00 f6 00 00  00 f0 00 00 00 fe 00 00                    
00023943    01 02 00 00 00 e9 00 00  00 ec 00 00 00 ec 00 00                    
00023944    00 e7 00 00 00 ea 00 00  00 de 00 00 00 e2 00 00                    
00023945    00 c9 00 00 00 d4 00 00  00 d4 00 00 00 c7 00 00                    
00023946    00 c9 00 00 00 c8 00 00  00 c1 00 00 00 c0 00 00                    
00023947    00 bd 00 00 00 de 00 00  00 cb 00 00 00 cd 00 00                    
00023948    00 d4 00 00 00 6d 00 00  00 28 73 74 73 63 00 00         m   (stsc  
00023949    00 00 00 00 00 02 00 00  00 01 00 00 00 07 00 00                    
00023950    00 01 00 00 00 26 00 00  00 02 00 00 00 01 00 00         &          
00023951    00 a8 73 74 63 6f 00 00  00 00 00 00 00 26 00 00      stco       &  
00023952    6d 8d 00 00 85 9b 00 00  9d e4 00 00 b1 21 00 00    m            !  
00023953    c2 a7 00 00 d2 8e 00 00  df 8e 00 00 f3 54 00 00                 T  
00023954    fe e8 00 01 1a 05 00 01  4a c7 00 01 7e 28 00 01            J   ~(  
00023955    c5 a1 00 01 fc 89 00 02  35 5c 00 02 83 6a 00 02            5\   j  
00023956    ae 62 00 02 e1 ae 00 03  1d 6d 00 03 58 04 00 03     b       m  X   
00023957    99 4d 00 03 e4 b1 00 04  21 99 00 04 39 67 00 04     M      !   9g  
00023958    5d c6 00 04 78 18 00 04  92 6a 00 04 a7 67 00 04    ]   x    j   g  
00023959    b8 7e 00 04 d0 6c 00 04  e7 0c 00 05 07 c6 00 05     ~   l          
00023960    53 5c 00 05 74 bc 00 05  8e 99 00 05 ae 19 00 05    S\  t           
00023961    c0 eb 00 05 cb 47 00 00  00 16 75 64 74 61 00 00         G    udta  
00023962    00 0e 6e 61 6d 65 53 74  65 72 65 6f 00 00 00 6f      nameStereo   o
00023963    75 64 74 61 00 00 00 67  6d 65 74 61 00 00 00 00    udta   gmeta    
00023964    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023965    6d 64 69 72 00 00 00 00  00 00 00 00 00 00 00 00    mdir            
00023966    00 00 00 00 3a 69 6c 73  74 00 00 00 32 a9 74 6f        :ilst   2 to
00023967    6f 00 00 00 2a 64 61 74  61 00 00 00 01 00 00 00    o   *data       
00023968    00 48 61 6e 64 42 72 61  6b 65 20 30 2e 39 2e 34     HandBrake 0.9.4
00023969    20 32 30 30 39 31 31 32  33 30 30                    2009112300
{% endcodeblock %}

####### **stsd**

{% codeblock lang:shell %}
00023777                                00 00 00 ab 73 74 73                 sts
00023778    64 00 00 00 00 00 00 00  01 00 00 00 9b 61 76 63    d            avc
00023779    31 00 00 00 00 00 00 00  01 00 00 00 00 00 00 00    1               
00023780    00 00 00 00 00 00 00 00  00 02 30 01 40 00 48 00              0 @ H 
00023781    00 00 48 00 00 00 00 00  00 00 01 0e 4a 56 54 2f      H         JVT/
00023782    41 56 43 20 43 6f 64 69  6e 67 00 00 00 00 00 00    AVC Coding      
00023783    00 00 00 00 00 00 00 00  00 00 00 00 18 ff ff 00                    
00023784    00 00 33 61 76 63 43 01  42 c0 1e ff e1 00 1b 67      3avcC B      g
00023785    42 c0 1e 9e 21 81 18 53  4d 40 40 40 50 00 00 03    B   !  SM@@@P   
00023786    00 10 00 00 03 03 c8 f1  62 ee 01 00 05 68 ce 06            b    h  
00023787    cb 20 00 00 00 12 63 6f  6c 72 6e 63 6c 63 00 01          colrnclc  
00023788    00 01 00 01
{% endcodeblock %}

####### **stts**

{% codeblock lang:shell %}
00023788                00 00 00 18  73 74 74 73 00 00 00 00            stts    
00023789    00 00 00 01 00 00 00 a6  00 00 0b b8 
{% endcodeblock %}

####### **stsz**

{% codeblock lang:shell %}
00023789                                         00 00 02 ac                    
00023790    73 74 73 7a 00 00 00 00  00 00 00 00 00 00 00 a6    stsz            
00023791    00 00 56 27 00 00 0b 20  00 00 05 bc 00 00 05 e2      V'            
00023792    00 00 05 c1 00 00 04 37  00 00 04 07 00 00 03 b6           7        
00023793    00 00 06 45 00 00 03 73  00 00 05 12 00 00 03 26       E   s       &
00023794    00 00 02 e9 00 00 03 7b  00 00 03 4a 00 00 03 6b           {   J   k
00023795    00 00 02 b6 00 00 03 4c  00 00 02 7a 00 00 02 c7           L   z    
00023796    00 00 02 2e 00 00 03 16  00 00 02 26 00 00 02 7f       .       &    
00023797    00 00 01 ec 00 00 01 ea  00 00 01 f5 00 00 01 eb                    
00023798    00 00 01 fa 00 00 01 e7  00 00 01 fc 00 00 01 dd                    
00023799    00 00 01 c6 00 00 01 ae  00 00 01 c8 00 00 01 b9                    
00023800    00 00 01 90 00 00 01 93  00 00 01 8c 00 00 01 da                    
00023801    00 00 01 c2 00 00 05 d0  00 00 07 b8 00 00 06 7a                   z
00023802    00 00 09 a9 00 00 0a 2c  00 00 0a 7c 00 00 0c b3           ,   |    
00023803    00 00 09 8c 00 00 09 52  00 00 0c 04 00 00 0d c1           R        
00023804    00 00 0f 74 00 00 10 48  00 00 11 06 00 00 10 61       t   H       a
00023805    00 00 0c 63 00 00 0c 31  00 00 0b 42 00 00 0c 0d       c   1   B    
00023806    00 00 0f 32 00 00 0a 7b  00 00 0d 0f 00 00 0a e0       2   {        
00023807    00 00 0a 0e 00 00 0b 6b  00 00 08 74 00 00 0c 36           k   t   6
00023808    00 00 09 e6 00 00 06 8d  00 00 04 f8 00 00 07 8a                    
00023809    00 00 07 c1 00 00 09 f3  00 00 07 c7 00 00 0a cb                    
00023810    00 00 0a d2 00 00 0b 74  00 00 0c 28 00 00 0a 9a           t   (    
00023811    00 00 0c 60 00 00 0d 6d  00 00 0c 3e 00 00 0f fc       `   m   >    
00023812    00 00 0e 82 00 00 0b 79  00 00 0d e4 00 00 0d 24           y       $
00023813    00 00 0a 17 00 00 11 aa  00 00 12 65 00 00 0d 7b               e   {
00023814    00 00 12 a0 00 00 13 d8  00 00 11 49 00 00 0e 59               I   Y
00023815    00 00 10 15 00 00 16 81  00 00 09 b4 00 00 06 eb                    
00023816    00 00 05 ef 00 00 05 8a  00 00 03 d7 00 00 04 0d                    
00023817    00 00 03 bb 00 00 04 6b  00 00 03 40 00 00 03 30           k   @   0
00023818    00 00 02 de 00 00 03 ae  00 00 05 cf 00 00 04 6c                   l
00023819    00 00 05 69 00 00 05 00  00 00 06 a1 00 00 03 35       i           5
00023820    00 00 04 1a 00 00 03 fa  00 00 06 3d 00 00 05 d6               =    
00023821    00 00 04 68 00 00 02 d6  00 00 04 b5 00 00 02 d9       h            
00023822    00 00 02 7f 00 00 02 4d  00 00 02 7d 00 00 03 8c           M   }    
00023823    00 00 02 06 00 00 02 01  00 00 07 7f 00 00 05 ef                    
00023824    00 00 05 b8 00 00 04 0a  00 00 02 99 00 00 03 1d                    
00023825    00 00 07 c5 00 00 05 ac  00 00 04 78 00 00 08 71               x   q
00023826    00 00 08 99 00 00 08 e9  00 00 08 99 00 00 05 73                   s
00023827    00 00 07 c7 00 00 08 3d  00 00 0b 59 00 00 0a 36           =   Y   6
00023828    00 00 06 ba 00 00 05 f9  00 00 07 2e 00 00 06 eb               .    
00023829    00 00 04 c6 00 00 04 ba  00 00 05 66 00 00 04 31               f   1
00023830    00 00 06 8a 00 00 06 cf  00 00 06 fe 00 00 04 97                    
00023831    00 00 02 43 00 00 03 e2  00 00 04 06 00 00 02 e6       C            
00023832    00 00 02 6b 00 00 02 75                                k   u
{% endcodeblock %}

####### **stsc**

{% codeblock lang:shell %}
00023832                             00 00 00 28 73 74 73 63               (stsc
00023833    00 00 00 00 00 00 00 02  00 00 00 01 00 00 00 04                    
00023834    00 00 00 01 00 00 00 2a  00 00 00 02 00 00 00 01           *        
{% endcodeblock %}

####### **stco**

{% codeblock lang:shell %}
00023835    00 00 00 b8 73 74 63 6f  00 00 00 00 00 00 00 2a        stco       *
00023836    00 00 00 a8 00 00 73 e6  00 00 8b f4 00 00 a4 08          s         
00023837    00 00 b7 64 00 00 c8 a5  00 00 d7 d8 00 00 e4 a5       d            
00023838    00 00 ec 5f 00 00 f8 5f  00 01 04 41 00 01 1f c3       _   _   A    
00023839    00 01 51 85 00 01 84 7e  00 01 cc a6 00 02 03 c0      Q    ~        
00023840    00 02 3c 52 00 02 66 75  00 02 8a 1c 00 02 b4 a6      <R  fu        
00023841    00 02 e7 66 00 03 23 01  00 03 5d ac 00 03 9e 97       f  #   ]     
00023842    00 03 ea 64 00 04 26 0a  00 04 3e 69 00 04 4c ff       d  &   >i  L 
00023843    00 04 63 d9 00 04 7e 43  00 04 98 9b 00 04 ad a9      c   ~C        
00023844    00 04 be f7 00 04 d7 94  00 04 ed 6c 00 05 0e 3b               l   ;
00023845    00 05 2d c9 00 05 59 f0  00 05 7b 82 00 05 95 2b      -   Y   {    +
00023846    00 05 b3 da 00 05 c6 67                                    g
{% endcodeblock %}

####### **stss**

{% codeblock lang:shell %}
00023846                             00 00 00 14 73 74 73 73                stss
00023847    00 00 00 00 00 00 00 01  00 00 00 01
{% endcodeblock %}

####### **sdtp**

{% codeblock lang:shell %}
00023847                                         00 00 00 b2                    
00023848    73 64 74 70 00 00 00 00  04 44 44 44 44 44 44 44    sdtp     DDDDDDD
00023849    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023850    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023851    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023852    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023853    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023854    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023855    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023856    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023857    44 44 44 44 44 44 44 44  44 44 44 44 44 44 44 44    DDDDDDDDDDDDDDDD
00023858    44 44 44 44 44 44 44 44  44 44 44 44 44 44          DDDDDDDDDDDDDD
{% endcodeblock %}

### **trak**

{% codeblock lang:shell %}
00023858                                               00 00    
00023859    06 7e 74 72 61 6b 00 00  00 5c 74 6b 68 64 00 00     ~trak   \tkhd  
00023860    00 03 c7 ca ee a7 c7 ca  ee a8 00 00 00 02 00 00                    
00023861    00 00 00 07 a5 80 00 00  00 00 00 00 00 00 00 00                    
00023862    00 00 01 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023863    00 00 00 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023864    00 00 00 00 00 00 40 00  00 00 00 00 00 00 00 00          @         
00023865    00 00 00 00 06 04 6d 64  69 61 00 00 00 20 6d 64          mdia    md
00023866    68 64 00 00 00 00 c7 ca  ee a7 c7 ca ee a8 00 00    hd              
00023867    bb 80 00 04 14 00 15 c7  00 00 00 00 00 21 68 64                 !hd
00023868    6c 72 00 00 00 00 00 00  00 00 73 6f 75 6e 00 00    lr        soun  
00023869    00 00 00 00 00 00 00 00  00 00 00 00 00 05 bb 6d                   m
00023870    69 6e 66 00 00 00 10 73  6d 68 64 00 00 00 00 00    inf    smhd     
00023871    00 00 00 00 00 00 24 64  69 6e 66 00 00 00 1c 64          $dinf    d
00023872    72 65 66 00 00 00 00 00  00 00 01 00 00 00 0c 75    ref            u
00023873    72 6c 20 00 00 00 01 00  00 05 7f 73 74 62 6c 00    rl         stbl 
00023874    00 00 67 73 74 73 64 00  00 00 00 00 00 00 01 00      gstsd         
00023875    00 00 57 6d 70 34 61 00  00 00 00 00 00 00 01 00      Wmp4a         
00023876    00 00 00 00 00 00 00 00  01 00 10 00 00 00 00 bb                    
00023877    80 00 00 00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02 00 00  00 18 73 74 74 73 00 00              stts  
00023881    00 00 00 00 00 01 00 00  01 05 00 00 04 00 00 00                    
00023882    04 28 73 74 73 7a 00 00  00 00 00 00 00 00 00 00     (stsz          
00023883    01 05 00 00 00 f7 00 00  00 db 00 00 00 e1 00 00                    
00023884    00 e5 00 00 00 e9 00 00  00 e8 00 00 00 f0 00 00                    
00023885    00 f1 00 00 00 ef 00 00  00 d8 00 00 00 e6 00 00                    
00023886    00 e7 00 00 00 e9 00 00  00 eb 00 00 00 ea 00 00                    
00023887    00 e8 00 00 00 e1 00 00  00 e7 00 00 00 d7 00 00                    
00023888    00 da 00 00 00 d9 00 00  00 db 00 00 00 e9 00 00                    
00023889    00 ee 00 00 00 e5 00 00  00 e1 00 00 00 e6 00 00                    
00023890    00 e5 00 00 00 d8 00 00  00 dd 00 00 00 dd 00 00                    
00023891    00 d5 00 00 00 ea 00 00  00 dd 00 00 00 d0 00 00                    
00023892    00 d6 00 00 00 e9 00 00  00 bc 00 00 00 ab 00 00                    
00023893    00 b3 00 00 00 b5 00 00  00 bc 00 00 00 ce 00 00                    
00023894    00 b4 00 00 00 b6 00 00  00 b3 00 00 00 b6 00 00                    
00023895    00 b7 00 00 00 bf 00 00  00 b7 00 00 00 cd 00 00                    
00023896    00 c1 00 00 00 ba 00 00  00 a7 00 00 00 b4 00 00                    
00023897    00 b1 00 00 00 be 00 00  00 d0 00 00 00 ba 00 00                    
00023898    00 bc 00 00 00 c4 00 00  00 c6 00 00 00 cb 00 00                    
00023899    00 c4 00 00 00 c3 00 00  00 c8 00 00 00 d2 00 00                    
00023900    00 d2 00 00 00 d6 00 00  00 f5 00 00 00 fa 00 00                    
00023901    00 f6 00 00 01 02 00 00  00 fc 00 00 00 fc 00 00                    
00023902    00 ee 00 00 00 e6 00 00  00 ea 00 00 00 ea 00 00                    
00023903    00 e8 00 00 00 de 00 00  00 df 00 00 00 e7 00 00                    
00023904    00 f6 00 00 00 ff 00 00  01 03 00 00 00 f6 00 00                    
00023905    01 08 00 00 01 03 00 00  00 fd 00 00 01 05 00 00                    
00023906    01 02 00 00 01 00 00 00  01 14 00 00 01 18 00 00                    
00023907    00 fd 00 00 00 fb 00 00  01 11 00 00 01 05 00 00                    
00023908    01 05 00 00 01 0a 00 00  01 01 00 00 00 f3 00 00                    
00023909    00 f7 00 00 00 f7 00 00  01 01 00 00 01 02 00 00                    
00023910    00 f8 00 00 00 f8 00 00  00 ef 00 00 00 ed 00 00                    
00023911    00 e3 00 00 00 ec 00 00  00 e2 00 00 00 e8 00 00                    
00023912    00 dc 00 00 00 e0 00 00  00 f3 00 00 00 df 00 00                    
00023913    00 e1 00 00 00 cf 00 00  00 ce 00 00 00 d8 00 00                    
00023914    00 ce 00 00 00 c7 00 00  00 cd 00 00 00 b7 00 00                    
00023915    00 af 00 00 00 c8 00 00  00 d7 00 00 00 e5 00 00                    
00023916    00 e4 00 00 00 c6 00 00  00 d1 00 00 00 d5 00 00                    
00023917    00 e5 00 00 00 d8 00 00  00 c8 00 00 00 be 00 00                    
00023918    00 bf 00 00 00 cb 00 00  00 d2 00 00 00 c8 00 00                    
00023919    00 ca 00 00 00 b1 00 00  00 a3 00 00 00 c7 00 00                    
00023920    00 dc 00 00 00 d9 00 00  00 dd 00 00 00 d1 00 00                    
00023921    00 d2 00 00 00 c2 00 00  00 bc 00 00 00 b1 00 00                    
00023922    00 9b 00 00 00 89 00 00  00 a2 00 00 00 9f 00 00                    
00023923    00 b5 00 00 00 a6 00 00  00 b2 00 00 00 b5 00 00                    
00023924    00 ae 00 00 00 b4 00 00  00 b0 00 00 00 c6 00 00                    
00023925    00 c3 00 00 00 d5 00 00  00 e4 00 00 00 f6 00 00                    
00023926    00 d6 00 00 00 db 00 00  00 cc 00 00 00 e7 00 00                    
00023927    00 f9 00 00 00 cb 00 00  00 d8 00 00 00 d6 00 00                    
00023928    00 e4 00 00 00 f1 00 00  00 e4 00 00 00 e6 00 00                    
00023929    00 df 00 00 00 ee 00 00  00 d7 00 00 00 c7 00 00                    
00023930    00 e7 00 00 00 f9 00 00  00 ed 00 00 00 cf 00 00                    
00023931    00 f1 00 00 00 e6 00 00  00 dc 00 00 00 e4 00 00                    
00023932    00 ef 00 00 00 e5 00 00  00 f1 00 00 00 e3 00 00                    
00023933    00 ec 00 00 00 ec 00 00  00 f3 00 00 00 f5 00 00                    
00023934    00 fd 00 00 01 0b 00 00  01 10 00 00 01 11 00 00                    
00023935    01 03 00 00 01 01 00 00  00 fb 00 00 00 fa 00 00                    
00023936    00 e7 00 00 00 e5 00 00  00 f0 00 00 00 d2 00 00                    
00023937    00 e5 00 00 00 f3 00 00  00 f1 00 00 00 f2 00 00                    
00023938    00 ff 00 00 00 f7 00 00  00 ee 00 00 00 d5 00 00                    
00023939    00 d9 00 00 00 ea 00 00  00 e3 00 00 00 df 00 00                    
00023940    00 f7 00 00 00 ff 00 00  00 f8 00 00 00 fa 00 00                    
00023941    00 fd 00 00 00 f7 00 00  00 f9 00 00 00 fb 00 00                    
00023942    00 f8 00 00 00 f6 00 00  00 f0 00 00 00 fe 00 00                    
00023943    01 02 00 00 00 e9 00 00  00 ec 00 00 00 ec 00 00                    
00023944    00 e7 00 00 00 ea 00 00  00 de 00 00 00 e2 00 00                    
00023945    00 c9 00 00 00 d4 00 00  00 d4 00 00 00 c7 00 00                    
00023946    00 c9 00 00 00 c8 00 00  00 c1 00 00 00 c0 00 00                    
00023947    00 bd 00 00 00 de 00 00  00 cb 00 00 00 cd 00 00                    
00023948    00 d4 00 00 00 6d 00 00  00 28 73 74 73 63 00 00         m   (stsc  
00023949    00 00 00 00 00 02 00 00  00 01 00 00 00 07 00 00                    
00023950    00 01 00 00 00 26 00 00  00 02 00 00 00 01 00 00         &          
00023951    00 a8 73 74 63 6f 00 00  00 00 00 00 00 26 00 00      stco       &  
00023952    6d 8d 00 00 85 9b 00 00  9d e4 00 00 b1 21 00 00    m            !  
00023953    c2 a7 00 00 d2 8e 00 00  df 8e 00 00 f3 54 00 00                 T  
00023954    fe e8 00 01 1a 05 00 01  4a c7 00 01 7e 28 00 01            J   ~(  
00023955    c5 a1 00 01 fc 89 00 02  35 5c 00 02 83 6a 00 02            5\   j  
00023956    ae 62 00 02 e1 ae 00 03  1d 6d 00 03 58 04 00 03     b       m  X   
00023957    99 4d 00 03 e4 b1 00 04  21 99 00 04 39 67 00 04     M      !   9g  
00023958    5d c6 00 04 78 18 00 04  92 6a 00 04 a7 67 00 04    ]   x    j   g  
00023959    b8 7e 00 04 d0 6c 00 04  e7 0c 00 05 07 c6 00 05     ~   l          
00023960    53 5c 00 05 74 bc 00 05  8e 99 00 05 ae 19 00 05    S\  t           
00023961    c0 eb 00 05 cb 47                                        G
{% endcodeblock %}

#### **tkhd**

{% codeblock lang:shell %}
00023859                      00 00  00 5c 74 6b 68 64 00 00             \tkhd  
00023860    00 03 c7 ca ee a7 c7 ca  ee a8 00 00 00 02 00 00                    
00023861    00 00 00 07 a5 80 00 00  00 00 00 00 00 00 00 00                    
00023862    00 00 01 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023863    00 00 00 00 00 00 00 01  00 00 00 00 00 00 00 00                    
00023864    00 00 00 00 00 00 40 00  00 00 00 00 00 00 00 00          @         
00023865    00 00
{% endcodeblock %}

#### **mdia**

{% codeblock lang:shell %}
00023865          00 00 06 04 6d 64  69 61 00 00 00 20 6d 64          mdia    md
00023866    68 64 00 00 00 00 c7 ca  ee a7 c7 ca ee a8 00 00    hd              
00023867    bb 80 00 04 14 00 15 c7  00 00 00 00 00 21 68 64                 !hd
00023868    6c 72 00 00 00 00 00 00  00 00 73 6f 75 6e 00 00    lr        soun  
00023869    00 00 00 00 00 00 00 00  00 00 00 00 00 05 bb 6d                   m
00023870    69 6e 66 00 00 00 10 73  6d 68 64 00 00 00 00 00    inf    smhd     
00023871    00 00 00 00 00 00 24 64  69 6e 66 00 00 00 1c 64          $dinf    d
00023872    72 65 66 00 00 00 00 00  00 00 01 00 00 00 0c 75    ref            u
00023873    72 6c 20 00 00 00 01 00  00 05 7f 73 74 62 6c 00    rl         stbl 
00023874    00 00 67 73 74 73 64 00  00 00 00 00 00 00 01 00      gstsd         
00023875    00 00 57 6d 70 34 61 00  00 00 00 00 00 00 01 00      Wmp4a         
00023876    00 00 00 00 00 00 00 00  01 00 10 00 00 00 00 bb                    
00023877    80 00 00 00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02 00 00  00 18 73 74 74 73 00 00              stts  
00023881    00 00 00 00 00 01 00 00  01 05 00 00 04 00 00 00                    
00023882    04 28 73 74 73 7a 00 00  00 00 00 00 00 00 00 00     (stsz          
00023883    01 05 00 00 00 f7 00 00  00 db 00 00 00 e1 00 00                    
00023884    00 e5 00 00 00 e9 00 00  00 e8 00 00 00 f0 00 00                    
00023885    00 f1 00 00 00 ef 00 00  00 d8 00 00 00 e6 00 00                    
00023886    00 e7 00 00 00 e9 00 00  00 eb 00 00 00 ea 00 00                    
00023887    00 e8 00 00 00 e1 00 00  00 e7 00 00 00 d7 00 00                    
00023888    00 da 00 00 00 d9 00 00  00 db 00 00 00 e9 00 00                    
00023889    00 ee 00 00 00 e5 00 00  00 e1 00 00 00 e6 00 00                    
00023890    00 e5 00 00 00 d8 00 00  00 dd 00 00 00 dd 00 00                    
00023891    00 d5 00 00 00 ea 00 00  00 dd 00 00 00 d0 00 00                    
00023892    00 d6 00 00 00 e9 00 00  00 bc 00 00 00 ab 00 00                    
00023893    00 b3 00 00 00 b5 00 00  00 bc 00 00 00 ce 00 00                    
00023894    00 b4 00 00 00 b6 00 00  00 b3 00 00 00 b6 00 00                    
00023895    00 b7 00 00 00 bf 00 00  00 b7 00 00 00 cd 00 00                    
00023896    00 c1 00 00 00 ba 00 00  00 a7 00 00 00 b4 00 00                    
00023897    00 b1 00 00 00 be 00 00  00 d0 00 00 00 ba 00 00                    
00023898    00 bc 00 00 00 c4 00 00  00 c6 00 00 00 cb 00 00                    
00023899    00 c4 00 00 00 c3 00 00  00 c8 00 00 00 d2 00 00                    
00023900    00 d2 00 00 00 d6 00 00  00 f5 00 00 00 fa 00 00                    
00023901    00 f6 00 00 01 02 00 00  00 fc 00 00 00 fc 00 00                    
00023902    00 ee 00 00 00 e6 00 00  00 ea 00 00 00 ea 00 00                    
00023903    00 e8 00 00 00 de 00 00  00 df 00 00 00 e7 00 00                    
00023904    00 f6 00 00 00 ff 00 00  01 03 00 00 00 f6 00 00                    
00023905    01 08 00 00 01 03 00 00  00 fd 00 00 01 05 00 00                    
00023906    01 02 00 00 01 00 00 00  01 14 00 00 01 18 00 00                    
00023907    00 fd 00 00 00 fb 00 00  01 11 00 00 01 05 00 00                    
00023908    01 05 00 00 01 0a 00 00  01 01 00 00 00 f3 00 00                    
00023909    00 f7 00 00 00 f7 00 00  01 01 00 00 01 02 00 00                    
00023910    00 f8 00 00 00 f8 00 00  00 ef 00 00 00 ed 00 00                    
00023911    00 e3 00 00 00 ec 00 00  00 e2 00 00 00 e8 00 00                    
00023912    00 dc 00 00 00 e0 00 00  00 f3 00 00 00 df 00 00                    
00023913    00 e1 00 00 00 cf 00 00  00 ce 00 00 00 d8 00 00                    
00023914    00 ce 00 00 00 c7 00 00  00 cd 00 00 00 b7 00 00                    
00023915    00 af 00 00 00 c8 00 00  00 d7 00 00 00 e5 00 00                    
00023916    00 e4 00 00 00 c6 00 00  00 d1 00 00 00 d5 00 00                    
00023917    00 e5 00 00 00 d8 00 00  00 c8 00 00 00 be 00 00                    
00023918    00 bf 00 00 00 cb 00 00  00 d2 00 00 00 c8 00 00                    
00023919    00 ca 00 00 00 b1 00 00  00 a3 00 00 00 c7 00 00                    
00023920    00 dc 00 00 00 d9 00 00  00 dd 00 00 00 d1 00 00                    
00023921    00 d2 00 00 00 c2 00 00  00 bc 00 00 00 b1 00 00                    
00023922    00 9b 00 00 00 89 00 00  00 a2 00 00 00 9f 00 00                    
00023923    00 b5 00 00 00 a6 00 00  00 b2 00 00 00 b5 00 00                    
00023924    00 ae 00 00 00 b4 00 00  00 b0 00 00 00 c6 00 00                    
00023925    00 c3 00 00 00 d5 00 00  00 e4 00 00 00 f6 00 00                    
00023926    00 d6 00 00 00 db 00 00  00 cc 00 00 00 e7 00 00                    
00023927    00 f9 00 00 00 cb 00 00  00 d8 00 00 00 d6 00 00                    
00023928    00 e4 00 00 00 f1 00 00  00 e4 00 00 00 e6 00 00                    
00023929    00 df 00 00 00 ee 00 00  00 d7 00 00 00 c7 00 00                    
00023930    00 e7 00 00 00 f9 00 00  00 ed 00 00 00 cf 00 00                    
00023931    00 f1 00 00 00 e6 00 00  00 dc 00 00 00 e4 00 00                    
00023932    00 ef 00 00 00 e5 00 00  00 f1 00 00 00 e3 00 00                    
00023933    00 ec 00 00 00 ec 00 00  00 f3 00 00 00 f5 00 00                    
00023934    00 fd 00 00 01 0b 00 00  01 10 00 00 01 11 00 00                    
00023935    01 03 00 00 01 01 00 00  00 fb 00 00 00 fa 00 00                    
00023936    00 e7 00 00 00 e5 00 00  00 f0 00 00 00 d2 00 00                    
00023937    00 e5 00 00 00 f3 00 00  00 f1 00 00 00 f2 00 00                    
00023938    00 ff 00 00 00 f7 00 00  00 ee 00 00 00 d5 00 00                    
00023939    00 d9 00 00 00 ea 00 00  00 e3 00 00 00 df 00 00                    
00023940    00 f7 00 00 00 ff 00 00  00 f8 00 00 00 fa 00 00                    
00023941    00 fd 00 00 00 f7 00 00  00 f9 00 00 00 fb 00 00                    
00023942    00 f8 00 00 00 f6 00 00  00 f0 00 00 00 fe 00 00                    
00023943    01 02 00 00 00 e9 00 00  00 ec 00 00 00 ec 00 00                    
00023944    00 e7 00 00 00 ea 00 00  00 de 00 00 00 e2 00 00                    
00023945    00 c9 00 00 00 d4 00 00  00 d4 00 00 00 c7 00 00                    
00023946    00 c9 00 00 00 c8 00 00  00 c1 00 00 00 c0 00 00                    
00023947    00 bd 00 00 00 de 00 00  00 cb 00 00 00 cd 00 00                    
00023948    00 d4 00 00 00 6d 00 00  00 28 73 74 73 63 00 00         m   (stsc  
00023949    00 00 00 00 00 02 00 00  00 01 00 00 00 07 00 00                    
00023950    00 01 00 00 00 26 00 00  00 02 00 00 00 01 00 00         &          
00023951    00 a8 73 74 63 6f 00 00  00 00 00 00 00 26 00 00      stco       &  
00023952    6d 8d 00 00 85 9b 00 00  9d e4 00 00 b1 21 00 00    m            !  
00023953    c2 a7 00 00 d2 8e 00 00  df 8e 00 00 f3 54 00 00                 T  
00023954    fe e8 00 01 1a 05 00 01  4a c7 00 01 7e 28 00 01            J   ~(  
00023955    c5 a1 00 01 fc 89 00 02  35 5c 00 02 83 6a 00 02            5\   j  
00023956    ae 62 00 02 e1 ae 00 03  1d 6d 00 03 58 04 00 03     b       m  X   
00023957    99 4d 00 03 e4 b1 00 04  21 99 00 04 39 67 00 04     M      !   9g  
00023958    5d c6 00 04 78 18 00 04  92 6a 00 04 a7 67 00 04    ]   x    j   g  
00023959    b8 7e 00 04 d0 6c 00 04  e7 0c 00 05 07 c6 00 05     ~   l          
00023960    53 5c 00 05 74 bc 00 05  8e 99 00 05 ae 19 00 05    S\  t           
00023961    c0 eb 00 05 cb 47                                        G
{% endcodeblock %}

##### **mdhd**

{% codeblock lang:shell %}
00023865                                   00 00 00 20 6d 64                  md
00023866    68 64 00 00 00 00 c7 ca  ee a7 c7 ca ee a8 00 00    hd              
00023867    bb 80 00 04 14 00 15 c7  00 00
{% endcodeblock %}

##### **hdlr**

{% codeblock lang:shell %}
00023867                                   00 00 00 21 68 64                 !hd
00023868    6c 72 00 00 00 00 00 00  00 00 73 6f 75 6e 00 00    lr        soun  
00023869    00 00 00 00 00 00 00 00  00 00 00
{% endcodeblock %}

##### **minf**

{% codeblock lang:shell %}
00023869                                      00 00 05 bb 6d                   m
00023870    69 6e 66 00 00 00 10 73  6d 68 64 00 00 00 00 00    inf    smhd     
00023871    00 00 00 00 00 00 24 64  69 6e 66 00 00 00 1c 64          $dinf    d
00023872    72 65 66 00 00 00 00 00  00 00 01 00 00 00 0c 75    ref            u
00023873    72 6c 20 00 00 00 01 00  00 05 7f 73 74 62 6c 00    rl         stbl 
00023874    00 00 67 73 74 73 64 00  00 00 00 00 00 00 01 00      gstsd         
00023875    00 00 57 6d 70 34 61 00  00 00 00 00 00 00 01 00      Wmp4a         
00023876    00 00 00 00 00 00 00 00  01 00 10 00 00 00 00 bb                    
00023877    80 00 00 00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02 00 00  00 18 73 74 74 73 00 00              stts  
00023881    00 00 00 00 00 01 00 00  01 05 00 00 04 00 00 00                    
00023882    04 28 73 74 73 7a 00 00  00 00 00 00 00 00 00 00     (stsz          
00023883    01 05 00 00 00 f7 00 00  00 db 00 00 00 e1 00 00                    
00023884    00 e5 00 00 00 e9 00 00  00 e8 00 00 00 f0 00 00                    
00023885    00 f1 00 00 00 ef 00 00  00 d8 00 00 00 e6 00 00                    
00023886    00 e7 00 00 00 e9 00 00  00 eb 00 00 00 ea 00 00                    
00023887    00 e8 00 00 00 e1 00 00  00 e7 00 00 00 d7 00 00                    
00023888    00 da 00 00 00 d9 00 00  00 db 00 00 00 e9 00 00                    
00023889    00 ee 00 00 00 e5 00 00  00 e1 00 00 00 e6 00 00                    
00023890    00 e5 00 00 00 d8 00 00  00 dd 00 00 00 dd 00 00                    
00023891    00 d5 00 00 00 ea 00 00  00 dd 00 00 00 d0 00 00                    
00023892    00 d6 00 00 00 e9 00 00  00 bc 00 00 00 ab 00 00                    
00023893    00 b3 00 00 00 b5 00 00  00 bc 00 00 00 ce 00 00                    
00023894    00 b4 00 00 00 b6 00 00  00 b3 00 00 00 b6 00 00                    
00023895    00 b7 00 00 00 bf 00 00  00 b7 00 00 00 cd 00 00                    
00023896    00 c1 00 00 00 ba 00 00  00 a7 00 00 00 b4 00 00                    
00023897    00 b1 00 00 00 be 00 00  00 d0 00 00 00 ba 00 00                    
00023898    00 bc 00 00 00 c4 00 00  00 c6 00 00 00 cb 00 00                    
00023899    00 c4 00 00 00 c3 00 00  00 c8 00 00 00 d2 00 00                    
00023900    00 d2 00 00 00 d6 00 00  00 f5 00 00 00 fa 00 00                    
00023901    00 f6 00 00 01 02 00 00  00 fc 00 00 00 fc 00 00                    
00023902    00 ee 00 00 00 e6 00 00  00 ea 00 00 00 ea 00 00                    
00023903    00 e8 00 00 00 de 00 00  00 df 00 00 00 e7 00 00                    
00023904    00 f6 00 00 00 ff 00 00  01 03 00 00 00 f6 00 00                    
00023905    01 08 00 00 01 03 00 00  00 fd 00 00 01 05 00 00                    
00023906    01 02 00 00 01 00 00 00  01 14 00 00 01 18 00 00                    
00023907    00 fd 00 00 00 fb 00 00  01 11 00 00 01 05 00 00                    
00023908    01 05 00 00 01 0a 00 00  01 01 00 00 00 f3 00 00                    
00023909    00 f7 00 00 00 f7 00 00  01 01 00 00 01 02 00 00                    
00023910    00 f8 00 00 00 f8 00 00  00 ef 00 00 00 ed 00 00                    
00023911    00 e3 00 00 00 ec 00 00  00 e2 00 00 00 e8 00 00                    
00023912    00 dc 00 00 00 e0 00 00  00 f3 00 00 00 df 00 00                    
00023913    00 e1 00 00 00 cf 00 00  00 ce 00 00 00 d8 00 00                    
00023914    00 ce 00 00 00 c7 00 00  00 cd 00 00 00 b7 00 00                    
00023915    00 af 00 00 00 c8 00 00  00 d7 00 00 00 e5 00 00                    
00023916    00 e4 00 00 00 c6 00 00  00 d1 00 00 00 d5 00 00                    
00023917    00 e5 00 00 00 d8 00 00  00 c8 00 00 00 be 00 00                    
00023918    00 bf 00 00 00 cb 00 00  00 d2 00 00 00 c8 00 00                    
00023919    00 ca 00 00 00 b1 00 00  00 a3 00 00 00 c7 00 00                    
00023920    00 dc 00 00 00 d9 00 00  00 dd 00 00 00 d1 00 00                    
00023921    00 d2 00 00 00 c2 00 00  00 bc 00 00 00 b1 00 00                    
00023922    00 9b 00 00 00 89 00 00  00 a2 00 00 00 9f 00 00                    
00023923    00 b5 00 00 00 a6 00 00  00 b2 00 00 00 b5 00 00                    
00023924    00 ae 00 00 00 b4 00 00  00 b0 00 00 00 c6 00 00                    
00023925    00 c3 00 00 00 d5 00 00  00 e4 00 00 00 f6 00 00                    
00023926    00 d6 00 00 00 db 00 00  00 cc 00 00 00 e7 00 00                    
00023927    00 f9 00 00 00 cb 00 00  00 d8 00 00 00 d6 00 00                    
00023928    00 e4 00 00 00 f1 00 00  00 e4 00 00 00 e6 00 00                    
00023929    00 df 00 00 00 ee 00 00  00 d7 00 00 00 c7 00 00                    
00023930    00 e7 00 00 00 f9 00 00  00 ed 00 00 00 cf 00 00                    
00023931    00 f1 00 00 00 e6 00 00  00 dc 00 00 00 e4 00 00                    
00023932    00 ef 00 00 00 e5 00 00  00 f1 00 00 00 e3 00 00                    
00023933    00 ec 00 00 00 ec 00 00  00 f3 00 00 00 f5 00 00                    
00023934    00 fd 00 00 01 0b 00 00  01 10 00 00 01 11 00 00                    
00023935    01 03 00 00 01 01 00 00  00 fb 00 00 00 fa 00 00                    
00023936    00 e7 00 00 00 e5 00 00  00 f0 00 00 00 d2 00 00                    
00023937    00 e5 00 00 00 f3 00 00  00 f1 00 00 00 f2 00 00                    
00023938    00 ff 00 00 00 f7 00 00  00 ee 00 00 00 d5 00 00                    
00023939    00 d9 00 00 00 ea 00 00  00 e3 00 00 00 df 00 00                    
00023940    00 f7 00 00 00 ff 00 00  00 f8 00 00 00 fa 00 00                    
00023941    00 fd 00 00 00 f7 00 00  00 f9 00 00 00 fb 00 00                    
00023942    00 f8 00 00 00 f6 00 00  00 f0 00 00 00 fe 00 00                    
00023943    01 02 00 00 00 e9 00 00  00 ec 00 00 00 ec 00 00                    
00023944    00 e7 00 00 00 ea 00 00  00 de 00 00 00 e2 00 00                    
00023945    00 c9 00 00 00 d4 00 00  00 d4 00 00 00 c7 00 00                    
00023946    00 c9 00 00 00 c8 00 00  00 c1 00 00 00 c0 00 00                    
00023947    00 bd 00 00 00 de 00 00  00 cb 00 00 00 cd 00 00                    
00023948    00 d4 00 00 00 6d 00 00  00 28 73 74 73 63 00 00         m   (stsc  
00023949    00 00 00 00 00 02 00 00  00 01 00 00 00 07 00 00                    
00023950    00 01 00 00 00 26 00 00  00 02 00 00 00 01 00 00         &          
00023951    00 a8 73 74 63 6f 00 00  00 00 00 00 00 26 00 00      stco       &  
00023952    6d 8d 00 00 85 9b 00 00  9d e4 00 00 b1 21 00 00    m            !  
00023953    c2 a7 00 00 d2 8e 00 00  df 8e 00 00 f3 54 00 00                 T  
00023954    fe e8 00 01 1a 05 00 01  4a c7 00 01 7e 28 00 01            J   ~(  
00023955    c5 a1 00 01 fc 89 00 02  35 5c 00 02 83 6a 00 02            5\   j  
00023956    ae 62 00 02 e1 ae 00 03  1d 6d 00 03 58 04 00 03     b       m  X   
00023957    99 4d 00 03 e4 b1 00 04  21 99 00 04 39 67 00 04     M      !   9g  
00023958    5d c6 00 04 78 18 00 04  92 6a 00 04 a7 67 00 04    ]   x    j   g  
00023959    b8 7e 00 04 d0 6c 00 04  e7 0c 00 05 07 c6 00 05     ~   l          
00023960    53 5c 00 05 74 bc 00 05  8e 99 00 05 ae 19 00 05    S\  t           
00023961    c0 eb 00 05 cb 47                                        G
{% endcodeblock %}

###### **smhd**

{% codeblock lang:shell %}
00023870             00 00 00 10 73  6d 68 64 00 00 00 00 00           smhd     
00023871    00 00 00
{% endcodeblock %}

###### **dinf**

{% codeblock lang:shell %}
00023871             00 00 00 24 64  69 6e 66 00 00 00 1c 64          $dinf    d
00023872    72 65 66 00 00 00 00 00  00 00 01 00 00 00 0c 75    ref            u
00023873    72 6c 20 00 00 00 01                                rl
{% endcodeblock %}

####### **dref**

{% codeblock lang:shell %}
00023871                                      00 00 00 1c 64                   d
00023872    72 65 66 00 00 00 00 00  00 00 01 00 00 00 0c 75    ref            u
00023873    72 6c 20 00 00 00 01                                rl
{% endcodeblock %}

###### **stbl**

{% codeblock lang:shell %}
00023873                         00  00 05 7f 73 74 62 6c 00               stbl 
00023874    00 00 67 73 74 73 64 00  00 00 00 00 00 00 01 00      gstsd         
00023875    00 00 57 6d 70 34 61 00  00 00 00 00 00 00 01 00      Wmp4a         
00023876    00 00 00 00 00 00 00 00  01 00 10 00 00 00 00 bb                    
00023877    80 00 00 00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02 00 00  00 18 73 74 74 73 00 00              stts  
00023881    00 00 00 00 00 01 00 00  01 05 00 00 04 00 00 00                    
00023882    04 28 73 74 73 7a 00 00  00 00 00 00 00 00 00 00     (stsz          
00023883    01 05 00 00 00 f7 00 00  00 db 00 00 00 e1 00 00                    
00023884    00 e5 00 00 00 e9 00 00  00 e8 00 00 00 f0 00 00                    
00023885    00 f1 00 00 00 ef 00 00  00 d8 00 00 00 e6 00 00                    
00023886    00 e7 00 00 00 e9 00 00  00 eb 00 00 00 ea 00 00                    
00023887    00 e8 00 00 00 e1 00 00  00 e7 00 00 00 d7 00 00                    
00023888    00 da 00 00 00 d9 00 00  00 db 00 00 00 e9 00 00                    
00023889    00 ee 00 00 00 e5 00 00  00 e1 00 00 00 e6 00 00                    
00023890    00 e5 00 00 00 d8 00 00  00 dd 00 00 00 dd 00 00                    
00023891    00 d5 00 00 00 ea 00 00  00 dd 00 00 00 d0 00 00                    
00023892    00 d6 00 00 00 e9 00 00  00 bc 00 00 00 ab 00 00                    
00023893    00 b3 00 00 00 b5 00 00  00 bc 00 00 00 ce 00 00                    
00023894    00 b4 00 00 00 b6 00 00  00 b3 00 00 00 b6 00 00                    
00023895    00 b7 00 00 00 bf 00 00  00 b7 00 00 00 cd 00 00                    
00023896    00 c1 00 00 00 ba 00 00  00 a7 00 00 00 b4 00 00                    
00023897    00 b1 00 00 00 be 00 00  00 d0 00 00 00 ba 00 00                    
00023898    00 bc 00 00 00 c4 00 00  00 c6 00 00 00 cb 00 00                    
00023899    00 c4 00 00 00 c3 00 00  00 c8 00 00 00 d2 00 00                    
00023900    00 d2 00 00 00 d6 00 00  00 f5 00 00 00 fa 00 00                    
00023901    00 f6 00 00 01 02 00 00  00 fc 00 00 00 fc 00 00                    
00023902    00 ee 00 00 00 e6 00 00  00 ea 00 00 00 ea 00 00                    
00023903    00 e8 00 00 00 de 00 00  00 df 00 00 00 e7 00 00                    
00023904    00 f6 00 00 00 ff 00 00  01 03 00 00 00 f6 00 00                    
00023905    01 08 00 00 01 03 00 00  00 fd 00 00 01 05 00 00                    
00023906    01 02 00 00 01 00 00 00  01 14 00 00 01 18 00 00                    
00023907    00 fd 00 00 00 fb 00 00  01 11 00 00 01 05 00 00                    
00023908    01 05 00 00 01 0a 00 00  01 01 00 00 00 f3 00 00                    
00023909    00 f7 00 00 00 f7 00 00  01 01 00 00 01 02 00 00                    
00023910    00 f8 00 00 00 f8 00 00  00 ef 00 00 00 ed 00 00                    
00023911    00 e3 00 00 00 ec 00 00  00 e2 00 00 00 e8 00 00                    
00023912    00 dc 00 00 00 e0 00 00  00 f3 00 00 00 df 00 00                    
00023913    00 e1 00 00 00 cf 00 00  00 ce 00 00 00 d8 00 00                    
00023914    00 ce 00 00 00 c7 00 00  00 cd 00 00 00 b7 00 00                    
00023915    00 af 00 00 00 c8 00 00  00 d7 00 00 00 e5 00 00                    
00023916    00 e4 00 00 00 c6 00 00  00 d1 00 00 00 d5 00 00                    
00023917    00 e5 00 00 00 d8 00 00  00 c8 00 00 00 be 00 00                    
00023918    00 bf 00 00 00 cb 00 00  00 d2 00 00 00 c8 00 00                    
00023919    00 ca 00 00 00 b1 00 00  00 a3 00 00 00 c7 00 00                    
00023920    00 dc 00 00 00 d9 00 00  00 dd 00 00 00 d1 00 00                    
00023921    00 d2 00 00 00 c2 00 00  00 bc 00 00 00 b1 00 00                    
00023922    00 9b 00 00 00 89 00 00  00 a2 00 00 00 9f 00 00                    
00023923    00 b5 00 00 00 a6 00 00  00 b2 00 00 00 b5 00 00                    
00023924    00 ae 00 00 00 b4 00 00  00 b0 00 00 00 c6 00 00                    
00023925    00 c3 00 00 00 d5 00 00  00 e4 00 00 00 f6 00 00                    
00023926    00 d6 00 00 00 db 00 00  00 cc 00 00 00 e7 00 00                    
00023927    00 f9 00 00 00 cb 00 00  00 d8 00 00 00 d6 00 00                    
00023928    00 e4 00 00 00 f1 00 00  00 e4 00 00 00 e6 00 00                    
00023929    00 df 00 00 00 ee 00 00  00 d7 00 00 00 c7 00 00                    
00023930    00 e7 00 00 00 f9 00 00  00 ed 00 00 00 cf 00 00                    
00023931    00 f1 00 00 00 e6 00 00  00 dc 00 00 00 e4 00 00                    
00023932    00 ef 00 00 00 e5 00 00  00 f1 00 00 00 e3 00 00                    
00023933    00 ec 00 00 00 ec 00 00  00 f3 00 00 00 f5 00 00                    
00023934    00 fd 00 00 01 0b 00 00  01 10 00 00 01 11 00 00                    
00023935    01 03 00 00 01 01 00 00  00 fb 00 00 00 fa 00 00                    
00023936    00 e7 00 00 00 e5 00 00  00 f0 00 00 00 d2 00 00                    
00023937    00 e5 00 00 00 f3 00 00  00 f1 00 00 00 f2 00 00                    
00023938    00 ff 00 00 00 f7 00 00  00 ee 00 00 00 d5 00 00                    
00023939    00 d9 00 00 00 ea 00 00  00 e3 00 00 00 df 00 00                    
00023940    00 f7 00 00 00 ff 00 00  00 f8 00 00 00 fa 00 00                    
00023941    00 fd 00 00 00 f7 00 00  00 f9 00 00 00 fb 00 00                    
00023942    00 f8 00 00 00 f6 00 00  00 f0 00 00 00 fe 00 00                    
00023943    01 02 00 00 00 e9 00 00  00 ec 00 00 00 ec 00 00                    
00023944    00 e7 00 00 00 ea 00 00  00 de 00 00 00 e2 00 00                    
00023945    00 c9 00 00 00 d4 00 00  00 d4 00 00 00 c7 00 00                    
00023946    00 c9 00 00 00 c8 00 00  00 c1 00 00 00 c0 00 00                    
00023947    00 bd 00 00 00 de 00 00  00 cb 00 00 00 cd 00 00                    
00023948    00 d4 00 00 00 6d 00 00  00 28 73 74 73 63 00 00         m   (stsc  
00023949    00 00 00 00 00 02 00 00  00 01 00 00 00 07 00 00                    
00023950    00 01 00 00 00 26 00 00  00 02 00 00 00 01 00 00         &          
00023951    00 a8 73 74 63 6f 00 00  00 00 00 00 00 26 00 00      stco       &  
00023952    6d 8d 00 00 85 9b 00 00  9d e4 00 00 b1 21 00 00    m            !  
00023953    c2 a7 00 00 d2 8e 00 00  df 8e 00 00 f3 54 00 00                 T  
00023954    fe e8 00 01 1a 05 00 01  4a c7 00 01 7e 28 00 01            J   ~(  
00023955    c5 a1 00 01 fc 89 00 02  35 5c 00 02 83 6a 00 02            5\   j  
00023956    ae 62 00 02 e1 ae 00 03  1d 6d 00 03 58 04 00 03     b       m  X   
00023957    99 4d 00 03 e4 b1 00 04  21 99 00 04 39 67 00 04     M      !   9g  
00023958    5d c6 00 04 78 18 00 04  92 6a 00 04 a7 67 00 04    ]   x    j   g  
00023959    b8 7e 00 04 d0 6c 00 04  e7 0c 00 05 07 c6 00 05     ~   l          
00023960    53 5c 00 05 74 bc 00 05  8e 99 00 05 ae 19 00 05    S\  t           
00023961    c0 eb 00 05 cb 47                                        G
{% endcodeblock %}

####### **stsd**

{% codeblock lang:shell %}
00023873                                                  00                
00023874    00 00 67 73 74 73 64 00  00 00 00 00 00 00 01 00      gstsd         
00023875    00 00 57 6d 70 34 61 00  00 00 00 00 00 00 01 00      Wmp4a         
00023876    00 00 00 00 00 00 00 00  01 00 10 00 00 00 00 bb                    
00023877    80 00 00 00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02
{% endcodeblock %}

######## **esds**

{% codeblock lang:shell %}
00023877             00 00 00 33 65  73 64 73 00 00 00 00 03          3esds     
00023878    80 80 80 22 00 00 00 04  80 80 80 14 40 15 00 01       "        @   
00023879    18 00 01 65 f0 00 01 44  6b 05 80 80 80 02 11 88       e   Dk       
00023880    06 80 80 80 01 02
{% endcodeblock %}

####### **stts**

{% codeblock lang:shell %}
00023880                      00 00  00 18 73 74 74 73 00 00              stts  
00023881    00 00 00 00 00 01 00 00  01 05 00 00 04 00
{% endcodeblock %}

####### **stsz**

{% codeblock lang:shell %}
00023881                                               00 00                    
00023882    04 28 73 74 73 7a 00 00  00 00 00 00 00 00 00 00     (stsz          
00023883    01 05 00 00 00 f7 00 00  00 db 00 00 00 e1 00 00                    
00023884    00 e5 00 00 00 e9 00 00  00 e8 00 00 00 f0 00 00                    
00023885    00 f1 00 00 00 ef 00 00  00 d8 00 00 00 e6 00 00                    
00023886    00 e7 00 00 00 e9 00 00  00 eb 00 00 00 ea 00 00                    
00023887    00 e8 00 00 00 e1 00 00  00 e7 00 00 00 d7 00 00                    
00023888    00 da 00 00 00 d9 00 00  00 db 00 00 00 e9 00 00                    
00023889    00 ee 00 00 00 e5 00 00  00 e1 00 00 00 e6 00 00                    
00023890    00 e5 00 00 00 d8 00 00  00 dd 00 00 00 dd 00 00                    
00023891    00 d5 00 00 00 ea 00 00  00 dd 00 00 00 d0 00 00                    
00023892    00 d6 00 00 00 e9 00 00  00 bc 00 00 00 ab 00 00                    
00023893    00 b3 00 00 00 b5 00 00  00 bc 00 00 00 ce 00 00                    
00023894    00 b4 00 00 00 b6 00 00  00 b3 00 00 00 b6 00 00                    
00023895    00 b7 00 00 00 bf 00 00  00 b7 00 00 00 cd 00 00                    
00023896    00 c1 00 00 00 ba 00 00  00 a7 00 00 00 b4 00 00                    
00023897    00 b1 00 00 00 be 00 00  00 d0 00 00 00 ba 00 00                    
00023898    00 bc 00 00 00 c4 00 00  00 c6 00 00 00 cb 00 00                    
00023899    00 c4 00 00 00 c3 00 00  00 c8 00 00 00 d2 00 00                    
00023900    00 d2 00 00 00 d6 00 00  00 f5 00 00 00 fa 00 00                    
00023901    00 f6 00 00 01 02 00 00  00 fc 00 00 00 fc 00 00                    
00023902    00 ee 00 00 00 e6 00 00  00 ea 00 00 00 ea 00 00                    
00023903    00 e8 00 00 00 de 00 00  00 df 00 00 00 e7 00 00                    
00023904    00 f6 00 00 00 ff 00 00  01 03 00 00 00 f6 00 00                    
00023905    01 08 00 00 01 03 00 00  00 fd 00 00 01 05 00 00                    
00023906    01 02 00 00 01 00 00 00  01 14 00 00 01 18 00 00                    
00023907    00 fd 00 00 00 fb 00 00  01 11 00 00 01 05 00 00                    
00023908    01 05 00 00 01 0a 00 00  01 01 00 00 00 f3 00 00                    
00023909    00 f7 00 00 00 f7 00 00  01 01 00 00 01 02 00 00                    
00023910    00 f8 00 00 00 f8 00 00  00 ef 00 00 00 ed 00 00                    
00023911    00 e3 00 00 00 ec 00 00  00 e2 00 00 00 e8 00 00                    
00023912    00 dc 00 00 00 e0 00 00  00 f3 00 00 00 df 00 00                    
00023913    00 e1 00 00 00 cf 00 00  00 ce 00 00 00 d8 00 00                    
00023914    00 ce 00 00 00 c7 00 00  00 cd 00 00 00 b7 00 00                    
00023915    00 af 00 00 00 c8 00 00  00 d7 00 00 00 e5 00 00                    
00023916    00 e4 00 00 00 c6 00 00  00 d1 00 00 00 d5 00 00                    
00023917    00 e5 00 00 00 d8 00 00  00 c8 00 00 00 be 00 00                    
00023918    00 bf 00 00 00 cb 00 00  00 d2 00 00 00 c8 00 00                    
00023919    00 ca 00 00 00 b1 00 00  00 a3 00 00 00 c7 00 00                    
00023920    00 dc 00 00 00 d9 00 00  00 dd 00 00 00 d1 00 00                    
00023921    00 d2 00 00 00 c2 00 00  00 bc 00 00 00 b1 00 00                    
00023922    00 9b 00 00 00 89 00 00  00 a2 00 00 00 9f 00 00                    
00023923    00 b5 00 00 00 a6 00 00  00 b2 00 00 00 b5 00 00                    
00023924    00 ae 00 00 00 b4 00 00  00 b0 00 00 00 c6 00 00                    
00023925    00 c3 00 00 00 d5 00 00  00 e4 00 00 00 f6 00 00                    
00023926    00 d6 00 00 00 db 00 00  00 cc 00 00 00 e7 00 00                    
00023927    00 f9 00 00 00 cb 00 00  00 d8 00 00 00 d6 00 00                    
00023928    00 e4 00 00 00 f1 00 00  00 e4 00 00 00 e6 00 00                    
00023929    00 df 00 00 00 ee 00 00  00 d7 00 00 00 c7 00 00                    
00023930    00 e7 00 00 00 f9 00 00  00 ed 00 00 00 cf 00 00                    
00023931    00 f1 00 00 00 e6 00 00  00 dc 00 00 00 e4 00 00                    
00023932    00 ef 00 00 00 e5 00 00  00 f1 00 00 00 e3 00 00                    
00023933    00 ec 00 00 00 ec 00 00  00 f3 00 00 00 f5 00 00                    
00023934    00 fd 00 00 01 0b 00 00  01 10 00 00 01 11 00 00                    
00023935    01 03 00 00 01 01 00 00  00 fb 00 00 00 fa 00 00                    
00023936    00 e7 00 00 00 e5 00 00  00 f0 00 00 00 d2 00 00                    
00023937    00 e5 00 00 00 f3 00 00  00 f1 00 00 00 f2 00 00                    
00023938    00 ff 00 00 00 f7 00 00  00 ee 00 00 00 d5 00 00                    
00023939    00 d9 00 00 00 ea 00 00  00 e3 00 00 00 df 00 00                    
00023940    00 f7 00 00 00 ff 00 00  00 f8 00 00 00 fa 00 00                    
00023941    00 fd 00 00 00 f7 00 00  00 f9 00 00 00 fb 00 00                    
00023942    00 f8 00 00 00 f6 00 00  00 f0 00 00 00 fe 00 00                    
00023943    01 02 00 00 00 e9 00 00  00 ec 00 00 00 ec 00 00                    
00023944    00 e7 00 00 00 ea 00 00  00 de 00 00 00 e2 00 00                    
00023945    00 c9 00 00 00 d4 00 00  00 d4 00 00 00 c7 00 00                    
00023946    00 c9 00 00 00 c8 00 00  00 c1 00 00 00 c0 00 00                    
00023947    00 bd 00 00 00 de 00 00  00 cb 00 00 00 cd 00 00                    
00023948    00 d4 00 00 00 6d                                        m
{% endcodeblock %}

####### **stsc**

{% codeblock lang:shell %}
00023948                      00 00  00 28 73 74 73 63 00 00             (stsc  
00023949    00 00 00 00 00 02 00 00  00 01 00 00 00 07 00 00                    
00023950    00 01 00 00 00 26 00 00  00 02 00 00 00 01               &
{% endcodeblock %}

####### **stco**

{% codeblock lang:shell %}
00023950                                               00 00                    
00023951    00 a8 73 74 63 6f 00 00  00 00 00 00 00 26 00 00      stco       &  
00023952    6d 8d 00 00 85 9b 00 00  9d e4 00 00 b1 21 00 00    m            !  
00023953    c2 a7 00 00 d2 8e 00 00  df 8e 00 00 f3 54 00 00                 T  
00023954    fe e8 00 01 1a 05 00 01  4a c7 00 01 7e 28 00 01            J   ~(  
00023955    c5 a1 00 01 fc 89 00 02  35 5c 00 02 83 6a 00 02            5\   j  
00023956    ae 62 00 02 e1 ae 00 03  1d 6d 00 03 58 04 00 03     b       m  X   
00023957    99 4d 00 03 e4 b1 00 04  21 99 00 04 39 67 00 04     M      !   9g  
00023958    5d c6 00 04 78 18 00 04  92 6a 00 04 a7 67 00 04    ]   x    j   g  
00023959    b8 7e 00 04 d0 6c 00 04  e7 0c 00 05 07 c6 00 05     ~   l          
00023960    53 5c 00 05 74 bc 00 05  8e 99 00 05 ae 19 00 05    S\  t           
00023961    c0 eb 00 05 cb 47                                        G
{% endcodeblock %}

### **udta**

{% codeblock lang:shell %}
00023961                      00 00  00 16 75 64 74 61 00 00              udta  
00023962    00 0e 6e 61 6d 65 53 74  65 72 65 6f                  nameStereo
{% endcodeblock %}

### **udta**

{% codeblock lang:shell %}
00023961                                         74 61 00 00              udta  
00023962    00 0e 6e 61 6d 65 53 74  65 72 65 6f 00 00 00 6f      nameStereo   o
00023963    75 64 74 61 00 00 00 67  6d 65 74 61 00 00 00 00    udta   gmeta    
00023964    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023965    6d 64 69 72 00 00 00 00  00 00 00 00 00 00 00 00    mdir            
00023966    00 00 00 00 3a 69 6c 73  74 00 00 00 32 a9 74 6f        :ilst   2 to
00023967    6f 00 00 00 2a 64 61 74  61 00 00 00 01 00 00 00    o   *data       
00023968    00 48 61 6e 64 42 72 61  6b 65 20 30 2e 39 2e 34     HandBrake 0.9.4
{% endcodeblock %}

#### **meta**

{% codeblock lang:shell %}
00023963                00 00 00 67  6d 65 74 61 00 00 00 00    udta   gmeta    
00023964    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023965    6d 64 69 72 00 00 00 00  00 00 00 00 00 00 00 00    mdir            
00023966    00 00 00 00 3a 69 6c 73  74 00 00 00 32 a9 74 6f        :ilst   2 to
00023967    6f 00 00 00 2a 64 61 74  61 00 00 00 01 00 00 00    o   *data       
00023968    00 48 61 6e 64 42 72 61  6b 65 20 30 2e 39 2e 34     HandBrake 0.9.4
00023969    20 32 30 30 39 31 31 32  33 30 30                    2009112300
{% endcodeblock %}

##### **hdlr**

{% codeblock lang:shell %}
00023964    00 00 00 21 68 64 6c 72  00 00 00 00 00 00 00 00       !hdlr        
00023965    6d 64 69 72 00 00 00 00  00 00 00 00 00 00 00 00    mdir            
00023966    00
{% endcodeblock %}

##### **ilst**

{% codeblock lang:shell %}
00023966       00 00 00 3a 69 6c 73  74 00 00 00 32 a9 74 6f        :ilst   2 to
00023967    6f 00 00 00 2a 64 61 74  61 00 00 00 01 00 00 00    o   *data       
00023968    00 48 61 6e 64 42 72 61  6b 65 20 30 2e 39 2e 34     HandBrake 0.9.4
00023969    20 32 30 30 39 31 31 32  33 30 30                    2009112300
{% endcodeblock %}

## **free**

{% codeblock lang:shell %}
00023969                                      00 00 00 84 66                   f
00023970    72 65 65 00 00 00 00 00  00 00 00 00 00 00 00 00    ree             
00023971    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023972    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023973    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023974    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023975    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023976    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00                    
00023977    00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 03                    
{% endcodeblock %}

---

# **ffmpeg 解封装 MP4**

参数|类型|说明
--|--
use_absolute_path|布尔|可以通过绝对路径加载外部的 track，可能会有安全因素的影响，默认不开启
seek_streams_individually|布尔|根据单独流进行 seek，默认开启
ignore_editlist|布尔|忽略 EditList 信息，默认不开启
ignore_chapters|布尔|忽略 Chapters 信息，默认不开启
enable_drefs|布尔|外部 track 支持，默认不开启


---

# **ffmpeg 封装 MP4**

<table><tr><td>参数</td><td>值</td><td>说明</td></tr><tr><td rowspan="18">movflags</td></tr><tr><td></td><td>MP4 Muxer 标记</td></tr><tr><td>rtphint</td><td>增加 RTP 的 hint track</td></tr><tr><td>empty_moov</td><td>初始化空的 moov box</td></tr><tr><td>frag_keyframe</td><td>在视频关键帧处切片</td></tr><tr><td>separate_moof</td><td>每一个 track 写独立的 moof/mdat box</td></tr><tr><td>frag_custom</td><td>每一个 caller 请求时 flush 一个片段</td></tr><tr><td>isml</td><td>创建实时流媒体（创建一个直播流发布点）</td></tr><tr><td>faststart</td><td>将 moov box 移动到文件头部</td></tr><tr><td>omit_tfhd_offset</td><td>忽略 tfhd 容器中的基础数据偏移</td></tr><tr><td>disable_chpl</td><td>关闭 Nero Chapter 容器</td></tr><tr><td>default_base_moof</td><td>在 tfhd 容器中设置 default-base-is-moof 标记</td></tr><tr><td>dash</td><td>兼容 DASH 格式的 MP4 分片</td></tr><tr><td>frag_discont</td><td>分片不连续式设置 discontinuous 信号</td></tr><tr><td>delay_moov</td><td>延迟写入 moov 信息，直到第一个分片切出来，或者第一片被刷掉</td></tr><tr><td>global_sidx</td><td>在文件的开头设置公共的 sidx 索引</td></tr><tr><td>write_colr</td><td>写入 colr 容器</td></tr><tr><td>write_gama</td><td>写被弃用的 gama 容器</td></tr><tr><td>moov_size</td><td>正整数</td><td>设置 moov 容器大小的最大值</td></tr><tr><td rowspan="7">rtpflags</td></tr><tr><td></td><td>设置 RTP 传输相关的标记</td></tr><tr><td>latm</td><td>使用 MP4A-LATM 方式传输 AAC 音频</td></tr><tr><td>rfc2190</td><td>使用 RFC2190 传输 H264、H263</td></tr><tr><td>skip_rtcp</td><td>忽略使用 RTCP</td></tr><tr><td>h264_mode0</td><td>使用 RTP 传输 mode0 的 H264</td></tr><tr><td>send_bye</td><td>当传输结束时发送 RTCP 的 BYE 包</td></tr><tr><td>skip_iods</td><td>布尔型</td><td>不写入 idos 容器</td></tr><tr><td>iods_audio_profile</td><td>0～255</td><td>设置 idos 的音频 profile 容器</td></tr><tr><td>oids_video_profile</td><td>0～255</td><td>设置 idos 的视频 profile 容器</td></tr><tr><td>frag_duration</td><td>正整数</td><td>切片最大的 duration</td></tr><tr><td>min_frag_duration</td><td>正整数</td><td>切片最小的 duration</td></tr><tr><td>frag_size</td><td>正整数</td><td>切片最大的大小</td></tr><tr><td>ism_lookahead</td><td>正整数</td><td>预读取 ISM 文件的数量</td></tr><tr><td>video_track_timescale</td><td>正整数</td><td>设置所有视频的时间计算方式</td></tr><tr><td>brand</td><td>字符串</td><td>写 major brand</td></tr><tr><td>use_editlist</td><td>布尔型</td><td>使用 edit list</td></tr><tr><td>fragment_index</td><td>正整数</td><td>下一个分片编号</td></tr><tr><td>mov_gamma</td><td>0～10</td><td>Gamma 容器的 gama 值</td></tr><tr><td>frag_interleave</td><td>正整数</td><td>交错分片样本</td></tr><tr><td>encryption_scheme</td><td>字符串</td><td>配置加密的方案</td></tr><tr><td>encryption_key</td><td>二进制</td><td>密钥</td></tr><tr><td>encryption_kid</td><td>二进制</td><td>密钥标识符</td></tr></table>

## **将 moov 移动到 mdat 前面**

1、输入命令：

{% codeblock lang:shell %}
ffmpeg -i sample.mp4 -c copy -f mp4 -movflags faststart sample_faststart.mp4
{% endcodeblock %}

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/MP4-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F3.png)

3、通过 mp4info 查看 box 树：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/MP4-%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F4.png)

---