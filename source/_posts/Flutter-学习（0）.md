---
layout: article
title: Flutter 学习（0）
date: 2018-12-06 21:50:04
tags:
categories: 
copyright: true
---

# **Reference**

* [Flutter](https://flutter.io/ "https://flutter.io/")
* [Android Studio Plugins](https://plugins.jetbrains.com/androidstudio "https://plugins.jetbrains.com/androidstudio")

---

# **搭建开发环境**

## **安装 Flutter SDK**

### **下载 SDK**

打开下载 SDK 的[网站](https://flutter.io/docs/get-started/install/windows "https://flutter.io/docs/get-started/install/windows")

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%891.png)

### **安装 SDK**

1、下载并解压缩 zip 包到本地，找到 \flutter\bin\flutter.bat，双击运行

2、在 \flutter\bin 路径下打开 cmd，运行 `flutter --version` 验证是否安装成功

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%892.png)

### **将 Flutter 加入环境变量**

1、创建 Flutter 环境变量

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%893.png)

2、将 Flutter 环境变量加入 Path

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%894.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%895.png)

3、重启 cmd，在非 \flutter\bin 路径下运行 `flutter --version` 验证是否加入成功

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%896.png)

### **查询 Flutter 依赖是否完全已安装**

1、运行 `flutter doctor`，未安装的依赖会有相关提示

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%897.png)

2、根据提示安装依赖、插入设备

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%898.png)

3、再次查询，则没有问题了

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%899.png)

## **Android Studio 关联 Flutter**

### **下载插件**

1、打开下载插件的[网站](https://plugins.jetbrains.com/androidstudio "https://plugins.jetbrains.com/androidstudio")

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8910.png)

2、查看 Android Studio 对应的 Intellij 版本

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8911.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8912.png)

3、搜索 Flutter，下载匹配的版本，下载下来的是个 zip 包

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8913.png)

4、因为 Flutter 依赖 Dart，所以参照 Flutter， 一并下载了

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8914.png)

### **安装插件**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8915.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8916.png)

选择下载好的 Dart 和 Flutter 的 zip 包并安装，重新启动 Android Studio。

### **验证关联成功**

Android Studio 可以创建 Flutter 项目。

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8917.png)

---

# **运行 demo**

1、新建 Flutter 项目

2、运行

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Flutter-%E5%AD%A6%E4%B9%A0%EF%BC%880%EF%BC%8918.jpeg)

---