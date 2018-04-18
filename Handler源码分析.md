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
Handler.sendMessage方法只是将该消息发送到消息队列，队列采用的链表的方式，按照when(也就是时间)排序

### 3、Looper消息循环

#### Looper.prepare()准备循环
原则上来说，在任何一个线程中使用Handler必须先调用Looper.prepare()，否则就会报错。但是主线程中并没有调用Looper.prepare()，原因是在应用程序的入口ActivityThread.main()方法中，系统已经为我们调用了Looper.prepareMainLooper()，所以我们可以正常使用。

```
sThreadLocal.set(new Looper(quitAllowed))
```

```
Thread t = Thread.currentThread();
//从线程中获取ThreadLocalMap集合，一个线程对应ThreadLocalMap集合
ThreadLocalMap map = getMap(t);
if (map != null) {
    map.set(this, value);
}
```
其实就是创建一个Looper对象，**并且保证一个Thread线程中，只有一个Looper对象**。

#### Looper.loop()循环
```
   /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        //获取线程的Looper对象
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        //获取消息队列
        final MessageQueue queue = me.mQueue;
        
        //死循环不断获取消息队列中的消息
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            try {
                //通过handler去执行Message，这个时候就调用了handleMessage方法
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            //回收消息
            msg.recycleUnchecked();
        }
    }
```
loop的过程就是不断的轮训消息队列，若有消息的话，就让handler去处理消息