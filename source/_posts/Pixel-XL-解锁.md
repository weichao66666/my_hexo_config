---
layout: article
title: Pixel XL 解锁
date: 2017-10-10 21:03:07
tags: 
categories: 
copyright: true
---

# **Reference**

* [从 Bootloader 解锁到必备应用推荐：我的 Google Pixel 折腾手记](https://sspai.com/post/38319 "https://sspai.com/post/38319")

---

# **Pixel XL**

![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_1.jpg)

---

# **解锁 BootLoader**

## **必要性**

* 只有解锁了 BootLoader，才能安装第三方的 Recovery。

## **操作步骤**

### **让手机具备翻墙能力**

1. 用 PC 下载 SS 的 Android 版 App

1. 安装 App

	Pixel XL 默认没有安装`文件管理器`等类似的 App，所以不能通过将 apk 文件导入内部存储空间后再使用`文件管理器`安装的方式进行安装。
	
	可将 apk 文件（比如 ss.apk）放到某个位置（比如 d 盘根目录）下，通过在命令行中执行`adb install d:/ss.apk`的方式进行安装。

1. 安装好 App 后，打开并配置服务器，连接网络

	配置完成后状态栏会出现一个钥匙的标志。

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_2.png)
	
### **解锁 OEM**

1. 打开`开发者选项`

	`设置`->`关于手机`，连续多次点击`版本号`。

1. 点击`OEM 解锁`（必须已联网，必须设定了图形锁或密码锁）

### **解锁 BootLoader**

1. 通过 USB 线连接手机和 PC

	如果没弹出调试许可，在命令行中执行`adb shell`。

1. 让手机进入 fastboot 模式

	在命令行中执行`adb reboot bootloader`。

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_3.jpg)

1. 进入 BootLoader 解锁界面

	在命令行中执行`fastboot oem unlock`。

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_4.jpg)

1. 解锁

	使用音量键控制选择`Yes`，使用电源键确定。

	解锁完成后，最后一行是`Device is UNLOCKED`。

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_5.jpg)

---

# **刷第三方 Recovery——TWRP**

## **操作步骤**

### **下载文件**

1. 在 [TWRP 官网](https://twrp.me/ "https://twrp.me/")中找到 [TWRP for marlin](https://eu.dl.twrp.me/marlin/ "https://eu.dl.twrp.me/marlin/")，下载最新版的 zip 文件和 img 文件。

### **刷 TWRP**

1. 将 zip 文件放入手机的内部存储空间中

1. 让手机进入 fastboot 模式

	在命令行中执行`adb reboot bootloader`。

1. 进入临时 TWRP

	可将 img 文件（比如 twrp-3.1.1-1-fastboot-marlin.img）放到某个位置（比如 d 盘根目录）下，在命令行中执行`fastboot boot d:/twrp-3.1.1-1-fastboot-marlin.img`，等待重启。

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_6.jpg)

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_7.jpg)

1. 刷 TWRP

	选择`Install`，选择内部存储空间中的 zip 文件并刷入。

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_8.jpg)

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_9.jpg)

---

# **获取 ROOT 权限**

## **操作步骤**

### **下载文件**

1. 在 [SuperSU 官网](http://www.supersu.com/ "http://www.supersu.com/")的 [Download](http://www.supersu.com/download "http://www.supersu.com/download") 中下载最新版的 zip 文件。

### **刷 SuperSU**

1. 将 zip 文件放入手机的内部存储空间中

1. 让手机进入 TWRP 模式

	在命令行中执行`adb reboot bootloader`，等待重启。

	使用音量键控制选择`Recovery Mode`。

1. 刷 SuperSU

	选择内部存储空间中的 zip 文件并刷入。

	![](http://otkw6sse5.bkt.clouddn.com/Pixel-XL-%E8%A7%A3%E9%94%81_10.jpg)

---
