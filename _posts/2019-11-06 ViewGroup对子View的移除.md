## ViewGroup对子View的移除

最近做了个动态布局，view和ViewGroup的布局关系大概如下
```html
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/viewGroup1"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:id="@+id/viewGroup2"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageView
            android:id="@+id/child"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

    </FrameLayout>
</FrameLayout>
```
这是一种情况，然后现在情况变了，这个child需要动态的直接add到viewGroup1，所以我在代码中如此做：
```java
	viewGroup1.removeAllViews();
	viewGroup1.addView(child);
```
这时候问题出现了，APP连报错打印都没有直接闪退
这个child在这之前如果没有添加到viewGroup2的话则没有这个问题

结论：ViewGroup的removeAllViews()仅是清空自身子view，对于自身子view中ViewGroup中的View是不管的，
viewGroup1调用了removeAllViews()后，虽然断开了和viewGroup2的联系移除了viewGroup2，但是viewGroup2和child之间的关系依然维持，这时child添加到viewGroup1是不被允许的。
