---
title: 使用 CircularReveal 动画实现页面扩展效果
date: 2016-03-21 07:45:00
categories: [Android]
tags: [Android,CircularReveal,动画]
---

Android的Material Design设计理念, 带来很多绚丽的动画效果. 在页面切换中, 最常用的就是**SharedElementTransition**, 通过设置控件的变换方式, 在进入时把控件变换为页面, 在退出时, 把页面变换为控件, 同时, 可以设置控件移动的轨迹. 这样的控件, 可以应用于**消息通知**, 或者**广告显示**, 提供非常好的用户体验. 那么是如何实现的呢?

<!-- more -->
> 更多: http://www.wangchenlong.org/

![Material Design](1603212-anim-circular-reveal/circle-md.png)

> 随着厂商的版本迭代, 超过三分之一的手机都是5.0以上的操作系统, 随着更多便宜的低端手机普及5.0+系统(如红米系列), 给用户带来更好的体验, 会大大增加应用留存率.

![Android系统(20160227)](1603212-anim-circular-reveal/circle-distrib.png)

本文主要内容:
(1) 修改分享元素的滑动轨迹, 以90°圆弧方式出现和返回, 即transition动画.
(2) 生成和销毁页面的爆炸和凝聚效果, 即CircularReveal的使用方式.

本文源码的GitHub[下载地址](https://github.com/SpikeKing/wcl-circle-reveal-demo).

---

# 首页

新建一个含义Fab按钮的HelloWorld工程. 添加ButterKnife和Lambda库.

设置Fab按钮的跳转事件.
```java
    // Fab的跳转事件
    public void startOtherActivity(View view) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            ActivityOptions options =
                    ActivityOptions.makeSceneTransitionAnimation(this, mFab, mFab.getTransitionName());
            startActivity(new Intent(this, OtherActivity.class), options.toBundle());
        } else {
            startActivity(new Intent(this, OtherActivity.class));
        }
    }
```
> 确保版本号大于5.0, 支持Material Design的动画效果.

设置Fab的TransitionName, 即变化名称.
```xml
    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:clickable="true"
        android:onClick="startOtherActivity"
        android:src="@android:drawable/ic_dialog_email"
        android:transitionName="@string/other_transition_name"
        tools:targetApi="lollipop"/>
```
> 注意android:transitionName属性, 表示变换名称.

---

# 跳转页

页面由背景, 变化控件, 关闭控件, 这三部分组成.
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/other_rl_container"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/transparent">

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/other_fab_circle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:transitionName="@string/other_transition_name"
        app:backgroundTint="@color/colorAccent"
        app:elevation="0dp"
        app:fabSize="normal"
        app:pressedTranslationZ="8dp"
        tools:targetApi="21"/>

    <TextView
        android:id="@+id/other_tv_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorAccent"
        android:gravity="center"
        android:text="@string/other_text"
        android:textColor="@android:color/white"
        android:textSize="40sp"
        android:textStyle="bold"
        android:visibility="invisible"
        tools:visibility="visible"/>

    <ImageView
        android:id="@+id/other_iv_close"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:clickable="true"
        android:contentDescription="@null"
        android:onClick="backActivity"
        android:src="@drawable/ic_close_white"
        android:visibility="invisible"/>

</RelativeLayout>
```

> 注意, 变换控件FloatingActionButton与主页的变换控件有相同的transitionName.

显示逻辑, 设置入场和退场动画, 爆炸的动画效果.
```java
    @Override protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_other);
        ButterKnife.bind(this);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            setupEnterAnimation(); // 入场动画
            setupExitAnimation(); // 退场动画
        } else {
            initViews();
        }
    }
```

退出逻辑, 退出动画, 凝聚的动画效果.
```java
    // 退出按钮
    public void backActivity(View view) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            onBackPressed();
        } else {
            defaultBackPressed();
        }
    }
```

下面仔细分析显示和退场动画.

---

# 显示动画

分享元素切换使用弧度规矩, 即arc_motion, 90°旋转过去和回来. 在动画结束后, 页面显示使用爆炸效果.
```java
    // 入场动画
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void setupEnterAnimation() {
        Transition transition = TransitionInflater.from(this)
                .inflateTransition(R.transition.arc_motion);
        getWindow().setSharedElementEnterTransition(transition);
        transition.addListener(new Transition.TransitionListener() {
            @Override public void onTransitionStart(Transition transition) {

            }

            @Override public void onTransitionEnd(Transition transition) {
                transition.removeListener(this);
                animateRevealShow();
            }

            @Override public void onTransitionCancel(Transition transition) {

            }

            @Override public void onTransitionPause(Transition transition) {

            }

            @Override public void onTransitionResume(Transition transition) {

            }
        });
    }

    // 动画展示
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void animateRevealShow() {
        GuiUtils.animateRevealShow(
                this, mRlContainer,
                mFabCircle.getWidth() / 2, R.color.colorAccent,
                new GuiUtils.OnRevealAnimationListener() {
                    @Override public void onRevealHide() {

                    }

                    @Override public void onRevealShow() {
                        initViews();
                    }
                });
    }

```
arc_motion角度变换
```xml
<transitionSet
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500"
    android:interpolator="@android:interpolator/linear_out_slow_in">

    <changeBounds>
        <!--suppress AndroidElementNotAllowed -->
        <arcMotion
            android:maximumAngle="90"
            android:minimumHorizontalAngle="90"
            android:minimumVerticalAngle="0"/>
    </changeBounds>
</transitionSet>
```
爆炸的动画效果
```java
    // 圆圈爆炸效果显示
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public static void animateRevealShow(
            final Context context, final View view,
            final int startRadius, @ColorRes int color,
            OnRevealAnimationListener listener) {
        int cx = (view.getLeft() + view.getRight()) / 2;
        int cy = (view.getTop() + view.getBottom()) / 2;

        float finalRadius = (float) Math.hypot(view.getWidth(), view.getHeight());

        // 设置圆形显示动画
        Animator anim = ViewAnimationUtils.createCircularReveal(view, cx, cy, startRadius, finalRadius);
        anim.setDuration(300);
        anim.setInterpolator(new AccelerateDecelerateInterpolator());
        anim.addListener(new AnimatorListenerAdapter() {
            @Override public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
                view.setVisibility(View.VISIBLE);
                listener.onRevealShow();
            }

            @Override public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
                view.setBackgroundColor(ContextCompat.getColor(context, color));
            }
        });

        anim.start();
    }
```

> 使用CircularReveal, 即圆形显示的动画效果.
> 第一个参数是显示的视图, 第二个和第三个是变换的中心位置, 第三个和第四个是变换的起始半径和结束半径. 

---

# 退出动画
退出动画是凝聚效果, 同样使用的是CircularReveal.
```java
    // 退出事件
    @Override public void onBackPressed() {
        GuiUtils.animateRevealHide(
                this, mRlContainer,
                mFabCircle.getWidth() / 2, R.color.colorAccent,
                new GuiUtils.OnRevealAnimationListener() {
                    @Override
                    public void onRevealHide() {
                        defaultBackPressed();
                    }

                    @Override
                    public void onRevealShow() {

                    }
                });
    }
```
退出和显示的区别就是起始和终止的半径不同.
```java
    // 圆圈凝聚效果
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public static void animateRevealHide(
            final Context context, final View view,
            final int finalRadius, @ColorRes int color,
            OnRevealAnimationListener listener
    ) {
        int cx = (view.getLeft() + view.getRight()) / 2;
        int cy = (view.getTop() + view.getBottom()) / 2;
        int initialRadius = view.getWidth();
        // 与入场动画的区别就是圆圈起始和终止的半径相反
        Animator anim = ViewAnimationUtils.createCircularReveal(view, cx, cy, initialRadius, finalRadius);
        anim.setDuration(300);
        anim.setInterpolator(new AccelerateDecelerateInterpolator());
        anim.addListener(new AnimatorListenerAdapter() {
            @Override public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
                view.setBackgroundColor(ContextCompat.getColor(context, color));
            }

            @Override public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
                listener.onRevealHide();
                view.setVisibility(View.INVISIBLE);
            }
        });
        anim.start();
    }
```

---

动画

![环形显示](1603212-anim-circular-reveal/circle-anim.gif)

最后说一些有关用户体验的事情, 对于一款应用而言, 好的性能固然重要, 但优秀的设计也非常关键, 优异的产品需要优异的展现方式.

OK, that's all! Enjoy it!

---

**生活**

> 有技术又要有生活, 美让生活更精彩!

[![丝袜](http://7xrsre.com1.z0.glb.clouddn.com/spike-ad-girl-socks-7.jpg)](http://s.click.taobao.com/t?e=m%3D2%26s%3DBgHOOo%2BuGfMcQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67sL74TtmOEySPLNzIt%2Fz56h1lK%2FY7wPaoHeQQxhDmA6IAe67oaxDEWp4DvOxtwmulwa0F0AWKbkgC1LHPteAx8i7Ng8yStHNWC995RSXr%2Fuk&pvid=12_117.73.144.43_332_1458433143248)

女生, 让自己更职业受欢迎! 男生, 送给心中女神或未来女友! [好物](http://s.click.taobao.com/t?e=m%3D2%26s%3DBgHOOo%2BuGfMcQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67sL74TtmOEySPLNzIt%2Fz56h1lK%2FY7wPaoHeQQxhDmA6IAe67oaxDEWp4DvOxtwmulwa0F0AWKbkgC1LHPteAx8i7Ng8yStHNWC995RSXr%2Fuk&pvid=12_117.73.144.43_332_1458433143248)

---

> 原始地址: 
> http://www.wangchenlong.org/2016/03/21/1603/212-anim-circular-reveal/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

