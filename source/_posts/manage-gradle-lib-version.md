---
title: 大型项目 Gradle 的常用库和版本管理
date: 2016-03-15 06:44:00
categories: [Android]
tags: [Android,Gradle]
---

随着Android开发的成熟, 模块越来越多, 引入库也随之增加, 需要统一管理这些库和版本号. 根据自己的开发经验, 本文介绍使用Gradle参数配置实现库的规范管理.

<!-- more -->
> 更多: http://www.wangchenlong.org/

![Gradle](manage-gradle-lib-version/manage-gradle.png)

主要
(1) 常用库的展示与配置.
(2) 统一管理项目和库的版本.
(3) 设置项目的私有参数.

---

# 常用库

编程三剑客, RxJava+Retrofit+Dagger. 
常用: ButterKnife依赖注解, Glide/Picasso图片处理.
使用根项目(rootProject)的参数管理子项目的版本.
```gradle
apply plugin: 'me.tatarka.retrolambda'      // Lambda表达式
apply plugin: 'com.android.application'     // Android应用
apply plugin: 'com.neenbedankt.android-apt' // 编译时类
apply plugin: 'com.android.databinding'     // 数据绑定

def cfg = rootProject.ext.configuration // 配置
def libs = rootProject.ext.libraries // 库

android {
    compileSdkVersion cfg.compileVersion
    buildToolsVersion cfg.buildToolsVersion

    defaultConfig {
        applicationId cfg.package
        minSdkVersion cfg.minSdk
        targetSdkVersion cfg.targetSdk
        versionCode cfg.version_code
        versionName cfg.version_name

        buildConfigField "String", "MARVEL_PUBLIC_KEY", "\"${marvel_public_key}\""
        buildConfigField "String", "MARVEL_PRIVATE_KEY", "\"${marvel_private_key}\""
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    // 注释冲突
    packagingOptions {
        exclude 'META-INF/services/javax.annotation.processing.Processor'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'

    // Android
    compile "com.android.support:design:${libs.supportVersion}"
    compile "com.android.support:appcompat-v7:${libs.supportVersion}"
    compile "com.android.support:cardview-v7:${libs.supportVersion}"
    compile "com.android.support:recyclerview-v7:${libs.supportVersion}"
    compile "com.android.support:palette-v7:${libs.supportVersion}"

    // Retrofit
    compile "com.squareup.retrofit:retrofit:${libs.retrofit}"
    compile "com.squareup.retrofit:converter-gson:${libs.retrofit}"
    compile "com.squareup.retrofit:adapter-rxjava:${libs.retrofit}"

    // ReactiveX
    compile "io.reactivex:rxjava:${libs.rxandroid}"
    compile "io.reactivex:rxandroid:${libs.rxandroid}"

    // Dagger
    compile "com.google.dagger:dagger:${libs.dagger}"
    apt "com.google.dagger:dagger-compiler:${libs.dagger}"
    compile "org.glassfish:javax.annotation:${libs.javax_annotation}"

    // Others
    compile "com.jakewharton:butterknife:${libs.butterknife}" // 资源注入
    compile "com.github.bumptech.glide:glide:${libs.glide}" // 图片处理
    compile "jp.wasabeef:recyclerview-animators:${libs.recycler_animators}" // Recycler动画
    compile "de.hdodenhof:circleimageview:${libs.circleimageview}" // 头像视图
}
```

> 项目版本:
> def cfg = rootProject.ext.configuration
> cfg.compileVersion
> 库版本:
> def libs = rootProject.ext.libraries
> ${libs.retrofit}

---

# 参数管理
buildConfigField管理私有参数, 配置在gradle.properties里面.
```gradle
android {
    defaultConfig {
        buildConfigField "String", "MARVEL_PUBLIC_KEY", "\"${marvel_public_key}\""
        buildConfigField "String", "MARVEL_PRIVATE_KEY", "\"${marvel_private_key}\""
    }
}
```

> 设置参数的**类型\变量名\位置**三个部分.

```gradle
marvel_public_key   = 74129ef99c9fd5f7692608f17abb88f9
marvel_private_key  = 281eb4f077e191f7863a11620fa1865f2940ebeb
```

> 未指定路径, 默认是配置在**gradle.properties**中.
> 两个地方可以配置参数, 一个是项目的build.gradle, 一个是gradle.properties.

项目中使用**BuildConfig.xxx**引入参数.
```gradle
        MarvelSigningIterceptor signingIterceptor = new MarvelSigningIterceptor(
                BuildConfig.MARVEL_PUBLIC_KEY, BuildConfig.MARVEL_PRIVATE_KEY);
```

---

# 版本管理
版本管理配置在项目的build.gradle中, 包含两个部分, 一个是项目的版本, 一个是库的版本. 把常用参数设置成为变量. 子项目使用**rootProject.ext.xxx**的形式引入.
```gradle
ext {
    configuration = [
            package          : "me.chunyu.spike.springrainnews",
            buildToolsVersion: "23.0.1",
            compileVersion   : 23,
            minSdk           : 14,
            targetSdk        : 23,
            version_code     : 1,
            version_name     : "0.0.1",
    ]

    libraries = [
            supportVersion    : "23.1.1",
            retrofit          : "2.0.0-beta2",
            rxandroid         : "1.1.0",
            dagger            : "2.0",
            javax_annotation  : "10.0-b28",
            butterknife       : "7.0.1",
            glide             : "3.6.1",
            recycler_animators: "2.1.0",
            circleimageview   : "2.0.0"
    ]
}

buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:2.0.0-alpha5'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        classpath 'me.tatarka:gradle-retrolambda:3.2.4'
        classpath 'com.android.databinding:dataBinder:1.0-rc4'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

---

# 补充
Retrolambda的最新配置方式
```gradle
plugins {
  id "me.tatarka.retrolambda" version "3.2.5"
}

android {
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

---

通过这些方式设置Android项目的Gradle配置, 可以便捷地修改库的版本号, 统一管理并控制风险.

OK, that's all! Enjoy it!

> 原始地址: 
> http://www.wangchenlong.org/2016/03/15/manage-gradle-lib-version/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动, 未经授权, 禁止转载, 授权或合作请留言.
