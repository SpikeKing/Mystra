---
title: 提取应用的签名和包名
date: 2016-03-12 18:20:07
categories: [Android]
tags: [Android,私钥,签名,MD5,SHA1,SHA256,包名]
---

Android应用在使用第三方的库时, 可能需要**申请密钥**, 表明应用身份, 如高德定位SDK等. **应用签名(printcert)**是公开的, 只要下载到Apk包, 就可以公开提取. 签名中包含**MD5**, **SHA1**, **SHA256**. **应用唯一性**就是表现为**签名+包名**, 就像人的指纹一样重要, 是确定应用属性的重要信息, 也是**应用商店**检测盗版应用的途径. 

本文讲解如何提取应用的**签名和包名**.

<!-- more -->
> 更多: http://www.wangchenlong.org/

---

![指纹](get-app-cert-and-package/fingerprint.jpg)

# 签名

获取签名包含两种方式:
**(1) Keystore**
**系统默认签名**: 存放位置: ``~/.android/debug.keystore``.
日常测试应用的签名, 均来自于此, 提取密钥.
```
keytool -list -v -keystore debug.keystore
```
输入默认密钥库口令: **android**
即可显示
```
证书指纹:
	 MD5: 97:0B:1C:...
	 SHA1: 47:DF:70:...
	 SHA256: 83:F9:04:...
	 签名算法名称: SHA256withRSA
	 版本: 3
```

**自定义签名**: 进入到存放keystore的文件夹，使用命令:
```
keytool -list -v -keystore [xxx] -keypass [xxx]
```
显示默认签名类似的效果.

**(2) RSA**
已经编译成Apk的包, 我们无法获取Keystore, 但是可以在RSA中获取签名.
修改Apk包的后缀名, 从".apk"变为"**.zip**", 解压缩.
进入**META-INF**文件夹, 即``cd META-INF``.
使用命令
```
keytool -printcert -file CERT.RSA
```
即可, 显示**Apk的签名**.

获取**MD5**, **SHA1**, **SHA256**.

---

# 包名

查看包名就一行命令, 显示Apk的信息.
```
aapt dump badging [xxx.apk]
```
输出, **package: name**, 即包名.
```
package: name='xxx.xxx.xxxxx' 
...
```

**注:** 也可以修改本地包名, 匹配已经存在的密匙. 
修改应用**包名**的方法, 在**build.gradle**中, 添加**applicationId**, 即
```gradle
android {
    defaultConfig {
        applicationId "com.amap.location.demo"
        // ...
    }
}
```
> 修改包名为**com.amap.location.demo**.

未添加gradle的参数, 默认位置是**AndroidManifest.xml**, 
其中manifest的package属性, 表示包名, 即
```xml
<manifest package="wangchenlong.chunyu.me.wcl_amap_demo">
```

> **build.gradle**的包名属性优先级高于**manifest**, 其他属性也是一样.

---

在第三方库的开发者平台输入签名和包名, 就可以生成唯一密钥, 放到程序中, 就可以使用库了. 

> **签名+包名**, 表明Apk的唯一身份, 防止盗版仿冒的Apk出现, 是Android的安全机制.

**PS:** 公司经常使用第三方库, 需要申请唯一的账号, 统一管理; 作为开发者, 在测试时, 也需要申请一些测试Key, 加快开发速度. 

OK, that's all! Enjoy it.

> 原始地址: 
> http://www.wangchenlong.org/2016/03/12/get-app-cert-and-package/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.