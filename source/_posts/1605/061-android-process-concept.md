---
title: 关于 Android 进程的简介
date: 2016-05-06 17:06:00
categories: [Android]
tags: [Android]
---

在Android系统中, 进程非常重要, 除了**主进程**运行App, 我们还可以使用**其他进程**处理独立任务.

<!-- more -->
> 更多: http://www.wangchenlong.org/
> 欢迎Follow我的GitHub: https://github.com/SpikeKing

进程, 即Process. 进程间通信, 即IPC(Inter-Process Communication). 

![Process](061-android-process-concept/process-concept-logo.jpg)

在Android中, 使用多进程只有一种方式, 在AndroidManifest中, 为四大组件(Activity, Service, Receiver, ContentProvider)指定``android:process``属性.

``` xml
<service
    android:name=".PedometerCounterService"
    android:exported="false"
    android:process=":cy_pedometer_set"/>
```

> ``exported="false"``表示只与本应用内的进程通信, 即包名相同.

默认进程的进程名是**包名**.

``` bash
➜  ~ adb shell ps | grep wangchenlong.chunyu.me.android_pedometer_set
u0_a354   28490 410   2259024 80272 ffffffff 00000000 S wangchenlong.chunyu.me.android_pedometer_set
u0_a354   28515 410   2191112 60080 ffffffff 00000000 S wangchenlong.chunyu.me.android_pedometer_set:cy_pedometer_set
```

> 进程ID是``28490``和``28515``. 父进程ID是``410``. ``ps -help``显示标题.

使用``":"``表示**私有进程**, 其他组件不能使用; 使用全称表示**全局进程**, 其他组件可以使用``ShareUID``共享进程.

多进程无法通过内存共享数据. 可以通过**Intent**传递数据.

不同进程的组件会拥有独立的**虚拟机**, **Application**, **内存空间**.

> 多个进程, Application会创建多次.

``Serializable``和``Parcelable``接口处理对象序列化过程. 使用``ObjectOutputStream``和``ObjectInputStream``处理对象的``Serializable``序列化与反序列化.

> Intent可以使用``Serializable``和``Parcelable``接口传递复杂对象数据, 参与进程间的通信.

OK, that's all! Enjoy it!

---

> 最初发布地址: 
> http://www.wangchenlong.org/2016/05/06/1605/061-android-process-concept/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

