## EventBus源码详解

### 1、介绍
一个个事件(event)发送到总线上，然后EventBus根据已注册的订阅者(subscribers)来匹配相应的事件，进而把事件传递给订阅者

### 2、描述
**(1)、**onCreate方法中注册EventBus——>`EventBus.getDefault().register(this)`

**(2)、**onDestroy方法中取消注册EventBus——>`EventBus.getDefault().unregister(this)`

**(3)、**把需要接收的方法采用注解Subscribe

**(4)、**在任意的地方只要调用EventBus.post，就会执行Subscribe注解方法（前提是同类型的对象）

### 3、步骤
#### register()注册

**第一步：findUsingReflectionInSingleClass()**

去解析注册者对象的所有方法并且找出带有Subscriber注解的方法，让通过Annotation解析所有细节参数（线程，优先级，粘性事件，参数类型），把这些参数封装成一个SubscriberMethod对象，添加到集合并返回。

**第二步：调用subscribe()方法进行订阅**

解析所有SubscriberMethod的eventType（参数的class），然后按照要求解析成Map<Class<?>, CopyOnWriteArrayList<Subscription>>格式，key是指eventType，value就是Subscription的列表，Subscription包含两个属性Subscriber和SubscriberMethod

#### post发送消息
**第一步：postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();**

首先获取是否在主线程post消息的标志（这里稍后会用到），然后循环调用postSingleEvent()方法，然后再调用postSingleEventForEventType()方法，通过参数event和参数的class来获取一个CopyOnWriteArrayList<Subscription>集合，然后遍历这个集合，拿到相应的信息去进行发送操作（调用postToSubscription()方法），然后通过判断threadMode的类型，通过反射去执行。

#### ThreadMode类型

**(1)、POSTING**

同一个线程；在哪个线程发送事件那么该方法就在哪个线程执行，EventBus默认就是这种类型

**(2)、MAIN**

主线程；如果发布者线程是主线程，那么直接在发布者线程(主线程)里边调用事件处理方法；如果发布者线程不是主线程，就把此事件送到主线程消息循环处理队列，在主线程中处理此事件

**(3)、BACKGROUND**

子线程；如果发布者线程是主线程，那么把此事件发送到一个专门处理后台线程的消息循环处理队列，该队列管理多个后台线程；如果发布者不是主线程，那么在发布者线程中直接调用事件处理方法

**(4)、ASYNC**

异步线程；并不使用队列管理多个事件，也不管发布者处在主线程与否，为每一个事件单独开辟一个线程处理

#### unregister()移除
将订阅者从集合中移除，根据传入的参数（Activity），然后通过之前存的Map<Class<?>, CopyOnWriteArrayList<Subscription>>集合，挨个移除即可。

#### 粘性事件
简单讲，就是在发送事件之后再订阅该事件也能收到该事件。它的使用场景是我们要把一个Event发送到一个还没有初始化的Activity/Fragment，即尚未订阅事件。那么如果只是简单的post一个事件，那么是无法收到的，这时候，你需要用到粘性事件,它可以帮你解决这类问题。
