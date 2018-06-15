## setContentView源码

### 1、为什么要分析？
(1)、为每个页面添加一个标题栏（不想每次都在`XML`文件中include），能不能在布局文件的上一层进行统一处理呢？我们的`XML`是加载到哪里去了呢？

(2)、为每个页面添加一个Loading页和Error页，考虑到页面有titlebar的存在（有可能是(1)中统一的titlebar，也有可能是自己include的titlebar）

### 2、源码解析

```
Activity中：

//加载资源
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}

加载View
public void setContentView(View view) {
    getWindow().setContentView(view);
    initWindowDecorActionBar();
}

Window中：

public abstract void setContentView(View view);

```
>不管继承什么`XXXActivity`，最终setContentView都会调用`Activity`中的方法，然后再调用`Window`中的方法，`Window`中setContentView是一个抽象方法，那我们肯定要去找到他的实现类：

```
PhoneWindow中：

@Override
public void setContentView(int layoutResID) {
	 
	 //mContentParent是Window内容放置的视图，也就是我们的根布局
    if (mContentParent == null) {
        installDecor();   //创建一个DecorView
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.removeAllViews();
    }
    
    ...
}

private void installDecor() {
    
    if (mDecor == null) {
    	 //判断DecorView为空时去手动创建一个
        mDecor = generateDecor(-1);
        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        mDecor.setIsRootNamespace(true);
        if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
            mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
        }
    } else {
    	 //不为空时将DecorView与Window绑定
        mDecor.setWindow(this);
    }
    if (mContentParent == null) {
    	 //当根布局为空时，系统也默认为其创建一个根布局
        mContentParent = generateLayout(mDecor);
        
        ...
    }
}

protected ViewGroup generateLayout(DecorView decor) {

    ...

    int layoutResource;
    int features = getLocalFeatures();
    //根据设置的style属性来获取不同的layoutResource
    if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
        layoutResource = R.layout.screen_swipe_dismiss;
        setCloseOnSwipeEnabled(true);
    } else if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
        if (mIsFloating) {
            TypedValue res = new TypedValue();
            getContext().getTheme().resolveAttribute(
                    R.attr.dialogTitleIconsDecorLayout, res, true);
            layoutResource = res.resourceId;
        } else {
            layoutResource = R.layout.screen_title_icons;
        }
        // XXX Remove this once action bar supports these features.
        removeFeature(FEATURE_ACTION_BAR);
        // System.out.println("Title Icons!");
    } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
            && (features & (1 << FEATURE_ACTION_BAR)) == 0) {
        // Special case for a window with only a progress bar (and title).
        // XXX Need to have a no-title version of embedded windows.
        layoutResource = R.layout.screen_progress;
        // System.out.println("Progress!");
    } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
        // Special case for a window with a custom title.
        // If the window is floating, we need a dialog layout
        if (mIsFloating) {
            TypedValue res = new TypedValue();
            getContext().getTheme().resolveAttribute(
                    R.attr.dialogCustomTitleDecorLayout, res, true);
            layoutResource = res.resourceId;
        } else {
            layoutResource = R.layout.screen_custom_title;
        }
        // XXX Remove this once action bar supports these features.
        removeFeature(FEATURE_ACTION_BAR);
    } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
        // If no other features and not embedded, only need a title.
        // If the window is floating, we need a dialog layout
        if (mIsFloating) {
            TypedValue res = new TypedValue();
            getContext().getTheme().resolveAttribute(
                    R.attr.dialogTitleDecorLayout, res, true);
            layoutResource = res.resourceId;
        } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
            layoutResource = a.getResourceId(
                    R.styleable.Window_windowActionBarFullscreenDecorLayout,
                    R.layout.screen_action_bar);
        } else {
            layoutResource = R.layout.screen_title;
        }
        // System.out.println("Title!");
    } else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
        layoutResource = R.layout.screen_simple_overlay_action_mode;
    } else {
        // Embedded, so no decoration is needed.
        layoutResource = R.layout.screen_simple;
        // System.out.println("Simple!");
    }

    mDecor.startChanging();
    mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);

    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);

    ...

	 //最后返回得到一个根布局（也就是id/content）
    return contentParent;
}

XML中：

//下面列举一个最简单的布局
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:orientation="vertical">
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
    <FrameLayout
         android:id="@android:id/content"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         android:foregroundInsidePadding="false"
         android:foregroundGravity="fill_horizontal|top"
         android:foreground="?android:attr/windowContentOverlay" />
</LinearLayout>
```
>把每一个系统给的`XML`文件都看了一遍，发现他都是这种类似的，LinearLayout包一个FrameLayout，这个FrameLayout就是我们经常看到的id/content（心里一阵喜）,这样的话那我们就可以在这个LinearLayout里面做文章了。接着往下看，拿到这个根布局之后，然后会将我们setContentView过来的View添加到根布局上，这么一来，我们就有以下的视图树了：

![](http://ooaap25kv.bkt.clouddn.com/18-6-15/38756758.jpg)

