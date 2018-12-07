---
layout: article
title: 使用 Service 解耦 Activity
date: 2017-11-15 21:23:49
tags: 
categories: 
copyright: true
---

# **需求**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/%E4%BD%BF%E7%94%A8-Service-%E8%A7%A3%E8%80%A6-Activity_1.png)

# **高耦合 Activity 搞定需求**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/%E4%BD%BF%E7%94%A8-Service-%E8%A7%A3%E8%80%A6-Activity_2.png)

# **解耦 Activity 搞定需求**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/%E4%BD%BF%E7%94%A8-Service-%E8%A7%A3%E8%80%A6-Activity_3.png)

## **问题**

在第一个 Activity（A）中启动第二个 Activity（B）后，未调用 finish()，所以在启动 B 后，A、B 是共存的，B 盖在 A 上面，用户可见的是 B。当用户点击【上传】后，需立刻显示 A，并进行上传，此时有几种选择：

1、直接杀掉 B，显示 A。该方式上传不能进行。

2、将图片通过 intent 或内置存储传递，在 A 上传。该方式受 intent 传递大小小于 1 MB 限制，通过内置存储传递写入、再读取的时间损失比较大。

3、等待上传完成，再杀掉 B。该方式不能立刻显示 A。

4、直接杀掉 A，在 B 中启动第一个 Activity（A'），等待上传完成，再杀掉 B，上传结果通过 BroadcastReceiver 告诉 A'。该方式可行，但是第一个 Activity 需要再初始化一次，且上传过程不易控（因为上传是在 B，现在切换到 A' 了，而 A' 和 B 交互比较麻烦）。

5、使用 Service。

## **使用 Service 解决问题**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/%E4%BD%BF%E7%94%A8-Service-%E8%A7%A3%E8%80%A6-Activity_4.png)

A 绑定 Service，并设置上传成功和失败的回调，当用户点击【上传】后，B 绑定 Service，将图片通过 Service 上传，并直接关闭 B。

1、用于上传的 Service（以存到内置存储中为例）

{% codeblock lang:java %}
public class MyService extends Service {
    private static String TAG = "MyService";

    private MyBinder mMyBinder = new MyBinder();

    @Override
    public IBinder onBind(Intent intent) {
        return mMyBinder;
    }

    public class MyBinder extends Binder {
        private MyListener mMyListener;

        public void setListener(MyListener myListener) {
            mMyListener = myListener;
        }

        public void submit(final Bitmap bitmap) {
            if (bitmap != null) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        FileOutputStream out = null;
                        try {
                            String filePath = Environment.getExternalStorageDirectory().getAbsolutePath() + File.separator + "temp.jpg";
                            File file = new File(filePath);
                            file.createNewFile();
                            out = new FileOutputStream(filePath);
                            bitmap.compress(Bitmap.CompressFormat.PNG, 100, out);
                            out.flush();
                            if (mMyListener != null) {
                                mMyListener.onSuccess();
                            }
                        } catch (Exception e) {
                            if (mMyListener != null) {
                                mMyListener.onFailed(e.getMessage());
                            }
                        } finally {
                            if (out != null) {
                                try {
                                    out.close();
                                } catch (IOException e) {
                                    e.printStackTrace();
                                }
                            }
                        }
                    }
                }).start();
            }
        }
    }

    public interface MyListener {
        void onSuccess();

        void onFailed(String msg);
    }
}
{% endcodeblock %}

2、A 绑定 Service，并设置上传成功和失败的回调

{% codeblock lang:java %}
private ServiceConnection mServiceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        mMyBinder = (MyService.MyBinder) service;
        mMyBinder.setListener(MainActivity.this);
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        mMyBinder.setListener(null);
    }
};

private void bindService() {
    Intent intent = new Intent(this, MyService.class);
    bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
}

private void unbindService() {
    if (mServiceConnection != null) {
        unbindService(mServiceConnection);
    }
}

@Override
public void onSuccess() {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            Toast.makeText(MainActivity.this, "success!", Toast.LENGTH_SHORT).show();
        }
    });
}

@Override
public void onFailed(final String msg) {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            Toast.makeText(MainActivity.this, "failed!" + msg, Toast.LENGTH_SHORT).show();
        }
    });
}
{% endcodeblock %}

3、当用户点击【上传】后，B 绑定 Service，将图片通过 Service 上传，并直接关闭 B

{% codeblock lang:java %}
private ServiceConnection mServiceConnection;

private void bindService() {
    Intent intent = new Intent(this, MyService.class);
    bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
}

private void unbindService() {
    if (mServiceConnection != null) {
        unbindService(mServiceConnection);
    }
}

public void submit(final Bitmap bitmap) {
    mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mMyBinder = (MyService.MyBinder) service;
            mMyBinder.submit(bitmap);
            CameraActivity.this.finish();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
        }
    };

    bindService();
}
{% endcodeblock %}

---