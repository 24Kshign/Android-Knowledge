## Activity的启动流程


#### 注：本篇文章只分析打开应用后页面间的跳转

启动某一个页面的时候我们是调用Activity的startActivity系列方法：

```
Activity类：

public void startActivity(Intent intent) {
    
}

public void startActivity(Intent intent, @Nullable Bundle options) {

}

public void startActivityForResult(@RequiresPermission Intent intent, int requestCode, @Nullable Bundle options) {
		
	 Instrumentation.ActivityResult ar = mInstrumentation.execStartActivity(xxx);

	 ...

}
```

```
Instrumentation类：

public ActivityResult execStartActivity(xxx) {

	 ...
	 int result = ActivityManager.getService().startActivity(xxx);
	 ...

}
```

```
ActivityManagerService类：

public final int startActivity(xxx) {
	 return startActivityAsUser(xxx);        
}

public final int startActivityAsUser(xxx) {
            
	 ...
	 return mActivityStarter.startActivityMayWait(xxx);
}
```

```
ActivityStarter类：

final int startActivityMayWait(xxx) {

	 ...
	 int res = startActivityLocked(xxx);
	 ...

}

int startActivityLocked(xxx) {

	 ...
	 mLastStartActivityResult = startActivity(xxx);
	 ...
}

private int startActivity(xxx) {

	 ...
	 return startActivity(xxx);
}

private int startActivity(xxx) {

	 ...
	 result = startActivityUnchecked(xxx);
	 ...
}

private int startActivityUnchecked(xxx) {

	 ...
	 mSupervisor.resumeFocusedStackTopActivityLocked(xxx);
	 ...
}
```

```
ActivityStackSupervisor类：

boolean resumeFocusedStackTopActivityLocked(xxx) {

	 ...
	 return targetStack.resumeTopActivityUncheckedLocked(xxx);
	 ...
}
```

```
ActivityStack类：

boolean resumeTopActivityUncheckedLocked(xxx) {

	 ...
	 result = resumeTopActivityInnerLocked(prev, options);
	 ...
}

private boolean resumeTopActivityInnerLocked(xxx) {

	 ...
	 mStackSupervisor.startSpecificActivityLocked(xxx);
	 ...
}
```

```
ActivityStackSupervisor类：

void startSpecificActivityLocked(xxx) {

	 ...
	 realStartActivityLocked(xxx);
	 ...
}

final boolean realStartActivityLocked(xxx) {

	 ...
	 app.thread.scheduleLaunchActivity(xxx);
	 
	 app.thread为IApplicationThread，它的实现类是ApplicationThread
	 ...
}
```

```
ActivityThread类：

private class ApplicationThread extends IApplicationThread.Stub {

	 ...
	 
	 public final void scheduleLaunchActivity(xxx){
	 
	 	 ...
	 	 sendMessage(H.LAUNCH_ACTIVITY, r);
	 }
	 
	 ...
}

private class H extends Handler{

	 ...
	 public void handleMessage(Message msg) {
	 
	 	 ...
	 	 case LAUNCH_ACTIVITY: {
	 	 	 ...
	 	 	 handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
	 	 }
	 }
}
```

——> 先调用handleLauncherActivity

——> 再调用performLauncherActivity

——> 然后调Activity.onCreate方法

——> 然后调handleResumeActivity方法

——> 然后调performResumeActivity

——> Activity的onResume方法

——> wm.addView(decor,1);  这时候才开始把DecorView加载到WindowManager上来，这时候才开始View的绘制流程，才开始执行measure()，layout()，draw()

##### 之所以能拿到控件的宽高是因为调用了onMeasure方法，setContentView只是创建了DecorView 把我们的布局加载到DecorView


### View的绘制流程：

——> wm.addView(decor,1);

——> WindowmanagerImpl.addView();

——> root.setView(view, wparams, panelParentView);

——> requestLayout();

——> scheduleTraversals();

——> 调用mTraversalRunnable里面的doTraversal();

——> **performTraversals();**   这里才开始View的测量，摆放，绘制等操作

第一个调用的方法：performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);

——> mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);

——> onMeasure(widthMeasureSpec, heightMeasureSpec);测量开始

——> for循环，拿ViewGroup的子布局挨个进行测量

——> wrap_content对应的测量模式为AT_MOST；match_parent fill_parent 100dp对应的测量模式为EXACTLY，该模式和大小是由父布局和自己决定的

1、如果父布局是wrap_content，就算子布局是match_parent，这个时候计算的测量模式还是AT_MOST

2、如果父布局是match_parent，子布局是match_parent，这个时候计算的测量模式还是EXACTLY

——> child.measure(childWidthMeasureSpec, childHeightMeasureSpec);

——> 然后调setMeasureDimension()方法，这个时候布局控件才算真正有了宽高;

——> 接着执行ViewGroup的onMeasure()方法，这个时候要指定自己（外层布局）的宽高了，childWidth = child.getMeasuredWidth() + share;

——> 同样，在**performTraversals()**方法中，也分别调用了OnLayout()和OnDraw()方法

### getWidth和getMeasureWidth
1、getWidth在onLayout方法执行完之后才会有数据，getMeasureWidth在onMeasure方法执行完之后才会有数据，所以在onMeasure之后可以调用getMeasuredWidth获取宽度，在onLayout()之后，可以用getWitdth获取宽度

2、当布局继承ViewGroup时，重写onLayout方法时，getWidth和getMeasureWidth返回的值才会不一致

### measure，layout和draw
measure和layout都是通过遍历的方式进行，而draw是通过递归的方式进行的
















