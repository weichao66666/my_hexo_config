---
layout: article
title: Pixel XL 刷机
date: 2017-10-11 20:49:25
tags: 
categories: 
copyright: true
---

# **Reference**

* [从 Bootloader 解锁到必备应用推荐：我的 Google Pixel 折腾手记](https://sspai.com/post/38319 "https://sspai.com/post/38319")

---

# **手动刷入工厂镜像**

## **操作步骤**

### **下载文件**

1. 	从[工厂镜像官网](https://developers.google.cn/android/images "https://developers.google.cn/android/images")下载对应版本的 zip 文件

	![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Pixel-XL-%E5%88%B7%E6%9C%BA_1.png)

### **刷机**

1. 解压 zip 文件

	比如将 zip 文件解压到 f 盘根目录。

	![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Pixel-XL-%E5%88%B7%E6%9C%BA_2.png)

1. 修改文件，用于刷机时保留系统数据

	使用文本编辑器修改 `flash-all.bat` 中的 `fastboot -w update image-marlin-opr3.170623.008.zip` 为 `fastboot update image-marlin-opr3.170623.008.zip`，即去掉 `-w`。

1. 通过 USB 线连接手机和 PC

1. 让手机进入 fastboot 模式

	在命令行中执行`adb reboot bootloader`。

1. 进入 `flash-all.bat` 所在目录

	![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Pixel-XL-%E5%88%B7%E6%9C%BA_3.png)

1. 执行 `flash-all.bat`，等待刷机完成

	![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Pixel-XL-%E5%88%B7%E6%9C%BA_4.png)

	重启后进入`设置`->`系统`可以看到系统版本。

	![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Pixel-XL-%E5%88%B7%E6%9C%BA_5.png)

---