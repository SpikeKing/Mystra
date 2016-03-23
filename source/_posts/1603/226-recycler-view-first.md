---
title: 使用 RecyclerView 控件实现瀑布流
date: 2016-03-22 08:14:00
categories: [Android]
tags: [Android,控件]
---

RecyclerView相比于ListView, 在回收重用时更具有灵活性, 也就是低耦合, 并且提供了扩展. 加载多个视图时, 应该多用RecyclerView代替ListView.

<!-- more -->
> 更多: http://www.wangchenlong.org/

那么我们来看看这东西应该怎么用? 比如生成一个瀑布流的视图.

![瀑布流](225-recycler-view-first/recycler-waterfall.jpg)

首先我们从一个HelloWorld写起, 看看如何构建一个RecyclerView.

---

# 依赖

Gradle配置, 添加RecyclerView库的依赖.

```gradle
compile 'com.android.support:recyclerview-v7:+'
```

---

# 布局

布局文件
```xml
    <android.support.v7.widget.RecyclerView
        android:id="@+id/test_recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
```

---

# 代码

LayoutManager: 管理RecyclerView的结构.
Adapter: 处理每个Item的显示.
ItemDecoration: 添加每个Item的装饰.
ItemAnimator: 负责添加\移除\重排序时的动画效果.
> LayoutManager\Adapter是必须, ItemDecoration\ItemAnimator是可选.

```java
    /**
     * 初始化RecyclerView
     *
     * @param recyclerView 主控件
     */
    private void initRecyclerView(RecyclerView recyclerView) {
        recyclerView.setHasFixedSize(true); // 设置固定大小
        initRecyclerLayoutManager(recyclerView); // 初始化布局
        initRecyclerAdapter(recyclerView); // 初始化适配器
        initItemDecoration(recyclerView); // 初始化装饰
        initItemAnimator(recyclerView); // 初始化动画效果
    }
```

---

# LayoutManager

管理RecyclerView的布局结构.
```java
    private void initRecyclerLayoutManager(RecyclerView recyclerView) {
        // 错列网格布局
        recyclerView.setLayoutManager(new StaggeredGridLayoutManager(4,
                StaggeredGridLayoutManager.VERTICAL));
    }
```
>  提供了多种LayoutManager, 瀑布流使用错列网格布局.

---

# Adapter

适配器, 处理RecyclerView的Item事务.
```java
    private void initRecyclerAdapter(RecyclerView recyclerView) {
        mAdapter = new MyAdapter(getData());
        recyclerView.setAdapter(mAdapter);
    }
```

对于Adapter, 我们需要展开来说, 先看看类.
```java
public class MyAdapter extends RecyclerView.Adapter<MyViewHolder> {

    private List<DataModel> mDataModels;
    private List<Integer> mHeights;

    MyAdapter(List<DataModel> dataModels) {
        if (dataModels == null) {
            throw new IllegalArgumentException("DataModel must not be null");
        }
        mDataModels = dataModels;
        mHeights = new ArrayList<>();
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_recycler_view, parent, false);
        return new MyViewHolder(itemView);
    }

    @Override
    public void onBindViewHolder(MyViewHolder holder, int position) {
        DataModel dataModel = mDataModels.get(position);

        // 随机高度, 模拟瀑布效果.
        if (mHeights.size() <= position) {
            mHeights.add((int) (100 + Math.random() * 300));
        }

        ViewGroup.LayoutParams lp = holder.getTvLabel().getLayoutParams();
        lp.height = mHeights.get(position);

        holder.getTvLabel().setLayoutParams(lp);

        holder.getTvLabel().setText(dataModel.getLabel());
        holder.getTvDateTime().setText(new SimpleDateFormat("yyyy-MM-dd", Locale.ENGLISH)
                .format(dataModel.getDateTime()));
    }

    @Override
    public int getItemCount() {
        return mDataModels.size();
    }

    public void addData(int position) {
        DataModel model = new DataModel();
        model.setDateTime(getBeforeDay(new Date(), position));
        model.setLabel("No. " + (int) (new Random().nextDouble() * 20.0f));

        mDataModels.add(position, model);
        notifyItemInserted(position);
    }

    public void removeData(int position) {
        mDataModels.remove(position);
        notifyItemRemoved(position);
    }

    /**
     * 获取日期的前一天
     *
     * @param date 日期
     * @param i    偏离
     * @return 新的日期
     */
    private static Date getBeforeDay(Date date, int i) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        calendar.add(Calendar.DAY_OF_YEAR, i * (-1));
        return calendar.getTime();
    }
}
```

onCreateViewHolder创建ViewHolder.
onBindViewHolder绑定每一项数据.
getItemCount返回列表长度.

RecyclerView强制使用ViewHolder.
```java
public class MyViewHolder extends RecyclerView.ViewHolder {

    private TextView mTvLabel; // 标签
    private TextView mTvDateTime; // 日期

    public MyViewHolder(View itemView) {
        super(itemView);
        mTvLabel = (TextView) itemView.findViewById(R.id.item_text);
        mTvDateTime = (TextView) itemView.findViewById(R.id.item_date);
    }

    public TextView getTvLabel() {
        return mTvLabel;
    }

    public TextView getTvDateTime() {
        return mTvDateTime;
    }

}
```
> 在onCreateViewHolder方法, 创建类; 在onBindViewHolder方法, 绑定数据.

DataModel
```java
public class DataModel {

    private String mLabel;
    private Date mDateTime;

    public String getLabel() {
        return mLabel;
    }

    public void setLabel(String label) {
        mLabel = label;
    }

    public Date getDateTime() {
        return mDateTime;
    }

    public void setDateTime(Date dateTime) {
        mDateTime = dateTime;
    }
}
```

# ItemDecoration

项的装饰, 比如ListView中的分割线, 在本例中, 左右两条粉线.
```java
    private void initItemDecoration(RecyclerView recyclerView) {
        recyclerView.addItemDecoration(new MyItemDecoration(this));
    }
```

ItemDecoration,  注意parent和child的使用方式.
```java
public class MyItemDecoration extends RecyclerView.ItemDecoration {

    private static final int[] ATTRS = new int[]{android.R.attr.listDivider};
    private Drawable mDivider;

    public MyItemDecoration(Context context) {
        final TypedArray array = context.obtainStyledAttributes(ATTRS);
        mDivider = array.getDrawable(0);
        array.recycle();
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent, State state) {
        drawHorizontal(c, parent);
        drawVertical(c, parent);
    }

    // 水平线
    public void drawHorizontal(Canvas c, RecyclerView parent) {

        final int childCount = parent.getChildCount();

        // 在每一个子控件的底部画线
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);

            final int left = child.getLeft() + child.getPaddingLeft();
            final int right = child.getWidth() + child.getLeft() - child.getPaddingRight();
            final int top = child.getBottom() - mDivider.getIntrinsicHeight() - child.getPaddingBottom();
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    // 竖直线
    public void drawVertical(Canvas c, RecyclerView parent) {

        final int childCount = parent.getChildCount();

        // 在每一个子控件的右侧画线
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            int right = child.getRight() - child.getPaddingRight();
            int left = right - mDivider.getIntrinsicWidth();
            final int top = child.getTop() + child.getPaddingTop();
            final int bottom = child.getTop() + child.getHeight() - child.getPaddingBottom();

            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    // Item之间的留白
    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
        outRect.set(0, 0, mDivider.getIntrinsicWidth(), mDivider.getIntrinsicHeight());
    }
}
```

本例重写了listDivider
```xml
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    	...
        <item name="android:listDivider">@drawable/divider_bg</item>
    </style>
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle">
    <solid android:color="#ff00ff"/>
    <size android:height="4dp"/>
    <size android:width="4dp"/>
</shape>
```

---

# ItemAnimator
动画效果比较复杂, 使用默认动画. 如要定制的话, 继承DefaultItemAnimator; 如设置null, 则不显示任何动画.
```java
    private void initItemAnimator(RecyclerView recyclerView) {
        recyclerView.setItemAnimator(new DefaultItemAnimator()); // 默认动画
    }
``` 

---

# 最终
最终的代码逻辑
```java
public class MainActivity extends AppCompatActivity {

    private MyAdapter mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                        .setAction("Action", null).show();
            }
        });

        // 初始化RecyclerView
        initRecyclerView((RecyclerView) findViewById(R.id.test_recycler_view));
    }

    /**
     * 初始化RecyclerView
     *
     * @param recyclerView 主控件
     */
    private void initRecyclerView(RecyclerView recyclerView) {
        recyclerView.setHasFixedSize(true); // 设置固定大小
        initRecyclerLayoutManager(recyclerView); // 初始化LayoutManager
        initRecyclerAdapter(recyclerView); // 初始化Adapter
        initItemDecoration(recyclerView); // 初始化边界装饰
        initItemAnimator(recyclerView); // 初始化动画效果
    }

    /**
     * 初始化RecyclerView的LayoutManager
     *
     * @param recyclerView 主控件
     */
    private void initRecyclerLayoutManager(RecyclerView recyclerView) {
        // 错列网格布局
        recyclerView.setLayoutManager(new StaggeredGridLayoutManager(4,
                StaggeredGridLayoutManager.VERTICAL));
    }

    /**
     * 初始化RecyclerView的Adapter
     *
     * @param recyclerView 主控件
     */
    private void initRecyclerAdapter(RecyclerView recyclerView) {
        mAdapter = new MyAdapter(getData());
        recyclerView.setAdapter(mAdapter);
    }

    /**
     * 初始化RecyclerView的(ItemDecoration)项目装饰
     *
     * @param recyclerView 主控件
     */
    private void initItemDecoration(RecyclerView recyclerView) {
        recyclerView.addItemDecoration(new MyItemDecoration(this));
    }

    /**
     * 初始化RecyclerView的(ItemAnimator)项目动画
     *
     * @param recyclerView 主控件
     */
    private void initItemAnimator(RecyclerView recyclerView) {
        recyclerView.setItemAnimator(new DefaultItemAnimator()); // 默认动画
    }

    /**
     * 模拟的数据
     *
     * @return 数据
     */
    private ArrayList<DataModel> getData() {
        int count = 57;
        ArrayList<DataModel> data = new ArrayList<>();
        for (int i = 0; i < count; i++) {
            DataModel model = new DataModel();

            model.setDateTime(getBeforeDay(new Date(), i));
            model.setLabel("No. " + i);

            data.add(model);
        }

        return data;
    }

    /**
     * 获取日期的前一天
     *
     * @param date 日期
     * @param i    偏离
     * @return 新的日期
     */
    private static Date getBeforeDay(Date date, int i) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        calendar.add(Calendar.DAY_OF_YEAR, i * (-1));
        return calendar.getTime();
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

        switch (item.getItemId()) {
            case R.id.id_action_add:
                mAdapter.addData(1);
                break;
            case R.id.id_action_delete:
                mAdapter.removeData(1);
                break;
        }

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;

        }

        return super.onOptionsItemSelected(item);
    }
}
```

> 为了测试动画, Menu额外添加两个按钮.

---

最终效果
![瀑布流](225-recycler-view-first/recycler-demo.png)

RecyclerView的基本要点就是这些了.

OK, that's all! Enjoy it!

---

**生活**

> 有技术又要有生活, 美让生活更精彩!

[![丝袜](http://7xrsre.com1.z0.glb.clouddn.com/spike-ad-girl-socks-6.jpg)](http://s.click.taobao.com/t?e=m%3D2%26s%3D095YJZQ1%2BQUcQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67sL74TtmOEyS6EFRCN7EKmx1lK%2FY7wPaoHeQQxhDmA6IAe67oaxDEWp4DvOxtwmul3Iwqzx5RfB%2Byogwp%2Fp0ZrHz07Zdf9LZNcYMXU3NNCg%2F&pvid=12_117.73.144.43_332_1458433143248)

女生, 让自己更职业受欢迎! [好物](http://s.click.taobao.com/t?e=m%3D2%26s%3D095YJZQ1%2BQUcQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67sL74TtmOEyS6EFRCN7EKmx1lK%2FY7wPaoHeQQxhDmA6IAe67oaxDEWp4DvOxtwmul3Iwqzx5RfB%2Byogwp%2Fp0ZrHz07Zdf9LZNcYMXU3NNCg%2F&pvid=12_117.73.144.43_332_1458433143248)

---

> 原始地址: 
> http://www.wangchenlong.org/2016/03/22/1603/226-recycler-view-first/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58), [Slides](https://slides.com/spikeking). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

