---
layout: article
title: app 换肤
date: 2020-08-13 09:22:52
tags:
categories: 
copyright: true
---

# **Reference**
* [Android资源篇1：资源编译与打包](https://www.jianshu.com/p/d487f0aa9818 "https://www.jianshu.com/p/d487f0aa9818")
* [Android进阶——从源文件到APK背后的所有主要流程小结](https://blog.csdn.net/CrazyMo_/article/details/94741088 "https://blog.csdn.net/CrazyMo_/article/details/94741088")
* [Android资源查找分析](https://www.jianshu.com/p/ef64106ca1d3 "https://www.jianshu.com/p/ef64106ca1d3")

---

# **需求**
1. 替换 res 目录中的资源；
2. 不重启；
3. 无闪烁。

---

# **换肤原理**
1、设置 Activity 的 LayoutInflater 为自定义的 Factory2，这样当调用 Activity#setContentView() 方法时会接管创建 View 的操作；
2、在创建 View 成功后，查看 View 是否包含在换肤范围内的属性，如果有，就将 View 存起来；
3、当执行换肤时，会先下载皮肤包，然后将皮肤包中的资源合并到宿主 app 的资源中，形成新的资源包；遍历（2）中保存的 View，将资源替换为新的资源包中的资源。

源码：[skin_demo](http://colors.black:10086/0/skin_demo "http://colors.black:10086/0/skin_demo")

---

# **资源**

## **分类**

### **assets**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E6%8D%A2%E8%82%A4/1.png)

### **res**
* animator：属性动画 XML 文件
* anim：视图动画 XML 文件
* color：颜色 XML 文件
* drawable：drawable XML 文件、Bitmap 文件（.png、.9.png、.jpg、.gif 等）
* layout：布局 XML 文件
* menu：菜单 XML 文件
* raw：原始资源文件
* values：字符串、颜色、尺寸、样式、属性等 XML 文件
* xml：XML 文件

## **资源打包流程**
资源打包是由 AAPT 工具完成，assert、res/raw、Bitmap 文件会被原封不动地打包到 APK 里，XML 会被编译成二进制文件，最终生成 R.java 和 resources.arsc。

详细流程：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E6%8D%A2%E8%82%A4/2.png)

R.java 存储资源 ID（系统资源以 0x01 开头，应用资源以 0x7f 开头）：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E6%8D%A2%E8%82%A4/3.png)

resources.arsc 存储资源 ID 和文件的映射关系：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E6%8D%A2%E8%82%A4/4.png)

## **布局 XML 解析流程**

### **在 Activity 中加载布局 XML 文件**
```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

### **Activity：直接交给 Window 处理**
此处的 Window 是 PhoneWindow。
```
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    ...
}
```

### **PhoneWindow：创建 DecorView**
创建 DecorView 作为 Window 的顶层 View，然后交给 LayoutInflater 处理，同时 DecorView 作为 parent。
```java
public void setContentView(int layoutResID) {
    if (mContentParent == null) {
        installDecor();
    }
    ...
    mLayoutInflater.inflate(layoutResID, mContentParent);
    ...
}
```

### **LayoutInflater 处理**
1、获取 Context 绑定的 Resources；
2、创建 XML 资源解析器，解析 XML；
3、通过 createViewFromTag() 方法创建 View。

```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
    final Resources res = getContext().getResources();
    final XmlResourceParser parser = res.getLayout(resource);
    try {
        return inflate(parser, root, attachToRoot);
    } finally {
        parser.close();
    }
}

public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
    ...
    View result = root;
    ...
    final String name = parser.getName();
    ...
    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

    ViewGroup.LayoutParams params = null;

    if (root != null) {
        params = root.generateLayoutParams(attrs);
        if (!attachToRoot) {
            temp.setLayoutParams(params);
        }
    }

    rInflateChildren(parser, temp, attrs, true);

    if (root != null && attachToRoot) {
        root.addView(temp, params);
    }

    if (root == null || !attachToRoot) {
        result = temp;
    }

    return result;
}

View createViewFromTag(View parent, String name, Context context, AttributeSet attrs, boolean ignoreThemeAttr) {
    ...
    View view;
    // view 创建顺序是 mFactory2、mFactory、mPrivateFactory、onCreateView、createView
    if (mFactory2 != null) {
        view = mFactory2.onCreateView(parent, name, context, attrs);
    } else if (mFactory != null) {
        view = mFactory.onCreateView(name, context, attrs);
    } else {
        view = null;
    }

    if (view == null && mPrivateFactory != null) {
        view = mPrivateFactory.onCreateView(parent, name, context, attrs);
    }

    if (view == null) {
        final Object lastContext = mConstructorArgs[0];
        mConstructorArgs[0] = context;
        try {
            if (-1 == name.indexOf('.')) {
                view = onCreateView(parent, name, attrs);
            } else {
                view = createView(name, null, attrs);
            }
        } finally {
            mConstructorArgs[0] = lastContext;
        }
    }

    return view;
}
```

所以自定义 Factory2 就可以实现抢在系统之前创建 View。

## **Resources 初始化流程**
在 app 启动流程中，当 zygote fork 出 app 进程后，会在 app 进程执行 ActivityThread#main() 方法。

### **ActivityThread：除了启动主线程 Looper，还创建了 ContextImpl 对象**
```java
public static void main(String[] args) {
    ...
    
    Looper.prepareMainLooper();
    
    ...
    
    ActivityThread thread = new ActivityThread();
    thread.attach(false, startSeq);

    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    ...
    
    Looper.loop();
}

private void attach(boolean system, long startSeq) {
    sCurrentActivityThread = this;
    mSystemThread = system;
    
    ...
    
    mInstrumentation = new Instrumentation();
    mInstrumentation.basicInit(this);
    ContextImpl context = ContextImpl.createAppContext(this, getSystemContext().mPackageInfo);
    mInitialApplication = context.mPackageInfo.makeApplication(true, null);
    mInitialApplication.onCreate();
    
    ...
}
```

### **ContextImpl：通过 LoadedApk 获取 Resources，并绑定**
```java
static ContextImpl createAppContext(ActivityThread mainThread, LoadedApk packageInfo) {
    ...
    
    ContextImpl context = new ContextImpl(null, mainThread, packageInfo, null, null, null, 0, null);
    context.setResources(packageInfo.getResources());
    return context;
}
```

### **LoadedApk：将获取 Resources 的工作转给 ResourcesManager**
```java
public Resources getResources() {
    if (mResources == null) {
        final String[] splitPaths;
        try {
            splitPaths = getSplitPaths(null);
        } catch (NameNotFoundException e) {
            throw new AssertionError("null split not found");
        }

        mResources = ResourcesManager.getInstance().getResources(null, mResDir, splitPaths, mOverlayDirs, mApplicationInfo.sharedLibraryFiles, Display.DEFAULT_DISPLAY, null, getCompatibilityInfo(), getClassLoader());
    }
    return mResources;
}
```

### **ResourcesManager：创建 ResourcesImpl 中的 AssetManager；创建 ResourcesImpl**
```java
public @Nullable Resources getResources(@Nullable IBinder activityToken, @Nullable String resDir, @Nullable String[] splitResDirs, @Nullable String[] overlayDirs, @Nullable String[] libDirs, int displayId, @Nullable Configuration overrideConfig, @NonNull CompatibilityInfo compatInfo, @Nullable ClassLoader classLoader) {
    final ResourcesKey key = new ResourcesKey(resDir, splitResDirs, overlayDirs, libDirs, displayId, overrideConfig != null ? new Configuration(overrideConfig) : null, compatInfo);
    classLoader = classLoader != null ? classLoader : ClassLoader.getSystemClassLoader();
    return getOrCreateResources(activityToken, key, classLoader);
}

private @Nullable Resources getOrCreateResources(@Nullable IBinder activityToken, @NonNull ResourcesKey key, @NonNull ClassLoader classLoader) {
    ...
    
    ResourcesImpl resourcesImpl = createResourcesImpl(key);
    if (resourcesImpl == null) {
        return null;
    }
    
    ...

    final Resources resources;
    if (activityToken != null) {
        resources = getOrCreateResourcesForActivityLocked(activityToken, classLoader, resourcesImpl, key.mCompatInfo);
    } else {
        resources = getOrCreateResourcesLocked(classLoader, resourcesImpl, key.mCompatInfo);
    }
    return resources;
}

private @Nullable ResourcesImpl createResourcesImpl(@NonNull ResourcesKey key) {
    final DisplayAdjustments daj = new DisplayAdjustments(key.mOverrideConfiguration);
    daj.setCompatibilityInfo(key.mCompatInfo);

    final AssetManager assets = createAssetManager(key);
    if (assets == null) {
        return null;
    }

    final DisplayMetrics dm = getDisplayMetrics(key.mDisplayId, daj);
    final Configuration config = generateConfig(key, dm);
    final ResourcesImpl impl = new ResourcesImpl(assets, dm, config, daj);
    return impl;
}

@VisibleForTesting
protected @Nullable AssetManager createAssetManager(@NonNull final ResourcesKey key) {
    final AssetManager.Builder builder = new AssetManager.Builder();

    if (key.mResDir != null) {
        try {
            builder.addApkAssets(loadApkAssets(key.mResDir, false /*sharedLib*/, false /*overlay*/));
        } catch (IOException e) {
            Log.e(TAG, "failed to add asset path " + key.mResDir);
            return null;
        }
    }

    if (key.mSplitResDirs != null) {
        for (final String splitResDir : key.mSplitResDirs) {
            try {
                builder.addApkAssets(loadApkAssets(splitResDir, false /*sharedLib*/, false /*overlay*/));
            } catch (IOException e) {
                Log.e(TAG, "failed to add split asset path " + splitResDir);
                return null;
            }
        }
    }

    if (key.mOverlayDirs != null) {
        for (final String idmapPath : key.mOverlayDirs) {
            try {
                builder.addApkAssets(loadApkAssets(idmapPath, false /*sharedLib*/, true /*overlay*/));
            } catch (IOException e) {
                Log.w(TAG, "failed to add overlay path " + idmapPath);
            }
        }
    }

    if (key.mLibDirs != null) {
        for (final String libDir : key.mLibDirs) {
            if (libDir.endsWith(".apk")) {
                try {
                    builder.addApkAssets(loadApkAssets(libDir, true /*sharedLib*/, false /*overlay*/));
                } catch (IOException e) {
                    Log.w(TAG, "Asset path '" + libDir +
                            "' does not exist or contains no resources.");
                }
            }
        }
    }

    return builder.build();
}
```

### **AssetManager：将系统资源和用户资源合并**
```java
public static class Builder {
    private ArrayList<ApkAssets> mUserApkAssets = new ArrayList<>();

    public Builder addApkAssets(ApkAssets apkAssets) {
        mUserApkAssets.add(apkAssets);
        return this;
    }

    public AssetManager build() {
        final ApkAssets[] systemApkAssets = getSystem().getApkAssets();

        final int totalApkAssetCount = systemApkAssets.length + mUserApkAssets.size();
        final ApkAssets[] apkAssets = new ApkAssets[totalApkAssetCount];

        System.arraycopy(systemApkAssets, 0, apkAssets, 0, systemApkAssets.length);

        final int userApkAssetCount = mUserApkAssets.size();
        for (int i = 0; i < userApkAssetCount; i++) {
            apkAssets[i + systemApkAssets.length] = mUserApkAssets.get(i);
        }

        final AssetManager assetManager = new AssetManager(false /*sentinel*/);
        assetManager.mApkAssets = apkAssets;
        AssetManager.nativeSetApkAssets(assetManager.mObject, apkAssets, false /*invalidateCaches*/);
        return assetManager;
    }
}
```

## **资源查找流程**
以查找字符串为例，依次经过 Context -> ContextImpl-> Resources -> ResourcesImpl -> AssetManager，其中 ContextImpl、ResourcesImpl、AssetManager 在 app 启动时初始化 Resource 时创建。
AssetManager 调用 native 层代码会获取 ResTable 资源表，然后根据机器配置在 ResTable 的 mPackageGroups 数组中查找到一个最合适的值。详细流程参考[Android资源查找分析](https://www.jianshu.com/p/ef64106ca1d3 "https://www.jianshu.com/p/ef64106ca1d3")。
```java
getString(R.string.app_name);
```

### **Context**
```java
@NonNull
public final String getString(@StringRes int resId) {
    return getResources().getString(resId);
}
```

### **ContextImpl**
```java
@Override
public Resources getResources() {
    return mResources;
}
```

### **Resources**
```java
@NonNull
public String getString(@StringRes int id) throws NotFoundException {
    return getText(id).toString();
}

@NonNull public CharSequence getText(@StringRes int id) throws NotFoundException {
    CharSequence res = mResourcesImpl.getAssets().getResourceText(id);
    if (res != null) {
        return res;
    }
    throw new NotFoundException("String resource ID #0x" + Integer.toHexString(id));
}
```

### **ResourcesImpl**
```java
@UnsupportedAppUsage
public AssetManager getAssets() {
    return mAssets;
}
```

### **AssetManager**
```java
@Nullable CharSequence getResourceText(@StringRes int resId) {
    synchronized (this) {
        final TypedValue outValue = mValue;
        if (getResourceValue(resId, 0, outValue, true)) {
            return outValue.coerceToString();
        }
        return null;
    }
}

boolean getResourceValue(@AnyRes int resId, int densityDpi, @NonNull TypedValue outValue, boolean resolveRefs) {
    Preconditions.checkNotNull(outValue, "outValue");
    synchronized (this) {
        ensureValidLocked();
        final int cookie = nativeGetResourceValue(mObject, resId, (short) densityDpi, outValue, resolveRefs);
        if (cookie <= 0) {
            return false;
        }

        outValue.changingConfigurations = ActivityInfo.activityInfoConfigNativeToJava(outValue.changingConfigurations);

        if (outValue.type == TypedValue.TYPE_STRING) {
            outValue.string = mApkAssets[cookie - 1].getStringFromPool(outValue.data);
        }
        return true;
    }
}
```

---

# **效果**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/app-%E6%8D%A2%E8%82%A4/5.gif)

---