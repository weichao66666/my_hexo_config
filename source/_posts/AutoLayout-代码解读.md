---
layout: article
title: AutoLayout 代码解读
date: 2018-02-16 16:09:11
tags:
categories: 
copyright: true
---

# <center>**AutoLayout 代码解读**</center>

---

# **Reference**

* [Android AutoLayout全新的适配方式 堪称适配终结者](http://blog.csdn.net/lmj623565791/article/details/49990941 "http://blog.csdn.net/lmj623565791/article/details/49990941")

---

# **AutoLayoutConifg**

该类的作用是获取屏幕宽高、`AndroidManifest.xml`中的`design_width`和`design_width`字段的值。

{% codeblock lang:java %}
public class AutoLayoutConifg {
    private static AutoLayoutConifg sIntance = new AutoLayoutConifg();

    private static final String KEY_DESIGN_WIDTH = "design_width";
    private static final String KEY_DESIGN_HEIGHT = "design_height";

    private int mScreenWidth;
    private int mScreenHeight;

    private int mDesignWidth;
    private int mDesignHeight;

    private boolean useDeviceSize;

    private AutoLayoutConifg() {
    }

    public static AutoLayoutConifg getInstance() {
        return sIntance;
    }

    public void init(Context context) {
        getMetaData(context);

        int[] screenSize = ScreenUtils.getScreenSize(context, useDeviceSize);
        mScreenWidth = screenSize[0];
        mScreenHeight = screenSize[1];
        L.e("screenWidth = " + mScreenWidth + ", screenHeight = " + mScreenHeight);
    }

    private void getMetaData(Context context) {
        PackageManager packageManager = context.getPackageManager();
        ApplicationInfo applicationInfo;
        try {
            applicationInfo = packageManager.getApplicationInfo(context.getPackageName(), PackageManager.GET_META_DATA);
            if (applicationInfo != null && applicationInfo.metaData != null) {
                mDesignWidth = (int) applicationInfo.metaData.get(KEY_DESIGN_WIDTH);
                mDesignHeight = (int) applicationInfo.metaData.get(KEY_DESIGN_HEIGHT);
            }
        } catch (PackageManager.NameNotFoundException e) {
            throw new RuntimeException("you must set " + KEY_DESIGN_WIDTH + " and " + KEY_DESIGN_HEIGHT + " in your manifest file.", e);
        }
        L.e("designWidth = " + mDesignWidth + ", designHeight = " + mDesignHeight);
    }

    public void checkParams() {
        if (mDesignHeight <= 0 || mDesignWidth <= 0) {
            throw new RuntimeException("you must set " + KEY_DESIGN_WIDTH + " and " + KEY_DESIGN_HEIGHT + " in your manifest file.");
        }
    }

    public AutoLayoutConifg useDeviceSize() {
        useDeviceSize = true;
        return this;
    }

    public int getScreenWidth() {
        return mScreenWidth;
    }

    public int getScreenHeight() {
        return mScreenHeight;
    }

    public int getDesignWidth() {
        return mDesignWidth;
    }

    public int getDesignHeight() {
        return mDesignHeight;
    }
}
{% endcodeblock %}

## **init(Context context)**

获取屏幕宽高。

## **getMetaData(Context context)**

获取`AndroidManifest.xml`中位于`application`节点下的`design_width`和`design_width`字段的值。

---

# **AutoLayoutHelper**

该类的作用是计算属性的值并保存到`AutoLayoutInfo`对象中，在需要的时候将`AutoLayoutInfo`对象中值替换给对应的属性。

{% codeblock lang:java %}
public class AutoLayoutHelper {
    private static final String TAG = "AutoLayoutHelper";

    private final ViewGroup mHost;

    private static final int[] LL = new int[]{
            android.R.attr.textSize,
            android.R.attr.padding,
            android.R.attr.paddingLeft,
            android.R.attr.paddingTop,
            android.R.attr.paddingRight,
            android.R.attr.paddingBottom,
            android.R.attr.layout_width,
            android.R.attr.layout_height,
            android.R.attr.layout_margin,
            android.R.attr.layout_marginLeft,
            android.R.attr.layout_marginTop,
            android.R.attr.layout_marginRight,
            android.R.attr.layout_marginBottom,
            android.R.attr.maxWidth,
            android.R.attr.maxHeight,
            android.R.attr.minWidth,
            android.R.attr.minHeight,
            android.R.attr.layout_marginStart,
            android.R.attr.layout_marginEnd,
            android.R.attr.paddingStart,
            android.R.attr.paddingEnd,
    };

    private static final int INDEX_TEXT_SIZE = 0;
    private static final int INDEX_PADDING = 1;
    private static final int INDEX_PADDING_LEFT = 2;
    private static final int INDEX_PADDING_TOP = 3;
    private static final int INDEX_PADDING_RIGHT = 4;
    private static final int INDEX_PADDING_BOTTOM = 5;
    private static final int INDEX_WIDTH = 6;
    private static final int INDEX_HEIGHT = 7;
    private static final int INDEX_MARGIN = 8;
    private static final int INDEX_MARGIN_LEFT = 9;
    private static final int INDEX_MARGIN_TOP = 10;
    private static final int INDEX_MARGIN_RIGHT = 11;
    private static final int INDEX_MARGIN_BOTTOM = 12;
    private static final int INDEX_MAX_WIDTH = 13;
    private static final int INDEX_MAX_HEIGHT = 14;
    private static final int INDEX_MIN_WIDTH = 15;
    private static final int INDEX_MIN_HEIGHT = 16;
    private static final int INDEX_MARGIN_START = 17;
    private static final int INDEX_MARGIN_END = 18;
    private static final int INDEX_PADDING_START = 19;
    private static final int INDEX_PADDING_END = 20;

    public AutoLayoutHelper(ViewGroup host) {
        mHost = host;

        initAutoLayoutConfig(host);
    }

    private void initAutoLayoutConfig(ViewGroup host) {
        if (host != null) {
            AutoLayoutConifg autoLayoutConifg = AutoLayoutConifg.getInstance();
            autoLayoutConifg.init(host.getContext());
        } else {
            Log.e(TAG, "host == null");
        }
    }

    public void adjustChildren() {
        AutoLayoutConifg.getInstance().checkParams();
        for (int i = 0, n = mHost.getChildCount(); i < n; i++) {
            View view = mHost.getChildAt(i);
            ViewGroup.LayoutParams params = view.getLayoutParams();
            if (params instanceof AutoLayoutParams) {
                AutoLayoutInfo info = ((AutoLayoutParams) params).getAutoLayoutInfo();
                if (info != null) {
                    info.fillAttrs(view);
                }
            }
        }
    }

    public static AutoLayoutInfo getAutoLayoutInfo(Context context, AttributeSet attrs) {
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.AutoLayout_Layout);
        int baseWidth = a.getInt(R.styleable.AutoLayout_Layout_layout_auto_basewidth, 0);
        int baseHeight = a.getInt(R.styleable.AutoLayout_Layout_layout_auto_baseheight, 0);
        a.recycle();

        AutoLayoutInfo info = new AutoLayoutInfo();
        TypedArray array = context.obtainStyledAttributes(attrs, LL);
        for (int i = 0, n = array.getIndexCount(); i < n; i++) {
            int index = array.getIndex(i);

            if (!DimenUtils.isPxVal(array.peekValue(index))) {
                continue;
            }

            int pxVal;
            try {
                pxVal = array.getDimensionPixelOffset(index, 0);
            } catch (Exception e) {
                e.printStackTrace();
                continue;
            }
            switch (index) {
                case INDEX_TEXT_SIZE:
                    info.addAttr(new TextSizeAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_PADDING:
                    info.addAttr(new PaddingAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_PADDING_LEFT:
                case INDEX_PADDING_START:
                    info.addAttr(new PaddingLeftAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_PADDING_TOP:
                    info.addAttr(new PaddingTopAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_PADDING_RIGHT:
                case INDEX_PADDING_END:
                    info.addAttr(new PaddingRightAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_PADDING_BOTTOM:
                    info.addAttr(new PaddingBottomAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_WIDTH:
                    info.addAttr(new WidthAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_HEIGHT:
                    info.addAttr(new HeightAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MARGIN:
                    info.addAttr(new MarginAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MARGIN_LEFT:
                case INDEX_MARGIN_START:
                    info.addAttr(new MarginLeftAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MARGIN_TOP:
                    info.addAttr(new MarginTopAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MARGIN_RIGHT:
                case INDEX_MARGIN_END:
                    info.addAttr(new MarginRightAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MARGIN_BOTTOM:
                    info.addAttr(new MarginBottomAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MAX_WIDTH:
                    info.addAttr(new MaxWidthAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MAX_HEIGHT:
                    info.addAttr(new MaxHeightAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MIN_WIDTH:
                    info.addAttr(new MinWidthAttr(pxVal, baseWidth, baseHeight));
                    break;
                case INDEX_MIN_HEIGHT:
                    info.addAttr(new MinHeightAttr(pxVal, baseWidth, baseHeight));
                    break;
            }
        }
        array.recycle();
        L.e("getAutoLayoutInfo " + info.toString());
        return info;
    }

    public interface AutoLayoutParams {
        AutoLayoutInfo getAutoLayoutInfo();
    }
}
{% endcodeblock %}

## **adjustChildren()**

遍历子 View，如果其布局参数是`AutoLayoutParams`，则将计算后的属性的值替换给对应的属性。

## **getAutoLayoutInfo(Context context, AttributeSet attrs)**

遍历属性，重新计算属性的值后保存到`AutoLayoutInfo`对象中。

---

# **AutoLayoutInfo**

该类的作用是替换属性的值为计算后的值。

{% codeblock lang:java %}
public class AutoLayoutInfo {
    private List<AutoAttr> autoAttrs = new ArrayList<>();

    public void fillAttrs(View view) {
        for (AutoAttr autoAttr : autoAttrs) {
            autoAttr.apply(view);
        }
    }

    public static AutoLayoutInfo getAttrFromView(View view, int attrs, int base) {
        ViewGroup.LayoutParams params = view.getLayoutParams();
        if (params == null) {
            return null;
        }

        AutoLayoutInfo autoLayoutInfo = new AutoLayoutInfo();

        // width, height
        if ((attrs & Attrs.WIDTH) != 0 && params.width > 0) {
            autoLayoutInfo.addAttr(WidthAttr.generate(params.width, base));
        }

        if ((attrs & Attrs.HEIGHT) != 0 && params.height > 0) {
            autoLayoutInfo.addAttr(HeightAttr.generate(params.height, base));
        }

        // margin
        if (params instanceof ViewGroup.MarginLayoutParams) {
            if ((attrs & Attrs.MARGIN) != 0) {
                autoLayoutInfo.addAttr(MarginLeftAttr.generate(((ViewGroup.MarginLayoutParams) params).leftMargin, base));
                autoLayoutInfo.addAttr(MarginTopAttr.generate(((ViewGroup.MarginLayoutParams) params).topMargin, base));
                autoLayoutInfo.addAttr(MarginRightAttr.generate(((ViewGroup.MarginLayoutParams) params).rightMargin, base));
                autoLayoutInfo.addAttr(MarginBottomAttr.generate(((ViewGroup.MarginLayoutParams) params).bottomMargin, base));
            }
            if ((attrs & Attrs.MARGIN_LEFT) != 0) {
                autoLayoutInfo.addAttr(MarginLeftAttr.generate(((ViewGroup.MarginLayoutParams) params).leftMargin, base));
            }
            if ((attrs & Attrs.MARGIN_TOP) != 0) {
                autoLayoutInfo.addAttr(MarginTopAttr.generate(((ViewGroup.MarginLayoutParams) params).topMargin, base));
            }
            if ((attrs & Attrs.MARGIN_RIGHT) != 0) {
                autoLayoutInfo.addAttr(MarginRightAttr.generate(((ViewGroup.MarginLayoutParams) params).rightMargin, base));
            }
            if ((attrs & Attrs.MARGIN_BOTTOM) != 0) {
                autoLayoutInfo.addAttr(MarginBottomAttr.generate(((ViewGroup.MarginLayoutParams) params).bottomMargin, base));
            }
        }

        // padding
        if ((attrs & Attrs.PADDING) != 0) {
            autoLayoutInfo.addAttr(PaddingLeftAttr.generate(view.getPaddingLeft(), base));
            autoLayoutInfo.addAttr(PaddingTopAttr.generate(view.getPaddingTop(), base));
            autoLayoutInfo.addAttr(PaddingRightAttr.generate(view.getPaddingRight(), base));
            autoLayoutInfo.addAttr(PaddingBottomAttr.generate(view.getPaddingBottom(), base));
        }
        if ((attrs & Attrs.PADDING_LEFT) != 0) {
            autoLayoutInfo.addAttr(MarginLeftAttr.generate(view.getPaddingLeft(), base));
        }
        if ((attrs & Attrs.PADDING_TOP) != 0) {
            autoLayoutInfo.addAttr(MarginTopAttr.generate(view.getPaddingTop(), base));
        }
        if ((attrs & Attrs.PADDING_RIGHT) != 0) {
            autoLayoutInfo.addAttr(MarginRightAttr.generate(view.getPaddingRight(), base));
        }
        if ((attrs & Attrs.PADDING_BOTTOM) != 0) {
            autoLayoutInfo.addAttr(MarginBottomAttr.generate(view.getPaddingBottom(), base));
        }

        // minWidth, maxWidth, minHeight, maxHeight
        if ((attrs & Attrs.MIN_WIDTH) != 0) {
            autoLayoutInfo.addAttr(MinWidthAttr.generate(MinWidthAttr.getMinWidth(view), base));
        }
        if ((attrs & Attrs.MAX_WIDTH) != 0) {
            autoLayoutInfo.addAttr(MaxWidthAttr.generate(MaxWidthAttr.getMaxWidth(view), base));
        }
        if ((attrs & Attrs.MIN_HEIGHT) != 0) {
            autoLayoutInfo.addAttr(MinHeightAttr.generate(MinHeightAttr.getMinHeight(view), base));
        }
        if ((attrs & Attrs.MAX_HEIGHT) != 0) {
            autoLayoutInfo.addAttr(MaxHeightAttr.generate(MaxHeightAttr.getMaxHeight(view), base));
        }

        // textSize
        if (view instanceof TextView) {
            if ((attrs & Attrs.TEXTSIZE) != 0) {
                autoLayoutInfo.addAttr(TextSizeAttr.generate((int) ((TextView) view).getTextSize(), base));
            }
        }
        return autoLayoutInfo;
    }

    public void addAttr(AutoAttr autoAttr) {
        autoAttrs.add(autoAttr);
    }

    @Override
    public String toString() {
        return "AutoLayoutInfo { autoAttrs = " + autoAttrs + " }";
    }
}
{% endcodeblock %}

## **getAttrFromView(View view, int attrs, int base)**

遍历属性，重新计算属性的值并保存。

## **fillAttrs(View view)**

遍历属性，将属性应用到给定的 View 上。

---

# **AutoUtils**

{% codeblock lang:java %}
public class AutoUtils {
    /**
     * 会直接将view的LayoutParams上设置的width，height直接进行百分比处理
     *
     * @param view
     */
    public static void auto(View view) {
        autoSize(view);
        autoPadding(view);
        autoMargin(view);
        autoTextSize(view, AutoAttr.BASE_DEFAULT);
    }

    /**
     * @param view
     * @param attrs #Attrs.WIDTH|Attrs.HEIGHT
     * @param base  AutoAttr.BASE_WIDTH|AutoAttr.BASE_HEIGHT|AutoAttr.BASE_DEFAULT
     */
    public static void auto(View view, int attrs, int base) {
        AutoLayoutInfo autoLayoutInfo = AutoLayoutInfo.getAttrFromView(view, attrs, base);
        if (autoLayoutInfo != null) {
            autoLayoutInfo.fillAttrs(view);
        }
    }

    public static void autoTextSize(View view) {
        auto(view, Attrs.TEXTSIZE, AutoAttr.BASE_DEFAULT);
    }

    public static void autoTextSize(View view, int base) {
        auto(view, Attrs.TEXTSIZE, base);
    }

    public static void autoMargin(View view) {
        auto(view, Attrs.MARGIN, AutoAttr.BASE_DEFAULT);
    }

    public static void autoMargin(View view, int base) {
        auto(view, Attrs.MARGIN, base);
    }

    public static void autoPadding(View view) {
        auto(view, Attrs.PADDING, AutoAttr.BASE_DEFAULT);
    }

    public static void autoPadding(View view, int base) {
        auto(view, Attrs.PADDING, base);
    }

    public static void autoSize(View view) {
        auto(view, Attrs.WIDTH | Attrs.HEIGHT, AutoAttr.BASE_DEFAULT);
    }

    public static void autoSize(View view, int base) {
        auto(view, Attrs.WIDTH | Attrs.HEIGHT, base);
    }

    public static boolean autoed(View view) {
        Object tag = view.getTag(R.id.id_tag_autolayout_size);
        if (tag != null) {
            return true;
        }

        view.setTag(R.id.id_tag_autolayout_size, "Just Identify");
        return false;
    }

    public static float getPercentWidth1px() {
        int screenWidth = AutoLayoutConifg.getInstance().getScreenWidth();
        int designWidth = AutoLayoutConifg.getInstance().getDesignWidth();
        return 1.0f * screenWidth / designWidth;
    }

    public static float getPercentHeight1px() {
        int screenHeight = AutoLayoutConifg.getInstance().getScreenHeight();
        int designHeight = AutoLayoutConifg.getInstance().getDesignHeight();
        return 1.0f * screenHeight / designHeight;
    }

    public static int getPercentWidthSize(int val) {
        int screenWidth = AutoLayoutConifg.getInstance().getScreenWidth();
        int designWidth = AutoLayoutConifg.getInstance().getDesignWidth();
        return (int) (val * 1.0f / designWidth * screenWidth);
    }

    public static int getPercentHeightSize(int val) {
        int screenHeight = AutoLayoutConifg.getInstance().getScreenHeight();
        int designHeight = AutoLayoutConifg.getInstance().getDesignHeight();
        return (int) (val * 1.0f / designHeight * screenHeight);
    }

    public static int getPercentWidthSizeBigger(int val) {
        int screenWidth = AutoLayoutConifg.getInstance().getScreenWidth();
        int designWidth = AutoLayoutConifg.getInstance().getDesignWidth();

        int res = val * screenWidth;
        if (res % designWidth == 0) {
            return res / designWidth;
        } else {
            return res / designWidth + 1;
        }
    }

    public static int getPercentHeightSizeBigger(int val) {
        int screenHeight = AutoLayoutConifg.getInstance().getScreenHeight();
        int designHeight = AutoLayoutConifg.getInstance().getDesignHeight();

        int res = val * screenHeight;
        if (res % designHeight == 0) {
            return res / designHeight;
        } else {
            return res / designHeight + 1;
        }
    }
}
{% endcodeblock %}

## **getPercentWidth1px()**

获取设计图中的`1px`在当前设备上的宽度。

## **getPercentHeight1px()**

获取设计图中的`1px`在当前设备上的高度。

## **getPercentWidthSize(int val)**

获取基于宽度计算的设计图中的指定的值在当前设备上的值。

## **getPercentHeightSize(int val)**

获取基于高度计算的设计图中的指定的值在当前设备上的值。

## **getPercentWidthSizeBigger(int val)**

获取基于宽度计算的设计图中的指定的值在当前设备上的值，当为`0`时，修改为`1`。

## **getPercentHeightSizeBigger(int val)**

获取基于高度计算的设计图中的指定的值在当前设备上的值，当为`0`时，修改为`1`。

---

# **AutoAttr**

判断是否未设置`app:layout_auto_basewidth`或`app:layout_auto_baseheight`，如果未设置，则判断该属性是基于宽度还是基于高度计算，并进行相应的计算；如果设置了基于宽度或高度计算，则进行对应的计算。

{% codeblock lang:java %}
public abstract class AutoAttr {
    public static final int BASE_WIDTH = 1;
    public static final int BASE_HEIGHT = 2;
    public static final int BASE_DEFAULT = 3;

    protected int pxVal;
    protected int baseWidth;
    protected int baseHeight;

    public AutoAttr(int pxVal, int baseWidth, int baseHeight) {
        this.pxVal = pxVal;
        this.baseWidth = baseWidth;
        this.baseHeight = baseHeight;
    }

    public void apply(View view) {
        boolean log = view.getTag() != null && view.getTag().toString().equals("auto");
        if (log) {
            L.e("pxVal = " + pxVal + " ," + getClass().getSimpleName());
        }
        int val;
        if (useDefault()) {
            val = defaultBaseWidth() ? getPercentWidthSize() : getPercentHeightSize();
            if (log) {
                L.e("useDefault val = " + val);
            }
        } else if (baseWidth()) {
            val = getPercentWidthSize();
            if (log) {
                L.e("baseWidth val = " + val);
            }
        } else {
            val = getPercentHeightSize();
            if (log) {
                L.e("baseHeight val = " + val);
            }
        }
        if (val > 0) {
            val = Math.max(val, 1);// for very thin divider
        }
        execute(view, val);
    }

    protected boolean useDefault() {
        return !contains(baseHeight, attrVal()) && !contains(baseWidth, attrVal());
    }

    protected boolean baseWidth() {
        return contains(baseWidth, attrVal());
    }

    protected boolean contains(int baseVal, int flag) {
        return (baseVal & flag) != 0;
    }

    protected int getPercentWidthSize() {
        return AutoUtils.getPercentWidthSizeBigger(pxVal);
    }

    protected int getPercentHeightSize() {
        return AutoUtils.getPercentHeightSizeBigger(pxVal);
    }

    @Override
    public String toString() {
        return "AutoAttr{" +
                "pxVal = " + pxVal +
                ", baseWidth = " + baseWidth() +
                ", defaultBaseWidth = " + defaultBaseWidth() +
                '}';
    }

    protected abstract int attrVal();

    protected abstract boolean defaultBaseWidth();

    protected abstract void execute(View view, int val);
}
{% endcodeblock %}

## **attrVal()**

该属性在`attrs.xml`中对应的值。

## **defaultBaseWidth()**

是否默认依赖于宽度进行百分比计算。

## **execute(View view, int val)**

将属性修改后的值替换给对应的属性。

## **apply(View view)**

计算属性的值并将属性修改后的值替换给对应的属性。

### **useDefault()**

判断是否未设置`app:layout_auto_basewidth`或`app:layout_auto_baseheight`。

### **baseWidth()**

判断是否设置了`app:layout_auto_basewidth`。

### **contains(int baseVal, int flag)**

判断是否对应的标志位是否置为了`1`。

### **getPercentWidthSize()**

基于宽度计算百分比的值。

### **getPercentHeightSize()**

基于高度计算百分比的值。

---

# **AutoLayoutActivity**

继承`AutoLayoutActivity`的 Activity 可以使布局实现百分比化。

{% codeblock lang:java %}
public class AutoLayoutActivity extends AppCompatActivity {
    private static final String LAYOUT_LINEARLAYOUT = "LinearLayout";
    private static final String LAYOUT_FRAMELAYOUT = "FrameLayout";
    private static final String LAYOUT_RELATIVELAYOUT = "RelativeLayout";

    @Override
    public View onCreateView(String name, Context context, AttributeSet attrs) {
        View view = null;
        switch (name) {
            case LAYOUT_FRAMELAYOUT:
                view = new AutoFrameLayout(context, attrs);
                break;
            case LAYOUT_LINEARLAYOUT:
                view = new AutoLinearLayout(context, attrs);
                break;
            case LAYOUT_RELATIVELAYOUT:
                view = new AutoRelativeLayout(context, attrs);
                break;
        }
        if (view != null) {
            return view;
        }

        return super.onCreateView(name, context, attrs);
    }
}
{% endcodeblock %}

## **onCreateView(String name, Context context, AttributeSet attrs)**

替换`LinearLayout`为`AutoLinearLayout`，替换`FrameLayout`为`AutoFrameLayout`，替换`RelativeLayout`为`AutoRelativeLayout`。

---

# **AutoRelativeLayout**

{% codeblock lang:java %}
public class AutoRelativeLayout extends RelativeLayout {
    private final AutoLayoutHelper mHelper = new AutoLayoutHelper(this);

    public AutoRelativeLayout(Context context) {
        super(context);
    }

    public AutoRelativeLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public AutoRelativeLayout(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new LayoutParams(getContext(), attrs);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (!isInEditMode()) {
            mHelper.adjustChildren();
        }
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    public static class LayoutParams extends RelativeLayout.LayoutParams implements AutoLayoutHelper.AutoLayoutParams {
        private AutoLayoutInfo mAutoLayoutInfo;

        public LayoutParams(Context c, AttributeSet attrs) {
            super(c, attrs);
            mAutoLayoutInfo = AutoLayoutHelper.getAutoLayoutInfo(c, attrs);
        }

        public LayoutParams(int width, int height) {
            super(width, height);
        }

        public LayoutParams(ViewGroup.LayoutParams source) {
            super(source);
        }

        public LayoutParams(MarginLayoutParams source) {
            super(source);
        }

        @Override
        public AutoLayoutInfo getAutoLayoutInfo() {
            return mAutoLayoutInfo;
        }
    }
}
{% endcodeblock %}

## **generateLayoutParams(AttributeSet attrs)**

当父容器添加子 View 时调用。返回内部类`LayoutParams`的对象。

---
