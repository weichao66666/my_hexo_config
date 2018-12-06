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

![](leanote://file/getImage?fileId=5c08992fab64414bdf001830)

### **安装 SDK**

1、下载并解压缩 zip 包到本地，找到 \flutter\bin\flutter.bat，双击运行

2、在 \flutter\bin 路径下打开 cmd，运行 `flutter --version` 验证是否安装成功

![](leanote://file/getImage?fileId=5c08b8c5ab64417973000338)

### **将 Flutter 加入环境变量**

1、创建 Flutter 环境变量

![](leanote://file/getImage?fileId=5c08bafeab644179730003b2)

2、将 Flutter 环境变量加入 Path

![](leanote://file/getImage?fileId=5c08bafdab644179730003b1)
![](leanote://file/getImage?fileId=5c08ba5cab64417785000382)

3、重启 cmd，在非 \flutter\bin 路径下运行 `flutter --version` 验证是否加入成功

![](leanote://file/getImage?fileId=5c08bbd0ab644179730003d3)

### **查询 Flutter 依赖是否完全已安装**

1、运行 `flutter doctor`，未安装的依赖会有相关提示

![](leanote://file/getImage?fileId=5c08bd2aab64417973000417)

2、根据提示安装依赖、插入设备

![](leanote://file/getImage?fileId=5c08bdc6ab64417973000438)

3、再次查询，则没有问题了

![](leanote://file/getImage?fileId=5c08be3dab64417973000450)

## **Android Studio 关联 Flutter**

### **下载插件**

1、打开下载插件的[网站](https://plugins.jetbrains.com/androidstudio "https://plugins.jetbrains.com/androidstudio")

![](leanote://file/getImage?fileId=5c08941aab644149ed001752)

2、查看 Android Studio 对应的 Intellij 版本

![](leanote://file/getImage?fileId=5c08946bab64414bdf001736)
![](leanote://file/getImage?fileId=5c08946bab64414bdf001735)

3、搜索 Flutter，下载匹配的版本，下载下来的是个 zip 包

![](leanote://file/getImage?fileId=5c08952eab64414bdf001754)

4、因为 Flutter 依赖 Dart，所以参照 Flutter， 一并下载了

![](leanote://file/getImage?fileId=5c0895f7ab64414bdf00177f)

### **安装插件**

![](leanote://file/getImage?fileId=5c0891cdab644149ed0016cb)
![](leanote://file/getImage?fileId=5c0896bdab644149ed0017db)

选择下载好的 Dart 和 Flutter 的 zip 包并安装，重新启动 Android Studio。

### **验证关联成功**

Android Studio 可以创建 Flutter 项目。

![](leanote://file/getImage?fileId=5c08bf3cab64417785000479)

---

# **运行 demo**

1、新建 Flutter 项目

2、运行

![](leanote://file/getImage?fileId=5c08e07eab64417785000b38)

---