## LayoutInflater源码解析

### 序言：最近在做一个类似商城的app（电商类），然后我就用了阿里新开源的VLayout布局来做，简直不要太方便啊，有空的话我会写一篇关于VLayout的文章，它的底层也是RecyclerView，只不过对LayoutManager和Adapter进行了封装而已，但是我们用的时候也需要对它进行再一次封装，因为像addHeader和addFooter这些都是没有的，在这个过程中我偶然发现用LayoutInflater实例的header和footer宽高竟然没用，这让我很捉急（因为项目很赶），然后百度才发现LayoutInflater通过XML加载布局的时候会产生不同的现象。

#### 1、使用方法

(1)、获取LayoutInflater实例有两种方法：

```
LayoutInflater layoutInflater = LayoutInflater.from(context);

LayoutInflater layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

(2)、inflate方法：

```
layoutInflater.inflate(int layoutId, ViewGroup root);

/**
 * layoutId      布局文件
 * root          在该布局的最外层嵌套一层父布局，如果不需要就传null
 * attachToRoot  true表示将layout的view添加到root中
 */
layoutInflater.inflate(int layoutId, ViewGroup root, attachToRoot);

layoutInflater.inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot)
```
>**注意：这里重点解释一下第三个参数attachToRoot，有以下几种情况：**
>
>**(1)、如果root为null，无论attachToRoot为true或者false，效果都是一样的**
>
>**(2)、如果root不为null，attachToRoot为true，表示将layout布局添加到root布局中**
>
>**(3)、如果root不为null，attachToRoot为false，表示不将layout布局添加到root布局，若要添加则需要手动addView**
>
>**(4)、如果root不为null，不设置attachToRoot(即调用两个参数的方法)，情况和(2)中一样**

前两种方法是最常用的加载布局的方法，最终都会调用最后一种方法来加载布局：

```
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
	...
	
	if (TAG_MERGE.equals(name)) {
        if (root == null || !attachToRoot) {
            throw new InflateException("<merge /> can be used only with a valid "+ "ViewGroup root and attachToRoot=true");
        }
        rInflate(parser, root, inflaterContext, attrs, false);
    }else {
        // Temp is the root view that was found in the xml
        final View temp = createViewFromTag(root, name, inflaterContext, attrs);
        ViewGroup.LayoutParams params = null;
        if (root != null) {
        	if (DEBUG) {
                System.out.println("Creating params from root: " + root);
            }
            // Create layout params that match root, if supplied
            params = root.generateLayoutParams(attrs);
            if (!attachToRoot) {
                // Set the layout params for temp if we are not
                // attaching. (If we are, we use addView, below)
                temp.setLayoutParams(params);
            }      
        }
        
        ...
        
        // We are supposed to attach all the views we found (int temp)
        // to root. Do that now.
        if (root != null && attachToRoot) {
           root.addView(temp, params);
        }
                    
        ...
        
    }
    
    ...
}
```
>LayoutInflater是通过Pull解析（XmlPullParser解析器）方式来解析XML布局文件，解析出节点名之后，然后会调用rInflate方法，这个方法里面会遍历这个根布局下的子元素。然后会调用createViewFromTag方法，这个方法是干嘛的呢？顾名思义创建View的呗，创建View 之后会判断root是否为null，若不为null会为其生成一个params，然后设置给该View，如果root为null，则不会生成params，所以就没有宽高这些数据了。最后若root不为null会调用addView方法，将该生成的View添加到root布局中。

```
    void rInflate(XmlPullParser parser, View parent, Context context, AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {
        final int depth = parser.getDepth();
        int type;
        boolean pendingRequestFocus = false;

        while (((type = parser.next()) != XmlPullParser.END_TAG ||
                parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }

            final String name = parser.getName();

            if (TAG_REQUEST_FOCUS.equals(name)) {
                pendingRequestFocus = true;
                consumeChildElements(parser);
            } else if (TAG_TAG.equals(name)) {
                parseViewTag(parser, parent, attrs);
            } else if (TAG_INCLUDE.equals(name)) {
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                parseInclude(parser, context, parent, attrs);
            } else if (TAG_MERGE.equals(name)) {
                throw new InflateException("<merge /> must be the root element");
            } else {
                final View view = createViewFromTag(parent, name, context, attrs);
                final ViewGroup viewGroup = (ViewGroup) parent;
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                rInflateChildren(parser, view, attrs, true);
                viewGroup.addView(view, params);
            }
        }

        if (pendingRequestFocus) {
            parent.restoreDefaultFocus();
        }

        if (finishInflate) {
            parent.onFinishInflate();
        }
    }
```
>细心的朋友可能会发现，上面两段代码都调用了同一个方法createViewFromTag，每次都会递归调用这个方法来创建这个View下的子元素并且添加到根布局中。那我们来看一下这个方法里面都干了些什么：

```
    View createViewFromTag(View parent, String name, Context context, AttributeSet attrs, boolean ignoreThemeAttr) {
        if (name.equals("view")) {
            name = attrs.getAttributeValue(null, "class");
        }

        // Apply a theme wrapper, if allowed and one is specified.
        if (!ignoreThemeAttr) {
            final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
            final int themeResId = ta.getResourceId(0, 0);
            if (themeResId != 0) {
                context = new ContextThemeWrapper(context, themeResId);
            }
            ta.recycle();
        }

        if (name.equals(TAG_1995)) {
            // Let's party like it's 1995!
            return new BlinkLayout(context, attrs);
        }

        try {
            View view;
            if (mFactory2 != null) {
                view = mFactory2.onCreateView(parent, name, context, attrs);
            } else if (mFactory != null) {
                view = mFactory.onCreateView(name, context, attrs);
            } else {
                view = null;
            }

            if (view == null && mPrivateFactory != null) {
                view = mPrivateFactory.onCreateView(parent, name, context, attrs);
            }

            if (view == null) {
                final Object lastContext = mConstructorArgs[0];
                mConstructorArgs[0] = context;
                try {
                    if (-1 == name.indexOf('.')) {
                        view = onCreateView(parent, name, attrs);
                    } else {
                        view = createView(name, null, attrs);
                    }
                } finally {
                    mConstructorArgs[0] = lastContext;
                }
            }

            return view;
        } catch (InflateException e) {
            throw e;

        } catch (ClassNotFoundException e) {
            final InflateException ie = new InflateException(attrs.getPositionDescription()
                    + ": Error inflating class " + name, e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;

        } catch (Exception e) {
            final InflateException ie = new InflateException(attrs.getPositionDescription()
                    + ": Error inflating class " + name, e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        }
    }
```
>代码不是很多，这个方法把我们传进去的xml的节点名来生成一个View对象，如何生成的呢，调用了一个createView方法（onCreateView方法中也会调用createView方法）：

```
public final View createView(String name, String prefix, AttributeSet attrs) throws ClassNotFoundException, InflateException {
	Constructor<? extends View> constructor = sConstructorMap.get(name);
    if (constructor != null && !verifyClassLoader(constructor)) {
        constructor = null;
        sConstructorMap.remove(name);
    }
    ...
    final View view = constructor.newInstance(args);
    ...
}
```
>看到这里大家都知道了吧，通过反射的方法来创建出最终的View。

好了，相信看到这里大家应该都能明白为什么有时候LayoutInflater出来的View宽高会失效了吧！

### 总结：LayoutInflater有两个加载布局的方法，分别是两个参数和三个参数的，这里有以下几种情况：

####(1)、如果root为null，无论attachToRoot为true或者false，返回的都是一个不带LayoutParams的View。
####(2)、如果root不为null，attachToRoot为true，会返回一个带有LayoutParams的View，并且该View会添加到root布局。
####(3)、如果root不为null，attachToRoot为false，也会返回一个带有LayoutParams的View，但不会添加到root布局。
####(4)、如果root不为null，不设置attachToRoot(即调用两个参数的方法)，情况和(2)中一样。