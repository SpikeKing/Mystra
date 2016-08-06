---
title: Android Tips 8
date: 2016-05-18 10:22:00
categories: [Tips]
tags: [Android,Tips]
---

本文是Tips的第8节, 记录一些有趣的知识点, 再加一些有用的代码段, 精心准备, 来源于实践.

<!-- more -->
> 更多: http://www.wangchenlong.org/
> 欢迎Follow我的GitHub: https://github.com/SpikeKing

![Android](181-android-tips-8/android-tips.png)

其余: [第一篇](http://www.wangchenlong.org/2016/02/23/tips/1603/231-android-tips-1/), [第二篇](http://www.wangchenlong.org/2016/02/23/tips/1603/232-android-tips-2/), [第三篇](http://www.wangchenlong.org/2016/02/23/tips/1603/233-android-tips-3/), [第四篇](http://www.wangchenlong.org/2016/02/24/tips/1603/241-android-tips-4/), [第五篇](http://www.wangchenlong.org/2016/02/24/tips/1603/242-android-tips-5/), [第六篇](http://www.wangchenlong.org/2016/02/25/tips/1605/071-android-tips-6/), [第七篇](http://www.wangchenlong.org/2016/02/25/tips/1605/072-android-tips-7/), [第八篇](http://www.wangchenlong.org/2016/05/18/tips/1605/181-android-tips-8/).

---

## Android 5.0 Status Bar 图标显示白色方块

Android 5.0 是样式改版, 引入``Material Design``, 统一风格, 其中就包含``状态栏(Status Bar)``的风格统一. 所有图标, 均以相同颜色覆盖, 如果是方形图标, 就会显示一个**均色方块**, 需要重新设计, 非重点部分使用透明处理.

代码位置不变, 仍是setSmallIcon方法.

``` java
builder.setSmallIcon(R.drawable.notification_icon);
```

---

## Gradle Daemon 异常

``AS 2.0`` + ``Gradle 2.10``, 报错: 

``` bash
To run dex in process, the Gradle daemon needs a larger heap.
```

原因: 在``Gradle 2.10``中, Dex运行在Gradle构建进程而不是分离进程.

> Dex runs inside gradle build process as opposed to a separate process.

解决方案:

增大守护进程的内存, 在``gradle.properties``中设置

``` gradle
org.gradle.jvmargs=-Xmx4g -XX:MaxPermSize=512m
```

> JVM最大使用4G内存, 每次增大512M. 

在build.gradle中设置

``` gradle
android {
    dexOptions {
        preDexLibraries true
        javaMaxHeapSize "3g"
        incremental true
        dexInProcess = true
    }
}
```

> 允许预增Dex, Dex的大小是3G, 允许Dex在进程中运行.

或 **直接屏蔽**.

``` gradle
android {
	 dexOptions {
        dexInProcess = false
    }
}
```

> Dex不在进程中运行.

---

## 毫秒转换具体时间

时间(小时:分钟:秒), 使用``TimeUnit.MILLISECONDS``工具转换.

``` java
/**
 * 转换毫秒到具体时间, 小时:分钟:秒
 * 参考: http://stackoverflow.com/questions/625433/how-to-convert-milliseconds-to-x-mins-x-seconds-in-java
 *
 * @param millis 毫秒
 * @return 时间字符串
 */
public static String convertMillis2Time(long millis) {
    return String.format("%02d:%02d:%02d",
            TimeUnit.MILLISECONDS.toHours(millis),
            TimeUnit.MILLISECONDS.toMinutes(millis) - TimeUnit.HOURS.toMinutes(TimeUnit.MICROSECONDS.toHours(millis)),
            TimeUnit.MILLISECONDS.toSeconds(millis) - TimeUnit.HOURS.toSeconds(TimeUnit.MICROSECONDS.toHours(millis)) - TimeUnit.MINUTES.toSeconds(TimeUnit.MILLISECONDS.toMinutes(millis))
    );
}
```

---

## 判断服务是否启动

**服务检测**比较具有迷惑性, 不能直接通过类名检查, 一定要判断**UID**是否相同, 否则多个应用使用相同的服务, 会出现检查错误, 有一个启动就会成功. 添加UID检查, 才可以正确使用.

用于在重启动服务时, 进行服务保活, 防止重复启动.

``` java
/**
 * 判断服务是否启动, 注意只要名称相同, 会检测任何服务.
 *
 * @param context      上下文
 * @param serviceClass 服务类
 * @return 是否启动服务
 */
public static boolean isServiceRunning(Context context, Class<?> serviceClass) {
    if (context == null) {
        return false;
    }

    Context appContext = context.getApplicationContext();
    ActivityManager manager = (ActivityManager) appContext.getSystemService(Context.ACTIVITY_SERVICE);

    if (manager != null) {
        List<ActivityManager.RunningServiceInfo> infos = manager.getRunningServices(Integer.MAX_VALUE);
        if (infos != null && !infos.isEmpty()) {
            for (ActivityManager.RunningServiceInfo service : infos) {
                // 添加Uid验证, 防止服务重名, 当前服务无法启动
                if (getUid(context) == service.uid) {
                    if (serviceClass.getName().equals(service.service.getClassName())) {
                        return true;
                    }
                }
            }
        }
    }
    return false;
}

/**
 * 获取应用的Uid, 用于验证服务是否启动
 *
 * @param context 上下文
 * @return uid
 */
public static int getUid(Context context) {
    if (context == null) {
        return -1;
    }

    int pid = android.os.Process.myPid();
    ActivityManager manager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);

    if (manager != null) {
        List<ActivityManager.RunningAppProcessInfo> infos = manager.getRunningAppProcesses();
        if (infos != null && !infos.isEmpty()) {
            for (ActivityManager.RunningAppProcessInfo processInfo : infos) {
                if (processInfo.pid == pid) {
                    return processInfo.uid;
                }
            }
        }
    }
    return -1;
}
```

---

## 午夜定时器

午夜定时器, 午夜12点发送广播. 用于计步器的日期更新, 或者其他与本地日期有关的功能.

``` java
/**
 * 设置午夜定时器, 午夜12点发送广播, MIDNIGHT_ALARM_FILTER.
 * 实际测试可能会有一分钟左右的偏差.
 *
 * @param context 上下文
 */
public static void setMidnightAlarm(Context context) {
    Context appContext = context.getApplicationContext();
    Intent intent = new Intent(IntentConsts.MIDNIGHT_ALARM_FILTER);

    PendingIntent pi = PendingIntent.getBroadcast(appContext, 0, intent, PendingIntent.FLAG_CANCEL_CURRENT);
    AlarmManager am = (AlarmManager) appContext.getSystemService(Context.ALARM_SERVICE);

    // 午夜12点的标准计时, 来源于SO, 实际测试可能会有一分钟左右的偏差.
    Calendar calendar = Calendar.getInstance();
    calendar.setTimeInMillis(System.currentTimeMillis());
    calendar.set(Calendar.SECOND, 0);
    calendar.set(Calendar.MINUTE, 0);
    calendar.set(Calendar.HOUR, 0);
    calendar.set(Calendar.AM_PM, Calendar.AM);
    calendar.add(Calendar.DAY_OF_MONTH, 1);

    // 显示剩余时间
    long now = Calendar.getInstance().getTimeInMillis();
    showLogs("剩余时间(秒): " + ((calendar.getTimeInMillis() - now) / 1000));

    // 设置之前先取消前一个PendingIntent
    am.cancel(pi);

    // 设置每一天的计时器
    am.setRepeating(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), AlarmManager.INTERVAL_DAY, pi);
}
```

---

## 非重复数

用于多个通知的ID, 或者其他非重复ID.

``` java
// 获取通知的ID, 防止重复, 可以用于通知的ID
public static class NotificationID {
    // 随机生成一个数
    private final static AtomicInteger c = new AtomicInteger(0);

    // 获取一个不重复的数, 从0开始
    public static int getID() {
        return c.incrementAndGet() 
    }
}
```

---

## 检测屏幕是否开启

除了此方法, 也可以通过监听系统广播, 判断屏幕的亮灭, 即``Intent.ACTION_SCREEN_ON``或``Intent.ACTION_SCREEN_OFF``.


``` java
/**
 * 检测屏幕是否开启
 *
 * @param context 上下文
 * @return 是否屏幕开启
 */
public static boolean isScreenOn(Context context) {
    Context appContext = context.getApplicationContext();
    PowerManager pm = (PowerManager) appContext.getSystemService(Context.POWER_SERVICE);

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT_WATCH) {
        return pm.isInteractive();
    } else {
        // noinspection all
        return pm.isScreenOn();
    }
}
```

---

## 判断计步传感器是否可用

判断的时候需要注意, 毕竟与硬件相关, 随时可能出现异常, 而且每个厂商都不同, 需要综合考虑.

``` java
/**
 * 检测计步传感器是否可以使用
 *
 * @param context 上下文
 * @return 是否可用计步传感器
 */
public static boolean hasStepSensor(Context context) {
    if (context == null) {
        return false;
    }

    Context appContext = context.getApplicationContext();
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT) {
        return false;
    } else {
        boolean hasSensor = false;
        Sensor sensor = null;
        try {
            hasSensor = appContext.getPackageManager().hasSystemFeature("android.hardware.sensor.stepcounter");
            SensorManager sm = (SensorManager) appContext.getSystemService(Context.SENSOR_SERVICE);
            sensor = sm.getDefaultSensor(Sensor.TYPE_STEP_COUNTER);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return hasSensor && sensor != null;
    }
}
```

---

## 获取进程名称

获取当前进程名称, 用于进程保活.

``` java
/**
 * 获取进程名称
 *
 * @param context 上下文
 * @return 进程名称
 */
public static String getProcessName(Context context) {
    int pid = android.os.Process.myPid();
    ActivityManager manager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    List<ActivityManager.RunningAppProcessInfo> infos = manager.getRunningAppProcesses();
    if (infos != null) {
        for (ActivityManager.RunningAppProcessInfo processInfo : infos) {
            if (processInfo.pid == pid) {
                return processInfo.processName;
            }
        }
    }
    return null;
}
```

---

## 检测应用是否运行

判断应用是否存活, 通过唯一包名判断.

``` java
/**
 * 检测应用是否运行
 *
 * @param packageName 包名
 * @param context     上下文
 * @return 是否存在
 */
public static boolean isAppAlive(String packageName, Context context) {
    if (context == null || TextUtils.isEmpty(packageName)) {
        return false;
    }

    ActivityManager activityManager = (ActivityManager)
            context.getSystemService(Context.ACTIVITY_SERVICE);

    if (activityManager != null) {
        List<ActivityManager.RunningAppProcessInfo> procInfos = activityManager.getRunningAppProcesses();
        if (procInfos != null && !procInfos.isEmpty()) {
            for (int i = 0; i < procInfos.size(); i++) {
                if (procInfos.get(i).processName.equals(packageName)) {
                    return true;
                }
            }
        }
    }

    return false;
}
```

---

## 转换16进制字符串

含有字母的字符串, 转换16进制的数字字符串, 用于鉴别判断.

``` java
/**
 * String转换40位16进制.
 *
 * @param arg 字母字符串
 * @return 16进制数字字符串
 */
private String toHex(String arg) {
    if (TextUtils.isEmpty(arg)) {
        return null;
    }
    return String.format("%040x", new BigInteger(1, arg.getBytes(/*YOUR_CHARSET?*/)));
}
```

---

OK, that's all! Enjoy it!

---

> 最初发布地址: 
> http://www.wangchenlong.org/2016/05/18/tips/1605/181-android-tips-8/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

