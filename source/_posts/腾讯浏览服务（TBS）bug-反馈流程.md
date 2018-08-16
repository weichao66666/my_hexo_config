---
layout: article
title: 腾讯浏览服务（TBS）bug 反馈流程
date: 2018-08-16 23:03:09
tags:
categories: 
copyright: true
---

# **Reference**

* [腾讯浏览服务](https://x5.tencent.com/ "https://x5.tencent.com/")
* [QQ 浏览器移动产品论坛-X5专区](http://bbs.mb.qq.com/forum-110-1.html "http://bbs.mb.qq.com/forum-110-1.html")

---

# **bug 示例**

快速点击按钮，两次加载同一 url，部分时候会出现：

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B1.png)

---

# **确认 bug 是否由 TBS 导致**

## **确认 bug 是否属于原生**

打开【开发者选项】中的【显示布局边界】。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B2.png)

## **复现 bug**

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8Bdemo480p.gif)

如果可以看见布局边框，说明 bug 属于原生，否则可能是原生自定义控件或前端页面。

## **确认 bug 在系统 WebView 中是否存在**

1、长按 WebView 中的文字部分，如果未出现【选择复制】说明是系统 WebView（第一次启动app肯定是系统 WebView，如果第一次使用时间足够长，会将 TBS 下载到本地，从第二次开始 app 使用的就是 TBS）。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B3.png)

在该场景下尝试复现 bug。

2、长按 WebView 中的文字部分，如果出现【选择复制】说明是 TBS。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B4.png)

在该场景下尝试复现 bug。

---

# **通过论坛反馈 bug**

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B5.png)

## **生成网络日志**

参考官方文档[《合作方网络日志抓取方法（反馈时附带抓取的日志，可优先处理）》](http://bbs.mb.qq.com/thread-1945241-1-1.html "http://bbs.mb.qq.com/thread-1945241-1-1.html")。

1、可以在 Log 层添加加载 http://debugx5.qq.com 的开关。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B6.png)

2、如果本地没有 TBS，加载 http://debugx5.qq.com 时会有相关提示：

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B7.png)

3、点击【进入DebugTbs安装或打开X5内核】，安装线上内核。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B8.png)

4、重启 app，再次加载 http://debugx5.qq.com。

5、点击【日志功能开启】。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B9.png)

6、点击 Log 层的 WebView 返回，再复现 bug。

7、点击【日志功能关闭并上传】。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B10.png)

8、记录 GUID，将 GUID 输入反馈页面的【操作步骤】中。

![](http://otkw6sse5.bkt.clouddn.com/%E8%85%BE%E8%AE%AF%E6%B5%8F%E8%A7%88%E6%9C%8D%E5%8A%A1%EF%BC%88TBS%EF%BC%89bug-%E5%8F%8D%E9%A6%88%E6%B5%81%E7%A8%8B11.png)

---

# **等待客服人员联系**

联系我的时间在晚上 10 点左右。

---

# **bug 修复结果**

bug 解决后在对应贴子下回复修复结果。

---
