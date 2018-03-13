---
title: Android自定义View之组合控件
date: 2018-03-06 16:40:38
tags:
    - Android
    - Custom View
    - Combine Component
---

在项目开发中应用界面经常会使用一些布局相同但内容不同的界面，为了减少不必要的重复代码，首先想到的方案是写一个通用的 xml 布局文件，通过 <code>include</code> 标签复用布局，毫无疑问这种方式是可行的，但是由于显示内容的不同，需要在每个使用到的界面先获取到相关控件后再设置属性。通常我们只是想显示不同的文字或图标，不需要获取相关控件再设置属性，采用自定义组合控件可以轻松的实现布局复用，也可以减少代码的冗余量。

<!-- more -->

## 自定义组合控件的两种实现方式

网上介绍自定义组合控件的文章很多，绝大部分文章都是介绍其中的一种实现方式，也有少部分文章虽然介绍了两种实现方式，但大都没有说两种实现方式有什么区别。下面我会介绍使用组合控件创建自定义 View 的两种实现方式，并通过运行示例来研究两种实现方式的区别，欢迎阅读并指出其中的错误。

为了便于区分将采用第一种方式实现的组合控件命名为 <code>ItemView</code>，采用第二种方式实现的组合控件命名为 <code>ItemView2</code>，请阅读文章时注意区分。

## 第一种方式

在构造函数中通过 <code>LayoutInflater.from(context).inflate(R.layout.xxx, this)</code> 将布局与当前 View 进行绑定。

### 创建自定义组合控件

<code>*ItemView.java*</code>
```java
package cn.neo.test;

import android.content.Context;
import android.support.annotation.Nullable;
import android.util.AttributeSet;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.LinearLayout;

public class ItemView extends LinearLayout {
    private static final String TAG = ItemView.class.getSimpleName();

    public ItemView(Context context) {
        this(context, null);
        Log.i(TAG, "[ItemView] " + "one param");
    }

    public ItemView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
        Log.i(TAG, "[ItemView] " + "two params");
    }

    public ItemView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        LayoutInflater.from(context).inflate(R.layout.item_view, this);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        Log.i(TAG, "[onFinishInflate] " + TAG);
    }

    @Override
    public void onViewAdded(View child) {
        super.onViewAdded(child);
        Log.i(TAG, "[onViewAdded] " + child);
    }

    @Override
    public void onViewRemoved(View child) {
        super.onViewRemoved(child);
        Log.i(TAG, "[onViewRemoved] " + child);
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        Log.i(TAG, "[onAttachedToWindow] " + TAG);
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        Log.i(TAG, "[onDetachedFromWindow] " + TAG);
    }
}
```

<code>*item_view.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ll_item_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center_vertical"
    android:orientation="horizontal">

    <ImageView
        android:id="@+id/iv_icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"/>

    <LinearLayout
        android:id="@+id/ll_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="16dp"
        android:orientation="vertical">

        <TextView
            android:id="@+id/tv_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:text="@string/title1"
            android:textColor="@color/colorPrimaryText"
            android:textSize="14sp"/>

        <TextView
            android:id="@+id/tv_summary"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:ellipsize="end"
            android:maxLines="1"
            android:text="@string/summary1"
            android:textColor="@color/colorSecondaryText"
            android:textSize="12sp"/>

    </LinearLayout>

</LinearLayout>
```

创建好 <code>ItemView</code> 后，接下来看看通过不同方式使用 <code>ItemView</code> 的区别。

### 在布局中引用控件

<code>*MainActivity.java*</code>
```java
package cn.neo.test;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

<code>*activity_main.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cn.neo.test.MainActivity">

    <cn.neo.test.ItemView
        android:id="@+id/item_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

运行结果：
{% img http://p5ia12npj.bkt.clouddn.com/2018-03-06/method_01_case_01.jpg 320 %}

运行日志：
```
03-08 14:50:23.795 3371-3371/cn.neo.test I/ItemView: [onViewAdded] android.widget.LinearLayout{a73c472 V.E...... ......I. 0,0-0,0 #7f070046 app:id/ll_item_view}
03-08 14:50:23.795 3371-3371/cn.neo.test I/ItemView: [ItemView] two params
03-08 14:50:23.795 3371-3371/cn.neo.test I/ItemView: [onFinishInflate] ItemView
03-08 14:50:23.847 3371-3371/cn.neo.test I/ItemView: [onAttachedToWindow] ItemView
```

### 在代码中新建控件

<code>*MainActivity.java*</code>
```java
package cn.neo.test;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.FrameLayout;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FrameLayout container = findViewById(R.id.container);
        ItemView itemView = new ItemView(this);
        container.addView(itemView);
    }
}
```

<code>*activity_main.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cn.neo.test.MainActivity">

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

运行结果：
{% img http://p5ia12npj.bkt.clouddn.com/2018-03-06/method_01_case_02.jpg 320 %}

运行日志：
```
03-08 14:53:03.422 3734-3734/cn.neo.test I/ItemView: [onViewAdded] android.widget.LinearLayout{a73c472 V.E...... ......I. 0,0-0,0 #7f070046 app:id/ll_item_view}
03-08 14:53:03.422 3734-3734/cn.neo.test I/ItemView: [ItemView] two params
03-08 14:53:03.422 3734-3734/cn.neo.test I/ItemView: [ItemView] one param
03-08 14:53:03.456 3734-3734/cn.neo.test I/ItemView: [onAttachedToWindow] ItemView
```

从运行结果来看，以上两种方式都能正常显示出自定义 View。从运行日志中可以看到在布局中直接引用控件时，系统在添加完子 View 后会调用 <code>onFinishInflate </code> 方法；而通过 <code>new</code> 方式创建对象时系统不会执行 <code>onFinishInflate</code> 方法。因此在使用这种方式创建自定义组合控件时，不要在 <code>onFinishInflate</code> 方法中执行代码，因为使用 <code>new</code> 方式创建对象时系统根本不会执行该方法。

## 第二种方式

将自定义 View 作为布局根节点使用，通过 <code>inflate</code> 方式创建自定义 View 的对象。

### 创建自定义组合控件

<code>*ItemView2.java*</code>
```java
package cn.neo.test;

import android.content.Context;
import android.support.annotation.Nullable;
import android.util.AttributeSet;
import android.util.Log;
import android.view.View;
import android.widget.LinearLayout;

public class ItemView2 extends LinearLayout {
    private static final String TAG = ItemView2.class.getSimpleName();

    public ItemView2(Context context) {
        this(context, null);
        Log.i(TAG, "[ItemView2] " + "one param");
    }

    public ItemView2(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
        Log.i(TAG, "[ItemView2] " + "two params");
    }

    public ItemView2(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public void onViewAdded(View child) {
        super.onViewAdded(child);
        Log.i(TAG, "[onViewAdded] " + child);
    }

    @Override
    public void onViewRemoved(View child) {
        super.onViewRemoved(child);
        Log.i(TAG, "[onViewRemoved] " + child);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        Log.i(TAG, "[onFinishInflate] " + TAG);
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        Log.i(TAG, "[onAttachedToWindow] " + TAG);
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        Log.i(TAG, "[onDetachedFromWindow] " + TAG);
    }
}
```

<code>*item_view2.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<cn.neo.test.ItemView2
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ll_item_view2"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center_vertical"
    android:orientation="horizontal">

    <ImageView
        android:id="@+id/iv_icon2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"/>

    <LinearLayout
        android:id="@+id/ll_content2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="16dp"
        android:orientation="vertical">

        <TextView
            android:id="@+id/tv_title2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:text="@string/title2"
            android:textColor="@color/colorPrimaryText"
            android:textSize="14sp"/>

        <TextView
            android:id="@+id/tv_summary2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:ellipsize="end"
            android:maxLines="1"
            android:text="@string/summary2"
            android:textColor="@color/colorSecondaryText"
            android:textSize="12sp"/>

    </LinearLayout>

</cn.neo.test.ItemView2>
```

在创建好 <code>ItemView2</code> 后，同样先来看看通过不同方式使用 <code>ItemView2</code> 的区别。

### 在布局中引用控件

<code>*MainActivity.java*</code>
```java
package cn.neo.test;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

<code>*activity_main.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cn.neo.test.MainActivity">

    <cn.neo.test.ItemView2
        android:id="@+id/item_view2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

运行结果：
{% img http://p5ia12npj.bkt.clouddn.com/2018-03-06/method_02_case_01.jpg 320 %}

运行日志：
```
03-08 15:07:15.789 1551-1551/? I/ItemView2: [ItemView2] two params
03-08 15:07:15.789 1551-1551/? I/ItemView2: [onFinishInflate] ItemView2
03-08 15:07:15.821 1551-1551/? I/ItemView2: [onAttachedToWindow] ItemView2
```

可以看到界面上没有显示任何控件，<code>ItemView2</code> 没有被显示出来。


### 在代码中新建控件

<code>*activity_main.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cn.neo.test.MainActivity">

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

#### 使用 new 的方式新建控件对象

<code>*MainActivity.java*</code>
```java
package cn.neo.test;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.FrameLayout;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FrameLayout container = findViewById(R.id.container);
        ItemView2 itemView2 = new ItemView2(this);
        container.addView(itemView2);
    }
}
```

运行日志：
```
03-08 15:14:01.098 3145-3145/cn.neo.test I/ItemView2: [ItemView2] two params
03-08 15:14:01.098 3145-3145/cn.neo.test I/ItemView2: [ItemView2] one param
03-08 15:14:01.171 3145-3145/cn.neo.test I/ItemView2: [onAttachedToWindow] ItemView2
```

运行代码可以看到界面上也没有显示任何控件，这里就不展示运行结果了，请自行尝试查看效果。

#### 使用 inflate 的方式新建控件对象

<code>*MainActivity.java*</code>
```java
package cn.neo.test;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.widget.FrameLayout;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FrameLayout container = findViewById(R.id.container);
        ItemView2 itemView2 = (ItemView2) LayoutInflater.from(this)
                .inflate(R.layout.item_view2, null);
        container.addView(itemView2);
    }
}
```

运行结果：
{% img http://p5ia12npj.bkt.clouddn.com/2018-03-06/method_02_case_03.jpg 320 %}

运行日志：
```
03-08 15:38:14.075 4482-4482/cn.neo.test I/ItemView2: [ItemView2] two params
03-08 15:38:14.083 4482-4482/cn.neo.test I/ItemView2: [onViewAdded] android.support.v7.widget.AppCompatImageView{f93c804 V.ED..... ......ID 0,0-0,0 #7f07003f app:id/iv_icon2}
03-08 15:38:14.097 4482-4482/cn.neo.test I/ItemView2: [onViewAdded] android.widget.LinearLayout{8538822 V.E...... ......I. 0,0-0,0 #7f070045 app:id/ll_content2}
03-08 15:38:14.097 4482-4482/cn.neo.test I/ItemView2: [onFinishInflate] ItemView2
03-08 15:38:14.172 4482-4482/cn.neo.test I/ItemView2: [onAttachedToWindow] ItemView2
```

从日志信息可以看到两个注意点：
- 通过 <code>inflate</code> 创建对象时调用的是两个参数的构造函数。  
- xml 布局中所有子 View 都被添加成功后，系统会调用 <code>onFinishInflate()</code> 方法通知绘制完毕。

第一点解释了为什么使用组合控件创建自定义 View 时必须需要重写两个参数的构造函数。第二点可以看到如果想要在自定义 View 中做一些初始化设置，使用 <code>findViewById()</code> 获取对象的操作必须放在 <code>super.onFinishInflate()</code> 调用之后，否则获取对象设置属性时会报 <code>**NullPointerException**</code> 异常，有兴趣可自行尝试在 <code>super.onFinishInflate()</code> 之前获取对象设置属性，查看运行结果是否如上所述。

## 分析

使用第一种方式创建自定义组合控件时，可以在布局中直接引用控件，也可以在代码中通过 <code>new</code> 的方式新建控件的对象。如果使用第二种方式创建自定义组合控件，那么既无法在布局中直接引用控件，也不能通过 <code>new</code> 的方式新建对象，只能使用 <code>inflate</code> 新建控件的对象，然后才可以使用控件，否则控件在界面上不会显示出来。

从两种不同创建方式的使用区别来看，使用组合控件创建自定义 View 的关键在于如何将布局填充到 ViewGroup 中。第一种方式在创建 View 时直接在构造函数中将布局填充到当前 ViewGroup 中，所以可以直接在布局中引用控件，也可以直接 <code>new</code> 控件的对象。而第二种方式在创建自 View 时没有与布局直接关联，所以只能通过 <code>inflate</code> 将布局与自定义 View 绑定后再使用。

以上就是两种不同方式创建自定义组合控件的使用差别，从上面的示例可以看出第一种方式比第二种方式使用起来更加便捷。另外当需要给组合控件添加一些通用属性时，使用第一种方式创建的组合控件使用起来更加便捷。因此我个人推荐使用组合控件创建自定义 View 时使用第一种方法，第二种方法作为了解即可。

## 完整示例
<code>*ItemView.java*</code>
```java
package cn.neo.test;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.drawable.Drawable;
import android.support.annotation.LayoutRes;
import android.support.annotation.Nullable;
import android.support.annotation.StringRes;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.TextView;

public class ItemView extends LinearLayout {
    private static final String TAG = ItemView.class.getSimpleName();

    private Drawable mItemIcon;
    private CharSequence mItemTitle;
    private CharSequence mItemSummary;

    private ImageView mIconIv;
    private TextView mTitleTv;
    private TextView mSummaryTv;

    public ItemView(Context context) {
        this(context, null);
    }

    public ItemView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ItemView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    private void initView(Context context, AttributeSet attrs) {
        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.ItemView);
        mItemIcon = array.getDrawable(R.styleable.ItemView_itemIcon);
        mItemTitle = array.getString(R.styleable.ItemView_itemTitle);
        mItemSummary = array.getString(R.styleable.ItemView_itemSummary);
        array.recycle();

        LayoutInflater.from(context).inflate(R.layout.item_view, this);
        mIconIv = findViewById(R.id.iv_icon);
        mTitleTv = findViewById(R.id.tv_title);
        mSummaryTv = findViewById(R.id.tv_summary);

        mIconIv.setImageDrawable(mItemIcon);
        mTitleTv.setText(mItemTitle);
        mSummaryTv.setText(mItemSummary);
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        // register listener ...
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        // unregister listener ...
    }

    public void setIcon(Drawable icon) {
        mIconIv.setImageDrawable(icon);
    }

    public Drawable getIcon() {
        return mItemIcon;
    }

    public void setTitle(@StringRes int resId) {
        setTitle(getResources().getString(resId));
    }

    public void setTitle(CharSequence title) {
        mItemTitle = title;
        mTitleTv.setText(title);
    }

    public CharSequence getTitle() {
        return mItemTitle;
    }

    public void setSummary(@StringRes int resId) {
        setSummary(getResources().getString(resId));
    }

    public void setSummary(CharSequence summary) {
        mItemSummary = summary;
        mSummaryTv.setText(summary);
    }

    public CharSequence getSummary() {
        return mItemSummary;
    }
}
```

<code>*item_view.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ll_item_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center_vertical"
    android:orientation="horizontal">

    <ImageView
        android:id="@+id/iv_icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher" />

    <LinearLayout
        android:id="@+id/ll_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="16dp"
        android:orientation="vertical">

        <TextView
            android:id="@+id/tv_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:text="@string/title1"
            android:textColor="@color/colorPrimaryText"
            android:textSize="14sp" />

        <TextView
            android:id="@+id/tv_summary"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:ellipsize="end"
            android:maxLines="1"
            android:text="@string/summary1"
            android:textColor="@color/colorSecondaryText"
            android:textSize="12sp" />

    </LinearLayout>

</LinearLayout>
```

<code>*attrs.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="ItemView">
        <attr name="itemIcon" format="reference" />
        <attr name="itemTitle" format="string" />
        <attr name="itemSummary" format="string" />
    </declare-styleable>
</resources>
```

<code>*MainActivity.java*</code>
```java
package cn.neo.test;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.FrameLayout;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FrameLayout container = findViewById(R.id.container);
        ItemView itemView = new ItemView(this);
        itemView.setIcon(getResources().getDrawable(R.mipmap.ic_launcher_round));
        itemView.setTitle(R.string.test_title2);
        itemView.setSummary(R.string.test_summary2);
        container.addView(itemView);
    }
}
```

<code>*activity_main.xml*</code>
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cn.neo.test.MainActivity">

    <cn.neo.test.ItemView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:itemIcon="@mipmap/ic_launcher_round"
        app:itemSummary="@string/test_summary1"
        app:itemTitle="@string/test_title1" />

    <View
        android:layout_width="match_parent"
        android:layout_height="3px"
        android:layout_marginBottom="8dp"
        android:layout_marginTop="8dp"
        android:background="@color/colorPrimary" />

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```

从上面的完整示例可以看到，使用第一种方式创建组合控件自定义 View 时，无论使用哪种方法引用控件都可以很方便的设置相关属性，下面是完整示例的运行结果：

{% img http://p5ia12npj.bkt.clouddn.com/2018-03-05/sample_result.jpg 320 %}

## 参考文章
[onFinishInflate() never gets called - stackoverflow](https://stackoverflow.com/questions/9650906/onfinishinflate-never-gets-called)
