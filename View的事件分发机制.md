### Android的事件分发机制
#### 1 事件分发

* 定义: 将点击事件(MotionEvent)传递给某个View或者处理&整个过程.
* 解决的问题: 点击事件由那个对象发出,经过哪些对象,最终达到哪些对象&最终得到处理.
* 顺序: Activity->ViewGroup->View.
* 核心方法: dispatchTouchEvent,onInterceptTouchEvent,onTouchEvent.
* 还有一个设置setOnTouchListener的方法.

#### 2 事件的总线有两条
* 事件从上到下分发,也就是最外层的ViewGroup1->View1...
如果中间没有任何View或者ViewGroup消费掉次事件的话,事件将一直传下去.
方法的传递顺序就是:
```
ViewGroup:dispatchTouchEvent->
ViewGroup:onInterceptTouchEvent->
     View:dispatchTouchEvent->
     View:onInterceptTouchEvent->
     View:onTouchEvent...
```
* 时间从下往上分发,当事件向下传递的过程中没有被任何View消费,那么事件会传到最后的那个View里面,然后再返回到最上层的View中.最后的终点就是一开始传递事件的那个View.View2->ViewGroup2...方法的传递顺序就是:
```
View:onTouchEvent -> ViewGroup:onTouchEvent...
```


#### 3 涉及到的方法

**(1)、dispatchTouchEvent**</br>
该方法是View和ViewGroup共有的方法,当点击事件传递过来的时候是最先接收到事件的.该方法决定了该View是否继续往下分发事件.也就是它的onInterceptTouchEvent和onTouchEvent是否会执行到.返回值有三种:
* true</br>
当返回true的时候,此次事件将不会再往下分发.也就是此次事件消费结束.

* false </br>
当返回false的时候,本身不会再处理任何事件,事件将被返回到上一级的onTouchEvent方法里面.也就是本身的onInterceptTouchEvent或者onTouchEvent都不会执行.

* super</br>
将事件分发到自己的onInterceptTouchEvent方法里面.



**(2)、onInterceptTouchEvent**
该方法是ViewGroup特有的,它可以决定你手否拦截次事件.再dispatchTouchEvent方法里面会调用.如果拦截事件(返回值为true),则该事件将交给自己的onTouchEvent方法处理,不再继续分发事件.返回值有三种:
* true</br>
当返回true的时候,表明拦截此事件,把事件交给自己的onTouchEvent处理,事件将不会再往子View分发.

* false </br>
当返回false的时候,不拦截事件,事件将继续分发下去.

* super</br>
等同于返回值为false的时候,super.onInterceptTouchEvent(ev)该方法返回的也是false.不拦截事件,事件将继续分发下去.

**(3)、onTouchEvent**

该方法一般是消费事件的.可以再该方法中处理很多事情.该方法的返回值有三种:

* true</br>
当返回true的时候,表示消费事件,一旦事件被消费了则事件就会停止了,不会继续往上分发.同样返回true的话也表示消费任何事件.

* false </br>
当返回false的时候,说明不消费事件,事件将继续往上分发,直到被消费&直到最顶层.如果返回false的话,此事件以后的事件就不会再接收.

* super</br>
等同于返回值为false的时候.

**(4)、setOnTouchListener**
该方法再dispatchTouchEvent中调用,再ViewGroup中是在onInterceptTouchEvent之后调用.该方法决定了onTouchEvent方法是否会执行到.

#### 4 事件的类型

常用的事件有以下几种:
* ACTION_DOWN :一个按下的手势已经启动,该事件里面包含初始的起始位置信息.
* ACTION_UP :一个按下的手势已经结束,该事件里面包含最后的位置信息.
* ACTION_MOVE :发生在DOWN和UP之间的时间,该事件包含了移动中的任何一个点.
* ACTION_CANCEL :当前的事件中止,可以当成一个UP事件.
* ACTION_OUTSIDE :当移动事件已经滑出该View的边界的事件.
* ACTION_POINTER_DOWN :双指按下.
* ACTION_POINTER_UP :双指抬起.
```

这里要注意的是当双指按下的时候事件的走向:ACTION_DOWN->ACTION_POINTER_DOWN.
双指抬起的时候是:ACTION_POINTER_UP->ACTION_UP.
当我们想要拦截到双指事件的时候必须加上一个添加判断,
switch (event.getAction() & event.getActionMasked()).
```


#### 问题1 只拦截点击事件?
#### 问题2 只拦截滑动事件?
#### 问题3 requestDisallowInterceptTouchEvent的原理









