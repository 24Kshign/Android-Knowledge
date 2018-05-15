## Handler源码分析
### 1、介绍
Handler是用来结合线程的消息队列来发送、处理**Message对象**和**Runnable对象**的工具。每一个Handler实例之后会关联一个线程和该线程的消息队列。当你创建一个Handler的时候，从这时开始，它就会自动关联到所在的线程/消息队列，然后它就会陆续把Message/Runnalbe分发到消息队列，并在它们出队的时候处理掉。

### 2、MessageQueue消息队列
Handler.sendMessage(Message message)

-->sendMessageDelayed(Message msg, long delayMillis)

-->MessageQueue.enqueueMessage(msg, uptimeMillis)

```
Message p = mMessages;
boolean needWake;
if (p == null || when == 0 || when < p.when) {
     // New head, wake up the event queue if blocked.
     msg.next = p;
     mMessages = msg;
     needWake = mBlocked;
} else {
     // Inserted within the middle of the queue.  Usually we don't have to wake
     // up the event queue unless there is a barrier at the head of the queue
    // and the message is the earliest asynchronous message in the queue.
    needWake = mBlocked && p.target == null && msg.isAsynchronous();
    Message prev;
    for (;;) {
        prev = p;
        p = p.next;
        if (p == null || when < p.when) {
            break;
        }
        if (needWake && p.isAsynchronous()) {
            needWake = false;
        }
    }
    msg.next = p; // invariant: p == prev.next
    prev.next = msg;
}
```
Handler.sendMessage方法只是将该消息发送到消息队列，队列采用的链表的方式，按照when(也就是时间)排序。

##### 注意：链表是增删快，数组是查询快

##### 问题：能不能在线程中创建一个handler？
答：答案是不能直接创建，还需要调用Looper.prepare()方法，详情可以看下面。

### 3、Looper消息循环

#### Looper.prepare()准备循环
原则上来说，在任何一个线程中使用Handler必须先调用Looper.prepare()，否则就会报错。但是主线程中并没有调用Looper.prepare()，原因是在应用程序的入口ActivityThread.main()方法中，系统已经为我们调用了Looper.prepareMainLooper()，所以我们可以正常使用，下面来看看Handler的构造方法：

```
public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}

```
我们可以看到，创建handler的时候会调用Looper.myLooper()方法来获取一个Looper，如果Looper为空的话就会报错，那么来myLooper里面看看：

```
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```
首先通过当前线程获取一个ThreadLocal的集合，因为我们什么也没干，所以这里拿到的会是null，所以会调用setInitialValue()方法，这个方法会创建一个集合，并且将以当前线程为key的value值置为null，所以我们再myLooper中获取到的也就是null了；那么怎么样才能让它不报错呢，在创建handler之前加一个Looper.prepare()方法，那么这个方法又是干嘛的呢：

```
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```
这里其实就是创建了一个Looper对象，并且把它加到集合中，这样的话我们在前面调用Looper.myLooper()方法时就不会报错了。那么我们又是怎么获取消息进行处理的呢：

#### Looper.loop()循环

```
public static void loop() {
    final Looper me = myLooper();
    
    ...

    for (;;) {
        Message msg = queue.next(); // might block
        ...
        try {
            msg.target.dispatchMessage(msg);
            end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
        ...
    }
    ...
}

public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```
从上面我们可以看到，loop的过程就是不断的轮训消息队列（for死循环），然后会调用handler.dispatchMessage()方法去处理消息，这里有三种情况：

**第一，msg中的callback（Runnable）不为空，也就是handler是通过handler.post(Runnable)的形式去发送一个消息的，这种情况会回调Runnable中的run方法；**

**第二，初始化handler的时候穿了CallBack参数，这个时候会回调CallBack中的handleMessage方法**

**第三，不满足上述两种情况的最终会回调handleMessage()方法，所以最终我们重写handlerMessage()方法就能处理消息了。**


### 4、面试题

#### Q：handler发送一个延迟消息之后，在发送完到处理的这段时间，Looper处于什么状态（handler延时发送消息的流程）？

A：上面我们都说过了，Handler在sendMessage的时候，消息队列中是根据delay延迟时间进行排序的，如果延迟时间过长，那么这个时间Looper在干什么呢？我们可以从Looper中的loop()方法中找答案：

```
public static void loop() {
    ...
    
    final MessageQueue queue = me.mQueue;
    
    ...

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
        ...
    }
}
```

我们可以看到，loop方法中是一个死循环，通过quene.next方法去拿到一个message，只有当messsage为null的情况下才会return，我们来看看MessageQueue里面的next方法：

```
Message next() {
    ...

    int pendingIdleHandlerCount = -1;
    int nextPollTimeoutMillis = 0;
    for (;;) {
        //阻塞操作
        nativePollOnce(ptr, nextPollTimeoutMillis);
        //循环消息队列中拿消息
        synchronized (this) {
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            //检索消息，如果找到了想要找的消息就返回
            if (msg != null && msg.target == null) {
                // 在队列中查找下一个异步消息
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                //如果延迟时间大于当前时间
                if (now < msg.when) {
                    //为下一个还未准备好的消息（delay时间还没到）设置一个超时，以便在准备好的时候唤醒
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 获取到了一个消息
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                //没有消息
                nextPollTimeoutMillis = -1;
            }

            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }

            /下面是关于PendingIdleHandlers的处理
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // loop需要等待
                mBlocked = true;    // 可以控制是否需要唤醒线程
                continue;
            }

            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        ...

        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}
```
通过上述源码我们不难看出，Looper在消息还未准备好时会被阻塞，等待下一个准备好的消息（也就是delay时间到了），自动唤醒，将该消息取出给handler处理，然后又会进入next方法，继续判断当前时间是否小于下一条消息的delay时间。