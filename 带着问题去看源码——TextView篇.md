
### 序言：为什么会分析这个问题呢，因为上次钉钉电话面试中被面试官问到了，很尴尬的没回答出来，View的绘制流程看过一点源码，但是感觉还不够，像这种View的问题能够延伸出很多问题，下面正文开始：
##### Q1：在一个RelativeLayout中有一个TextView和一个Button，当点击Button的时候给TextView设置文本，这时RelativeLayout会重新测量吗？如果会，为什么？

首先我们先大致的想一下这个问题问的是关于哪一块的知识，如果毫不犹豫上去就是一通回答，这样显得太不明智了，我也知道会重新测量，为什么？下面我们从源码的角度去看。既然是设置文本，那么我们就从TextView的setText中去看看吧：

**TextView：**

```

private void setText(CharSequence text, BufferType type, boolean notifyBefore, int oldlen) {

	...
	
	if (mLayout != null) {
        checkForRelayout();
    }
    
    ...

}

```
在setText中，我找到了这个`checkForRelayout`方法，由于我们初始化过了，所以setText肯定会执行该方法：

**TextView：**

```
   /**
     * Check whether entirely new text requires a new view layout
     * or merely a new text layout.
     * 检查新文本是否需要一个新的视图布局
     */
    private void checkForRelayout() {
        // If we have a fixed width, we can just swap in a new text layout
        // if the text height stays the same or if the view height is fixed.
        //如果textview的宽度和高度固定不变的话
        if ((mLayoutParams.width != LayoutParams.WRAP_CONTENT
                || (mMaxWidthMode == mMinWidthMode && mMaxWidth == mMinWidth))
                && (mHint == null || mHintLayout != null)
                && (mRight - mLeft - getCompoundPaddingLeft() - getCompoundPaddingRight() > 0)) {
            // Static width, so try making a new text layout.

            int oldht = mLayout.getHeight();
            int want = mLayout.getWidth();
            int hintWant = mHintLayout == null ? 0 : mHintLayout.getWidth();

            /*
             * No need to bring the text into view, since the size is not
             * changing (unless we do the requestLayout(), in which case it
             * will happen at measure).
             * 因为大小不会变，所以不需要将文本放入视图中
             */
            makeNewLayout(want, hintWant, UNKNOWN_BORING, UNKNOWN_BORING,
                          mRight - mLeft - getCompoundPaddingLeft() - getCompoundPaddingRight(),
                          false);

            if (mEllipsize != TextUtils.TruncateAt.MARQUEE) {
                // In a fixed-height view, so use our new text layout.
                if (mLayoutParams.height != LayoutParams.WRAP_CONTENT
                        && mLayoutParams.height != LayoutParams.MATCH_PARENT) {
                    autoSizeText();
                    invalidate();
                    return;
                }

                // Dynamic height, but height has stayed the same,
                // so use our new text layout.
                //动态高度，但是高度不变
                if (mLayout.getHeight() == oldht
                        && (mHintLayout == null || mHintLayout.getHeight() == oldht)) {
                    autoSizeText();
                    invalidate();
                    return;
                }
            }

            // We lose: the height has changed and we have a dynamic height.
            // Request a new view layout using our new text layout.
            requestLayout();
            invalidate();
        } else {
            // Dynamic width, so we have no choice but to request a new
            // view layout with a new text layout.
            //动态宽度，我们只能请求一个新的布局
            nullLayouts();
            requestLayout();
            invalidate();
        }
    }

```
说实话，看源码真的是一件很累的过程，我们可能很难找到下手的点，在这里给大家分享一个方法，找你觉得是重点的代码或者方法去看（和你本次看源码想要研究的方向相同），一旦你看着看着发现看不太懂了，你就倒回来再看其他地方。从上面这段代码我们不难看出，在这里调用了`requestLayout()`和`invalidate()`这两个方法，看到这里相信大家应该就明白了吧，是他是他就是他，`requestLayout()`，好，我们也顺便来看一下这个方法：

**TextView：**

```

   public void requestLayout() {
   		//判断是否正在布局
        if (mMeasureCache != null) mMeasureCache.clear();

        if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == null) {
            // Only trigger request-during-layout logic if this is the view requesting it,
            // not the views in its parent hierarchy
            ViewRootImpl viewRoot = getViewRootImpl();
            if (viewRoot != null && viewRoot.isInLayout()) {
                if (!viewRoot.requestLayoutDuringLayout(this)) {
                    return;
                }
            }
            mAttachInfo.mViewRequestingLayout = this;
        }

        mPrivateFlags |= PFLAG_FORCE_LAYOUT;
        mPrivateFlags |= PFLAG_INVALIDATED;

        if (mParent != null && !mParent.isLayoutRequested()) {
            //向父容器请求布局
            mParent.requestLayout();
        }
        if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == this) {
            mAttachInfo.mViewRequestingLayout = null;
        }
    }

```
在`requestLayout()`方法中调用了`mParent.requestLayout()`，也就是说调用了父View的`requestLayout()`方法，然后一级一级往上传，最终会调用ViewRootImpl中的`requestLayout()`方法：

**RootViewImpl：**

```
@Override
public void requestLayout() {
    if (!mHandlingLayoutInLayoutRequest) {
        checkThread();
        mLayoutRequested = true;
        scheduleTraversals();
    }
}
```
首先会先去判断一下是否是在当前线程，然后会调用`scheduleTraversals()`方法：

**RootViewImpl：**

```
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
```

看到这里估计有很多人会懵逼了，这里好像也没有什么嘛，别急老铁，这里有一个名叫`mTraversalRunnable`的参数，那我们就点进去看看他的实现：

**RootViewImpl：**

```
    final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }
    final TraversalRunnable mTraversalRunnable = new TraversalRunnable();
```

接下来会调用doTraversal()方法：

**RootViewImpl：**

```
   void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```

然后终于到了我们所期待的地方了，这里又调用了`performTraversals()`方法，相信看过View的绘制流程源码的童鞋应该就知道了，这里才真正开始View的测量，摆放，绘制等操作。在`performTraversals()`这个方法中分别调用了onMeasure，onLayout，onDraw等方法，有兴趣的童鞋可以自行去看。至此我们应该就能知道上述问题该如何回答了。

##### Q2：为什么TextView的宽高设置成wrap_content，在Activity中获取的时候宽度为0，高度不为0？

![](http://ooaap25kv.bkt.clouddn.com/18-3-23/28928051.jpg)

这个问题呢，是我在找上面那个问题的答案的过程中发现的，既然是宽高的问题，那么我们当然得要去看onMeasure方法了：

```
if (widthMode == MeasureSpec.EXACTLY) {
     // Parent has told us how big to be. So be it.
     width = widthSize;
} else {
     //宽度设置成wrap_content会走这里
     if (mLayout != null && mEllipsize == null) {
           des = desired(mLayout);
     }

     if (des < 0) {
          boring = BoringLayout.isBoring(mTransformed, mTextPaint, mTextDir, mBoring);
          if (boring != null) {
               mBoring = boring;
           }
      } else {
           fromexisting = true;
      }

      if (boring == null || boring == UNKNOWN_BORING) {
          if (des < 0) {
              des = (int) Math.ceil(Layout.getDesiredWidth(mTransformed, 0, mTransformed.length(), mTextPaint, mTextDir));
          }
      } else {
           width = boring.width;
      }
      ...
      这下面是计算设置drawable和hint的宽度，所以我们可以忽略
}
```
关于宽度的我们只需要看这一段就好了，TextView设置wrap_content，会走下面的else，然后第一次进来，这个onMeasue里面的mLayout还没有初始化，所以`mLayout = null`，然后由于`des = -1`，所以boring会被初始化，`boring != null`，所以`width = boring.width`，而boring.width这个东西初始值为0，所以`width = 0`；同样的高度也是这样分析就可以了，要注意的是，高度和textSize和行数有关，所以设置不同的行数和textSize（默认是有TextSize的）得到的hight都不一样的。

**总结：**其实大家可以这样理解，宽高都设置成wrap_content，没设置文本的情况下，宽度肯定为0，但是单行的高度是固定的（和TextSize也有关，一旦设置也是固定的了）。