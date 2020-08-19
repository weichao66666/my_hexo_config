---
layout: article
title: app 保活与拉活
date: 2020-08-19 16:53:04
tags:
categories: 
copyright: true
---

# **Reference**
* [解读Android进程优先级ADJ算法](http://gityuan.com/2018/05/19/android-process-adj/ "http://gityuan.com/2018/05/19/android-process-adj/")
* [2020年了，Android后台保活还有戏吗？看我如何优雅的实现！](https://juejin.im/post/6844904032809517070 "https://juejin.im/post/6844904032809517070")
* [深度剖析APP保活案例](http://gityuan.com/2018/02/24/process-keep-forever/#%E5%9B%9B-%E6%80%BB%E7%BB%93 "http://gityuan.com/2018/02/24/process-keep-forever/#%E5%9B%9B-%E6%80%BB%E7%BB%93")

---

# **源码**
[keep_alive_demo](http://localhost:10086/0/keep_alive_demo.git "http://localhost:10086/0/keep_alive_demo.git")

---

# **进程的优先级与回收机制**
[ProcessList.java](http://androidos.net.cn/android/9.0.0_r8/xref/frameworks/base/services/core/java/com/android/server/am/ProcessList.java "http://androidos.net.cn/android/9.0.0_r8/xref/frameworks/base/services/core/java/com/android/server/am/ProcessList.java")

```java
private final int[] mOomAdj = new int[] {
        FOREGROUND_APP_ADJ, VISIBLE_APP_ADJ, PERCEPTIBLE_APP_ADJ,
        BACKUP_APP_ADJ, CACHED_APP_MIN_ADJ, CACHED_APP_MAX_ADJ
};
```

ADJ 级别|取值（Android 7.0 以前的值）|含义
--|--
NATIVE_ADJ|-1000（-17）|native 进程（不被系统管理）
SYSTEM_ADJ|-900（-16）|仅指 system_server 进程
PERSISTENT_PROC_ADJ|-800（-12）|系统 persistent 进程，比如 telephony
PERSISTENT_SERVICE_ADJ|-700（-11）|关联着系统或 persistent 进程
`FOREGROUND_APP_ADJ`|0（0）|前台进程
`VISIBLE_APP_ADJ`|100（1）|可见进程
`PERCEPTIBLE_APP_ADJ`|200（2）|可感知进程，比如后台音乐播放
`BACKUP_APP_ADJ`|300（3）|备份进程
HEAVY_WEIGHT_APP_ADJ|400（4）|后台的重量级进程
SERVICE_ADJ|500（5）|服务进程
HOME_APP_ADJ|600（6）|Home 进程
PREVIOUS_APP_ADJ|700（7）|上一个 app 的进程（往往通过按返回键）
SERVICE_B_ADJ|800（8）|B List 中的 Service
`CACHED_APP_MIN_ADJ`|900（9）|不可见进程的 ADJ 最小值
`CACHED_APP_MAX_ADJ`|906（15）|不可见进程的 ADJ 最大值
UNKNOWN_ADJ|1001（16）|一般指将要会缓存进程，无法获取确定值


## **进程回收机制：Low Memory Killer**
当系统剩余空闲内存低于某阈值（比如 147 MB），则从 ADJ 大于或等于相应阈值（比如 900）的进程中，杀死 ADJ 值最大的进程，如果存在多个 ADJ 相同的进程，则杀死内存最大的进程。

### **查看内存阈值**
```cmd
cat /sys/module/lowmemorykiller/parameters/minfree
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/1.png)
单位是 4KB（一页），所以对于该设备：

* 前台进程：18432 * 4KB / 1024 = 72 MB
* 可见进程：90 MB
* 可感知进程：108 MB
* 备份进程：126 MB
* 不可见进程的 ADJ 最小值：216 MB
* 不可见进程的 ADJ 最大值：315 MB

## **查看进程的优先级**
1、确定进程值，是 3107：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/2.png)

2、查看进程的优先级：
```cmd
cat /proc/3107/oom_adj
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/3.png)
启动 app 后，查看 app 进程的优先级是前台进程；
1、如果按返回键，是 8；
2、如果按 HOME 键，是 6；
3、锁屏不会改变优先级。

## **ADJ 算法**
[ActivityManagerService.java](http://androidos.net.cn/android/9.0.0_r8/xref/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java "http://androidos.net.cn/android/9.0.0_r8/xref/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java")

* updateOomAdjLocked：更新 ADJ，当目标进程为空，或者被杀则返回 false；否则返回 true；
* computeOomAdjLocked：计算 ADJ，返回计算后的 RawAdj 值；
* applyOomAdjLocked：使用 ADJ，当需要杀掉目标进程则返回 false；否则返回 true。

---

# **app 保活**

## **提升进程的优先级**
操作流程：启动 app -> 按 HOME 键回到桌面 -> 锁屏 -> 解锁

默认进程优先级的变化：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/4.png)

### **1 像素法**
使用 1 像素法后进程优先级的变化：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/5.png)

原理：在 Application 里注册广播，监听锁屏和解锁的事件；当收到锁屏广播时启动一个 1 像素的页面，当收到解锁广播时关闭这个页面。
缺点：只在锁屏时有效。

1、注册广播：
```java
public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        KeepAliveManager.getInstance().registerBroadcast(this);
    }
}
```

```java
public class KeepAliveReceiver extends BroadcastReceiver {
    private static final String TAG = "KeepReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        Log.e(TAG, "onReceive：" + action);
        if (TextUtils.equals(action, Intent.ACTION_SCREEN_OFF)) {
            KeepAliveManager.getInstance().startKeepAlive(context);
        } else if (TextUtils.equals(action, Intent.ACTION_SCREEN_ON)) {
            KeepAliveManager.getInstance().finishKeepAlive();
        }
    }
}
```

```java
public class KeepAliveManager {
    private static KeepAliveManager sInstance;

    private KeepAliveReceiver mKeepAliveReceiver;
    private WeakReference<Activity> mKeepAliveActivity;

    private KeepAliveManager() {
    }

    public static KeepAliveManager getInstance() {
        if (sInstance == null) {
            synchronized (KeepAliveManager.class) {
                if (sInstance == null) {
                    sInstance = new KeepAliveManager();
                }
            }
        }
        return sInstance;
    }

    public void registerBroadcast(Context context) {
        IntentFilter filter = new IntentFilter();

        filter.addAction(Intent.ACTION_SCREEN_ON);
        filter.addAction(Intent.ACTION_SCREEN_OFF);

        mKeepAliveReceiver = new KeepAliveReceiver();
        context.registerReceiver(mKeepAliveReceiver, filter);
    }

    public void unregisterBroadcast(Context context) {
        if (mKeepAliveReceiver != null) {
            context.unregisterReceiver(mKeepAliveReceiver);
        }
    }

    public void startKeepAlive(Context context) {
        Intent intent = new Intent(context, KeepAliveActivity.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
    }

    public void finishKeepAlive() {
        if (mKeepAliveActivity != null) {
            Activity activity = mKeepAliveActivity.get();
            if (activity != null) {
                activity.finish();
            }
            mKeepAliveActivity = null;
        }
    }

    public void setActivity(KeepAliveActivity keep) {
        mKeepAliveActivity = new WeakReference<Activity>(keep);
    }
}
```

2、1 像素的页面：
```java
public class KeepAliveActivity extends Activity {
    private static final String TAG = "KeepAliveActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(TAG, "onCreate");

        Window window = getWindow();
        window.setGravity(Gravity.START | Gravity.TOP);
        WindowManager.LayoutParams params = window.getAttributes();
        params.width = 1;
        params.height = 1;
        params.x = 0;
        params.y = 0;
        window.setAttributes(params);

        KeepAliveManager.getInstance().setActivity(this);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");
    }
}
```

### **前台进程法**
使用前台进程法后进程优先级的变化：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/6.png)

原理：在 Application 里启动前台进程。
缺点：自 Android 8.0 开始会在通知栏显示消息。

```java
public class ForegroundService extends Service {
    private static final String TAG = "ForegroundService";

    private static final int SERVICE_ID = 1;

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "ForegroundService 服务创建了");

        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR2) {
            startForeground(SERVICE_ID, new Notification());
        } else if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
            startForeground(SERVICE_ID, new Notification());
            // 删除通知栏消息
            startService(new Intent(this, InnerService.class));
        } else {
            NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
            NotificationChannel channel = new NotificationChannel("channel", "title", NotificationManager.IMPORTANCE_NONE);
            if (manager != null) {
                manager.createNotificationChannel(channel);
                Notification notification = new NotificationCompat.Builder(this, "channel").build();
                startForeground(SERVICE_ID, notification);
            }
        }
    }

    public static class InnerService extends Service {
        @Override
        public void onCreate() {
            super.onCreate();
            Log.d(TAG, "InnerService 服务创建了");

            startForeground(SERVICE_ID, new Notification());
            stopSelf();
        }

        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }
    }
}
```

## **加入系统白名单**
以华为为例：
```java
public boolean isHuawei() {
    if (Build.BRAND == null) {
        return false;
    } else {
        return Build.BRAND.toLowerCase().equals("huawei") || Build.BRAND.toLowerCase().equals("honor");
    }
}

private void goHuaweiSetting() {
    try {
        showActivity("com.huawei.systemmanager", "com.huawei.systemmanager.startupmgr.ui.StartupNormalAppListActivity");
    } catch (Exception e) {
        showActivity("com.huawei.systemmanager", "com.huawei.systemmanager.optimize.bootstart.BootStartActivity");
    }
}

private void showActivity(String packageName, String activityDir) {
    Intent intent = new Intent();
    intent.setComponent(new ComponentName(packageName, activityDir));
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    startActivity(intent);
}
```

---

# **app 拉活**

## **全家桶**
监听其他大厂的 app 发出的广播进行拉活。

## **账号同步**
原理：系统有自动同步账号数据的机制。
缺点：时间不确定。

添加账号：
```java
public class AccountHelper {
    private static final String TAG = "AccountHelper";

    private static final String ACCOUNT_TYPE = "io.weichao.lib_keep_alive.account";
    private static final String ACCOUNT_NAME = "测试";
    private static final String ACCOUNT_PASSWORD = "111111";
    private static final String ACCOUNT_AUTHORITY = "io.weichao.lib_keep_alive.provider";

    public static void addAccount(Context context) {
        AccountManager accountManager = (AccountManager) context.getSystemService(Context.ACCOUNT_SERVICE);
        // 获得此类型的账户，需要增加权限：
        // <uses-permission
        //        android:name="android.permission.GET_ACCOUNTS"
        //        android:maxSdkVersion="22" />
        Account[] accounts = accountManager.getAccountsByType(ACCOUNT_TYPE);
        if (accounts.length > 0) {
            Log.e(TAG, "账户已存在");
            return;
        }

        Account account = new Account(ACCOUNT_NAME, ACCOUNT_TYPE);
        // 给这个账户类型添加一个账户，需要增加权限：
        // <uses-permission
        //        android:name="android.permission.AUTHENTICATE_ACCOUNTS"
        //        android:maxSdkVersion="22" />
        accountManager.addAccountExplicitly(account, ACCOUNT_PASSWORD, new Bundle());
    }

    public static void autoSync() {
        Account account = new Account(ACCOUNT_NAME, ACCOUNT_TYPE);
        // 以下 3 个需要增加权限：<uses-permission android:name="android.permission.WRITE_SYNC_SETTINGS" />
        // 设置同步
        ContentResolver.setIsSyncable(account, ACCOUNT_AUTHORITY, 1);
        // 自动同步
        ContentResolver.setSyncAutomatically(account, ACCOUNT_AUTHORITY, true);
        // 设置同步周期
        ContentResolver.addPeriodicSync(account, ACCOUNT_AUTHORITY, new Bundle(), 1);
    }
}
```

```java
public class SyncService extends Service {
    private static final String TAG = "SyncService";

    private SyncAdapter mSyncAdapter;

    @Override
    public void onCreate() {
        super.onCreate();
        mSyncAdapter = new SyncAdapter(getApplicationContext(), true);
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mSyncAdapter.getSyncAdapterBinder();
    }

    public static class SyncAdapter extends AbstractThreadedSyncAdapter {
        public SyncAdapter(Context context, boolean autoInitialize) {
            super(context, autoInitialize);
        }

        @Override
        public void onPerformSync(Account account, Bundle extras, String authority, ContentProviderClient provider, SyncResult syncResult) {
            Log.d(TAG, "同步账户数据--do something");
        }
    }
}
```

当用户主动同步或者系统自动同步账号数据时，会进行拉活：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/7.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E4%BF%9D%E6%B4%BB%E4%B8%8E%E6%8B%89%E6%B4%BB/8.png)

## **推送**
接入各个手机厂商的推送。

---