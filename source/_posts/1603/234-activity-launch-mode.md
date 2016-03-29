---
title: 分析 Activity 的启动模式
date: 2016-02-23 20:12:00
categories: [Android]
tags: [Android,基础,Activity]
---

在Android应用中, Activity是最核心的组件, 如何生成一个Activity实例, 可以选择不同的启动模式, 即LaunchMode. 启动模式主要包括: standard, singleTop, singleTask, singleInstance. 

<!-- more -->
> 更多: http://www.wangchenlong.org/

标准模式在每次启动时, 都会创建实例; 三种单例模式, 会根据情况选择创建还是复用实例. 在Activity启动中, 创建实例的生命周期: onCreate -> onStart -> onResume; 重用实例的生命周期: onNewIntent -> onResume. 

在AndroidManifest的Activity中, 使用launchMode属性, 可以设置启动模式, 默认是standard模式; 使用taskAffinity属性, 并添加包名, 可以设置Activity栈, 默认是当前包名, 只能应用于single模式.

希望通过本文, 可以更好的理解Activity的启动模式(LaunchMode).

![Android](234-activity-launch-mode/launch-logo.png)

观察Activity栈的脚本.
```shell
adb shell dumpsys activity | sed -n -e '/Stack #/p' -e '/Running activities/,/Run #0/p'
```

本文示例的GitHub[下载地址](https://github.com/SpikeKing/wcl-activity-launchmode-demo)

---

# Standard

标准模式, 启动Activity的默认模式, **被启动的Activity**会运行于**启动的Activity**栈, 因此必须使用Activity的Context启动, 不能使用Application, 否则会报错.

如MainActivity启动TestAActivity.
```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{3caa65e3 #2711 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=2}
        Run #1: ActivityRecord{36b06e99 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2711}
        Run #0: ActivityRecord{27396226 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2711}
  Stack #0:
    Running activities (most recent first):
      TaskRecord{27d796c9 #2695 A=com.miui.home U=0 sz=1}
        Run #0: ActivityRecord{2e5712cb u0 com.miui.home/.launcher.Launcher t2695}
```

> 栈内由下到上: MainActivity -> TestAActivity.

---

# SingleTop
栈顶复用模式. 只有Activity位于栈顶, 重复启动时, 会使用默认实例, 即单例模式; 如果位于栈内, 则仍然会创建实例.

MainActivity启动TestA, TestA启动TestB, TestB启动自身, TestB是单例. 观察栈内情况, TestB只有一份实例, 第二次创建复用.
```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{12abf566 #2712 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=3}
        Run #2: ActivityRecord{187d7ff7 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2712}
        Run #1: ActivityRecord{a551034 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2712}
        Run #0: ActivityRecord{22f9cce4 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2712}
```
> 栈内: MainActivity -> TestAActivity -> TestBActivity

MainActivity启动TestA, TestA启动TestB, TestB启动TestC, TestC启动TestB, TestB是单例. 观察栈内情况, 由于TestC是栈顶, TestC启动TestB, TestB不是栈顶, 重新创建TestB实例, 则保留两份TestB.

```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{1792f5f0 #2715 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=5}
        Run #4: ActivityRecord{1e70110b u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2715}
        Run #3: ActivityRecord{c7f4dce u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestCActivity t2715}
        Run #2: ActivityRecord{254536cd u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2715}
        Run #1: ActivityRecord{36b2da15 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2715}
        Run #0: ActivityRecord{3a1c4a6a u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2715}
```

> 栈内: MainActivity -> TestAActivity -> TestBActivity -> 
TestCActivity -> TestBActivity

---

# SingleTask

栈内复用模式, 只要Activity在一个栈中存在, 多次调用时, 都不会创建实例, 即单例模式.

情况包含以下几种:

**(1) 任务栈不存在, 初次启动SingleTask实例, 会创建任务栈和实例.**

MainActivity启动TestA, TestA启动TestB, TestB是SingleTask, 并且任务栈不同. 观察可知, 系统包含两个任务栈, TestB位于其他任务栈中.

```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{d5d53d4 #2727 A=me.chunyu.spike.stack U=0 sz=1}
        Run #2: ActivityRecord{1d720e55 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2727}
      TaskRecord{a3f797d #2726 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=2}
        Run #1: ActivityRecord{ffd689d u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2726}
        Run #0: ActivityRecord{192310ac u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2726}
```

> 使用taskAffinity属性, 添加新的Activity栈, 与SingleTask配合使用, Standard模式无效.
> 新任务栈是`me.chunyu.spike.stack`.

**(2) 任务栈存在, 初次启动SingleTask实例, 会直接入栈, 与Standard模式相同.**

**(3) 任务栈相同, 再次启动SingleTask实例, 实例会置于栈顶, 并清除其上面实例, 具有clearTop的效果.**

MainActivity启动TestA, TestA启动TestB, TestB是SingleTask, TestB启动TestC, TestC重新启动TestB, 则TestC会出栈. 观察可知, TestC出栈, TestB位于栈顶.

```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{18230815 #2737 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=3}
        Run #4: ActivityRecord{1126c300 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2737}
        Run #3: ActivityRecord{3114fee8 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2737}
        Run #2: ActivityRecord{f8e235d u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2737}
```

> TestC启动TestB, SingleTask模式, 导致clearTop, TestC出栈.

**(4) 任务栈不同, 再次启动SingleTask实例, 会导致任务栈切换, 后台置于前台. **

这比较难理解.
MainActivity启动TestA, TestA启动TestB(SingleTask实例, 不同任务栈), TestB启动TestC(与B类似), 则MainActivity和TestA相同栈, TestB和TestC相同栈, 此时栈顶是TestC. 按Home键, 再次启动应用, 则默认任务栈会启动, TestA启动, TestA启动TestC. 应用当前状态如下.

```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{1d05e6c9 #2754 A=me.chunyu.spike.stack U=0 sz=2}
        Run #4: ActivityRecord{3f77e822 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestCActivity t2754}
      TaskRecord{3fe736d0 #2753 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=2}
        Run #3: ActivityRecord{15f0470e u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2753}
      TaskRecord{1d05e6c9 #2754 A=me.chunyu.spike.stack U=0 sz=2}
        Run #2: ActivityRecord{181229e6 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2754}
      TaskRecord{3fe736d0 #2753 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=2}
        Run #1: ActivityRecord{28628d61 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2753}
      TaskRecord{2d646058 #2719 A=com.android.incallui U=0 sz=1}
```

TestC至于栈顶, 点击Back键, 不是返回TestA(启动TestC的实例), 而是TestB, 即优先返回相同栈的实例. 再次是TestA, 然后是MainActivity, 依次出栈.

---

# SingleInstance

单实例模式, 启动时, 系统会为其创造一个单独的任务栈, 以后每次使用, 都会使用这个单例, 直到其被销毁, 属于真正的单例模式.

示例: MainActivity启动TestA, TestA启动TestB(SingleInstance模式), 
TestB启动TestC, TestC再启动TestB, 则仍启动上一次的TestB, 
TestC合并入默认栈(MainActivity+TestA).
```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{384e3928 #2765 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=1}
        Run #3: ActivityRecord{1ffc5b6b u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2765}
      TaskRecord{2ad03544 #2764 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=3}
        Run #2: ActivityRecord{293d8c37 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestCActivity t2764}
        Run #1: ActivityRecord{158bc0f3 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2764}
        Run #0: ActivityRecord{77691cf u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2764}
```

---

# startActivityForResult

Thx@回调的幸福时光

startActivityForResult不同于startActivity, 使用LaunchMode模式启动Activity时, 也会有一些不同, 可以正常传递数据, 但是无法连续创建自己时, 会生成多份实例. 

TestB(singleTask模式)使用startActivity创建自己时, 会使用默认实例, 即单例; 而使用startActivityForResult创建自己时, 会生成一份新的示例.
```shell
  Stack #1:
    Running activities (most recent first):
      TaskRecord{323200ac #2786 A=me.chunyu.clwang.stack U=0 sz=3}
        Run #4: ActivityRecord{3f9e14f3 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2786}
        Run #3: ActivityRecord{30d8f17b u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2786}
        Run #2: ActivityRecord{11b95b5c u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestBActivity t2786}
      TaskRecord{c86e175 #2785 A=me.chunyu.spike.wcl_activity_launchmode_demo U=0 sz=2}
        Run #1: ActivityRecord{3558d7c4 u0 me.chunyu.spike.wcl_activity_launchmode_demo/.TestAActivity t2785}
        Run #0: ActivityRecord{1b8620c u0 me.chunyu.spike.wcl_activity_launchmode_demo/.MainActivity t2785}
```

由此可知, 因为startActivityForResult需要返回值, 会保留实例, 覆盖单例效果.

> 注意: 4.x版本通过startActivityForResult启动singleTask, 无法正常获取返回值, [参考](http://stackoverflow.com/questions/8960072/onactivityresult-with-launchmode-singletask).
> 5.x以上版本修复此问题, 考虑兼容性, 不推荐使用startActivityForResult和singleTask.

---

# Intent设置标志位

Intent可以设置启动标志位, 即Flag.
```java
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
```
AndroidManifest无法设置FLAG_ACTIVITY_CLEAR_TOP, 即清除栈上其他实例; Intent无法设置singleInstance启动模式. 两者选其一使用即可, 如都使用, Intent的优先级大于AndroidManifest的优先级.

常用的标志位:
**FLAG_ACTIVITY_NEW_TASK**: 同singleTask启动模式.
**FLAG_ACTIVITY_SINGLE_TOP**: 同singleTop启动模式.
**FLAG_ACTIVITY_CLEAR_TOP**: 一般和singleTask启动模式出现. 如果是singleTask启动模式, 会清除栈上其他实例, 复用实例, 调用onNewIntent; 如果是standard启动模式, 即默认模式, 则会清除自己和其他实例, 并重新创建, 调用 onCreate.

---

# 显示栈的Shell命令

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
```shell
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

启动模式做为Activity的重要属性, 还是需要比较透彻的掌握.

OK, that's all! Enjoy it!

---

> 原始地址: 
> http://www.wangchenlong.org/2016/02/23/1603/234-activity-launch-mode/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

