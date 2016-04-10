---
title: 运行 React Native 的开源工程
date: 2016-04-11 07:26:00
categories: [ReactNative]
tags: [ReactNative]
---

在学习 React Native 的过程中, 需要多学习其他人的代码, 则运行其他项目必不可少. 完整的 ``React Native`` 包含两个部分, ``Android`` 和 ``iOS`` .  本文介绍一些在运行时的注意事项. 

<!-- more -->
> 更多: http://www.wangchenlong.org/
> 欢迎Follow我的GitHub: https://github.com/SpikeKing

![React Native](111-rn-open-project/open-project-logo.png)

---

## 运行 iOS 版

React Native 的规范或编码更偏向于 iOS, 同时一些开源项目仅支持 iOS版本. 因此, 在运行时, 优先从 iOS 版本入手.

安装NPM, ``npm install``, 下载 Node 库的依赖.

> 未安装库, 错误: ``'RCTRootView.h' file not found``

进入工程目录的 ``ios`` 文件夹, 启动``xxx.xcodeproj``文件. 使用 ``Xcode`` 启动, 或 使用 ``open xxx.xcodeproj`` 均可. 

直接运行项目, 即可显示. iOS 工程会自启动服务, 加载本地数据.

### 错误

#### **``react-tools → aft``无法下载**

依赖的 RN 版本过低. 升级 RN 的版本, 即可, 目前低于版本17, 均无法下载 ``aft`` . RN 还在迭代过程中, 低版本不会向下兼容, 要抱有探索精神.

``` bash
loadDep:react-tools → aft ▌ ╢██████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
```

依赖 RN 的位置, ``package.json`` , 修改 ``react-native`` 项为较高版本. 高版本与低版本可能会不兼容.

> 在版本号前需要添加``^``, 如``^0.18.0``.


``` json
{
  "name": "WeatherProject",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node_modules/react-native/packager/packager.sh"
  },
  "dependencies": {
    "react-native": "^0.18.0"
  }
}
```

> RN 的版本号列表, [参考](https://github.com/facebook/react-native/tags)

#### **低版本兼容**

Log类接口修改, 添加 ``RCTLogSource source`` 即可.

``` objc
RCTSetLogFunction(^(RCTLogLevel level, RCTLogSource source, NSString *fileName, NSNumber *lineNumber, NSString *message)
```

---

## 运行 Android 版

多数 Demo 不包含 Android 版本. 直接于项目中, 观察 ``index.android.js`` 文件. 如果文件中添加新的模块, 则表示支持 Android 版; 如果是默认的``HelloWorld``, 则表示不支持. 不支持就不必启动, 使用 iOS 版本即可.

不支持 Android 版本的原因, 主要是 React Native 的控件有一部分是有区分的, 专门开发 Android 版或 iOS 版. 默认控件通过后缀区分, **IOS** 表示 iOS 独有, **Android** 表示 Android 独有, 未添加则表示两者均可.

使用 ``react-native`` 命令启动 Android 项目.

```
react-native run-android
```

即shell命令: ``cd android && ./gradlew installDebug``

注意修改 Gradle 的构建版本

```
classpath 'com.android.tools.build:gradle:1.2.3'
```

> 否则报错: ``Error while uploading app-debug.apk : Unknown failure``

低版本 RN 运行 Android 项目, 问题多多, 请耐心解决. 推荐使用 iOS 进行学习与演示.

---

运行项目只是开始, 要想熟练使用, 还要多多练习 RN 的编码风格.

OK, that's all! Enjoy it!

---

> 原始地址: 
> http://www.wangchenlong.org/2016/04/11/1604/111-rn-open-project/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

