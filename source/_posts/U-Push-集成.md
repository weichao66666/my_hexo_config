---
layout: article
title: U-Push 集成
date: 2018-08-31 22:06:55
tags:
categories: 
copyright: true
---

# **Reference**

* [U-Push集成文档](https://developer.umeng.com/docs/66632/detail/66744#h1-u91CDu8981u66F4u65B02 "https://developer.umeng.com/docs/66632/detail/66744#h1-u91CDu8981u66F4u65B02")

---

# **在控制台创建 app**

---

# **导入 PushSDK**

1、工程添加仓库：

```gradle
allprojects {
    repositories {
        mavenCentral()
    }
}
```

2、app 添加依赖：

```gradle
dependencies {
    implementation 'com.umeng.sdk:common:1.5.3'     // 友盟推送
    implementation 'com.umeng.sdk:utdid:1.1.5.3'
    implementation 'com.umeng.sdk:push:4.2.0'
}
```

3、如果运行时报错：

> Program type already present: com.alibaba.sdk.android.utils.AMSDevReporter
Message{kind=ERROR, text=Program type already present: com.alibaba.sdk.android.utils.AMSDevReporter, sources=[Unknown source file], tool name=Optional.of(D8)}

说明存在 jar 包冲突，即引用了同一 jar 包的不同版本，解决方法是去掉其中一个 jar 包的引用，比如：

```gradle
dependencies {
    implementation('com.aliyun.ams:alicloud-android-hotfix:3.1.2') {    // 热修复
        // 关闭传递性依赖
        exclude(module: 'alicloud-android-utdid')
        exclude(module: 'alicloud-android-utils')
    }
}
```

---

# **初始化 PushSDK**

修改 Application：

```java
@Override
public void onCreate() {
    super.onCreate();
    initUPush();
}

private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

---

# **统计应用启动数据**

修改 BaseActivity：

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Log.d("life", TAG + " onCreate");
    PushAgent.getInstance(this).onAppStart();
}
```

如果不调用此方法，不仅会导致按照“几天不活跃”条件来推送失效，还将导致广播发送不成功以及设备描述红色等问题发生。

---

# **混淆配置**

```pro
-dontwarn com.umeng.**
-dontwarn com.taobao.**
-dontwarn anet.channel.**
-dontwarn anetwork.channel.**
-dontwarn org.android.**
-dontwarn org.apache.thrift.**
-dontwarn com.xiaomi.**
-dontwarn com.huawei.**
-dontwarn com.meizu.**
```

# **功能集成**

## **设置通知点击后的行为**

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    // 自定义行为的回调处理，参考文档：高级功能-通知的展示及提醒-自定义通知打开动作
    // UmengNotificationClickHandler是在BroadcastReceiver中被调用，故如果需启动Activity，需添加Intent.FLAG_ACTIVITY_NEW_TASK
    final UmengNotificationClickHandler umengNotificationClickHandler = new UmengNotificationClickHandler() {
        /**
         * 启动 app
         */
        @Override
        public void launchApp(Context context, UMessage msg) {
            super.launchApp(context, msg);
        }

        /**
         * 打开 URL
         */
        @Override
        public void openUrl(Context context, UMessage msg) {
            super.openUrl(context, msg);
        }

        /**
         * 打开指定页面
         */
        @Override
        public void openActivity(Context context, UMessage msg) {
            super.openActivity(context, msg);
        }

        /**
         * 自定义行为
         */
        @Override
        public void dealWithCustomAction(Context context, UMessage msg) {
            if (Constant.DEBUG) {
                Toast.makeText(context, msg.custom, Toast.LENGTH_LONG).show();
            }
        }
    };
    pushAgent.setNotificationClickHandler(umengNotificationClickHandler);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **设置是否展示通知**

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    final UmengMessageHandler umengMessageHandler = new UmengMessageHandler() {
        /**
         * 通知的回调方法，通知送达时触发
         */
        @Override
        public void dealWithNotificationMessage(Context context, UMessage msg) {
            // 调用super，会展示通知；不调用super，则不展示通知。
            super.dealWithNotificationMessage(context, msg);
        }
    };
    pushAgent.setMessageHandler(umengMessageHandler);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **设置通知免打扰时间段**

默认是 23:00~7:00。

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.setNoDisturbMode(23, 0, 7, 0); // 关闭免打扰：pushAgent.setNoDisturbMode(0, 0, 0, 0);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **设置通知冷却时间**

默认是 60 秒。

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.setMuteDurationSeconds(10);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **设置合并通知数**

可以设置为 0~10 之间任意整数，默认是 1。

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.setDisplayNotificationNumber(2); // 参数为 0 时，表示不合并通知。
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、效果

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%901.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%902.jpeg)

## **自定义通知栏图标**

* 如果在发送后台没有指定通知栏图标，SDK 将使用本地的默认图标，其中，大图标默认使用 drawable 下的 umeng_push_notification_default_large_icon，小图标默认使用 drawable 下的 umeng_push_notification_default_small_icon。
* ~~若开发者没有设置这两个图标，则默认使用应用的图标（&lt;application android:icon="@mipmap/ic_launcher"&gt;&lt;/application&gt;）。~~
* ~~若开发者在发送后台指定了图片的链接，则该链接将通过 Message 的 img 字段进行传输。当 SDK 接受到消息后，会先下载图片，下载成功后显示通知；若下载失败，会自动重试两次（总共下载 3 次）；若仍失败，则采用默认图片进行显示。~~
* ~~如果使用 API 方式来发送消息, 可以指定 icon 和 largeIcon 字段，分别对应状态栏图标和通知栏下拉之后左侧大图标 id（[R.drawable].icon）, 请填写对应图片文件名，不包含扩展名。~~
* MIUI 暂时无法支持自定义通知栏图标，若开发者有需求，也可通过自定义通知栏样式来解决。

* 小图标 smallIcon：要求为 48*48 像素，图片各边至少留 1 个像素的透明，图标主体使用颜色，背景均使用透明。
* 大图标 largeIcon：要求为 64*64 像素。
* 对于超高分辨率的机型的适配，可以适当提高图标的尺寸。

## **自定义通知栏样式**

1、新建布局 notification.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="64dp"
    android:background="@drawable/bg_new_word_list_recyclerview">

    <ImageView
        android:id="@+id/notification_large_icon"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:scaleType="fitXY"
        android:src="@mipmap/ic_launcher" />

    <TextView
        android:id="@+id/notification_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="10dp"
        android:layout_marginTop="5dp"
        android:layout_toEndOf="@+id/notification_large_icon"
        android:text="Title"
        android:textColor="@color/black" />

    <TextView
        android:id="@+id/notification_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/notification_title"
        android:layout_marginStart="10dp"
        android:layout_toEndOf="@+id/notification_large_icon"
        android:text="Message"
        android:textColor="@color/black" />


    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/notification_text"
        android:layout_marginStart="10dp"
        android:layout_toEndOf="@+id/notification_large_icon"
        android:text="(嘿嘿，偷偷听告诉你，我是自定义的)"
        android:textColor="@color/black"
        android:textSize="12sp" />

    <ImageView
        android:id="@+id/notification_small_icon"
        android:layout_width="24dp"
        android:layout_height="24dp"
        android:layout_alignParentBottom="true"
        android:layout_alignParentEnd="true"
        android:layout_marginBottom="5dp"
        android:layout_marginEnd="5dp"
        android:scaleType="fitXY"
        android:src="@mipmap/ic_launcher" />
</RelativeLayout>
```

2、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    final UmengMessageHandler umengMessageHandler = new UmengMessageHandler() {
        /**
         * 自定义通知栏样式的回调方法，通知展示到通知栏时触发
         */
        @Override
        public Notification getNotification(Context context, UMessage msg) {
            switch (msg.builder_id) {
                case 1:
                    final Notification.Builder builder;
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                        final NotificationChannel channel = new NotificationChannel("channel_id", "channel_name", NotificationManager.IMPORTANCE_HIGH);
                        final NotificationManager manager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
                        if (manager != null) {
                            manager.createNotificationChannel(channel);
                        }
                        builder = new Notification.Builder(context, "channel_id");
                    } else {
                        builder = new Notification.Builder(context);
                    }
                    final RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.notification);
                    remoteViews.setTextViewText(R.id.notification_title, msg.title);
                    remoteViews.setTextViewText(R.id.notification_text, msg.text);
                    remoteViews.setImageViewBitmap(R.id.notification_large_icon, getLargeIcon(context, msg));
                    remoteViews.setImageViewResource(R.id.notification_small_icon, getSmallIconId(context, msg));
                    return builder.setContent(remoteViews)
                            .setSmallIcon(getSmallIconId(context, msg))
                            .setTicker(msg.ticker)
                            .setAutoCancel(true)
                            .build();
                default:
                    // 默认为0，若填写的builder_id并不存在，也使用默认。
                    return super.getNotification(context, msg);
            }
        }
    };
    pushAgent.setMessageHandler(umengMessageHandler);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

3、效果

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%903.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%904.jpeg)

## **设置通知到达时响铃、震动、呼吸灯策略**

* MsgConstant.NOTIFICATION_PLAY_SERVER：通过服务端推送状态来设置客户端响铃、震动、呼吸灯的状态。
* MsgConstant.NOTIFICATION_PLAY_SDK_ENABLE：不关心服务端推送状态，客户端都会响铃、震动、呼吸灯亮。
* MsgConstant.NOTIFICATION_PLAY_SDK_DISABLE：不关心服务端推送状态，客户端不会响铃、震动、呼吸灯亮。

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.setNotificationPlaySound(MsgConstant.NOTIFICATION_PLAY_SERVER);       // 声音
    pushAgent.setNotificationPlayLights(MsgConstant.NOTIFICATION_PLAY_SDK_ENABLE);  // 呼吸灯
    pushAgent.setNotificationPlayVibrate(MsgConstant.NOTIFICATION_PLAY_SDK_DISABLE);// 振动
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **自定义通知栏声音**

*TODO 适配 8.0+。*

* 如果在发送后台没有指定通知栏的声音，SDK 将采用本地默认的声音，即 res/raw/umeng_push_notification_default_sound。若无此文件，则默认使用系统的 Notification 声音。
* 若需要在线配置声音，则需先将与配置的声音文件放置在 res/raw 下，然后自发送后台指定声音的 id，即 R.raw.[sound] 里的 sound。

## **设置应用在前台时是否显示通知**

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.setNotificaitonOnForeground(false);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **自定义消息**

自定义消息的内容存放在 UMessage.custom 字段里。

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    final UmengMessageHandler umengMessageHandler = new UmengMessageHandler() {
        /**
         * 自定义消息的回调方法，通知被点击时触发
         */
        @Override
        public void dealWithCustomMessage(final Context context, final UMessage msg) {
            Log.i(TAG, "dealWithCustomMessage()");
            if (Constant.DEBUG) {
                Toast.makeText(context, msg.custom, Toast.LENGTH_LONG).show();
            }
        }
    };
    pushAgent.setMessageHandler(umengMessageHandler);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **设置用户标签（Tag）**

方便推送时按标签过滤。

### **添加标签**

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.getTagManager().addTags(new TagManager.TCallBack() {
        @Override
        public void onMessage(final boolean isSuccess, final ITagManager.Result result) {
            Log.i(TAG, "isSuccess: " + isSuccess + ";result: " + result.toString());
        }
    }, "movie", "sport");
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%905.png)

3、添加完成后，如果之前没有用户添加过该标签，现在就可以从控制台可以看到该标签并可以按该标签推送了：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%906.png)

### **获取服务器端的所有标签**

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.getTagManager().getTags(new TagManager.TagListCallBack() {
        @Override
        public void onMessage(boolean isSuccess, List<String> result) {
            Log.i(TAG, result.toString());
        }
    });
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%907.png)

### **删除标签**

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.getTagManager().deleteTags(new TagManager.TCallBack() {
        @Override
        public void onMessage(final boolean isSuccess, final ITagManager.Result result) {
            Log.i(TAG, "isSuccess: " + isSuccess + ";result: " + result.toString());
        }
    }, "movie");
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%908.png)

## **设置用户加权标签（Weighted Tag）**

*TODO 该功能尚未开通。*

### **添加加权标签**

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    final Hashtable<String, Integer> hashtable = new Hashtable<>();
    hashtable.put("财经", 5);
    pushAgent.getTagManager().addWeightedTags(new TagManager.TCallBack() {
        @Override
        public void onMessage(final boolean isSuccess, final ITagManager.Result result) {
            Log.i(TAG, "isSuccess: " + isSuccess + ";result: " + result.toString());
        }
    }, hashtable);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%909.png)

### **获取服务器端的所有加权标签**

### **删除加权标签**

## **设置用户别名（Alias）**

如果你的应用有自有的用户 id 体系，可以 在SDK 中通过 Alias 字段上传自有用户 id，按用户 id 向用户推送消息。
方便推送时按别名过滤（单点推送）。

### **添加用户别名**

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.addAlias("659xxx", "ALIAS_TYPE_DEMO", new UTrack.ICallBack() {
        @Override
        public void onMessage(boolean isSuccess, String message) {
            Log.i(TAG, "isSuccess: " + isSuccess + ";message: " + message);
        }
    });
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%9010.png)

### **删除用户别名**

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.deleteAlias("659xxx", "ALIAS_TYPE_DEMO", new UTrack.ICallBack() {
        @Override
        public void onMessage(boolean isSuccess, String message) {
            Log.i(TAG, "isSuccess: " + isSuccess + ";message: " + message);
        }
    });
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%9011.png)

## **自定义参数**

### **SDK 回调**

#### **getNotification(Context context, UMessage msg)**

通知展示到通知栏时触发。

1、修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    final UmengMessageHandler umengMessageHandler = new UmengMessageHandler() {
        @Override
        public Notification getNotification(Context context, UMessage msg) {
            Log.i(TAG, "getNotification()");
            if (msg != null && msg.extra != null) {
                for (Map.Entry<String, String> entry : msg.extra.entrySet()) {
                    final String key = entry.getKey();
                    final String value = entry.getValue();
                    Log.i(TAG, "key: " + key + ";value: " + value);
                }
            }
        }
    };
    pushAgent.setMessageHandler(umengMessageHandler);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

2、Logcat 查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%9012.png)

#### **dealWithCustomAction(Context context, UMessage msg)**

通知被点击时触发。

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    final UmengNotificationClickHandler umengNotificationClickHandler = new UmengNotificationClickHandler() {
        @Override
        public void dealWithCustomAction(Context context, UMessage msg) {
            Log.i(TAG, "dealWithCustomAction()");
            if (msg != null && msg.extra != null) {
                for (Map.Entry<String, String> entry : msg.extra.entrySet()) {
                    final String key = entry.getKey();
                    final String value = entry.getValue();
                    Log.i(TAG, "key: " + key + ";value: " + value);
                }
            }
        }
    };
    pushAgent.setNotificationClickHandler(umengNotificationClickHandler);
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

### **Intent 回调**

1、修改 Activity：

```java
@Override
protected void onResume() {
    super.onResume();
    final Bundle bundle = getIntent().getExtras();
    if (bundle != null) {
        final Set<String> keySet = bundle.keySet();
        for (String key : keySet) {
            final String value = bundle.getString(key);
            Log.i(TAG, "key: " + key + ";value: " + value);
        }
    }
}

@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
    setIntent(intent);
}
```

2、修改 Activity 启动方式为 singleTask。

3、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%9013.png)

## **自定义资源包名**

修改 Application：

```java
private void initUPush() {
    UMConfigure.init(this, "5xxxxxxxxxxxxxxxxxxxxxx0", AppUtil.getMetaDataValueFromApplication(this, "UMENG_CHANNEL"), UMConfigure.DEVICE_TYPE_PHONE, "4xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx3");
    final PushAgent pushAgent = PushAgent.getInstance(this);
    pushAgent.setResourcePackageName("com.xx.xx");
    // 注册
    pushAgent.register(new IUmengRegisterCallback() {
        @Override
        public void onSuccess(String deviceToken) {
            Log.i(TAG, "device token: " + deviceToken);
        }

        @Override
        public void onFailure(String s, String s1) {
            Log.i(TAG, "register failed: " + s + " " + s1);
        }
    });
}
```

## **关闭推送**

不能在刚刚注册后开启或关闭推送。

1、修改 Application：

```java
pushAgent.disable(new IUmengCallback() {
    @Override
    public void onSuccess() {
        Log.i(TAG, "onSuccess()");
    }

    @Override
    public void onFailure(String s, String s1) {
        Log.i(TAG, "onFailure(s" + s + ", " + s1 + ")");
    }
});
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%9014.png)

## **开启推送**

不能在刚刚注册后开启或关闭推送。

1、修改 Application：

```java
pushAgent.enable(new IUmengCallback() {
    @Override
    public void onSuccess() {
        Log.i(TAG, "onSuccess()");
    }

    @Override
    public void onFailure(String s, String s1) {
        Log.i(TAG, "onFailure(s" + s + ", " + s1 + ")");
    }
});
```

2、断点查看：

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%9015.png)

## **支持多包名**

若一个 APP 针对不同渠道有不同的包名，则可通过开通多包名支持一个 AppKey 对应多个包名发送消息。
如果同一设备安装同一应用的不同包名的多个安装包，只有最后安装的应用可以正常收到推送消息。

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/U-Push-%E9%9B%86%E6%88%9016.png)

## **完全自定义处理（透传）**

*TODO 未实现。*

使用完全自定义处理后，PushSDK 只负责下发消息体且只统计送达数，展示逻辑需由开发者自己写代码实现，点击数和忽略数需由开发者调用 UTrack 类的 trackMsgClick 和 trackMsgDismissed 方法进行统计。

## **应用内消息**

### **全屏消息**

*TODO 未实现。*

Activity 需要继承 UmengSplashMessageActivity，而不是实现某个接口。

### **插屏消息**

*TODO 未实现。*

### **自定义插屏**

*TODO 未实现。*

### **纯文本**

*TODO 未实现。*

---