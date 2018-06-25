## 是时候好好管理一下啊你的RxJava2.x

### 序言，最近在测试的时候发现项目中有的网络请求没有带上项目中封装的RxJava生命周期管理器，然后就导致了某个Activity在销毁之后，网络请求依然在，等到响应的时候View已经为空了（Unbinder.unbind），这个时候就报空指针了，吓得我立马去检查还有没有某些漏网之鱼也没有加的。这个就是我们常说的RxJava内存泄漏。

![](http://ooaap25kv.bkt.clouddn.com/18-6-22/24237331.jpg)

如果你还不知道RxJava2.0会发生内存泄漏或者你还不知道网络请求会出现上述情况的话，那么，你还是装死好了，本篇博客将针对Retrofit2.0+RxJava2.0来进行介绍，有两种方法可以避免内存泄漏问题。

### 一、Disposable / CompositeDisposable

首先我们来写一段代码，在Activity中每隔一段时间发送一个事件，然后用TextView来显示：

```
        Observable.interval(2, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable d) {

                    }

                    @Override
                    public void onNext(Long aLong) {
                        textView.setText(String.valueOf(aLong));
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });
```

![](http://ooaap25kv.bkt.clouddn.com/18-6-22/78624525.jpg)

相信大家都看到了，退出activity的时候，我们把做了unbind操作，但是rxjava发送事件还没有停止，然后就发生了上述悲剧。接下来我们使用官方回调给我们的Disposable这个东西来让Rxjava的事件停止，只需要在onDestroy中切断Observable（被观察者）和Observer（观察者）的联系，使用Disposable.dispose()方法即可，那么在