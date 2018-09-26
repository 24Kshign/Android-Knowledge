## 浅谈Activity、Window、View之间的关系

### 序言：很多人都会用Activity、Window、View，但是你知道他们是怎样加载出来并呈现在你眼前的吗？你知道他们之间有着鲜为人知的关系吗？

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1537872442296&di=7411d17ac60eb7be6dcd9c99e134eda7&imgtype=0&src=http%3A%2F%2Fhimg2.huanqiu.com%2Fattachment2010%2F2017%2F0411%2F20170411011851880.jpg)

讲个很简单的例子，这一天天气甚好，小明外出写生，小明背了一包东西，画板啊，纸啊，笔啊什么的，然后小明找了一处风景甚好的地方，从包里拿出画板，纸，笔然后开始画画，不一会儿小明就画完了一幅风景图。在这个例子当中，画板就好比`Activity`，纸就好比`Window`，而笔就是`View`，我们所看到的就是这幅画，是通过笔一点一点画出来的，在哪里画呢？当然是纸上了，而最终承载这幅画的东西就是画板了。这么说可能不太生动，下面，我们从源码的角度来看看这三者的关系。

### Activity的创建过程

我们都知道，Activity启动的时候是从ActivityThread中的Handler中发起的，然后经过handlerLauncher等一系列方法，如果还不知道的话可以去参考我之前写的[一步一步带你探索Activity的启动流程](https://github.com/24Kshign/Android-Knowledge/blob/a8b38bf4765df13fee3a573249a1dc472f3a7e6c/Android%E6%BA%90%E7%A0%81%E7%9B%B8%E5%85%B3/Activity%E7%9A%84%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.md)

```
ActivityThread类：

private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {

	...
	WindowManagerGlobal.initialize();
	Activity a = performLaunchActivity(r, customIntent);
	...
}
```

在这里先调用了`WindowManagerGlobal`中的初始化方法初始化了`WindowManagerService`，看名字大概就能知道这是一个`WindowManager`的服务，通过这个服务可以对页面进行操作；然后通过调用`performLaunchActivity`方法生成了一个Activity。


### Window的创建过程

上面通过`performLaunchActivity`方法生成了一个Activity，我们来看看是怎样生成的：

```
ActivityThread类：

private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {

	...
	Activity activity = null;
	try {
		activity = mInstrumentation.newActivity(cl, 
				component.getClassName(), r.intent);
	} catch (Exception e) {
		...
	}
	...
	
	if (activity != null) {
		activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);
	}

	...
}
```
在这个方法中，通过`newActivity`这个方法（反射）来生成了一个`Activity`，生成好了`Activity`之后就调用`Activity`中的`attach`方法，来看一下这个方法里面干了些什么：

```
final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback) {
	        
	    mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(this);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
            mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {
            mWindow.setUiOptions(info.uiOptions);
        }           
}
```
果然，在`Activity`的`attach`方法中创建了一个`Window`，这个`Window`就是我们经常听到的`PhoneWindow`

### View的创建过程

我们大胆的猜测一下，`View`应该是被添加到`Window`中的，那么我们来看一下，到底是怎样添加的呢？上面说到在`handlerLauncher`中调用了`performLaunchActivity`方法，源码中还调用了`handleResumeActivity`方法，这个方法是在生命周期`onCreate`之后，`onResume`之前调用的，我们来看一下在这个方法中干了些什么：

```
final void handleResumeActivity(IBinder token,
            boolean clearHide, boolean isForward, boolean reallyResume, int seq, String reason) {
	
	...
	r = performResumeActivity(token, clearHide, reason);
	...
	if (r.window == null && !a.mFinished && willBeVisible) {
		 r.window = r.activity.getWindow();
         View decor = r.window.getDecorView();
         decor.setVisibility(View.INVISIBLE);
         ViewManager wm = a.getWindowManager();
         WindowManager.LayoutParams l = r.window.getAttributes();
         a.mDecor = decor;
         
         ...
         wm.addView(decor, l);
         ...
	}           
}
```
这里会先获取一个`Window`和`DecorView`，然后拿到`ViewManager`（`WindowManager`的父类），然后调用`addView`方法，`ViewManager`和`WindowManager`都是接口，那么我们只要到他的实现类`WindowManagerImpl`中去找`addView`方法就可以了：

```
    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
    }
```
这个`mGlobal`就是我们之前的`WindowManagerGlobal`，看到这里相信大家应该有点眉目了吧，最终是由这货负责把`DecorView`添加到Window中，在`WindowManagerGlobal`中的`addView`方法中还会初始化`ViewRootImpl`，有兴趣的可以自行看源码了解一下

`XML`中的`View`是如何添加到`DecorView`中的这个也不在这里分析了，可以参考我之前写的[一步一步带你解析setContentView源码](https://github.com/24Kshign/Android-Knowledge/blob/a8b38bf4765df13fee3a573249a1dc472f3a7e6c/Android%E6%BA%90%E7%A0%81%E7%9B%B8%E5%85%B3/setContentView%E6%BA%90%E7%A0%81.md)

### 总结

啥也不说了，上图

![](http://ooaap25kv.bkt.clouddn.com/18-6-15/38756758.jpg)