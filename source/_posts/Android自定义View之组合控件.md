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

网上介绍自定义组合控件的文章很多，绝大部分文章都是介绍其中的一种实现方式，少部分文章虽然介绍了两种实现方式，但基本都没有对比两种实现方式的使用差异，我会尝试在这篇文章里介绍两种实现方式的差别，欢迎大家讨论并指出其中的错误。

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

运行效果：

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

查看运行结果：


运行日志：
```
03-08 14:53:03.422 3734-3734/cn.neo.test I/ItemView: [onViewAdded] android.widget.LinearLayout{a73c472 V.E...... ......I. 0,0-0,0 #7f070046 app:id/ll_item_view}
03-08 14:53:03.422 3734-3734/cn.neo.test I/ItemView: [ItemView] two params
03-08 14:53:03.422 3734-3734/cn.neo.test I/ItemView: [ItemView] one param
03-08 14:53:03.456 3734-3734/cn.neo.test I/ItemView: [onAttachedToWindow] ItemView
```

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

在创建好 <code>ItemView2</code> 后，同样的先来看看通过不同方式使用 <code>ItemView2</code> 的区别。

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

运行代码可以看到界面上也没有显示任何控件，这里就不再贴运行结果截图了，请自行尝试查看效果。

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


运行日志：
```
03-08 15:38:14.075 4482-4482/cn.neo.test I/ItemView2: [ItemView2] two params
03-08 15:38:14.083 4482-4482/cn.neo.test I/ItemView2: [onViewAdded] android.support.v7.widget.AppCompatImageView{f93c804 V.ED..... ......ID 0,0-0,0 #7f07003f app:id/iv_icon2}
03-08 15:38:14.097 4482-4482/cn.neo.test I/ItemView2: [onViewAdded] android.widget.LinearLayout{8538822 V.E...... ......I. 0,0-0,0 #7f070045 app:id/ll_content2}
03-08 15:38:14.097 4482-4482/cn.neo.test I/ItemView2: [onFinishInflate] ItemView2
03-08 15:38:14.172 4482-4482/cn.neo.test I/ItemView2: [onAttachedToWindow] ItemView2
```

从日志信息可以看到两个注意点：
- 通过 inflate 方法操作创建的对象调用的是两个参数的构造函数。  
- 使用 findViewById() 获取对象的操作需要放在 super.onFinishInflate() 调用完成之后，否则获取到的对象会为空，设置属性时会报 NullPointerException 异常。

## 分析

使用第一种方式创建自定义组合控件时，可以在布局中直接引用控件，也可以在代码中通过 <code>new</code> 的方式新建控件的对象。如果使用第二种方式创建自定义组合控件，则既无法在布局中直接引用控件，也不能通过 <code>new</code> 的方式新建对象，只能使用 <code>inflate</code> 新建控件的对象，然后才可以使用控件，否则控件在界面上不会显示出来。

以上就是两种不同方式创建自定义组合控件的使用差别，从上面的示例可以看出第一种方式比第二种方式使用起来更加便捷。另外当需要给组合控件添加一些通用属性时，使用第一种方式创建的组合控件使用起来更加便捷。因此我个人推荐使用组合控件创建自定义 View 时使用第一种方法，第二种方法作为了解即可。
