## Android基础题目

### 选择题

#### 1、Android四大组件中哪一个主要功能是负责与用户交互？ ([B](https://blog.csdn.net/m0_37989980/article/details/78681367))

- A. Servce

- B. Activity     

- C. Content Provider     
 
- D. BroadCast Receiver

#### 2、以下哪种不是Android数据存储的方式？  ([C](https://www.cnblogs.com/pxsbest/p/5068482.html))

- A. Shared Preferences

- B. Content Provider

- C. Bundle

- D. SQLite

#### 3、下列关于Activity的启动模式描述错误的是  （[C](https://www.cnblogs.com/lwbqqyumidi/p/3771542.html)）

- A. 设置属性为singleTop时，若目标Activity位于task栈顶，则不再重复生成，直接使用栈顶Activity。

- B. 设置属性为singleTask时，若task栈中已经存在目标Activity，则将其之上的Acticity全部出栈，并使其变为栈顶。

- C. android:taskAffinity与singleTask搭配使用可以使Activity运行在指定的栈中。

- D. 通过Intent.setFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)能实现android：launchMode=”singleTop”相同的效果。

#### 4、通过 startService() 和 bindService()，以下说法错误的是      ([A](https://blog.csdn.net/dfskhgalshgkajghljgh/article/details/51471108))

- A. 如果是调用 bindService() 启动服务：会调用如下生命周期方法：onCreate() —> onBind —> onDestory() —> onUnBind()

- B. 通过 startService() 启动服务：会调用如下生命周期方法：onCreate() —> onStart() —> onDestory()

- C. 当采用 startService() 方法启动服务，访问者与服务之间是没有绑定在一起的，访问者退出，服务还在运行

- D. 采用 bindService() 方法启动服务时，访问者与服务是绑定在一起的，即访问者退出，服务也就终止，解除绑定。

#### 5、下面关于 Android dvm 的进程和 Linux 的进程 , 应用程序的进程说法正确的是？      （[B](https://blog.csdn.net/lin111000713/article/details/52459710)）

- A. DVM 指 Dalvik 的虚拟机。每一个 Android 应用程序都在它自己的进程中运行，都拥有一个独立的 Dalvik 虚拟机实例。而每一个 DVM 不一定都是在 Linux 中的一个进程，所以说不是一个概念。

- B. DVM 指 Dalvik 的虚拟机。每一个 Android 应用程序都在它自己的进程中运行，都拥有一个独立的 Dalvik 虚拟机实例。而每一个 DVM 都是在 Linux 中的一个进程，所以说可以认为是同一个概念。

- C. DVM 指 Dalvik 的虚拟机，每一个 Android 应用程序都在它自己的进程中运行，不一定拥有一个独立的 Dalvik 虚拟机实例。而每一个 DVM 都是在 Linux 中的一个进程，所以说可以认为是同一个概念。

- D. DVM 指 Dalvik 的虚拟机，每一个 Android 应用程序都在它自己的进程中运行，不一定拥有一个独立的 Dalvik 虚拟机实例。而每一个DVM不一定都是在 Linux 中的一个进程，所以说不是一个概念。

#### 6、以下哪种场景不会导致内存泄漏？   （[D](https://www.jianshu.com/p/65f914e6a2f8)）

- A. 查询数据库时未关闭游标Cursor。

- B. 使用计时器时,未在不使用的时候关闭。

- C. 动态注册的广播未在不使用的时候unregister。

- D. 在Activity被destory的时候未将全局变量设成null。

#### 7、在滴滴打车APP中点击支付到支付宝APP中进行支付操作，出现密码输入框，到此时相关的Activity会发生的生命周期回调依次为？       ([C](https://www.cnblogs.com/lwbqqyumidi/p/3769113.html))

- A. onpause() —> ondestroy() —> oncreate() —> onresume()

- B. ondestroy() —> oncreate() —> onstart() —> onresume()

- C. onpause() —> oncreate() —> onstart() —> onresume()

- D. onstop() —> ondestroy() —> oncreate() —> onstart()

#### 8、以下关于内存回收的说明正确的是？      ([B](https://www.cnblogs.com/ganchuanpu/p/8479518.html))

- A. 程序员必须创建一个线程来释放内存。

- B. 内存回收程序负责释放无用内存。

- C. 内存回收程序允许程序员直接释放内存。

- D. 内存回收程序可以在指定的时间释放内存对象。

#### 9、关于res/raw目录说法正确的是？   ([A](https://blog.csdn.net/ztchun/article/details/60809640))

- A. 这里的文件是原封不动的存储到设备上不会转换为二进制的格式。

- B. 这里的文件是原封不动的存储到设备上会转换为二进制的格式。

- C. 这里的文件最终以二进制的格式存储到指定的包中。

- D. 这里的文件最终不会以二进制的格式存储到指定的包中。

#### 10、关于Handler的说法不正确的是？  ([A](https://www.cnblogs.com/wlming/p/5553207.html))

- A. 它实现不同进程间通信的一种机制。

- B. 它避免了在新线程中刷新UI的操作。

- C. 它采用队列的方式来存储Message。

- D. 它实现不同线程间通信的一种机制。