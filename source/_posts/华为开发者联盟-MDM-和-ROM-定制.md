---
layout: article
title: 华为开发者联盟 MDM 和 ROM 定制
date: 2018-08-16 23:12:11
tags:
categories: 
copyright: true
---

# **Reference**

* [安全类授权开放SDK](https://developer.huawei.com/consumer/cn/devservice/doc/10101 "https://developer.huawei.com/consumer/cn/devservice/doc/10101")

---

# **MDM**

## **在官网下载资料**

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B61.png)

下载的 SecurityAuthorizationOpenSDK-2.0.rar 包里面提供了部分资料，但是《目录结构说明.txt》中提到的【华为证书HUAWEI.cer】和【MDMSample.apk】均没有提供。

### **coding 前**

1、开发指导书：
docs/HW安全类授权开放SDK开发指导书V2.0.pdf

2、功能概述：
docs/功能简介V2.0.pdf
docs/HW安全类授权开放SDK功能介绍及展示文档V2.0.pdf

3、demo（该项目直接导入 IDE 后不能正常使用，修改后的版本可[下载](https://github.com/weichao66666/MDMSample "https://github.com/weichao66666/MDMSample")）：
samples/MDMSample

### **coding 中**

1、调用华为接口所需的 jar 包：
libs/hwsdk-mdm-openapi-2.0.0.jar

2、API：
docs/HW安全类授权开放SDK API接口(EMUI4.1-8.0)V2.0.pdf

### **coding 后**

1、APK 签名与华为证书签名要求：
docs/关于签名.pdf

2、华为证书签名工具和使用说明：
tools/DevPack.exe
tools/Devpack使用说明.pdf

## **SDK 集成步骤**

### **添加基本权限**

在 AndroidManifest.xml 中添加：

```xml
<uses-permission android:name="com.huawei.permission.sec.MDM" />
```

### **引入 jar 包**

1、将 hwsdk-mdm-openapi-2.0.0.jar 放入 libs 目录。

2、确保 build.gradle 中有：

```gradle
implementation fileTree(dir: 'libs', include: ['*.jar'])
```

或：

```gradle
implementation files('libs/hwsdk-mdm-openapi-2.0.0.jar')
```

### **调用接口（比如：最基本的静默激活管控）**

1、创建 ComponentName 时可以添加回调：

```java
public class SampleDeviceReceiver extends DeviceAdminReceiver {
    private static final String TAG = "SampleDeviceReceiver";

    @Override
    public void onEnabled(Context context, Intent intent) {
        Log.v(TAG, "onEnabled()");
    }

    @Override
    public void onDisabled(Context context, Intent intent) {
        Log.v(TAG, "onDisabled()");
    }

    @Override
    public CharSequence onDisableRequested(Context context, Intent intent) {
        Log.v(TAG, "onDisableRequested()");
        return "onDisableRequested()";
    }
}
```

2、创建 ComponentName，调用接口静默激活管控：

```java
try {
    final ComponentName componentName = new ComponentName(this, SampleDeviceReceiver.class);
    DevicePolicyManager devicePolicyManager = (DevicePolicyManager) getSystemService(Context.DEVICE_POLICY_SERVICE);
    if (devicePolicyManager != null && !devicePolicyManager.isAdminActive(componentName)) {
        DeviceControlManager deviceControlManager = new DeviceControlManager();
        deviceControlManager.setSilentActiveAdmin(componentName);
    }
} catch (Exception e) {
    e.printStackTrace();
}
```

### **避免混淆**

```pro
-keep class com.huawei.android.app.**{*;}
```

## **跳转到华为桌面**

直接使用：

```java
Intent intent = getPackageManager().getLaunchIntentForPackage("com.huawei.android.launcher");
startActivity(intent);
```

返回的 intent 为 null。

需要直接指定包名和类名：

```java
Intent intent = new Intent();
intent.setClassName("com.huawei.android.launcher", "com.huawei.android.launcher.unihome.UniHomeLauncher");
startActivity(intent);
```

## **打包时签名**

打包时签名只使用 V1 签名，不使用 V2 签名：

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B62.png)

## **华为证书签名**

首先需要一个华为开发者账号，这个账号和应用在华为应用市场上架是同一个。

### **证书申请**

1、在官网找到【权签】：

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B63.png)

2、申请权签服务（第一次申请的是开发证书）：

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B64.png)

3、与代理商签订《API授权证书申请承诺函》并将电子扫描件发送。

该文件需要盖公章、骑缝章，法人授权代码签名。

#### **开发证书申请**

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B65.png)

证书和设备绑定，在同一设备上可以正常使用开发证书签名后的任意版本的同一包名的 app，也就是实现了修改 app 后签名有效。
证书有效期最长半年。
审批时间大概 1 天。

#### **商用证书申请**

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B66.png)

证书和 app 绑定，在任意设备上可以正常使用商用证书签名后的唯一 app，也就是实现了更换设备后签名有效。
证书有效期无限。
审批时间大概 3 天。

### **证书签名**

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B67.png)

---

# **ROM 定制**

## **开机动画前的 Logo**

![](http://otkw6sse5.bkt.clouddn.com/%E5%8D%8E%E4%B8%BA%E5%BC%80%E5%8F%91%E8%80%85%E8%81%94%E7%9B%9F-MDM-%E5%92%8C-ROM-%E5%AE%9A%E5%88%B68.jpeg)

仅支持修改黑色背景图。

## **开机动画**

需要提供 30~50 张 JPG 或 PNG 格式的图片，形成帧动画。
支持单次/循环播放。

## **开机向导界面裁剪**

最少保留：选择语言、协议与条款、应用权限开关说明。

## **预置 app 到 Pad 中**

1、填写《【开发者联盟专用】第三方应用安全自检checklist V2.1》并发送。

2、与代理商签订《预置软件免责协议》并将电子扫描件发送。

该文件需要盖公章、骑缝章，法人授权代码签名。

## **app 开机自启**

## **硬件定制 Logo**

需要找第三方厂商，需要单独收费，如果订货量在 3000 台以上厂商可以做。

---