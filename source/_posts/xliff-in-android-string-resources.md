---
title: Android xliff 介绍
date: 2018-08-29 10:55:21
toc: true
tags:
    - Android
    - Localization
    - xliff
categories:
    - Android
    - Localization
---

## 标记字符串中不需要翻译的文字
Android 系统通过在字符串中添加 <code><xliff:g></code> 占位符来标记字符串中不需要翻译的文字，使用 <code><xliff:g></code> 占位符需要在 <code>resources</code> 节点添加相应的 <code>xmlns</code>：
```xml
xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2"
```
<!---- more ---->

## 使用方法
### strings.xml
```xml
<xliff:g example="xxx" id="xxx">format_specifiers</xliff:g>
```
- example： 如果程序在运行时会替换占位符的内容，最好提供此属性来明确预期使用结果
- id： 解释占位符的作用
- format_specifiers： 格式说明符，用来指定文字的输出格式

### 代码

通过 <code>Resources.getString(@StringRes int id, Object... formatArgs)</code> 方法动态加载字符串的内容，<code>formatArgs</code> 根据参数下标替代对应占位符中的内容。

## 格式说明符语法
```
%[argument_index$][flags][width][.precision]conversion
```
- 可选参数 <code>argument_index</code> 是一个十进制整数，表示参数在参数列表中的位置。第一个参数下标是 <code>1$</code>，第二个参数下标是 <code>2$</code>，依此类推。
- 可选参数 <code>flags</code> 是一组字符，用来修改输出格式。有效转换集取决于 <code>conversion</code>。
- 可选参数 <code>width</code> 是一个十进制正整数，表示输出结果的最小字符数。
- 可选参数 <code>precision</code> 是一个非负十进制整数，通常用来限制字符数。输出结果取决于 <code>conversion</code>。
- 必要参数 <code>conversion</code> 是一个字符，用来表示参数如何被格式化。给定参数的有效转换集取决于参数的数据类型。

查看源码可以知道 <code>Resources.getString(@StringRes int id, Object... formatArgs)</code> 实际上调用的是 <code>String.format(...)</code> 方法：
```java
@NonNull
public String getString(@StringRes int id, Object... formatArgs) throws NotFoundException {
    final String raw = getString(id);
    return String.format(mResourcesImpl.getConfiguration().getLocales().get(0), raw,
            formatArgs);
}
```

因此字符串如何格式化实际上就是 <code>String.format()</code> 如何格式化。关于如何 <code>String.format()</code> 如何格式化字符串请参考 [Java String Format Examples](https://dzone.com/articles/java-string-format-examples) 或者 [JAVA字符串格式化-String.format()的使用](https://www.cnblogs.com/Dhouse/p/7776780.html)，文章里面有详细的介绍和示例。

## 示例
### 字符串资源只有一个占位符
字符串只有一个占位符时，<code>1$</code> 下标可以不写：
```xml
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
    <string name="app_name">Test</string>
    <string name="name_introduction">My name is <xliff:g example="jack" id="name">%s</xliff:g>.</string>
    <string name="age_introduction">I am <xliff:g example="20" id="age">%d</xliff:g> years old.</string>
</resources>
```

在代码中动态加载字符串：
```java
String name = getResources().getString(R.string.name_introduction, "alex");
System.out.println(name);
String age = getResources().getString(R.string.age_introduction, 20);
System.out.println(age);
```

查看输出结果：
```
08-30 17:28:27.655 6628-6628/cn.neo.test I/System.out: My name is alex.
08-30 17:28:27.655 6628-6628/cn.neo.test I/System.out: I am 20 years old.
```

### 字符串资源有多个占位符
当字符串包含多个占位符时，需要指定每个参数的下标：
```xml
<string name="self_introduction">My name is <xliff:g example="jack" id="name">%1$s</xliff:g> and I am <xliff:g example="20" id="age">%2$d</xliff:g> years old.</string>
```

```java
String selfInfo = getResources().getString(R.string.self_introduction, "alex", 20);
System.out.println(selfInfo);
```

```
08-30 17:35:20.379 7923-7923/cn.neo.test I/System.out: My name is alex and I am 20 years old.
```

## 参考文章
[Formatter - Android Developer](https://developer.android.google.cn/reference/java/util/Formatter)
[Referring to resources in code - Android Developer](https://developer.android.google.cn/guide/topics/resources/localization#referring-to-resources)
[What does “xmlns” in XML mean? - Stack Overflow](https://stackoverflow.com/questions/1181888/what-does-xmlns-in-xml-mean)
[XLIFF - Wikipedia](https://en.wikipedia.org/wiki/XLIFF)
[Java String Format Examples](https://dzone.com/articles/java-string-format-examples)
[JAVA字符串格式化-String.format()的使用](https://www.cnblogs.com/Dhouse/p/7776780.html)
