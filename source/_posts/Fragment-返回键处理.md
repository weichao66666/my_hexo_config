---
layout: article
title: Fragment 返回键处理
date: 2017-12-31 13:39:49
tags: 
categories: 
copyright: true
---

# **逻辑**

![](http://otkw6sse5.bkt.clouddn.com/Fragment-%E8%BF%94%E5%9B%9E%E9%94%AE%E5%A4%84%E7%90%861514695944977_2.png)

启动后进入 FirstFragment，在 FirstFragment 按返回键退出，可以通过点击按钮显示 SecondFragment、ThirdFragment、FourthFragment：

* 进入 SecondFragment 后，按返回键返回 FirstFragment，但是不更新 FirstFragment。
* 进入 ThirdFragment 后，按返回键退出，可以通过点击按钮显示 FourthFragment。
* 进入 FourthFragment 后，按返回键返回 FirstFragment 或 ThirdFragment（取决于从 FirstFragment 或 ThirdFragment 进入），同时更新 FirstFragment 或 ThirdFragment。

---

# **点击按钮显示不同 Fragment**

## **布局文件**

* fragment_first.xml

{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="io.weichao.fragment_demo.activity.MainActivity">

    <TextView
        android:id="@+id/tv_time"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/btn_change2SecondFragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/tv_time"
        android:text="@string/change2SecondFragment" />

    <Button
        android:id="@+id/btn_change2ThirdFragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/btn_change2SecondFragment"
        android:text="@string/change2ThirdFragment" />

    <Button
        android:id="@+id/btn_change2FourthFragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/btn_change2ThirdFragment"
        android:text="@string/change2FourthFragment" />
</RelativeLayout>
{% endcodeblock %}

## **添加点击监听**

* FirstFragment.java

{% codeblock lang:java %}
public class FirstFragment extends Fragment {
    private Button mChange2SecondFragmentBtn;
    private Button mChange2ThirdFragmentBtn;
    private Button mChange2FourthFragmentBtn;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        RelativeLayout rootView = (RelativeLayout) inflater.inflate(R.layout.fragment_first, null);
        mChange2SecondFragmentBtn = rootView.findViewById(R.id.btn_change2SecondFragment);
        mChange2ThirdFragmentBtn = rootView.findViewById(R.id.btn_change2ThirdFragment);
        mChange2FourthFragmentBtn = rootView.findViewById(R.id.btn_change2FourthFragment);
        return rootView;
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        mChange2SecondFragmentBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mMainActivity != null) {
                    mMainActivity.change2SecondFragment();
                } else {
                    Log.e(TAG, "mMainActivity == null");
                }
            }
        });
        mChange2ThirdFragmentBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mMainActivity != null) {
                    mMainActivity.change2ThirdFragment();
                } else {
                    Log.e(TAG, "mMainActivity == null");
                }
            }
        });
        mChange2FourthFragmentBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mMainActivity != null) {
                    mMainActivity.change2FourthFragment();
                } else {
                    Log.e(TAG, "mMainActivity == null");
                }
            }
        });
    }
}
{% endcodeblock %}

## **添加点击响应**

* MainActivity.java

{% codeblock lang:java %}
public class MainActivity extends AppCompatActivity {
    private Fragment mBeforeFragment;
    private Fragment mFragment;

    public void change2FirstFragment() {
        FirstFragment fragment = (FirstFragment) getSupportFragmentManager().findFragmentByTag("FirstFragment");
        if (fragment == null) {
            fragment = FirstFragment.newInstance(this);
            Bundle bundle = new Bundle();
            bundle.putString("time", DateUtil.getCurrentTime());
            fragment.setArguments(bundle);
            FragmentUtil.addFragment(getSupportFragmentManager(), R.id.fragment, mFragment, fragment, "FirstFragment");
        } else {
            if (mFragment != fragment) {
                FragmentUtil.showFragment(getSupportFragmentManager(), mFragment, fragment);
            }
            fragment.update(DateUtil.getCurrentTime());
        }
        mBeforeFragment = null;
        mFragment = fragment;
    }

    public void change2SecondFragment() {
        SecondFragment fragment = (SecondFragment) getSupportFragmentManager().findFragmentByTag("SecondFragment");
        if (fragment == null) {
            fragment = SecondFragment.newInstance(this);
            Bundle bundle = new Bundle();
            bundle.putString("time", DateUtil.getCurrentTime());
            fragment.setArguments(bundle);
            FragmentUtil.addFragment(getSupportFragmentManager(), R.id.fragment, mFragment, fragment, "SecondFragment");
        } else {
            if (mFragment != fragment) {
                FragmentUtil.showFragment(getSupportFragmentManager(), mFragment, fragment);
            }
            fragment.update(DateUtil.getCurrentTime());
        }
        mBeforeFragment = mFragment;
        mFragment = fragment;
    }

    public void change2ThirdFragment() {
        ThirdFragment fragment = (ThirdFragment) getSupportFragmentManager().findFragmentByTag("ThirdFragment");
        if (fragment == null) {
            fragment = ThirdFragment.newInstance(this);
            Bundle bundle = new Bundle();
            bundle.putString("time", DateUtil.getCurrentTime());
            fragment.setArguments(bundle);
            FragmentUtil.addFragment(getSupportFragmentManager(), R.id.fragment, mFragment, fragment, "ThirdFragment");
        } else {
            if (mFragment != fragment) {
                FragmentUtil.showFragment(getSupportFragmentManager(), mFragment, fragment);
            }
            fragment.update(DateUtil.getCurrentTime());
        }
        mBeforeFragment = null;
        mFragment = fragment;
    }

    public void change2FourthFragment() {
        FourthFragment fragment = (FourthFragment) getSupportFragmentManager().findFragmentByTag("FourthFragment");
        if (fragment == null) {
            fragment = FourthFragment.newInstance(this);
            Bundle bundle = new Bundle();
            bundle.putString("time", DateUtil.getCurrentTime());
            fragment.setArguments(bundle);
            FragmentUtil.addFragment(getSupportFragmentManager(), R.id.fragment, mFragment, fragment, "FourthFragment");
        } else {
            if (mFragment != fragment) {
                FragmentUtil.showFragment(getSupportFragmentManager(), mFragment, fragment);
            }
            fragment.update(DateUtil.getCurrentTime());
        }
        mBeforeFragment = mFragment;
        mFragment = fragment;
    }
}
{% endcodeblock %}

# **返回键逻辑实现**

* MainActivity.java

{% codeblock lang:java %}
@Override
public void onBackPressed() {
    if (mBeforeFragment == null) {
        finish();
    } else if (mFragment instanceof SecondFragment) {
        FragmentUtil.showFragment(getSupportFragmentManager(), mFragment, mBeforeFragment);
        mFragment = mBeforeFragment;
        mBeforeFragment = null;
    } else if (mFragment instanceof FourthFragment) {
        if (mBeforeFragment instanceof FirstFragment) {
            change2FirstFragment();
        } else if (mBeforeFragment instanceof ThirdFragment) {
            change2ThirdFragment();
        }
    }
}
{% endcodeblock %}

# **源码地址**

* [Fragment_Demo](https://github.com/weichao66666/Fragment_Demo "https://github.com/weichao66666/Fragment_Demo")

---

















