---
layout: article
title: NAS 搭建环境 & 部署 Git 服务端
date: 2020-07-12 13:58:33
tags:
categories: 
copyright: true
---

# **缘起**
起初购买 NAS 是因为 2018 年老家盖房，我负责弱电部分。除了电话、有线电视、网络、网络电视以外，还有监控系统。

对于监控系统，我原计划是在室外部分采用有线传输的[萤石C5S](https://item.jd.com/4415837.html "https://item.jd.com/4415837.html")及其公有云，在室内部分采用无线传输的[米家智能摄像机](https://item.jd.com/5083558.html "https://item.jd.com/5083558.html")及私有云，私有云由 NAS 实现。由于摄像机部署在室内时采集的数据属于敏感数据，所以私有云存储是很有必要的。

经过调研，最终 NAS 购买了[威廉通TS-451+](https://item.jd.com/2496077.html "https://item.jd.com/2496077.html")，并配置了[红盘](https://item.jd.com/100004198714.html "https://item.jd.com/100004198714.html")作为系统盘、[紫盘](https://item.jd.com/4934516.html "https://item.jd.com/4934516.html")作为摄像头数据专用盘、[APC BK650-CH](https://item.jd.com/1578736992.html "https://item.jd.com/1578736992.html")作为 NAS 后备电源。

后来因为长辈不喜室内监控，所以监控系统的室内部分就放弃了。到了 2019 年，我的房子装修，实现了室内监控，再次翻牌 NAS。

---

# **搭建环境**
由于价格原因，ISP 我放弃了联通，转投电信了。光纤入户后接光猫，由于光猫接口有限，所以接出来一个路由器，现在使用的是[华为路由AX3 Pro](https://item.jd.com/100012605828.html "https://item.jd.com/100012605828.html")，是 WiFi6 协议的。

## **配置内网访问**

### **确定路由器 IP**
登录光猫 192.168.1.1，可以看到接入的设备的 IP。我这里接了两个路由器，其中华为路由器的 IP 是 192.168.1.17。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF1.jpg)

### **确定 NAS IP**
登录路由器 192.168.1.17，可以看到 NAS IP 是 192.168.3.9。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF2.jpg)

同时将路由器 IP 设置为静态 IP。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF3.jpg)

### **登录 NAS**
此时，通过 192.168.3.9 就可以访问 NAS 了，前提是两个设备在同一内网中，也就是连接同一个路由器。

将 NAS IP 设置为静态 IP。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF4.jpg)

## **配置外网访问**

### **查看公网 IP**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF5.jpg)

### **扫描未被占用的端口**
因为 http 的 80 端口和 https 的 443 端口太容易被占用了，所以使用[站长工具](http://tool.chinaz.com/port/ "http://tool.chinaz.com/port/")扫描端口，找到两个未被占用的端口，比如 8888、4443。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF6.jpg)

### **修改 NAS 端口**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF7.jpg)

### **启动 NAS UPnP 转发**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF8.jpg)

启动后可以在路由器中看到同样的信息。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF9.jpg)

### **配置光猫端口映射**
将 8888 和 4443 端口映射给路由器。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF10.jpg)

此时，通过[IP:端口]就可以访问 NAS 了。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF11.jpg)

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF12.jpg)

---

# **部署 Git 服务端**

## **下载**
下载 ContainerStation。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF13.jpg)

下载 GitLab。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF14.jpg)

## **添加 NAS 服务**
我给 GitLab 设置的端口是 10086，所以在 NAS 中添加 NAS 服务。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF15.jpg)

## **配置光猫端口映射**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF16.jpg)

## **2GB RAM 时访问 GitLab**
大概率会报错。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF17.jpg)

这是因为 NAS 自带的 RAM 为 2GB，而 [GitLab 要求的最小 RAM 为 4GB](https://docs.gitlab.com/ee/install/requirements.html#memory "https://docs.gitlab.com/ee/install/requirements.html#memory")，所以又增加了[内存](https://item.jd.com/45678117643.html "https://item.jd.com/45678117643.html")到 8GB。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF18.jpg)

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF19.jpg)

## **8GB RAM 时访问 GitLab**
可以正常使用了。
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF/NAS%20%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83%20%26%20%E9%83%A8%E7%BD%B2%20Git%20%E6%9C%8D%E5%8A%A1%E7%AB%AF20.jpg)

---