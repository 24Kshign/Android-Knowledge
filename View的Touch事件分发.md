### View的Touch事件分发

#### 1、现象
OnTouch，OnTouchEvent，OnClickListener三者都有时：

**(1)、OnTouch返回false**

执行顺序为：OnTouch.DOWN---->OnTouchEvent.DOWN---->OnTouch.MOVE---->OnTouchEvent.MOVE---->OnTouch.UP---->OnTouchEvent.UP---->OnClickListener

**(2)、OnTouch返回true**

执行顺序为：OnTouch.DOWN---->OnTouch.MOVE---->OnTouch.UP，不会执行OnTouchEvent和OnClickListener方法

**(3)、没有OnTouch，OnTouchEvent返回true**
执行顺序为：OnTouchEvent.DOWN---->OnTouchEvent.MOVE---->OnTouchEvent.UP，不会执行OnClickListener

#### 2、View中有两个非常重要的与Touch相关的方法

**(1)、dispatchTouchEvent**

```
boolean result = false;
ListenerInfo li = mListenerInfo;
```
ListenerInfo存放了关于View的所有Listener信息，如OnTouchListener，OnClickListener

```
if (li != null && li.mOnTouchListener != null
               && (mViewFlags & ENABLED_MASK) == ENABLED
               && li.mOnTouchListener.onTouch(this, event)) {
    result = true;
}
```
ENABLE表示该View是否enable；若mOnTouchListener.onTouch(this, event)返回false，则result = false，否则result = true

```
if (!result && onTouchEvent(event)) {
    result = true;
}
```

如果result为false，则View会执行onTouchEvent事件，如果result为false，则View不会执行onYouchEvent事件

```
if (!post(mPerformClick)) {
    performClick();
}
```
在View的onTouchEvent()方法的ACTION_UP中调用了performClick()方法，然后调用li.mOnClickListener.onClick(this)方法(OnClickListener方法)，所以上述分析可以解释1中的现象

**(2)、onTouchEvent**

一般都会被我们复写

#### 3、在ViewGroup中还有一个和OnTouch相关的方法

**onInterceptTouchEvent方法**

```
if (!disallowIntercept) {
    intercepted = onInterceptTouchEvent(ev);
    ev.setAction(action); // restore action in case it was changed
} else {
    intercepted = false;
}
```

每次有事件时，都会按照Activity---->ViewGroup---->View这样的线路开始分发，而且最先调用的肯定是dispatchTouchEvent()方法，在ViewGroup的dispatchTouchEvent方法中会调用onInterceptTouchEvent()方法询问是否拦截事件(返回false表示不拦截事件，事件会继续往下一层传递；true表示拦截，事件不会往下传了，这时会调用ViewGroup的OnTouchEvent方法，自己消费，而且在同一个事件列的其他事件中也都直接自己消费，不会往下传递)，还会遍历该ViewGroup下的所有子View。