---
title: Android Tips 5
date: 2016-02-24 10:27:00
categories: [Tips]
tags: [Android,Tips]
---

本文介绍一些, 在Android开发中会经常使用的小知识点, 每篇10个. 第五篇.

<!-- more -->
> 更多: http://www.wangchenlong.org/

![Android](242-android-tips-5/android-tips.png)

系列
[第一篇](http://www.wangchenlong.org/2016/02/23/tips/1603/231-android-tips-1/), [第二篇](http://www.wangchenlong.org/2016/02/23/tips/1603/232-android-tips-2/), [第三篇](http://www.wangchenlong.org/2016/02/23/tips/1603/233-android-tips-3/), [第四篇](http://www.wangchenlong.org/2016/02/24/tips/1603/241-android-tips-4/), [第五篇](http://www.wangchenlong.org/2016/02/24/tips/1603/242-android-tips-5/).

---

# 模拟系统回收Activity

使用adb命令可以模拟Android系统自动回收Activity进程, 可以调试这个效果.
单进程
```
adb shell am force-stop [包名]
```
多进程
```
adb shell ps | grep [包名]
adb shell kill [PID]
```

---

# Android库动态权限建议

Android 6.0使用动态权限, 在创建第三方库时, 需要充分考虑这一特性.
(1) 在自动Merge库的AndroidManifest时, 需要提供**危险权限**的提示文档.
(2) 当权限未获取时, 用户可以使用库的某一部分, 并提示缺少权限.
(3) 需要提供在权限获取失败时, 库使用方式的文档.
(4) 确保所有权限都是必须的, 不含有未使用权限.

关于提示用户获取权限的解决方案, 可以[参考](http://www.jianshu.com/p/dbe4d37731e6).

---

# RxJava处理Retry请求

服务器经常会出现异常, 应用需要连续尝试请求, RxJava可以非常简单的实现.

模板
```java
Observable<Boolean> source = ...; // Something that eventually emits true

source
    .repeatWhen(completed -> completed.delay(1, TimeUnit.SECONDS))
    .takeUntil(result -> result)
    .filter(result -> result)
    .subscribe(
        res -> System.out.println("onNext(" + res + ")"),
        err -> System.out.println("onError()"),
        () -> System.out.println("onCompleted()")
    );
```

示例
```java
/**
 * This is a class that should be
 * mapped on your json response from the server
 */
class ServerPollingResponse {
    boolean isJobDone;

    @Override
    public String toString() {
        return "isJobDone=" + isJobDone;
    }
}

Subscription checkJobSubscription = mDataManager.pollServer(inputData)
        .repeatWhen(new Func1<Observable<? extends Void>, Observable<?>>() {
            @Override
            public Observable<?> call(Observable<? extends Void> observable) {
                Log.v(TAG, "repeatWhen, call");
                /**
                 * This is called only once.
                 * 5 means each repeated call will be delayed by 5 seconds
                 */
                return observable.delay(5, TimeUnit.SECONDS);
            }
        })
        .takeUntil(new Func1<ServerPollingResponse, Boolean>() {
            @Override
            public Boolean call(ServerPollingResponse response) {
                /** Here we can check if the responce is correct and if we should
                 *  finish polling
                 *  We finish polling when job is done.
                 *  In other words : "We stop taking when job is done"
                 */
                Log.v(TAG, "takeUntil, call response " + response);
                return response.isJobDone;
            }
        })
        .filter(new Func1<ServerPollingResponse, Boolean>() {
            @Override
            public Boolean call(ServerPollingResponse response) {
                /**
                 * We are filtering results if we return "false".
                 * Filtering means that onNext() will not be called.
                 * But onComplete() will be delivered.
                 */
                Log.v(TAG, "filter, call response " + response);
                return response.isJobDone;
            }
        })
        .subscribe(
                new Subscriber<ServerPollingResponse>() {
                    @Override
                    public void onCompleted() {
                        Log.v(TAG, "onCompleted ");
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.v(TAG, "onError ");
                    }

                    @Override
                    public void onNext(ServerPollingResponse response) {
                        Log.v(TAG, "onNext response " + response);
                        // Do whatever you need. Server polling has been finished
                    }
                }
        );
```

[参考](http://stackoverflow.com/questions/34943734/using-of-skipwhile-combined-with-repeatwhen-in-rxjava-to-implement-server-po/34948978)和[参考](https://medium.com/@v.danylo/server-polling-and-retrying-failed-operations-with-retrofit-and-rxjava-8bcc7e641a5a#.wy66xu6rd).

---

# Chrome的JsonView插件

添加插件之后, 可以格式化的显示Json数据.

![JsonView](242-android-tips-5/tips5-4.png)

---

# 显示Activity栈的Shell命令

Shell命令
```
adb shell dumpsys activity | sed -n -e '/Stack #/p' -e '/Running activities/,/Run #0/p'
```
直接获取Activity信息有些冗余, 我们只关注堆栈信息即可.
**sed**可以编辑显示的文字. 
``-n``, 从截取处开始连续处理.
``-e``, 多选参数.
``'/Stack #/p'``, 输出含有``Stack #``的行.
``-e '/Running activities/,/Run #0/p'``, 输出从``Running activities``至``Run #0``的所有行.

输出结果
```
  Stack #1:
    Running activities (most recent first):
      TaskRecord{299f41ea #2269 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=6}
        Run #5: ActivityRecord{33926043 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2269}
        Run #4: ActivityRecord{3f181566 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2269}
        Run #3: ActivityRecord{22737e45 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2269}
        Run #2: ActivityRecord{ce0a990 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2269}
        Run #1: ActivityRecord{3de8e378 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2269}
        Run #0: ActivityRecord{1cb28ec4 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2269}
  Stack #0:
    Running activities (most recent first):
      TaskRecord{bfee9cf #2241 A=com.miui.home U=0 sz=1}
        Run #0: ActivityRecord{279bc098 u0 com.miui.home/.launcher.Launcher t2241}
```

---

# dp和sp的区别

dp是Android页面常用的度量单位, sp主要用于字体度量.
在标准情况下, dp等于sp. 然而, Android系统允许用户设置字体大小, sp会随着字体的大小而改变, 放大或是缩小.
设置位置(红米): Android -> 设置 -> 字体大小 -> 标准(默认)或大小号.

---

# AlertDialog获取屏幕监听

在Android 4.0以上, AlertDialog在触摸对话框边缘外部时, 对话框消失.
在AlertDialog.Builder.create(), 可以设置属性获取屏幕监听.
方法一:
```java
setCanceledOnTouchOutside(false);
```
> 调用这个方法时, 按对话框以外的地方不起作用. 按返回键仍起作用.

方法二:
```java
setCancelable(false);
```
调用这个方法时, 按对话框以外的地方不起作用. 按返回键也不起作用.
 
---

# getColor遗弃

最新版本的getColor被遗弃(deprecated), 使用时, 需要添加主题. 
也可以使用兼容模式, 即
```java
ContextCompat.getColor(context, R.color.your_color);
```

``ContextCompat.getColor``的源码
```java
public static final int getColor(Context context, int id) {
    final int version = Build.VERSION.SDK_INT;
    if (version >= 23) {
        return ContextCompatApi23.getColor(context, id);
    } else {
        return context.getResources().getColor(id);
    }
}
```

---

# libarchive和expat简介

**[libarchive](http://www.libarchive.org/)**
Multi-format archive and compression library. 多格式存档和压缩库.
Android的toolchain使用libArchive.
[参考](https://github.com/libarchive/libarchive/issues/442)

**[expat](http://expat.sourceforge.net/)**
Expat is an XML parser library written in C. Expat是用C语言写的XML解析库.
Android的Platform的扩展.
[参考](https://android.googlesource.com/platform/external/expat/)

libarchive 2.8.4和expat 2.1.0会产生漏洞, 如需修复, 需要升级Android的编译版本.

---

# 网页重定向

默认链接会跳转其他链接, 根据链接内容, 进行相应操作, 如下载Apk等. 如果使用重定向, 则返回false; 如果非重定向, 则返回true.
```
        WebViewClient webClient = new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                if (url.endsWith(".apk")) {
                    DownloadUtils.downloadFiles(url);
                    if (mStartDownloadAppListener != null) {
                        mStartDownloadAppListener.doAfter();
                    }
                    return true;
                }
                return false;
            }
        };
        setWebViewClient(webClient);
```

---

OK, that's all! Enjoy it!

---

> 原始地址: 
> http://www.wangchenlong.org/2016/02/24/tips/1603/242-android-tips-5/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

