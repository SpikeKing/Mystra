---
title: 实现漫天飞雪的动画效果
date: 2016-03-22 06:36:00
categories: [Android]
tags: [Android,动画]
---

冬天来了, 大雪纷飞, 好冷啊. 在应用里, 也可以实现漫天飞雪的动画, 让我来介绍一下吧.

<!-- more -->
> 更多: http://www.wangchenlong.org/

![下雪](222-snowflake-anim/snow-logo.png)

我们的应用也可以有一些冬天的效果, 教大家做一个下雪的动画效果, [参考](http://www.openprocessing.org/sketch/84771).

主要
(1) 隐藏``status bar``, 全屏显示图片.
(2) 绘制多个点, 重绘页面, 实现移动效果.
(3) 回收点, 避免重复创建.

我喜欢用注释说话, 请大家多关注代码中的注释.

本文源码的GitHub[下载地址](https://github.com/SpikeKing/wcl-snowfall-demo)

> 欢迎Follow我的GitHub: https://github.com/SpikeKing

![动画](222-snowflake-anim/snow-anim.gif)

---

# 雪花类

雪花的属性包含: 随机量, 位置, 增量, 大小, 角度, 画笔.
绘画的过程中, 使用角度会移动点的位置, 每次速率都不同. 
当雪花移出屏幕时, 会重新使用, 在屏幕的顶端重新落下.
[算法参考](http://www.openprocessing.org/sketch/84771).

```java
/**
 * 雪花的类, 移动, 移出屏幕会重新设置位置.
 * <p/>
 * Created by wangchenlong on 16/1/24.
 */
public class SnowFlake {
    // 雪花的角度
    private static final float ANGE_RANGE = 0.1f; // 角度范围
    private static final float HALF_ANGLE_RANGE = ANGE_RANGE / 2f; // 一般的角度
    private static final float HALF_PI = (float) Math.PI / 2f; // 半PI
    private static final float ANGLE_SEED = 25f; // 角度随机种子
    private static final float ANGLE_DIVISOR = 10000f; // 角度的分母

    // 雪花的移动速度
    private static final float INCREMENT_LOWER = 2f;
    private static final float INCREMENT_UPPER = 4f;

    // 雪花的大小
    private static final float FLAKE_SIZE_LOWER = 7f;
    private static final float FLAKE_SIZE_UPPER = 20f;

    private final RandomGenerator mRandom; // 随机控制器
    private final Point mPosition; // 雪花位置
    private float mAngle; // 角度
    private final float mIncrement; // 雪花的速度
    private final float mFlakeSize; // 雪花的大小
    private final Paint mPaint; // 画笔

    private SnowFlake(RandomGenerator random, Point position, float angle, float increment, float flakeSize, Paint paint) {
        mRandom = random;
        mPosition = position;
        mIncrement = increment;
        mFlakeSize = flakeSize;
        mPaint = paint;
        mAngle = angle;
    }

    public static SnowFlake create(int width, int height, Paint paint) {
        RandomGenerator random = new RandomGenerator();
        int x = random.getRandom(width);
        int y = random.getRandom(height);
        Point position = new Point(x, y);
        float angle = random.getRandom(ANGLE_SEED) / ANGLE_SEED * ANGE_RANGE + HALF_PI - HALF_ANGLE_RANGE;
        float increment = random.getRandom(INCREMENT_LOWER, INCREMENT_UPPER);
        float flakeSize = random.getRandom(FLAKE_SIZE_LOWER, FLAKE_SIZE_UPPER);
        return new SnowFlake(random, position, angle, increment, flakeSize, paint);
    }

    // 绘制雪花
    public void draw(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        move(width, height);
        canvas.drawCircle(mPosition.x, mPosition.y, mFlakeSize, mPaint);
    }

    // 移动雪花
    private void move(int width, int height) {
        double x = mPosition.x + (mIncrement * Math.cos(mAngle));
        double y = mPosition.y + (mIncrement * Math.sin(mAngle));

        mAngle += mRandom.getRandom(-ANGLE_SEED, ANGLE_SEED) / ANGLE_DIVISOR; // 随机晃动

        mPosition.set((int) x, (int) y);

        // 移除屏幕, 重新开始
        if (!isInside(width, height)) {
            reset(width);
        }
    }

    // 判断是否在其中
    private boolean isInside(int width, int height) {
        int x = mPosition.x;
        int y = mPosition.y;
        return x >= -mFlakeSize - 1 && x + mFlakeSize <= width && y >= -mFlakeSize - 1 && y - mFlakeSize < height;
    }

    // 重置雪花
    private void reset(int width) {
        mPosition.x = mRandom.getRandom(width);
        mPosition.y = (int) (-mFlakeSize - 1); // 最上面
        mAngle = mRandom.getRandom(ANGLE_SEED) / ANGLE_SEED * ANGE_RANGE + HALF_PI - HALF_ANGLE_RANGE;
    }
}
```

随机数生成器, 包含区间随机和上界随机.

```java
/**
 * 随机生成器
 * <p/>
 * Created by wangchenlong on 16/1/24.
 */
public class RandomGenerator {
    private static final Random RANDOM = new Random();

    // 区间随机
    public float getRandom(float lower, float upper) {
        float min = Math.min(lower, upper);
        float max = Math.max(lower, upper);
        return getRandom(max - min) + min;
    }

    // 上界随机
    public float getRandom(float upper) {
        return RANDOM.nextFloat() * upper;
    }

    // 上界随机
    public int getRandom(int upper) {
        return RANDOM.nextInt(upper);
    }
}
```

---

# 雪花视图

雪花视图, DELAY时间重绘, 绘制NUM_SNOWFLAKES个雪花.
初始化在onSizeChanged中进行, 绘制在onDraw中进行.

```java
/**
 * 雪花视图, DELAY时间重绘, 绘制NUM_SNOWFLAKES个雪花
 * <p/>
 * Created by wangchenlong on 16/1/24.
 */
public class SnowView extends View {

    private static final int NUM_SNOWFLAKES = 150; // 雪花数量
    private static final int DELAY = 5; // 延迟
    private SnowFlake[] mSnowFlakes; // 雪花

    public SnowView(Context context) {
        super(context);
    }

    public SnowView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public SnowView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @TargetApi(21)
    public SnowView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }

    @Override protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (w != oldw || h != oldh) {
            initSnow(w, h);
        }
    }

    private void initSnow(int width, int height) {
        Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG); // 抗锯齿
        paint.setColor(Color.WHITE); // 白色雪花
        paint.setStyle(Paint.Style.FILL); // 填充;
        mSnowFlakes = new SnowFlake[NUM_SNOWFLAKES];
        for (int i = 0; i < NUM_SNOWFLAKES; ++i) {
            mSnowFlakes[i] = SnowFlake.create(width, height, paint);
        }
    }

    @Override protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        for (SnowFlake s : mSnowFlakes) {
            s.draw(canvas);
        }
        // 隔一段时间重绘一次, 动画效果
        getHandler().postDelayed(runnable, DELAY);
    }

    // 重绘线程
    private Runnable runnable = new Runnable() {
        @Override
        public void run() {
            invalidate();
        }
    };
}
```

> 使用``getHandler().postDelayed(runnable, DELAY);``刷新页面.

---

# 全屏布局

全屏布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CollapsingToolbarLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:contentDescription="@null"
            android:scaleType="centerCrop"
            android:src="@drawable/christmas"/>

        <me.chunyu.spike.wcl_snowfall_demo.views.SnowView
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

    </FrameLayout>

</android.support.design.widget.CollapsingToolbarLayout>
```

> ``status bar``默认是不会被透明化的, 需要使用``CollapsingToolbarLayout``, 
> 替换status bar的样式, 否则会留有一定高度, 即使透明也不会填充.

样式

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme.NoStatusBar">
        <item name="android:windowTranslucentStatus">true</item>
    </style>
</resources>
```

---

[参考](https://blog.stylingandroid.com/)

可以在冬天的时候, 为应用添加些有趣的东西, 让编程更有趣!

OK, that's all! Enjoy it!

---

> 原始地址: 
> http://www.wangchenlong.org/2016/03/22/1603/222-snowflake-anim/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

