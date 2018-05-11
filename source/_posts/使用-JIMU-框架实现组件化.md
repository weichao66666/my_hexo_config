---
layout: article
title: 使用 JIMU 框架实现组件化
date: 2018-04-19 21:44:58
tags:
categories: 
copyright: true
---

# **Reference**

* [Android彻底组件化方案实践](https://www.jianshu.com/p/1b1d77f58e84 "https://www.jianshu.com/p/1b1d77f58e84")
* [全面组件化---DDComponentForAndroid分析(1)](https://www.jianshu.com/p/5746e50ed572 "https://www.jianshu.com/p/5746e50ed572")

---

# **最后实现的效果**

一个模块既可以作为 application 进行单独打包，也可以作为其他模块的 library 进行打包。

---

# **从零开始实现功能**

## **下载 JIMU 项目**

JIMU 项目[下载地址](https://github.com/mqzhangw/JIMU/tree/master "https://github.com/mqzhangw/JIMU/tree/master")，后面需要用到其中的 module。

## **新建项目**

比如 component_demo。

## **新建 module**

比如 component1。

## **按照[ JIMU 使用指南](https://github.com/mqzhangw/JIMU#%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97 "https://github.com/mqzhangw/JIMU#%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97")配置**

### **主项目引用编译脚本**

1、修改`component_demo\gradle.properties`为：

{% codeblock lang:property %}
# Project-wide Gradle settings.

# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.

# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html

# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx1536m

# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true

mainmodulename=app
{% endcodeblock %}

2、修改`component_demo\build.gradle`为：

{% codeblock lang:gradle %}
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.1'
        classpath 'com.luojilab.ddcomponent:build-gradle:1.2.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
{% endcodeblock %}

### **设置组件可独立打包**

新建`component_demo\app\gradle.properties`和`component_demo\component1\gradle.properties`，并写入内容：

{% codeblock lang:property %}
# Project-wide Gradle settings.

# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.

# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html

# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx1536m

# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true

isRunAlone=true
debugComponent=sharecomponent
compileComponent=sharecomponent
{% endcodeblock %}

### **应用组件化编译脚本**

1、修改`component_demo\app\build.gradle`为：

{% codeblock lang:gradle %}
apply plugin: 'com.dd.comgradle'

android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "io.weichao.component_demo"
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:27.1.1'
}

combuild {
    applicationName = 'com.luojilab.reader.runalone.application.ReaderApplication'
    isRegisterCompoAuto = true
}
{% endcodeblock %}

2、修改`component_demo\component1\build.gradle`为：

{% codeblock lang:gradle %}
apply plugin: 'com.dd.comgradle'

android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "io.weichao.component1"
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:27.1.1'
}

combuild {
    applicationName = 'com.luojilab.reader.runalone.application.ReaderApplication'
    isRegisterCompoAuto = true
}
{% endcodeblock %}

### **混淆**

修改`component_demo\app\proguard-rules.pro`和`component_demo\component1\proguard-rules.pro`为：

{% codeblock lang:property %}
-keep interface * {
  <methods>;
}
-keep class com.luojilab.component.componentlib.** {*;}
-keep class com.luojilab.router.** {*;}
-keep class * implements com.luojilab.component.componentlib.router.ISyringe {*;}
-keep class * implements com.luojilab.component.componentlib.applicationlike.IApplicationLike {*;}
{% endcodeblock %}

## **application 式打包（运行 component1）**

### **报错：Project with path ':sharecomponent' could not be found in project ':xx'.**

修改`component_demo\app\gradle.properties`和`component_demo\component1\gradle.properties`为：

{% codeblock lang:property %}
# Project-wide Gradle settings.

# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.

# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html

# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx1536m

# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true

isRunAlone=true
{% endcodeblock %}

### **报错：Cannot read packageName from xx\src\main\runalone\AndroidManifest.xml**

新建`component_demo\component1\src\main\runalone`，在该文件夹中新建`AndroidManifest.xml`并写入：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.weichao.component1">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
{% endcodeblock %}

### **运行成功**

为了便于展示，修改`component_demo\component1\src\main\res\layout\activity_main.xml`为：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="component1" />
</RelativeLayout>
{% endcodeblock %}

## **library 式打包（运行 app）**

【非必须】修改`component_demo\component1\src\main\AndroidManifest.xml`为：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.weichao.component1" />
{% endcodeblock %}


### **app 集成 component1**

修改`component_demo\app\gradle.properties`为：

{% codeblock lang:property %}
# Project-wide Gradle settings.

# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.

# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html

# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx1536m

# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true

isRunAlone=true
debugComponent=component1
compileComponent=component1
{% endcodeblock %}

### **组件数据交互**

#### **新建专用于数据交互的 module**

新建`component_demo\component_service\src\main\java\io\weichao\component_service\Component1Service.java`用于其他 module 和 component1 的交互：

{% codeblock lang:java %}
package io.weichao.component_service;

public interface Component1Service {
    String getComponentStr();
}
{% endcodeblock %}

#### **component1 引入 component_service**

修改`component_demo\component1\build.gradle`为：

{% codeblock lang:gradle %}
apply plugin: 'com.dd.comgradle'

android {
    compileSdkVersion 27
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    compile project(':component_service')
    implementation 'com.android.support:appcompat-v7:27.1.1'
}

combuild {
    applicationName = 'com.luojilab.reader.runalone.application.ReaderApplication'
    isRegisterCompoAuto = true
}
{% endcodeblock %}

#### **component1 实现 Component1Service 接口**

新建`component_demo\component1\src\main\java\io\weichao\component1\Component1ServiceImpl.java`，并写入内容：

{% codeblock lang:java %}
package io.weichao.component1;

import io.weichao.component_service.Component1Service;

public class Component1ServiceImpl implements Component1Service {
    @Override
    public String getComponentStr() {
        return "component1";
    }
}
{% endcodeblock %}

#### **将 Component1ServiceImpl 注册到 Router 中**

1、导入已下载的 JIMU 项目中的 componentlib。

##### **报错：Plugin with id 'com.github.dcendents.android-maven' not found.**

修改`component_demo\componentlib\build.gradle`为：

{% codeblock lang:gradle %}
apply plugin: 'com.android.library'

sourceCompatibility = "1.7"
targetCompatibility = "1.7"


ext {
    bintrayName = 'componentlib'
    artifact = bintrayName
    libraryName = 'component build lib '
    libraryDescription = 'component build lib '
    libraryVersion = "1.3.1"
    licenseName = 'The Apache Software License, Version 2.0'
    licenseUrl = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
    allLicenses = ["Apache-2.0"]
}

android {
    compileSdkVersion 26
    buildToolsVersion "26.0.0"

    defaultConfig {
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile 'com.android.support:appcompat-v7:26.+'
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    testCompile 'junit:junit:4.12'
//    compile project(':router-annotation')
    compile 'com.github.jimu:router-annotation:1.0.1'
    compile 'com.google.code.gson:gson:2.8.2'

}


//apply from: 'https://raw.githubusercontent.com/nuuneoi/JCenter/master/installv1.gradle'
//apply from: 'https://raw.githubusercontent.com/nuuneoi/JCenter/master/bintrayv1.gradle'
{% endcodeblock %}

2、修改`component_demo\component_service\build.gradle`：

{% codeblock lang:gradle %}
apply plugin: 'com.android.library'

android {
    compileSdkVersion 27
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:27.1.1'
    compile project(':componentlib')
}
{% endcodeblock %}

3、新建`component_demo\component1\src\main\java\io\weichao\component1\Component1ApplicationLike.java`，并写入内容：

{% codeblock lang:java %}
package io.weichao.component1;

import com.luojilab.component.componentlib.applicationlike.IApplicationLike;
import com.luojilab.component.componentlib.router.Router;

import io.weichao.component_service.Component1Service;

public class Component1ApplicationLike implements IApplicationLike {
    Router router = Router.getInstance();
    @Override
    public void onCreate() {
        router.addService(Component1Service.class.getSimpleName(), new Component1ServiceImpl());
    }

    @Override
    public void onStop() {
        router.removeService(Component1Service.class.getSimpleName());
    }
}
{% endcodeblock %}

#### **app 调用 Component1**

1、修改`component_demo\app\build.gradle`为：

{% codeblock lang:gradle %}
apply plugin: 'com.dd.comgradle'

android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "io.weichao.component_demo"
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    compile project(':component_service')
    implementation 'com.android.support:appcompat-v7:27.1.1'
}

combuild {
    applicationName = 'io.weichao.component_demo.MainApplication'
    isRegisterCompoAuto = true
}
{% endcodeblock %}

2、修改`component_demo\app\src\main\java\io\weichao\component_demo\MainActivity.java`为：

{% codeblock lang:java %}
package io.weichao.component_demo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.Toast;

import com.luojilab.component.componentlib.router.Router;

import io.weichao.component_service.Component1Service;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Router router = Router.getInstance();
        if (router.getService(Component1Service.class.getSimpleName()) != null) {
            Component1Service service = (Component1Service) router.getService(Component1Service.class.getSimpleName());
            Toast.makeText(this, service.getComponentStr(), Toast.LENGTH_LONG).show();
        }
    }
}
{% endcodeblock %}

### **报错：Library projects cannot set applicationId. applicationId is set to 'xx' in default config.**

修改`component_demo\component1\build.gradle`为：

{% codeblock lang:gradle %}
apply plugin: 'com.dd.comgradle'

android {
    compileSdkVersion 27
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:27.1.1'
}

combuild {
    applicationName = 'com.luojilab.reader.runalone.application.ReaderApplication'
    isRegisterCompoAuto = true
}
{% endcodeblock %}

### **报错：Unable to find source java class: 'xx' because it does not belong to any of the source dirs: 'xx'**

clean 项目。

### **运行成功**

## **UI 跳转**

### **组件添加必要的依赖**

修改`component_demo\component1\build.gradle`为：

{% codeblock lang:gradle %}
apply plugin: 'com.dd.comgradle'

android {
    compileSdkVersion 27
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [host: "component1"]
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    compile project(':component_service')
    implementation 'com.android.support:appcompat-v7:27.1.1'
    annotationProcessor 'com.github.jimu:router-anno-compiler:1.0.1'
}

combuild {
    applicationName = 'io.weichao.component1.MainApplication'
    isRegisterCompoAuto = true
}
{% endcodeblock %}

### **注册组件到 UIRouter 中**

修改`component_demo\component1\src\main\java\io\weichao\component1\Component1ApplicationLike.java`为：

{% codeblock lang:java %}
package io.weichao.component1;

import com.luojilab.component.componentlib.applicationlike.IApplicationLike;
import com.luojilab.component.componentlib.router.Router;
import com.luojilab.component.componentlib.router.ui.UIRouter;

import io.weichao.component_service.Component1Service;

public class Component1ApplicationLike implements IApplicationLike {
    Router router = Router.getInstance();
    UIRouter uiRouter = UIRouter.getInstance();

    @Override
    public void onCreate() {
        router.addService(Component1Service.class.getSimpleName(), new Component1ServiceImpl());
        uiRouter.registerUI("component1");
    }

    @Override
    public void onStop() {
        router.removeService(Component1Service.class.getSimpleName());
        uiRouter.unregisterUI("component1");
    }
}
{% endcodeblock %}

### **目标页面添加注解**


修改`component_demo\component1\src\main\java\io\weichao\component1\MainActivity.java`为：

{% codeblock lang:java %}
package io.weichao.component1;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.text.TextUtils;
import android.widget.TextView;

import com.luojilab.component.componentlib.service.AutowiredService;
import com.luojilab.router.facade.annotation.Autowired;
import com.luojilab.router.facade.annotation.RouteNode;

@RouteNode(path = "/main", desc = "主页面")
public class MainActivity extends AppCompatActivity {
    @Autowired
    String arg;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.component1_activity_main);
        TextView tv = findViewById(R.id.textview);

        AutowiredService.Factory.getSingletonImpl().autowire(this);

        if (!TextUtils.isEmpty(arg)) {
            tv.setText(arg);
        }

        Intent intent = new Intent();
        intent.putExtra("callbackArg", "success");
        setResult(RESULT_OK, intent);
    }
}
{% endcodeblock %}

### **跳转**

1、修改`component_demo\app\src\main\res\layout\activity_main.xml`为：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btn_bundle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Bundle 方式" />

    <Button
        android:id="@+id/btn_uri"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="URI 方式" />

    <Button
        android:id="@+id/btn_callback"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="有 callback 方式" />
</LinearLayout>
{% endcodeblock %}

2、修改`component_demo\app\src\main\java\io\weichao\component_demo\MainActivity.java`为：

{% codeblock lang:java %}
package io.weichao.component_demo;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.text.TextUtils;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import com.luojilab.component.componentlib.router.Router;
import com.luojilab.component.componentlib.router.ui.UIRouter;

import io.weichao.component_service.Component1Service;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button bundleBtn = findViewById(R.id.btn_bundle);
        Button uriBtn = findViewById(R.id.btn_uri);
        Button callbackBtn = findViewById(R.id.btn_callback);
        bundleBtn.setOnClickListener(this);
        uriBtn.setOnClickListener(this);
        callbackBtn.setOnClickListener(this);

        Router router = Router.getInstance();
        if (router.getService(Component1Service.class.getSimpleName()) != null) {
            Component1Service service = (Component1Service) router.getService(Component1Service.class.getSimpleName());
            Toast.makeText(this, service.getComponentStr(), Toast.LENGTH_LONG).show();
        }
    }

    @Override
    public void onClick(View v) {
        String arg = "app arg";

        Bundle bundle = new Bundle();
        bundle.putString("arg", arg);

        final String URI_LEGAL = "DDComp://component1/main?arg=" + arg;

        switch (v.getId()) {
            case R.id.btn_bundle:
                UIRouter.getInstance().openUri(this, "DDComp://component1/main", bundle);
                break;
            case R.id.btn_uri:
                UIRouter.getInstance().openUri(this, URI_LEGAL, null);
                break;
            case R.id.btn_callback:
                UIRouter.getInstance().openUri(this, URI_LEGAL, null, 7777);
                break;
        }
    }

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (data != null) {
            if (requestCode == 7777 && resultCode == RESULT_OK) {
                String callbackArg = data.getStringExtra("callbackArg");
                if (!TextUtils.isEmpty(callbackArg)) {
                    Toast.makeText(this, callbackArg, Toast.LENGTH_LONG).show();
                }
            }
        }
    }
}
{% endcodeblock %}

#### **报错：android.content.ActivityNotFoundException: Unable to find explicit activity class {xxActivity}; have you declared this activity in your AndroidManifest.xml?**

修改`component_demo\component1\src\main\AndroidManifest.xml`为：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.weichao.component1">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity" />
    </application>
</manifest>
{% endcodeblock %}

#### **报错：java.lang.NoSuchFieldError: No field xx in class Lxx/R$id; or its superclasses (declaration of 'xx.R$id' appears in /data/app/xx/base.apk)**

重命名`component_demo\component1\src\main\res\layout\activity_main.xml`为`component_demo\component1\src\main\res\layout\component1_activity_main.xml`。

修改`component_demo\component1\src\main\res\layout\component1_activity_main.xml`为：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="component1" />
</RelativeLayout>
{% endcodeblock %}

### **运行成功**

---

