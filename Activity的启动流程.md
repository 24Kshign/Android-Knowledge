### Activity的启动流程：

——> 先调用performLauncherActivity

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
















