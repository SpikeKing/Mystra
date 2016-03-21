---
title: 使用 Espresso 和 Dagger 测试网络服务
date: 2016-03-13 14:14:00
categories: [Android]
tags: [Android,Espresso,Dagger,测试]
---

可靠的功能测试, 意味着在任何时候, 获取的测试结果均相同, 这就需要模拟(Mock)数据. 测试框架可以使用Android推荐的[Espresso](https://google.github.io/android-testing-support-library/docs/espresso/). 模拟数据可以使用[Dagger2](http://google.github.io/dagger/), 一种依赖注入框架.

单元测试通常会模拟所有依赖, 避免出现不可靠的情况, 而功能测试也可以这样做. 一个经典的例子是如何模拟稳定的网络数据, 可以使用Dagger2处理这种情况.

<!-- more -->
> 更多: http://www.wangchenlong.org/

本文源码的GitHub[下载地址](https://github.com/SpikeKing/wcl-espresso-dagger-demo)

> Dagger2已经成为众多Android开发者的必备工具, 是一个快速的[依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)框架，由[Square](https://squareup.com/)开发，并针对Android做了特别优化, 已经被Google进行Fork开发. 不像其他的依赖注入器, Dagger2没有使用反射, 而是使用预生成代码, 提高执行速度.H

![Android Test](use-dagger-espresso-test-network/android-test.png)

Talk is cheap! 我来讲解下如何实现.

---

# 配置依赖环境

主要:
(1) Lambda表达式支持.
(2) Dagger2依赖注入框架.
(3) RxAndroid响应式编程框架.
(4) Retrofit2网络库框架.
(5) Espresso测试框架.
(6) DataBinding数据绑定支持.

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}

// Lambda表达式
plugins {
    id "me.tatarka.retrolambda" version "3.2.4"
}

apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt' // 注释处理

final BUILD_TOOLS_VERSION = '23.0.1'

android {
    compileSdkVersion 23
    buildToolsVersion "${BUILD_TOOLS_VERSION}"

    defaultConfig {
        applicationId "clwang.chunyu.me.wcl_espresso_dagger_demo"
        minSdkVersion 16
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "clwang.chunyu.me.wcl_espresso_dagger_demo.runner.WeatherTestRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    // 注释冲突
    packagingOptions {
        exclude 'META-INF/services/javax.annotation.processing.Processor'
    }

    // 使用Java1.8
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    // 数据绑定
    dataBinding {
        enabled = true
    }
}

final DAGGER_VERSION = '2.0.2'
final RETROFIT_VERSION = '2.0.0-beta2'

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    // Warning:Conflict with dependency 'com.android.support:support-annotations'.
    // Resolved versions for app (23.1.1) and test app (23.0.1) differ.
    // See http://g.co/androidstudio/app-test-app-conflict for details.
    compile "com.android.support:appcompat-v7:${BUILD_TOOLS_VERSION}" // 需要与BuildTools保持一致

    compile 'com.jakewharton:butterknife:7.0.1' // 标注

    compile "com.google.dagger:dagger:${DAGGER_VERSION}" // dagger2
    compile "com.google.dagger:dagger-compiler:${DAGGER_VERSION}" // dagger2

    compile 'io.reactivex:rxandroid:1.1.0' // RxAndroid
    compile 'io.reactivex:rxjava:1.1.0' // 推荐同时加载RxJava

    compile "com.squareup.retrofit:retrofit:${RETROFIT_VERSION}" // Retrofit网络处理
    compile "com.squareup.retrofit:adapter-rxjava:${RETROFIT_VERSION}" // Retrofit的rx解析库
    compile "com.squareup.retrofit:converter-gson:${RETROFIT_VERSION}" // Retrofit的gson库
    compile 'com.squareup.okhttp:logging-interceptor:2.6.0' // 拦截器

    // 测试的编译
    androidTestCompile 'com.android.support.test:runner:0.4.1' // Android JUnit Runner
    androidTestCompile 'com.android.support.test:rules:0.4.1' // JUnit4 Rules
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1' // Espresso core

    provided 'javax.annotation:jsr250-api:1.0' // Java标注
}
```

Lambda表达式支持, 优雅整洁代码的关键.
```gradle
// Lambda表达式
plugins {
    id "me.tatarka.retrolambda" version "3.2.4"
}

android {
    // 使用Java1.8
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

Dagger2依赖注入框架, 实现依赖注入. android-apt使用生成代码的插件.
```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}

apply plugin: 'com.neenbedankt.android-apt' // 注释处理

dependencies {
    compile "com.google.dagger:dagger:${DAGGER_VERSION}" // dagger2
    compile "com.google.dagger:dagger-compiler:${DAGGER_VERSION}" // dagger2
    provided 'javax.annotation:jsr250-api:1.0' // Java标注
}
```

测试, 在默认配置中添加Runner, 在依赖中添加espresso库.
```gradle
android{
    defaultConfig {
        testInstrumentationRunner "clwang.chunyu.me.wcl_espresso_dagger_demo.runner.WeatherTestRunner"
    }
}

dependencies {
    testCompile 'junit:junit:4.12'

    // 测试的编译
    androidTestCompile 'com.android.support.test:runner:0.4.1' // Android JUnit Runner
    androidTestCompile 'com.android.support.test:rules:0.4.1' // JUnit4 Rules
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1' // Espresso core
}
```
数据绑定
```gradle
android{
    // 数据绑定 
    dataBinding { 
        enabled = true 
    }
}
```

---

# 设置项目

使用数据绑定, 实现了简单的搜索天功能.
```java
/**
 * 实现简单的查询天气的功能.
 *
 * @author wangchenlong
 */
public class MainActivity extends AppCompatActivity {

    private ActivityMainBinding mBinding; // 数据绑定
    private MenuItem mSearchItem; // 菜单项
    private Subscription mSubscription; // 订阅

    @Inject WeatherApiClient mWeatherApiClient; // 天气客户端

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ((WeatherApplication) getApplication()).getAppComponent().inject(this);
        mBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
    }


    @Override public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_activity_main, menu); // 加载目录资源
        mSearchItem = menu.findItem(R.id.menu_action_search);
        tintSearchMenuItem();
        initSearchView();
        return true;
    }

    // 搜索项着色, 会覆盖基础颜色, 取交集.
    private void tintSearchMenuItem() {
        int color = ContextCompat.getColor(this, android.R.color.white); // 白色
        mSearchItem.getIcon().setColorFilter(color, PorterDuff.Mode.SRC_IN); // 交集
    }

    // 搜索项初始化
    private void initSearchView() {
        SearchView searchView = (SearchView) MenuItemCompat.getActionView(mSearchItem);
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override public boolean onQueryTextSubmit(String query) {
                MenuItemCompat.collapseActionView(mSearchItem);
                loadWeatherData(query); // 加载查询数据
                return true;
            }

            @Override public boolean onQueryTextChange(String newText) {
                return false;
            }
        });
    }

    // 加载天气数据
    private void loadWeatherData(String cityName) {
        mBinding.progress.setVisibility(View.VISIBLE);
        mSubscription = mWeatherApiClient
                .getWeatherForCity(cityName)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(this::bindData, this::bindDataError);
    }

    // 绑定天气数据
    private void bindData(WeatherData weatherData) {
        mBinding.progress.setVisibility(View.INVISIBLE);
        mBinding.weatherLayout.setVisibility(View.VISIBLE);
        mBinding.setWeatherData(weatherData);
    }

    // 绑定数据失败
    private void bindDataError(Throwable throwable) {
        mBinding.progress.setVisibility(View.INVISIBLE);
    }

    @Override
    protected void onDestroy() {
        if (mSubscription != null) {
            mSubscription.unsubscribe();
        }
        super.onDestroy();
    }
}
```
数据绑定实现数据和显示分离, 解耦项目, 易于管理, 非常适合数据展示页面.

在layout中设置数据.
```xml
    <data>
        <variable
            name="weatherData"
            type="clwang.chunyu.me.wcl_espresso_dagger_demo.data.WeatherData"/>
    </data>
```

在代码中绑定数据.
```java
mBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
mBinding.setWeatherData(weatherData);
```

搜索框的设置.
```java
    @Override public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_activity_main, menu); // 加载目录资源
        mSearchItem = menu.findItem(R.id.menu_action_search);
        tintSearchMenuItem();
        initSearchView();
        return true;
    }

    // 搜索项着色, 会覆盖基础颜色, 取交集.
    private void tintSearchMenuItem() {
        int color = ContextCompat.getColor(this, android.R.color.white); // 白色
        mSearchItem.getIcon().setColorFilter(color, PorterDuff.Mode.SRC_IN); // 交集
    }

    // 搜索项初始化
    private void initSearchView() {
        SearchView searchView = (SearchView) MenuItemCompat.getActionView(mSearchItem);
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override public boolean onQueryTextSubmit(String query) {
                MenuItemCompat.collapseActionView(mSearchItem);
                loadWeatherData(query); // 加载查询数据
                return true;
            }

            @Override public boolean onQueryTextChange(String newText) {
                return false;
            }
        });
    }
```

---

# 功能测试

这一部分, 我会重点讲解.

既然使用Dagger2, 那么我们就来配置依赖注入.
三部曲: Module -> Component -> Application

Module, 使用模拟Api类, **MockWeatherApiClient**.
```java
/**
 * 测试App的Module, 提供AppContext, WeatherApiClient的模拟数据.
 * <p>
 * Created by wangchenlong on 16/1/16.
 */
@Module
public class TestAppModule {
    private final Context mContext;

    public TestAppModule(Context context) {
        mContext = context.getApplicationContext();
    }

    @AppScope
    @Provides
    public Context provideAppContext() {
        return mContext;
    }

    @Provides
    public WeatherApiClient provideWeatherApiClient() {
        return new MockWeatherApiClient();
    }
}
```

Component, 注入**MainActivityTest**.
```java
/**
 * 测试组件, 添加TestAppModule
 * <p>
 * Created by wangchenlong on 16/1/16.
 */
@AppScope
@Component(modules = TestAppModule.class)
public interface TestAppComponent extends AppComponent {
    void inject(MainActivityTest test);
}
```

Application, 继承**非测试的Application(WeatherApplication)**, 设置测试组件, 重写**获取组件的方法(getAppComponent)**.
```java
/**
 * 测试天气应用
 * <p>
 * Created by wangchenlong on 16/1/16.
 */
public class TestWeatherApplication extends WeatherApplication {
    private TestAppComponent mTestAppComponent;

    @Override public void onCreate() {
        super.onCreate();
        mTestAppComponent = DaggerTestAppComponent.builder()
                .testAppModule(new TestAppModule(this))
                .build();
    }

    // 组件
    @Override
    public TestAppComponent getAppComponent() {
        return mTestAppComponent;
    }
}
```

Mock数据类, 使用模拟数据创建Gson类, 延迟发送至监听接口.
```java
/**
 * 模拟天气Api客户端
 */
public class MockWeatherApiClient implements WeatherApiClient {
    @Override public Observable<WeatherData> getWeatherForCity(String cityName) {
        // 获得模拟数据
        WeatherData weatherData = new Gson().fromJson(TestData.MUNICH_WEATHER_DATA_JSON, WeatherData.class);
        return Observable.just(weatherData).delay(1, TimeUnit.SECONDS); // 延迟时间
    }
}
```

注册Application至TestRunner.
```java
/**
 * 更换Application, 设置TestRunner
 */
public class WeatherTestRunner extends AndroidJUnitRunner {
    @Override
    public Application newApplication(ClassLoader cl, String className, Context context) throws InstantiationException,
            IllegalAccessException, ClassNotFoundException {
        String testApplicationClassName = TestWeatherApplication.class.getCanonicalName();
        return super.newApplication(cl, testApplicationClassName, context);
    }
}
```

测试主类
```java
/**
 * 测试的Activity
 * <p>
 * Created by wangchenlong on 16/1/16.
 */
@LargeTest
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    private static final String CITY_NAME = "Beijing"; // 因为我们使用测试接口, 设置任何都可以.

    @Rule public ActivityTestRule<MainActivity> activityTestRule = new ActivityTestRule<>(MainActivity.class);

    @Inject WeatherApiClient weatherApiClient;

    @Before
    public void setUp() {
        ((TestWeatherApplication) activityTestRule.getActivity().getApplication()).getAppComponent().inject(this);
    }

    @Test
    public void correctWeatherDataDisplayed() {
        WeatherData weatherData = weatherApiClient.getWeatherForCity(CITY_NAME).toBlocking().first();

        onView(withId(R.id.menu_action_search)).perform(click());
        onView(withId(android.support.v7.appcompat.R.id.search_src_text)).perform(replaceText(CITY_NAME));
        onView(withId(android.support.v7.appcompat.R.id.search_src_text)).perform(pressKey(KeyEvent.KEYCODE_ENTER));

        onView(withId(R.id.city_name)).check(matches(withText(weatherData.getCityName())));
        onView(withId(R.id.weather_date)).check(matches(withText(weatherData.getWeatherDate())));
        onView(withId(R.id.weather_state)).check(matches(withText(weatherData.getWeatherState())));
        onView(withId(R.id.weather_description)).check(matches(withText(weatherData.getWeatherDescription())));
        onView(withId(R.id.temperature)).check(matches(withText(weatherData.getTemperatureCelsius())));
        onView(withId(R.id.humidity)).check(matches(withText(weatherData.getHumidity())));
    }
}
```

> **ActivityTestRule**设置MainActivity.class测试类.
> **setup**设置依赖注入, 注入TestWeatherApplication的组件.

使用WeatherApiClient的数据, 模拟类的功能. 由于数据是预设的, 不论有无网络, 都可以进行可靠的功能测试.

执行测试, 右键点击**MainActivityTest**, 使用**Run 'MainActivityTest'**.

---

OK, that's all! Enjoy it!

> 原始地址: 
> http://www.wangchenlong.org/2016/03/13/use-dagger-espresso-test-network/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.