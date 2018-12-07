---
layout: article
title: ffplay 命令常用功能
date: 2018-05-13 13:13:29
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》

---

# **ffplay 命令参数**

## **查询 ffplay 命令支持的参数**

1、输入命令：

{% codeblock lang:shell %}
ffplay --help
{% endcodeblock %}

如果出现：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD1.png)

需要确认 FFmpeg 文件夹中是否有可执行文件 ffplay：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD2.png)

如果没有，说明在编译 FFmpeg 源代码时，系统中不存在 SDL（旧版本 FFmpeg 需要 1.2，新版本 FFmpeg 需要 2.0），导致未自动编译出 ffplay，需要安装 SDL，且需要先删除掉已安装的 FFmpeg，再重新编译 FFmpeg 源代码并安装。

如果有，说明未配置 FFmpeg 环境变量，需要主动配置并重启 PC。

2、输出结果（只截取了部分）：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD3.png)

## **部分参数说明**

参数|说明
--|--
x|强制设置视频显示窗口的宽度
y|强制设置视频显示窗口的高度
s|设置视频显示的宽高
fs|强制全屏显示
an|屏蔽音频
vn|屏蔽视频
sn|屏蔽字幕
ss|根据设置的秒进行定位拖动
t|设置播放视频/音频的长度
bytes|设置定位拖动的策略，0 为不可拖动，1 为可拖动，-1 为自动
nodisp|关闭图形化显示窗口
f|强制使用设置的格式进行解析
window_title|设置显示窗口的标题
af|设置音频的滤镜
codec|强制使用设置的 codec 进行解码
autorotate|自动旋转视频
ast|设置将要播放的音频流
vst|设置将要播放的视频流
sst|设置将要播放的字幕流
stats|输出多媒体播放状态
fast|非标准化规范的多媒体兼容优化
sync|音视频同步设置可根据音频时间、视频时间或者外部扩展时间进行参考
autoexit|多媒体播放完毕之后自动退出 ffplay，ffplay 默认播放完毕之后不退出播放器
exitonkeydown|当有按键按下事件产生时退出 ffplay
exitonmousedown|当有鼠标按下事件产生时退出 ffplay
loop|设置多媒体文件循环播放的次数
framedrop|当 CPU 资源占用过高时，自动丢帧
infbuf|设置无极限的播放 buffer，这个选项常用于实时流媒体播放场景
vf|视频滤镜设置
acodec|强制使用设置的音频解码器
vcodec|强制使用设置的视频解码器
scodec|强制使用设置的字幕解码器


---

# **ffplay 命令应用**

## **从视频的第 5 秒开始播放，播放 10 秒**

{% codeblock lang:shell %}
ffplay -ss 5 -t 10 sample.mp4
{% endcodeblock %}

## **播放流媒体**

1、输入命令：

{% codeblock lang:shell %}
ffplay -window_title "播放测试" rtmp://live.hkstv.hk.lxdns.com/live/hks
{% endcodeblock %}

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD4.png)

3、播放成功的截图：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD5.png)

## **强制使用错误的解码器会导致解码失败**

1、输入命令：

{% codeblock lang:shell %}
ffplay -vcodec mpeg4 sample.mp4
{% endcodeblock %}

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD6.png)

## **加载字幕**

1、编辑字幕文件 sample.srt：

{% codeblock lang:srt %}
1
00:00:01,001 --> 00:00:02,000
test 1

2
00:00:02,001 --> 00:00:03,000
test 2

3
00:00:03,001 --> 00:00:04,000
test 3

3
00:00:04,001 --> 00:00:05,000
test 4

3
00:00:05,001 --> 00:00:06,000
test 5
{% endcodeblock %}

2、输入命令：

{% codeblock lang:shell %}
ffplay -window_title "播放测试" -vf subtitles=sample.srt sample.mp4
{% endcodeblock %}

3、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD7.png)

4、加载字幕成功的截图：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD8.png)

## **将音频数据以音频波形的形式显示**

1、输入命令：

{% codeblock lang:shell %}
ffplay -showmode 1 sample.mp4
{% endcodeblock %}

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD9.png)

3、显示成功的截图：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD10.png)

## **显示解码宏块（新版本 FFmpeg 已失效）**

1、输入命令：

{% codeblock lang:shell %}
ffplay -debug vis_mb_type sample.mp4
{% endcodeblock %}

## **显示运动估计**

1、输入命令：

{% codeblock lang:shell %}
ffplay -flags2 +export_mvs sample.mp4 -vf codecview=mv=pf+bf+bb
{% endcodeblock %}

2、输出结果：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD11.png)

3、显示成功的截图：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/ffplay-%E5%91%BD%E4%BB%A4%E5%B8%B8%E7%94%A8%E5%8A%9F%E8%83%BD12.png)

---