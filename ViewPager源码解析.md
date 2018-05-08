## ViewPager源码解析

### 1、setAdapter方法
```
public void setAdapter(@Nullable PagerAdapter adapter) {
	...
	
	populate();
	
	...
}
```
调用setAdapter方法将setAdapter方法里面主要调了populate方法，这个方法是用来创建和销毁View的：

创建View：mAdapter.instantiateItem()

销毁View：mAdapter.destroyItem()

从源码中可以看出，ViewPager中放多少页面都不会导致内存溢出问题，因为会不断的创建和销毁View

![ViewPager工作原理](https://upload-images.jianshu.io/upload_images/4314397-07dc5f93c8ef12ce?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### 2、setCurrentItem()方法
```
public void setCurrentItem(int item, boolean smoothScroll) {
    setCurrentItemInternal(item, smoothScroll, false);
}

void setCurrentItemInternal(int item, boolean smoothScroll, boolean always, int velocity) {
    scrollToItem(item, smoothScroll, velocity, dispatchSelected);
}

private void scrollToItem(int item, boolean smoothScroll, int velocity, boolean dispatchSelected) {
    smoothScrollTo(destX, 0, velocity);
}

void smoothScrollTo(int x, int y, int velocity) {
    mScroller.startScroll(sx, sy, dx, dy, duration);
}
```
mScroller.startScroll方法是切换页面并且执行动画


