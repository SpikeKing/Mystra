---
title: Android Tips 4
date: 2016-02-24 10:16:00
categories: [Tips]
tags: [Android,Tips]
---

本文介绍一些, 在Android开发中会经常使用的小知识点, 每篇10个. 第四篇.

<!-- more -->
> 更多: http://www.wangchenlong.org/

![Android](241-android-tips-4/android-tips.png)

系列
[第一篇](http://www.wangchenlong.org/2016/02/23/tips/1603/231-android-tips-1/), [第二篇](http://www.wangchenlong.org/2016/02/23/tips/1603/232-android-tips-2/), [第三篇](http://www.wangchenlong.org/2016/02/23/tips/1603/233-android-tips-3/), [第四篇](http://www.wangchenlong.org/2016/02/24/tips/1603/241-android-tips-4/), [第五篇](http://www.wangchenlong.org/2016/02/24/tips/1603/242-android-tips-5/).

---

# IDE版本管理

AS即Android Studio. 我从0.4版本就开始使用, 现在已经到2.0版本. 那么我们应该如何控制AS的版本呢? 最新版是升还是不升? 哪个版本对我来说很重要?

AS版本主要分为3个类型.
**Canary builds**: 构建版本, 会增加一些新特性, 但不是很稳定, 非常有意思.
**Beta versions**: 测试版本, Bug非常少, 逐步过渡到稳定版.
**Stable releases**: 稳定版本, 官方推荐版本.

AS支持多个应用并存, 不会相互污染. 如果你是热爱开发的人, 可以保留两个应用, 即Canary版和Stable版. Canary版引入新特性, 加速开发; Stable版避免IDE的Bug, 备份选择. 一个命名是``Android Studio Canary``, 另一个是``Android Studio Stable``.

> 注意: AS与Gradle构建版本绑定. 如果不是精通Android开发的人, 不要随意升级AS, 否则会出现Gradle问题, 请找他人解决. 不过, 我至今没发现不能解决的问题.

其实, 我每次都会升级AS到最新版本, 并没有发现严重的Bug, 反而是新特性会大幅提升生产力. 不过, 玩转IDE也是因水平而异的, 不要盲目尝试.

---

# NoActionBar样式

普通
```xml
    <style name="AppTheme.NoActionBar">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>
```

v21
```xml
    <style name="AppTheme.NoActionBar">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <item name="android:windowDrawsSystemBarBackgrounds">true</item>
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
```

> 注意: windowActionBar和windowNoTitle都不是**android:**开头的, 属于系统.

---

# org.apache.http遗弃

Android升级编译版本, 版本20以下升至高版本都会发生一些错误.
即**找不到org.apache.http.protocol.HTTP**. 
原因是org.apache.http被遗弃了, 需要额外补充或替换.

无损方法, 使用
```gradle
android {
    useLibrary 'org.apache.http.legacy'
}
```
即可.

---

# AppBarLayout显示内容

Android推荐使用AppBar替代ActionBar. AppBar是ActionBar的超集, 既包含常用的工具栏, 又可以使用图片页面. 上面是AppBar, 下面是内容布局. 需要注意的是, 内容布局需要设置额外的属性, 即
```xml
<LinearLayout
    app:layout_behavior="@string/appbar_scrolling_view_behavior">
```
才可以正常显示.

---

# Markdown流程图

[在线网址](http://mdp.tylingsoft.com/), 动态权限授权示例.
```
graph TD
    A[显示页面] --> B{检测权限}
    B --> |全部授权| C[启动页面]
    B --> |缺少权限| D{系统授权弹窗}
    D --> |允许| C[启动页面]
    D --> |拒绝| E{自定义设置权限弹窗}
    E --> |退出| F[退出页面]
    E --> |设置| G{添加应用权限}
    G --> |成功| C
    G --> |失败| D
```

---

# ADB启动Activity

启动Activity的shell命令是
```
adb shell am start  -n [包名]/[Activity名]
```
如
```
adb shell am start  -n me.chunyu.Pedometer/me.chunyu.Assistant.activity.AssistantShowFunction
```

---

# 修改Toolbar的Title颜色

Toolbar是Android推荐替代ActionBar的标题栏. 设置标题颜色, 使用样式覆盖.

布局
```
    <android.support.v7.widget.Toolbar
        android:id="@+id/main_t_toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimary"
        android:theme="@style/DemoActionBar"/>
```
样式
```
    <style name="DemoActionBar" parent="ThemeOverlay.AppCompat.ActionBar">
        <item name="android:textColorPrimary">@android:color/white</item>
    </style>
```

[参考](http://stackoverflow.com/questions/26852108/how-do-you-set-the-title-color-for-the-new-toolbar)

---

# 在线视频处理

非常不错的在线网站, 处理音频和视频. [参考](http://online-video-cutter.com/).

---

# 截获网页的Apk下载

在嵌入网页跳转时, Apk下载地址, 无法跳转, 需要截获, 自动下载, 并安装. 
```
    // 配置WebViewClient来处理网页加载的各种状态
    protected void setWebViewClient() {
        WebViewClient webClient = new WebViewClient() {
            @Override
            public void onPageFinished(WebView view, String url) { // 重定向会加载多次
                if (mPageFinishedListener != null) {
                    mPageFinishedListener.overridePageFinished(view, url);
                }
                Log.d(TAG, "overridePageFinished");
                loadUrl(HTML_CONTENT); // 加载JS内容
            }

            @Override
            public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
                if (mReceivedErrorListener != null) {
                    mReceivedErrorListener.overrideReceivedError(view, request, error);
                }
            }

            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                Log.d(TAG, "url: " + url);
                if (url.endsWith(".apk")) {
                    Log.d(TAG, "下载Apk: " + url);
                    DownloadUtils.downloadFiles(url);
                }
                return false;
            }
        };
        setWebViewClient(webClient);
    }
```

> 注意**shouldOverrideUrlLoading**和**url.endsWith(".apk")**.

---

# 自动化测试

推荐两个常用的测试工具[Espresso](https://google.github.io/android-testing-support-library/docs/espresso/)和[Robolectric](http://robolectric.org/).

---

OK, that's all! Enjoy it!

---

> 原始地址: 
> http://www.wangchenlong.org/2016/02/24/tips/1603/241-android-tips-4/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

