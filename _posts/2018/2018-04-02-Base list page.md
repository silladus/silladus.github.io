---
layout: post
title:  "包含上拉加载下拉刷新功能的通用列表页面封装"
date:   2018-04-02 15:23:35 +0200
categories: Android
---

我们在实现一些业务的时候常会碰到类似的场景：有一些集合数据，我们要以列表的形式展示出来，我们希望用户可以控制数据的刷新和显示，这类需求往往在多处多项目中实现。

为了实现复用和定制自由，需要设计模式封装抽取。

实现功能应包含四个要素：

- 数据

- 列表控件

- 列表适配器

- 下拉刷新上拉加载更多

数据类型、列表控件类型及其相对应的控件适配器均不可预知，下拉刷新上拉加载更多还是通用的好，确定引入第三方框架[android-Ultra-Pull-To-Refresh支持上拉加载更多的版本](https://github.com/captainbupt/android-Ultra-Pull-To-Refresh-With-Load-More)。

### 接下来就是实现了。

首先是抽象基类接口：

```java
public abstract class BasePage<T, LV extends View, A> implements PtrHandler2 {
    protected List<T> data;
    protected LV listView;
    protected A mListAdapter;

    public BasePage(...) {
        ...
            
        initView();
    }

    private void initView() {
        ...

        mListAdapter = createListAdapter(listView);
    }
    
    ...

    @Override
    public void onLoadMoreBegin(PtrFrameLayout frame) {
        // Do something.
    }

    @Override
    public void onRefreshBegin(PtrFrameLayout frame) {
        // Do something.
    }
    
    @Override
    public boolean checkCanDoLoadMore(PtrFrameLayout frame, View content, View footer) {
        return PtrDefaultHandler2.checkContentCanBePulledUp(frame, listView, footer);
    }

    @Override
    public boolean checkCanDoRefresh(PtrFrameLayout frame, View content, View header) {
        return PtrDefaultHandler2.checkContentCanBePulledDown(frame, listView, header);
    }

    /**
     * 该方法接受生成的数据适配器
     * 
     * @param listView 列表显示控件
     * @return adapter
     */
    public abstract A createListAdapter(LV listView);

    /**
     * Request data.
     * 
     * @param reqCode 请求码，一般用于不同分类的数据
     * @param pageIndex 分页加载第几页
     */
    public abstract void getData(int reqCode, int pageIndex);
    
    ...
}
```

列表控件引入继承View的泛型，不管是ListView、ReclclerView还是其他列表控件理论上均予支持，这里只是一个接口。另外在这里做PtrHandler2接口的具体实现，Adapter、数据也均在这里声明传入方法。基本的就是这样。

### 单独的使用，列表控件以ListView为例

```java
BasePage<String, ListView, CommonAdapter<String>> mPage = new BasePage<String, 
ListView, CommonAdapter<String>>(0, context) {
    @Override
    public CommonAdapter<String> createListAdapter() {
        return new CommonAdapter<String>(R.layout.item_order_listview_detail) {
            @Override
            public void convert(ViewHolder holder, String item, int position) {
                ...
            }
        };
    }
    
    @Override
    public void getData(int reqCode, int pageIndex) {
        ...  
    }
};

viewGroup.addView(mPage.rootView);

...
```

### 在ViewPager中使用

基本思路还是一样的，一开始生成相对数量的BasePage保存在集合里(我选择SparseArray)，然后生成页面的时候添加进去。

```java
    @NonNull
    @Override
    public Object instantiateItem(@NonNull ViewGroup container, final int position) {
        BasePage<T, LV, A> page = pageArr.get(position);
        container.addView(page.rootView);
        if (page.data == null) {
            page.requestData(1);
        }
        return page.rootView;
    }
```

主要还是面向自己做个偏思路方向的记录，很多东西都懒得去解释说明。