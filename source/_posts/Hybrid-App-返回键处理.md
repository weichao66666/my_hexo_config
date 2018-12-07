---
layout: article
title: Hybrid App 返回键处理
date: 2018-01-23 23:44:55
tags:
categories: 
copyright: true
---

# **逻辑**

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/Hybrid-App-%E8%BF%94%E5%9B%9E%E9%94%AE%E5%A4%84%E7%90%86QQ%E6%88%AA%E5%9B%BE20180124000230.png)

App启动后显示NormalFragment，在NormalFragment中有一个原生的对话框，通过按返回键会对退出App做二次确认，且默认为不退出，同时NormalFragment中有一个按钮，点击后可以跳转到WebWrapperFragment，在WebWrapperFragment中有一个WebView，WebView中有对话框，通过按返回键会对跳转到NormalFragment做二次确认，且默认为不退出。

---

# **NormalFragment**

## **布局文件**

仅包含一个表示当前Fragment为NormalFragment的TextView和一个用于点击后跳转的按钮。

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tv_normal_fragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/normal_fragment" />

    <Button
        android:id="@+id/btn_change2WebWrapperFragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/tv_normal_fragment"
        android:text="@string/change2WebWrapperFragment" />
</RelativeLayout>
{% endcodeblock %}

## **点击按钮跳转到WebWrapperFragment**

在NormalFragment中设置按钮的点击监听。

{% codeblock lang:java %}
mChange2WebWrapperFragmentBtn.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        if (mMainActivity != null) {
            mMainActivity.change2WebWrapperFragment();
        } else {
            Log.e(TAG, "mMainActivity == null");
        }
    }
});
{% endcodeblock %}

在MainActivity中跳转到WebWrapperFragment。

{% codeblock lang:java %}
public void change2WebWrapperFragment() {
    WebWrapperFragment fragment = (WebWrapperFragment) getSupportFragmentManager().findFragmentByTag("WebWrapperFragment");
    if (fragment == null) {
        fragment = WebWrapperFragment.newInstance();
        FragmentUtil.addFragment(getSupportFragmentManager(), R.id.fragment, mFragment, fragment, "WebWrapperFragment");
    } else {
        if (mFragment != fragment) {
            FragmentUtil.showFragment(getSupportFragmentManager(), mFragment, fragment);
        }
    }
}
{% endcodeblock %}

---

# **WebWrapperFragment**

## **布局文件**

仅包含WebView。

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
{% endcodeblock %}

## **加载html文件**

此处以放在assets中的html文件为例。

{% codeblock lang:java %}
WebSettings webSettings = mWebView.getSettings();
// 解决在用户调整手机字体大小/用户调整浏览器字体大小后，布局错乱问题。
webSettings.setTextZoom(100);
// 设置 Java 可调用 JS 方法
webSettings.setJavaScriptEnabled(true);
// 打开本地缓存
webSettings.setDomStorageEnabled(true);

webSettings.setAppCacheEnabled(true);
webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);
// 设置可以访问文件
webSettings.setAllowFileAccess(true);
webSettings.setAllowContentAccess(true);
webSettings.setAllowFileAccessFromFileURLs(true);

webSettings.setDatabaseEnabled(true);

mWebView.setWebChromeClient(new WebChromeClientWithFullLog());
mWebView.setWebViewClient(new WebViewClientWithFullLog());
// 设置 JS 可调用 Java 方法
mWebView.addJavascriptInterface(new WebWrapperJsInteration(this), WebWrapperJsInteration.JAVA_INTERFACE);

mWebView.loadUrl("file:///android_asset/demo.html");

mWebView.requestFocus();
{% endcodeblock %}

### **demo.js**

clickBack为Java调用JS的方法，传入参数为str，当str是"web_wrapper"，会触发id是"web_wrapper_back"的点击事件。
back为JS调用Java的方法，传入参数为"web_wrapper"，当该JS文件被html引用时，会触发back方法。

{% codeblock lang:javascript %}
window.clickBack = function (str) {
    if(str != null && typeof str == "string") {
        if(str == "web_wrapper") {
	        document.getElementById('web_wrapper_back').click();
	    }
	}
};
window.javaInterface.back("web_wrapper");
{% endcodeblock %}

### **demo.html**

仅包含表示当前Fragment为WebWrapperFragment的文字和一个用于点击后跳转的按钮。
该html引用了demo.js。
按钮的id是"web_wrapper_back"，当点击后会显示对话框，如果点击对话框中的确认会调用Java的exit方法，如果点击对话框中的取消会调用Java的back方法并传入参数"web_wrapper"，按返回键会默认触发取消。

{% codeblock lang:html %}
<html>
<head>
    <script type="text/javascript" src="demo.js"></script>
    <script type="text/javascript">
        function exitConfirm(){
            window.javaInterface.removeBack("web_wrapper");
            var r = confirm("是否退出？");
            if (r == true) {
    //          if(window.javaInterface && window.javaInterface.resetBack){
                    window.javaInterface.exit();
    //          }
            } else {
    //          if(window.javaInterface && window.javaInterface.back){
                    window.javaInterface.back("web_wrapper");
    //          }
            }
        }
    </script>
</head>
<body>
这是 WebWrapper Fragment 页面<br/>
<input type="button" id="web_wrapper_back" onclick="exitConfirm()" value="后退到 Normal Fragment 页面"/>
</body>
</html>
{% endcodeblock %}

### **WebWrapperJsInteration.java**

添加Java方法供JS调用。

{% codeblock lang:java %}
public static final String JAVA_INTERFACE = "javaInterface";

@JavascriptInterface
public void back(String str) {
    Log.d(TAG, "back(" + str + ")");
    mView.showToast("back(" + str + ")");
    mView.addBackStack(str);
}

@JavascriptInterface
public void removeBack(String str) {
    Log.d(TAG, "removeBack(" + str + ")");
    mView.showToast("removeBack(" + str + ")");
    mView.removeBackStack(str);
}

@JavascriptInterface
public void exit() {
    Log.d(TAG, "exit()");
    mView.showToast("exit()");
    mView.onBackPressed();
}
{% endcodeblock %}

---

# **返回键逻辑实现**

重写MainActivity的onBackPressed方法。

{% codeblock lang:java %}
if (mFragment instanceof NormalFragment) {
    NormalFragment fragment = (NormalFragment) mFragment;
    if (fragment.isDialogShown()) {
        finish();
    } else {
        fragment.showDialog();
    }
} else if (mFragment instanceof WebWrapperFragment) {
    WebWrapperFragment fragment = (WebWrapperFragment) mFragment;
    if (fragment.isBackStackEmpty()) {
        change2NormalFragment();
    } else {
        fragment.goBackStack();
    }
}
{% endcodeblock %}

## **显示NormalFragment的对话框**

当取消时将用于判断对话框是否为显示状态的标志置为false，按返回键会默认触发取消。

{% codeblock lang:java %}
AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
builder.setMessage("是否退出？")
        .setPositiveButton("确定", new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialog, int id) {
                if (mMainActivity != null) {
                    mMainActivity.onBackPressed();
                } else {
                    Log.e(TAG, "mMainActivity == null");
                }
            }
        })
        .setNegativeButton("取消", new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialog, int id) {
                dialog.cancel();
            }
        })
        .setOnCancelListener(new DialogInterface.OnCancelListener() {
            @Override
            public void onCancel(DialogInterface dialog) {
                mDialogShown = false;
            }
        })
        .show();
{% endcodeblock %}

## **WebWrapperFragment中后退栈的维护**

1、WebView加载html时，JS会调用Java的back("web_wrapper")，则此时后退栈添加元素"web_wrapper"。

2、按返回键会先判断后退栈中是否有元素，因为有"web_wrapper"，然后Java会调用JS的clickBack("web_wrapper")，clickBack("web_wrapper")对应于html中按钮的点击事件，即会弹出对话框，同时JS会调用Java的removeBack("web_wrapper")，删除后退栈中的元素"web_wrapper"。

3、再次点击返回键会直接触发对话框的取消，同时JS会调用Java的back("web_wrapper")，则此时后退栈添加元素"web_wrapper"，又回到最初加载时的状态。

---

# **源码地址**

[Hybrid_App_Back_Demo](https://github.com/weichao66666/Hybrid_App_Back_Demo "https://github.com/weichao66666/Hybrid_App_Back_Demo")

---