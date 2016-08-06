---
title: Android Tips 1
date: 2016-02-23 20:46:00
categories: [Tips]
tags: [Android, Tips]
---

本文介绍一些, 在Android开发中会经常使用的小知识点, 每篇10个. 第一篇.

<!-- more -->
> 更多: http://www.wangchenlong.org/

![Android](231-android-tips-1/android-tips.png)

系列
[第一篇](http://www.wangchenlong.org/2016/02/23/tips/1603/231-android-tips-1/), [第二篇](http://www.wangchenlong.org/2016/02/23/tips/1603/232-android-tips-2/), [第三篇](http://www.wangchenlong.org/2016/02/23/tips/1603/233-android-tips-3/), [第四篇](http://www.wangchenlong.org/2016/02/24/tips/1603/241-android-tips-4/), [第五篇](http://www.wangchenlong.org/2016/02/24/tips/1603/242-android-tips-5/).

---

# Download文件夹

绝对路径
```
/storage/emulated/0/Download/xxx
```

遍历
```java
        File file = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS);
        File[] files = file.listFiles();
        for (int i = 0; i < files.length; ++i) {
            Log.e(TAG, files[i].getAbsolutePath());
        }
```

---

# ButterKnife多参数

绑定多个参数
```java
    @OnClick({
            R.id.dialog_dau_share_wx,
            R.id.dialog_dau_share_wx_timeline,
            R.id.dialog_dau_share_weibo,
            R.id.dialog_dau_share_qq
    })
```

---

# Submodule的使用方式

submodule与git可以保持实时同步.
添加
```
git submodule add https://github.com/SpikeKing/DroidPlugin.git DroidPlugin
```
使用
```
git submodule update --init --recursive
```
导入, 路径多于一个, 前面不添加冒号(:).
```gradle
include ':app', 'DroidPlugin:project:Libraries:DroidPlugin'
```
引用
```gradle
compile project(':DroidPlugin:project:Libraries:DroidPlugin')
```

---

# 更新Github的Fork库

GitHub的教程, [参考](https://help.github.com/articles/syncing-a-fork/).

---

# 检测App是否安装

使用PackageManager
```java
// 检查App是否安装
private boolean appInstalledOrNot(String uri) {
    PackageManager pm = getPackageManager();
    boolean app_installed;
    try {
        pm.getPackageInfo(uri, PackageManager.GET_ACTIVITIES);
        app_installed = true;
    } catch (PackageManager.NameNotFoundException e) {
        app_installed = false;
    }
    return app_installed;
}
```

---

# Canvas重绘

参考函数[invalidate()](http://developer.android.com/reference/android/view/View.html#invalidate%28%29). 
[参考](http://stackoverflow.com/questions/16449039/how-to-redraw-canvas-in-a-view-from-activity).

---

# 按钮的默认点击效果

波纹效果(5.0+), 阴影效果(5.0-).
```xml
android:background="?android:attr/selectableItemBackground"
```
继承样式
```xml
    <!--按钮-->
    <style name="PersonInfoButton" parent="@android:style/ButtonBar">
        <item name="android:layout_width">@dimen/d80dp</item>
        <item name="android:layout_height">@dimen/d32dp</item>
        <item name="android:textSize">@dimen/d14sp</item>
    </style>
```
> 注意: @android:style/ButtonBar

---

# Proguard去除Log信息

默认删除log.i, .v, 可以指定删除.d, .e. [参考](http://stackoverflow.com/questions/12390466/android-proguard-not-removing-all-log-messages).
```proguard
# 删除Log
-assumenosideeffects class android.util.Log { *; }
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** e(...);
}
```

---

# 简化数据库的使用

在使用数据库时, 操作有些复杂, Sugar库简化使用方法. [参考](http://satyan.github.io/sugar/index.html).
```gradle
compile 'com.github.satyan:sugar:1.3'
```

---

# 完全填充EditView

完全填充链接的EditView是无法被点击的, 只能跳转, 通过在结尾处添加一个不占位的空格("\u200B"), 才可以点击到尾部.
```java
    // 设置可以点击和编辑的EditText
    private void setEditClickable() {
        mEtEditText.setMovementMethod(LinkMovementMethod.getInstance());
        Spannable spannable = new SpannableString("http://www.baidu.com");
        Linkify.addLinks(spannable, Linkify.WEB_URLS);

        // 添加了零宽度空格(​\u200B​​​), 才可以点击到最后的位置, 否则会触发链接
        CharSequence text = TextUtils.concat(spannable, "\u200B");

        mEtEditText.setText(text);
    }
```

---

OK, that's all! Enjoy it.

---

> 原始地址: 
> http://www.wangchenlong.org/2016/02/23/tips/1603/231-android-tips-1/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

