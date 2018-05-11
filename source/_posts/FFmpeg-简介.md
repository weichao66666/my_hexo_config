---
layout: article
title: FFmpeg 简介
date: 2018-05-11 22:42:02
tags:
categories: 
copyright: true
---

# **Reference**

* 《FFmpeg 从入门到精通》
* [Compile FFmpeg for Ubuntu, Debian, or Mint](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu "https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu")
* [Ubuntu 16.04安装编译FFmpeg](https://www.linuxidc.com/Linux/2017-10/147769.htm "https://www.linuxidc.com/Linux/2017-10/147769.htm")

---

# **基本组成**

* 封装模块 AVFormat；
* 编解码模块 AVCodec；
* 滤镜模块 AVFilter；
* 输入输出设备模块 AVDevice；
* 工具模块 AVUtil；
* 视频图像转换计算模块 swscale；
* 音频转换计算模块 swresample。

---

# **编解码工具 ffmpeg**

ffmpeg 是 FFmpeg 源代码编译后生成的一个可执行程序，其可以作为命令行工具使用。

ffmpeg 的主要工作流程是：
1、解封装；
2、解码；
3、编码；
4、封装。

---

# **播放器 ffplay**

ffplay 是 FFmpeg 源代码编译后生成的一个可执行程序，其提供了音视频显示和播放相关的图像信息、音频的波形信息等。

---

# **多媒体分析器 ffprobe**

ffprobe 是 FFmpeg 源代码编译后生成的一个可执行程序，其可以从媒体文件或者媒体流中获得你想要了解的媒体信息，比如音频的参数、视频的参数、媒体容器的参数信息等。

---

# **编译源代码**

## **安装依赖**

{% codeblock lang:shell %}
sudo apt-get update
sudo apt-get -y install autoconf automake build-essential libass-dev libfreetype6-dev libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev
{% endcodeblock %}

## **创建文件夹**

{% codeblock lang:shell %}
mkdir ~/ffmpeg_sources
{% endcodeblock %}

## **安装编译器（二选一）**

### **NASM**

{% codeblock lang:shell %}
cd ~/ffmpeg_sources
wget https://www.nasm.us/pub/nasm/releasebuilds/2.13.03/nasm-2.13.03.tar.bz2
tar xjvf nasm-2.13.03.tar.bz2
cd nasm-2.13.03
./autogen.sh
PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
make
make install
{% endcodeblock %}

### **YASM**

{% codeblock lang:shell %}
sudo apt-get install yasm
{% endcodeblock %}

## **安装编解码库**

### **libx264**

{% codeblock lang:shell %}
sudo apt-get install libx264-dev
{% endcodeblock %}

### **libx265**

{% codeblock lang:shell %}
sudo apt-get install libx265-dev
{% endcodeblock %}

### **libvpx**

{% codeblock lang:shell %}
sudo apt-get install libvpx-dev
{% endcodeblock %}

### **libfdk-aac**

{% codeblock lang:shell %}
sudo apt-get install libfdk-aac-dev
{% endcodeblock %}

### **libmp3lame**

{% codeblock lang:shell %}
sudo apt-get install libmp3lame-dev
{% endcodeblock %}

### **libopus**

{% codeblock lang:shell %}
sudo apt-get install libopus-dev
{% endcodeblock %}

## **编译**

{% codeblock lang:shell %}
cd ~/ffmpeg_sources
wget -O ffmpeg-snapshot.tar.bz2 https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2
tar xjvf ffmpeg-snapshot.tar.bz2
cd ffmpeg
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure
  --prefix="$HOME/ffmpeg_build"
  --pkg-config-flags="--static"
  --extra-cflags="-I$HOME/ffmpeg_build/include"
  --extra-ldflags="-L$HOME/ffmpeg_build/lib"
  --extra-libs="-lpthread -lm"
  --bindir="$HOME/bin"
  --enable-gpl
  --enable-libaom
  --enable-libass
  --enable-libfdk-aac
  --enable-libfreetype
  --enable-libmp3lame
  --enable-libopus
  --enable-libvorbis
  --enable-libvpx
  --enable-libx264
  --enable-libx265
  --enable-nonfree
PATH="$HOME/bin:$PATH" make
sudo make install
hash -r
{% endcodeblock %}

## **让 ffmpeg 命令立刻生效**

{% codeblock lang:shell %}
source ~/.profile
{% endcodeblock %}

## **查看 ffmpeg 版本**

{% codeblock lang:shell %}
ffmpeg -version
{% endcodeblock %}

![](http://otkw6sse5.bkt.clouddn.com/FFmpeg-%E7%AE%80%E4%BB%8B1526045783703_2.png)

---