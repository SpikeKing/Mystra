---
title: 实现网页链接跳转原生应用
date: 2016-03-20 17:37:00
categories: [Android]
tags: [Android]
---

人们每天都要访问大量的手机网页, 如果把手机网页(Web)和应用(App)紧密地联系起来, 就可以增大用户的访问量, 也有其他应用场景, 如``网页中调用支付链接``, ``新闻中启动问诊界面``, ``提供优质的原生功能``等等.

<!-- more -->
> 更多: http://www.wangchenlong.org/

本文源码的GitHub[下载地址](https://github.com/SpikeKing/TestWebIntent)

如何在网页(Web)中, 通过Intent直接启动应用(App)的Activity呢? 

本文主要有以下几点:
(1) 如何在Web中发送原生的Intent消息.
(1) 如何加载本地的HTML页面到浏览器.
(2) 如何创建半透明的Activity页面.

![展示](160320-implement-webview-goto-app/web-app-demo.png)

---

# 配置项目
新建HelloWorld工程. 添加ButterKnife支持.
```gradle
compile 'com.jakewharton:butterknife:7.0.1'
```

---

# BottomSheet

逻辑, 添加ShareIntent的监听, 即网页链接触发的Intent, 提取Link和Title信息, 底部出现或消失的动画.
```java
/**
 * 网页Activity
 * <p/>
 * Created by wangchenlong on 15/12/7.
 */
public class WebIntentActivity extends Activity {

    @Bind(R.id.web_intent_et_title) EditText mEtTitle;
    @Bind(R.id.web_intent_et_link) EditText mEtLink;

    @Override protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bottom_sheet);
        ButterKnife.bind(this);

        // 获取WebIntent信息
        if (isShareIntent()) {
            ShareCompat.IntentReader intentReader = ShareCompat.IntentReader.from(this);
            mEtLink.setText(intentReader.getText());
            mEtTitle.setText(intentReader.getSubject());
        }
    }

    @Override protected void onResume() {
        super.onResume();
        // 底部出现动画
        overridePendingTransition(R.anim.bottom_in, R.anim.bottom_out);
    }

    // 判断是不是WebIntent
    private boolean isShareIntent() {
        return getIntent() != null && Intent.ACTION_SEND.equals(getIntent().getAction());
    }

    @Override public void overridePendingTransition(int enterAnim, int exitAnim) {
        super.overridePendingTransition(enterAnim, exitAnim);
    }
}
```

动画属性, 沿Y轴变换.
```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="300"
        android:fromYDelta="100%p"
        android:toYDelta="0%p"/>
</set>
```
```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="300"
        android:fromYDelta="0%p"
        android:toYDelta="100%p"/>
</set>
```

BottomSheet页面, 由两个EditText组成.
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    android:id="@+id/web_intent_ll_popup_window"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|center"
    android:background="@android:color/white"
    android:orientation="vertical"
    android:padding="10dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="发送网页内容到应用"
        android:textSize="20sp"/>

    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:layout_marginStart="12dp"
        android:layout_marginTop="8dp">

        <EditText
            android:id="@+id/web_intent_et_title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Post Title"
            android:inputType="textCapWords"/>

    </android.support.design.widget.TextInputLayout>

    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:layout_marginStart="12dp"
        android:layout_marginTop="8dp">

        <EditText
            android:id="@+id/web_intent_et_link"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Link"
            android:inputType="textCapWords"/>

    </android.support.design.widget.TextInputLayout>

</LinearLayout>
```

> 注意
设置LinearLayout的``android:layout_gravity="bottom|center"``属性, 
> 配合样式(Styles)的``<item name="android:windowIsFloating">false</item>``属性,
> 可以在底部显示页面.

![BottomSheet](160320-implement-webview-goto-app/web-app-page.png)

声明, 添加``SEND``的Action, ``BROWSABLE``的Category, ``text/plain``的文件类型. 
主题设置透明主题. 启动时, 会保留上部半透明, 用于显示网页信息.
```xml
        <activity
            android:name=".WebIntentActivity"
            android:theme="@style/Theme.Transparent">
            <intent-filter>
                <action android:name="android.intent.action.SEND"/>

                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>

                <data android:mimeType="text/plain"/>
            </intent-filter>
        </activity>
```

透明主题, 注意一些关键属性, 参考注释, 不一一列举.
```xml
    <style name="Theme.Transparent" parent="AppTheme.NoActionBar">
        <!--背景色-->
        <item name="android:windowBackground">@color/page_background</item>
        <!--不使用背景缓存-->
        <item name="android:colorBackgroundCacheHint">@null</item>
        <!--控制窗口位置, 非流窗口, 固定位置, 用于非全屏窗口-->
        <item name="android:windowIsFloating">false</item>
        <!--窗口透明-->
        <item name="android:windowIsTranslucent">true</item>
        <!--窗口无标题-->
        <item name="android:windowNoTitle">true</item>
    </style>
```

背景颜色``windowBackground``非常重要, 不是常规颜色, 也可以设置为透明.
```xml
<!--最前两位是颜色厚度, 00透明, FF全黑-->
<color name="page_background">#99323232</color>
``` 

---

# 主页面

本地HTML文件存放在``assets``中, 提供在浏览器打开功能. 
浏览器打开Web链接非常简单, 打开本地HTML有很多难点.
```java
/**
 * 测试WebIntent的Demo
 *
 * @author C.L.Wang
 */
public class MainActivity extends AppCompatActivity {

    @SuppressWarnings("unused")
    private static final String TAG = "DEBUG-WCL: " + MainActivity.class.getSimpleName();

    private static final String FILE_NAME = "file:///android_asset/web_intent.html";

    @Bind(R.id.main_wv_web) WebView mWvWeb; // WebView

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 跳转WebIntentActivity
                startActivity(new Intent(MainActivity.this, WebIntentActivity.class));
            }
        });

        mWvWeb.loadUrl(FILE_NAME);
    }

    @Override public void onBackPressed() {
        // 优先后退网页
        if (mWvWeb.canGoBack()) {
            mWvWeb.goBack();
        } else {
            finish();
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        // 打开浏览器选项
        if (id == R.id.action_open_in_browser) {
            // 获取文件名, 打开assets文件使用文件名
            String[] as = FILE_NAME.split("/");
            openUrlInBrowser(as[as.length - 1]);
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    /**
     * 在浏览器中打开
     *
     * @param url 链接(本地HTML或者网络链接)
     */
    private void openUrlInBrowser(String url) {
        Uri uri;
        if (url.endsWith(".html")) { // 文件
            uri = Uri.fromFile(createFileFromInputStream(url));
        } else { // 链接
            if (!url.startsWith("http://") && !url.startsWith("https://")) {
                url = "http://" + url;
            }
            uri = Uri.parse(url);
        }

        try {
            Intent intent = new Intent(Intent.ACTION_VIEW, uri);
            // 启动浏览器, 谷歌浏览器, 小米手机浏览器支持, 其他手机或浏览器不支持.
            intent.setClassName("com.android.browser", "com.android.browser.BrowserActivity");
            startActivity(intent);
        } catch (ActivityNotFoundException e) {
            Toast.makeText(this, "没有应用处理这个请求. 请安装浏览器.", Toast.LENGTH_LONG).show();
            e.printStackTrace();
        }
    }

    /**
     * 存储assets内的文件
     *
     * @param url 文件名
     * @return 文件类(File)
     */
    private File createFileFromInputStream(String url) {
        try {
            // 打开Assets内的文件
            InputStream inputStream = getAssets().open(url);
            // 存储位置 /sdcard
            File file = new File(
                    Environment.getExternalStorageDirectory().getPath(), url);
            OutputStream outputStream = new FileOutputStream(file);
            byte buffer[] = new byte[1024];
            int length;
            while ((length = inputStream.read(buffer)) > 0) {
                outputStream.write(buffer, 0, length);
            }
            outputStream.close();
            inputStream.close();
            return file;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

> 注意:
> (1) 浏览器打开``assets``内文件的方式, 与``WebView``有所不同, 
具体参考``createFileFromInputStream``函数.
> (2) 在浏览器打开时, 需要指定包名, 而且各自浏览器的模式也不一样, 
小米支持Google原生调用, 参考``openUrlInBrowser``函数.
> (3) 回退事件的处理方式, 参考``onBackPressed``函数.

![动画](160320-implement-webview-goto-app/web-app-anim.gif)

就这些了, 在浏览器的HTML5页面中, 可以添加更多和本地应用的交互.

OK, that's all! Enjoy it.

---

**生活**

> 有技术又要有生活, 美让生活更精彩!

[![丝袜](http://7xrsre.com1.z0.glb.clouddn.com/spike-ad-girl-socks-10.jpg)](http://s.click.taobao.com/t?e=m%3D2%26s%3DY1RUHO0vTA0cQipKwQzePOeEDrYVVa64K7Vc7tFgwiFRAdhuF14FMWYM4BMqU8IYJ1gyddu7kN%2BD18qGwYGwMucFZRTbL%2Fvo%2BIoZChn2najN9dt%2FjNhJlNObIo0vIHz3QFuiTPmB1MeO9CYTlVrtTXEqY%2Bakgpmw&pvid=53_117.73.144.43_341_1458470034945)

女生, 让自己更职业受欢迎! 男生, 送给心中女神或未来女友! [好物](http://s.click.taobao.com/t?e=m%3D2%26s%3DY1RUHO0vTA0cQipKwQzePOeEDrYVVa64K7Vc7tFgwiFRAdhuF14FMWYM4BMqU8IYJ1gyddu7kN%2BD18qGwYGwMucFZRTbL%2Fvo%2BIoZChn2najN9dt%2FjNhJlNObIo0vIHz3QFuiTPmB1MeO9CYTlVrtTXEqY%2Bakgpmw&pvid=53_117.73.144.43_341_1458470034945)

---

> 原始地址: 
> http://www.wangchenlong.org/2016/03/20/1603/203-implement-webview-goto-app/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

