---
title: 实现翻转卡片的动画效果
date: 2016-03-21 19:59:00
categories: [Android]
tags: [Android,动画]
---

在Android设计中, 经常会使用**卡片元素**, 正面显示图片或主要信息, 背面显示详细内容, 如**网易有道词典的单词翻转**和**海底捞的食谱展示**. 实现卡片视图非常容易, 那么如何实现翻转动画呢?

<!-- more -->
> 更多: http://www.wangchenlong.org/

![Card](1603213-implement-flip-card/card-logo.png)

> 在TB吃海底捞时, 使用Pad点餐, 发现应用的食谱功能使用卡片控件, 就准备和大家分享一下实现方式. 有兴趣的朋友可以去海底捞看看:)

本文源码的GitHub[下载地址](https://github.com/SpikeKing/wcl-flip-anim-demo)

---

# 首页

首页由正面和背面两张卡片组成, 同时, 设置点击事件**flipCard**.
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    android:id="@+id/main_fl_container"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:onClick="flipCard"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="me.chunyu.spike.wcl_flip_anim_demo.MainActivity">

    <include
        layout="@layout/cell_card_back"/>

    <include
        layout="@layout/cell_card_front"/>

</FrameLayout>
```

逻辑, 初始化动画和镜头距离.
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        setAnimators(); // 设置动画
        setCameraDistance(); // 设置镜头距离
    }
```

---

# 动画

初始化**右出(RightOut)**和**左入(LeftIn)**动画, 使用动画集合AnimatorSet.
当右出动画开始时, 点击事件无效, 当左入动画结束时, 点击事件恢复.
```java
    // 设置动画
    private void setAnimators() {
        mRightOutSet = (AnimatorSet) AnimatorInflater.loadAnimator(this, R.animator.anim_out);
        mLeftInSet = (AnimatorSet) AnimatorInflater.loadAnimator(this, R.animator.anim_in);

        // 设置点击事件
        mRightOutSet.addListener(new AnimatorListenerAdapter() {
            @Override public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
                mFlContainer.setClickable(false);
            }
        });
        mLeftInSet.addListener(new AnimatorListenerAdapter() {
            @Override public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
                mFlContainer.setClickable(true);
            }
        });
    }
```

右出动画
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!--旋转-->
    <objectAnimator
        android:duration="@integer/anim_length"
        android:propertyName="rotationY"
        android:valueFrom="0"
        android:valueTo="180"/>

    <!--消失-->
    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:startOffset="@integer/anim_half_length"
        android:valueFrom="1.0"
        android:valueTo="0.0"/>
</set>
```
> 旋转180°, 当旋转一半时, 卡片消失.

左入动画
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <!--消失-->
    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:valueFrom="1.0"
        android:valueTo="0.0"/>

    <!--旋转-->
    <objectAnimator
        android:duration="@integer/anim_length"
        android:propertyName="rotationY"
        android:valueFrom="-180"
        android:valueTo="0"/>

    <!--出现-->
    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:startOffset="@integer/anim_half_length"
        android:valueFrom="0.0"
        android:valueTo="1.0"/>
</set>
```

> 在开始时是隐藏, 逆向旋转, 当旋转一半时, 显示卡片.

---

# 镜头视角

改变视角, 涉及到旋转卡片的Y轴, 即rotationY, 需要修改视角距离.
如果不修改, 则会超出屏幕高度, 影响视觉体验.
```java
    // 改变视角距离, 贴近屏幕
    private void setCameraDistance() {
        int distance = 16000;
        float scale = getResources().getDisplayMetrics().density * distance;
        mFlCardFront.setCameraDistance(scale);
        mFlCardBack.setCameraDistance(scale);
    }
```

---

# 旋转控制

设置右出和左入动画的目标控件, 两个动画同步进行, 并区分正反面朝上.
```java
    // 翻转卡片
    public void flipCard(View view) {
        // 正面朝上
        if (!mIsShowBack) {
            mRightOutSet.setTarget(mFlCardFront);
            mLeftInSet.setTarget(mFlCardBack);
            mRightOutSet.start();
            mLeftInSet.start();
            mIsShowBack = true;
        } else { // 背面朝上
            mRightOutSet.setTarget(mFlCardBack);
            mLeftInSet.setTarget(mFlCardFront);
            mRightOutSet.start();
            mLeftInSet.start();
            mIsShowBack = false;
        }
    }
```

---

动画效果

![旋转卡片](1603213-implement-flip-card/card-anim.gif)


动画效果非常简单, 全部逻辑都不足50行, 这么好玩的动画, 用起来吧.

OK, that's all! Enjoy it!

---

**生活**

> 有技术又要有生活, 美让生活更精彩!

[![丝袜](http://7xrsre.com1.z0.glb.clouddn.com/spike-ad-girl-socks-8.jpg)](http://s.click.taobao.com/t?e=m%3D2%26s%3DkuMRvLgFfW4cQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67l7Rhd61ZfyxUQTSx8a5hQd1lK%2FY7wPaoHeQQxhDmA6IAe67oaxDEWp4DvOxtwmul3VDMpR1XpI6%2BC4BVs0yhNE9WzFfQPcoCcYMXU3NNCg%2F&pvid=12_117.73.144.43_332_1458433143248)

女生, 让自己更职业受欢迎! 男生, 送给心中女神或未来女友! [好物](http://s.click.taobao.com/t?e=m%3D2%26s%3DkuMRvLgFfW4cQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67l7Rhd61ZfyxUQTSx8a5hQd1lK%2FY7wPaoHeQQxhDmA6IAe67oaxDEWp4DvOxtwmul3VDMpR1XpI6%2BC4BVs0yhNE9WzFfQPcoCcYMXU3NNCg%2F&pvid=12_117.73.144.43_332_1458433143248)

---

> 原始地址: 
> http://www.wangchenlong.org/2016/03/21/1603/213-implement-flip-card/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

