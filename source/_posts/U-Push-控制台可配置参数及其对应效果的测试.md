---
layout: article
title: U-Push 控制台可配置参数及其对应效果的测试
date: 2018-08-24 20:52:46
tags:
categories: 
copyright: true
---

# **Reference**

* [MultiFunctionAndroidDemo](https://github.com/umeng/MultiFunctionAndroidDemo "https://github.com/umeng/MultiFunctionAndroidDemo")
* [U-Push集成文档](https://developer.umeng.com/docs/66632/detail/66744#h2--appkey-umeng-message-secret3 "https://developer.umeng.com/docs/66632/detail/66744#h2--appkey-umeng-message-secret3")

---

# **全部默认**

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%951.png)

## **效果**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%952.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%953.jpeg)

---

# **基础内容/展示样式/定制样式**

## **预置在 app 内的样式**

```java
@Override
public Notification getNotification(Context context, UMessage msg) {
    switch (msg.builder_id) {
        case 1:
            Notification.Builder builder = new Notification.Builder(context);
            RemoteViews myNotificationView = new RemoteViews(context.getPackageName(),
                R.layout.notification_view);
            myNotificationView.setTextViewText(R.id.notification_title, msg.title);
            myNotificationView.setTextViewText(R.id.notification_text, msg.text);
            myNotificationView.setImageViewBitmap(R.id.notification_large_icon, getLargeIcon(context, msg));
            myNotificationView.setImageViewResource(R.id.notification_small_icon,
                getSmallIconId(context, msg));
            builder.setContent(myNotificationView)
                .setSmallIcon(getSmallIconId(context, msg))
                .setTicker(msg.ticker)
                .setAutoCancel(true);

            return builder.getNotification();
        default:
            //默认为0，若填写的builder_id并不存在，也使用默认。
            return super.getNotification(context, msg);
    }
}
```

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%954.png)

## **效果**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%955.png)

---

# **基础内容/展示样式/图片**

## **上传图片**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%956.png)

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%957.png)

## **效果**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%958.png)

---

# **基础内容/自定义图标/应用内图标文件**

## **大图标文件**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%959.png)

## **小图标文件**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9510.png)

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9511.png)

## **效果**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9512.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9513.jpeg)

---

# **基础内容/自定义图标/上传图标文件**

## **大图标文件**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9514.png)

## **小图标文件**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9515.png)

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9516.png)

## **效果**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9517.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9518.jpeg)

---

# **基础内容/是否展开/大图**

## **上传图片**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9519.png)

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9520.png)

## **效果**

点击下拉按钮可以显示图片。

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9521.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9522.jpeg)

---

# **目标人群/目标人群/部分用户**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9523.png)

按【标签】过滤时提示“无匹配结果”是因为没有用户添加过标签。

---

# **目标人群/目标人群/独立用户**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9524.png)

---

# **目标人群/目标人群/推送时间/定时推送**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9525.png)

---

# **目标人群/目标人群/推送时间/重复推送**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9526.png)

---

# **后续行为/后续动作/打开连接(URL)**

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9527.png)

## **效果**

显示效果和默认相同，但是点击后启动浏览器访问 URL。

---

# **后续行为/后续动作/打开指定页面**

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9528.png)

## **效果**

显示效果和默认相同，但是点击后打开 app 内的一个 activity，不能是其他 app 的。

---

# **后续行为/后续动作/自定义行为**

## **预置在 app 内的处理**

```java
@Override
public void dealWithCustomAction(Context context, UMessage msg) {
    Toast.makeText(context, msg.custom, Toast.LENGTH_LONG).show();
}
```

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9530.png)

## **效果**

显示效果和默认相同，但是点击后触发回调。

---

# **后续行为/后续动作/自定义参数**

## **消息详情**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9531.png)

## **效果**

显示效果和默认相同，但是点击后在触发 4 个可选【后续动作】中对应的回调的同时，会传递参数。

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9532.png)

---

# **后续行为/系统通道/MIUI、EMUI、Flyme系统设备离线转为系统下发**

*TODO 暂不清楚是干啥的。*

---

# **后续行为/提醒方式**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9533.png)

---

# **后续行为/高级设置/限制发送速度**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9534.png)

---

# **后续行为/高级设置/消息触发器**

为用户加上标签，方便推送时按照标签来筛选。

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E6%95%88%E6%9E%9C%E7%9A%84%E6%B5%8B%E8%AF%9535.png)

---