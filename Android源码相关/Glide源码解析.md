## Glide源码解析

### 用法
加载图片：Glide.with(this).load(url).into(imageView);

加载图片且占位：Glide.with(this).load(url).placeholder(placeholderRes).into(imageView);

### 解析
#### 1、Glide.with()方法

```
public static RequestManager with(Context context) {
    RequestManagerRetriever retriever = RequestManagerRetriever.get();
    return retriever.get(context);
}

public static RequestManager with(Activity activity) {
    RequestManagerRetriever retriever = RequestManagerRetriever.get();
    return retriever.get(activity);
}

public static RequestManager with(Fragment fragment) {
    RequestManagerRetriever retriever = RequestManagerRetriever.get();
    return retriever.get(fragment);
}
```

我们可以看到with()方法是一组重载方法，接收的参数有很多，Context，Activity，Fragment，每一个都会调用RequestManagerRetriever的get()方法：

```
    public RequestManager get(Context context) {
        if (context == null) {
            throw new IllegalArgumentException("You cannot start a load on a null Context");
        } else if (Util.isOnMainThread() && !(context instanceof Application)) {
            if (context instanceof FragmentActivity) {
                return get((FragmentActivity) context);
            } else if (context instanceof Activity) {
                return get((Activity) context);
            } else if (context instanceof ContextWrapper) {
                return get(((ContextWrapper) context).getBaseContext());
            }
        }
        return getApplicationManager(context);
    }
    
    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    public RequestManager get(Activity activity) {
        if (Util.isOnBackgroundThread() || Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
            return get(activity.getApplicationContext());
        } else {
            assertNotDestroyed(activity);
            android.app.FragmentManager fm = activity.getFragmentManager();
            return fragmentGet(activity, fm);
        }
    }
    
    public RequestManager get(Fragment fragment) {
        if (fragment.getActivity() == null) {
            throw new IllegalArgumentException("You cannot start a load on a fragment before it is attached");
        }
        if (Util.isOnBackgroundThread()) {
            return get(fragment.getActivity().getApplicationContext());
        } else {
            FragmentManager fm = fragment.getChildFragmentManager();
            return supportFragmentGet(fragment.getActivity(), fm);
        }
    }
```
上述方法中都是为了拿到一个RequestManager的对象，我们可以看到有三种方法可以拿到RequestManager的对象：

```
    private RequestManager getApplicationManager(Context context) {
        // Either an application context or we're on a background thread.
        if (applicationManager == null) {
            synchronized (this) {
                if (applicationManager == null) {
                    // Normally pause/resume is taken care of by the fragment we add to the fragment or activity.
                    // However, in this case since the manager attached to the application will not receive lifecycle
                    // events, we must force the manager to start resumed using ApplicationLifecycle.
                    applicationManager = new RequestManager(context.getApplicationContext(),
                            new ApplicationLifecycle(), new EmptyRequestManagerTreeNode());
                }
            }
        }

        return applicationManager;
    }

    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    RequestManager fragmentGet(Context context, android.app.FragmentManager fm) {
        RequestManagerFragment current = getRequestManagerFragment(fm);
        RequestManager requestManager = current.getRequestManager();
        if (requestManager == null) {
            requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
            current.setRequestManager(requestManager);
        }
        return requestManager;
    }
    
    RequestManager supportFragmentGet(Context context, FragmentManager fm) {
        SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm);
        RequestManager requestManager = current.getRequestManager();
        if (requestManager == null) {
            requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
            current.setRequestManager(requestManager);
        }
        return requestManager;
    }
```

>但是归根结底，只有两种（下面两种方法可以合并的）；一种是通过Application类型的Context去拿（**如果传的是Application或者当前线程是子线程**），另外一种是通过非Applicaiotn类型的Context去拿，然后再看RequestManager的构造方法：

```
    RequestManager(Context context, final Lifecycle lifecycle,RequestManagerTreeNode treeNode,
            RequestTracker requestTracker, ConnectivityMonitorFactory factory) {
        this.context = context.getApplicationContext();
        this.lifecycle = lifecycle;
        this.treeNode = treeNode;
        this.requestTracker = requestTracker;
        this.glide = Glide.get(context);
        this.optionsApplier = new OptionsApplier();

        ConnectivityMonitor connectivityMonitor = factory.build(context,
                new RequestManagerConnectivityListener(requestTracker));

        // If we're the application level request manager, we may be created on a background thread. In that case we
        // cannot risk synchronously pausing or resuming requests, so we hack around the issue by delaying adding
        // ourselves as a lifecycle listener by posting to the main thread. This should be entirely safe.
        if (Util.isOnBackgroundThread()) {
            new Handler(Looper.getMainLooper()).post(new Runnable() {
                @Override
                public void run() {
                    lifecycle.addListener(RequestManager.this);
                }
            });
        } else {
            lifecycle.addListener(this);
        }
        lifecycle.addListener(connectivityMonitor);
    }
```

>不知道大家有没有看到这个Lifecycle，我猜这个是和生命周期挂钩了，所以当我们使用的时候传的是Application的Context时，这个时候Glide的加载就和应用程序的生命周期同步，如果我们传的是非Application的Context时，这个时候Glide的加载和Fragment，Activity等的生命周期同步。那么在非Application的Context类型中是如何监听Activity的生命周期呢：

```
    //创建一个RequestManager对象
    RequestManager supportFragmentGet(Context context, FragmentManager fm) {
        SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm);
        RequestManager requestManager = current.getRequestManager();
        if (requestManager == null) {
            requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
            current.setRequestManager(requestManager);
        }
        return requestManager;
    }
    
    
    //RequestManager的构造方法
    RequestManager(Context context, final Lifecycle lifecycle,RequestManagerTreeNode treeNode,
            RequestTracker requestTracker, ConnectivityMonitorFactory factory) {
        this.context = context.getApplicationContext();
        this.lifecycle = lifecycle;
        this.treeNode = treeNode;
        this.requestTracker = requestTracker;
        this.glide = Glide.get(context);
        this.optionsApplier = new OptionsApplier();

        ConnectivityMonitor connectivityMonitor = factory.build(context,
                new RequestManagerConnectivityListener(requestTracker));

        // If we're the application level request manager, we may be created on a background thread. In that case we
        // cannot risk synchronously pausing or resuming requests, so we hack around the issue by delaying adding
        // ourselves as a lifecycle listener by posting to the main thread. This should be entirely safe.
        if (Util.isOnBackgroundThread()) {
            new Handler(Looper.getMainLooper()).post(new Runnable() {
                @Override
                public void run() {
                    lifecycle.addListener(RequestManager.this);
                }
            });
        } else {
            lifecycle.addListener(this);
        }
        lifecycle.addListener(connectivityMonitor);
    }    
```
>我们可以看到，在第一个方法中创建了一个fragment为SupportRequestManagerFragment，因为我们都知道Fragment的生命周期是和Activity的生命周期是同步的，所以这里将SupportRequestManagerFragment添加到当前页面，实例化一个RequestManager对象，将该RequestManager对象设置到fragment中，在RequestManager中传入Lifecycle对象，然后通过调用lifecycle.addListener(RequestManager.this)方法将requestmanager和Lifecycle进行绑定，因此fragment也和Lifecycle绑定了，所以我们可以通过fragment来获取Activity的生命周期。

##### 总结：Glide的with方法实际上就是为了创建一个RequestManager对象，这个对象用来管理和启动Glide，并且和应用程序或者Activity的生命周期相关联。

#### 2、Glide.load()方法
也就是RequestManager.load()方法，这个方法有很多重载方法：

```
//加载一个图片链接
public DrawableTypeRequest<String> load(String string) {
    return (DrawableTypeRequest<String>) fromString().load(string);
}

//加载一个图片Uri
public DrawableTypeRequest<Uri> load(Uri uri) {
    return (DrawableTypeRequest<Uri>) fromUri().load(uri);
}

//加载一个图片文件
public DrawableTypeRequest<File> load(File file) {
    return (DrawableTypeRequest<File>) fromFile().load(file);
}

//加载一个本地图片资源
public DrawableTypeRequest<Integer> load(Integer resourceId) {
    return (DrawableTypeRequest<Integer>) fromResource().load(resourceId);
}

//加载一个图片URL
public DrawableTypeRequest<URL> load(URL url) {
    return (DrawableTypeRequest<URL>) fromUrl().load(url);
}

//加载一个图片二进制流
public DrawableTypeRequest<byte[]> load(byte[] model) {
    return (DrawableTypeRequest<byte[]>) fromBytes().load(model);
}
```
上述每个方法都会调用对应的fromXXX()方法，而每个fromXXX()方法都会调用loadGeneric()这个方法，那我们重点来看一下这个方法：

```
    private <T> DrawableTypeRequest<T> loadGeneric(Class<T> modelClass) {
        ModelLoader<T, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
        ModelLoader<T, ParcelFileDescriptor> fileDescriptorModelLoader =
                Glide.buildFileDescriptorModelLoader(modelClass, context);
        if (modelClass != null && streamModelLoader == null && fileDescriptorModelLoader == null) {
            throw new IllegalArgumentException("Unknown type " + modelClass + ". You must provide a Model of a type for"
                    + " which there is a registered ModelLoader, if you are using a custom model, you must first call"
                    + " Glide#register with a ModelLoaderFactory for your custom model class");
        }

        return optionsApplier.apply(
                new DrawableTypeRequest<T>(modelClass, streamModelLoader, fileDescriptorModelLoader, context,
                        glide, requestTracker, lifecycle, optionsApplier));
    }
```
>这个方法主要用来创建一个DrawableTypeRequest对象，这个方法中创建了两个ModelLoader对象（是一个接口），这个接口的作用是将你传入的类型转换为可解码为资源的数据类型；DrawableTypeRequest这个对象的主要作用是加载请求，直接加载gif或者一个bitmap图片，这个类中这么两个方法：

```
public BitmapTypeRequest<ModelType> asBitmap() {
    return optionsApplier.apply(new BitmapTypeRequest<ModelType>(this, streamModelLoader, fileDescriptorModelLoader, optionsApplier));
}

public GifTypeRequest<ModelType> asGif() {
    return optionsApplier.apply(new GifTypeRequest<ModelType>(this, streamModelLoader, optionsApplier));
}
```
>这两个方法用来指定Glide加载的是静态图片（asBitmap）还是动态图片（asGif），如果没有指定，那么默认是加载静态图片。上面说的除了调用fromXXX方法之外还调用了DrawableTypeRequest的load方法，然后调用了父类DrawableRequestBuilder的load()方法，这个父类中有很多我们经常用到的方法：

```
public DrawableRequestBuilder<ModelType> thumbnail(DrawableRequestBuilder<?> thumbnailRequest) {
    super.thumbnail(thumbnailRequest);
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> priority(Priority priority) {
    super.priority(priority);
    return this;
}

public DrawableRequestBuilder<ModelType> transform(BitmapTransformation... transformations) {
    return bitmapTransform(transformations);
}

@SuppressWarnings("unchecked")
public DrawableRequestBuilder<ModelType> centerCrop() {
    return transform(glide.getDrawableCenterCrop());
}

@SuppressWarnings("unchecked")
public DrawableRequestBuilder<ModelType> fitCenter() {
    return transform(glide.getDrawableFitCenter());
}

public DrawableRequestBuilder<ModelType> bitmapTransform(Transformation<Bitmap>... bitmapTransformations) {
    GifBitmapWrapperTransformation[] transformations =
            new GifBitmapWrapperTransformation[bitmapTransformations.length];
    for (int i = 0; i < bitmapTransformations.length; i++) {
        transformations[i] = new GifBitmapWrapperTransformation(glide.getBitmapPool(), bitmapTransformations[i]);
    }
    return transform(transformations);
}

@Override
public DrawableRequestBuilder<ModelType> transform(Transformation<GifBitmapWrapper>... transformation) {
    super.transform(transformation);
    return this;
}

public final DrawableRequestBuilder<ModelType> crossFade() {
    super.animate(new DrawableCrossFadeFactory<GlideDrawable>());
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> dontAnimate() {
    super.dontAnimate();
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> animate(ViewPropertyAnimation.Animator animator) {
    super.animate(animator);
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> placeholder(int resourceId) {
    super.placeholder(resourceId);
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> error(int resourceId) {
    super.error(resourceId);
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> listener(
        RequestListener<? super ModelType, GlideDrawable> requestListener) {
    super.listener(requestListener);
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> diskCacheStrategy(DiskCacheStrategy strategy) {
    super.diskCacheStrategy(strategy);
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> skipMemoryCache(boolean skip) {
    super.skipMemoryCache(skip);
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> dontTransform() {
    super.dontTransform();
    return this;
}

@Override
public DrawableRequestBuilder<ModelType> load(ModelType model) {
    super.load(model);
    return this;
}

@Override
public Target<GlideDrawable> into(ImageView view) {
    return super.into(view);
}

@Override
void applyFitCenter() {
    fitCenter();
}

@Override
void applyCenterCrop() {
    centerCrop();
}
```

我列举了一些我们经常用到的Glide的Api方法，有兴趣的可以去深入学习一下全部的方法。

##### 总结：load方法就是将传入的图片参数类型转换为可用的资源类型，并且返回一个DrawableTypeRequest对象。

#### 3、Glide.into()
最终会调用到GenericRequestBuilder.into()方法：

```
public Target<TranscodeType> into(ImageView view) {
    Util.assertMainThread();
    if (view == null) {
        throw new IllegalArgumentException("You must pass in a non null View");
    }
    if (!isTransformationSet && view.getScaleType() != null) {
        switch (view.getScaleType()) {
            case CENTER_CROP:
                applyCenterCrop();
                break;
            case FIT_CENTER:
            case FIT_START:
            case FIT_END:
                applyFitCenter();
                break;
            //$CASES-OMITTED$
            default:
                // Do nothing.
        }
    }
    return into(glide.buildImageViewTarget(view, transcodeClass));
}
```
into方法中会调用glide.buildImageViewTarget()方法：

```
<R> Target<R> buildImageViewTarget(ImageView imageView, Class<R> transcodedClass) {
    return imageViewTargetFactory.buildTarget(imageView, transcodedClass);
}

@SuppressWarnings("unchecked")
public <Z> Target<Z> buildTarget(ImageView view, Class<Z> clazz) {
    if (GlideDrawable.class.isAssignableFrom(clazz)) {
        return (Target<Z>) new GlideDrawableImageViewTarget(view);
    } else if (Bitmap.class.equals(clazz)) {
        return (Target<Z>) new BitmapImageViewTarget(view);
    } else if (Drawable.class.isAssignableFrom(clazz)) {
        return (Target<Z>) new DrawableImageViewTarget(view);
    } else {
        throw new IllegalArgumentException("Unhandled class: " + clazz
                + ", try .as*(Class).transcode(ResourceTranscoder)");
    }
}
```
这里就是创建一个Target对象（GlideDrawableImageViewTarget，BitmapImageViewTarget，DrawableImageViewTarget）然后作为参数去调用in 国外一个into方法：

```
public <Y extends Target<TranscodeType>> Y into(Y target) {
    Util.assertMainThread();
    if (target == null) {
        throw new IllegalArgumentException("You must pass in a non null Target");
    }
    if (!isModelSet) {
        throw new IllegalArgumentException("You must first set a model (try #load())");
    }

    Request previous = target.getRequest();

    if (previous != null) {
        previous.clear();
        requestTracker.removeRequest(previous);
        previous.recycle();
    }

    Request request = buildRequest(target);
    target.setRequest(request);
    lifecycle.addListener(target);
    requestTracker.runRequest(request);

    return target;
}
```
这个方法中会创建一个Request对象，然后执行这个对象，我们来看看这个Request里面都有些什么：

```
Request类：

public interface Request {

    /**
     * Starts an asynchronous load.
     */
    void begin();

    /**
     * Identical to {@link #clear()} except that the request may later be restarted.
     */
    void pause();

    /**
     * Prevents any bitmaps being loaded from previous requests, releases any resources held by this request,
     * displays the current placeholder if one was provided, and marks the request as having been cancelled.
     */
    void clear();

    /**
     * Returns true if this request is paused and may be restarted.
     */
    boolean isPaused();

    /**
     * Returns true if this request is running and has not completed or failed.
     */
    boolean isRunning();

    /**
     * Returns true if the request has completed successfully.
     */
    boolean isComplete();

    /**
     * Returns true if a non-placeholder resource is set. For Requests that load more than one resource, isResourceSet
     * may return true even if {@link #isComplete()}} returns false.
     */
    boolean isResourceSet();

    /**
     * Returns true if the request has been cancelled.
     */
    boolean isCancelled();

    /**
     * Returns true if the request has failed.
     */
    boolean isFailed();

    /**
     * Recycles the request object and releases its resources.
     */
    void recycle();
}
```

这是一个接口，看到这些方法相信大家已经猜到这个接口是用来干什么的，没错，用来发起加载图片请求的，那这个Request接口对象又是如何创建的呢，接着往下看：

```
GenericRequestBuilder类：

private Request buildRequestRecursive(Target<TranscodeType> target, ThumbnailRequestCoordinator parentCoordinator) {
    if (thumbnailRequestBuilder != null) {
        if (isThumbnailBuilt) {
            throw new IllegalStateException("You cannot use a request as both the main request and a thumbnail, "
                    + "consider using clone() on the request(s) passed to thumbnail()");
        }
        // Recursive case: contains a potentially recursive thumbnail request builder.
        if (thumbnailRequestBuilder.animationFactory.equals(NoAnimation.getFactory())) {
            thumbnailRequestBuilder.animationFactory = animationFactory;
        }

        if (thumbnailRequestBuilder.priority == null) {
            thumbnailRequestBuilder.priority = getThumbnailPriority();
        }

        if (Util.isValidDimensions(overrideWidth, overrideHeight)
                && !Util.isValidDimensions(thumbnailRequestBuilder.overrideWidth,
                        thumbnailRequestBuilder.overrideHeight)) {
          thumbnailRequestBuilder.override(overrideWidth, overrideHeight);
        }

        ThumbnailRequestCoordinator coordinator = new ThumbnailRequestCoordinator(parentCoordinator);
        Request fullRequest = obtainRequest(target, sizeMultiplier, priority, coordinator);
        // Guard against infinite recursion.
        isThumbnailBuilt = true;
        // Recursively generate thumbnail requests.
        Request thumbRequest = thumbnailRequestBuilder.buildRequestRecursive(target, coordinator);
        isThumbnailBuilt = false;
        coordinator.setRequests(fullRequest, thumbRequest);
        return coordinator;
    } else if (thumbSizeMultiplier != null) {
        // Base case: thumbnail multiplier generates a thumbnail request, but cannot recurse.
        ThumbnailRequestCoordinator coordinator = new ThumbnailRequestCoordinator(parentCoordinator);
        Request fullRequest = obtainRequest(target, sizeMultiplier, priority, coordinator);
        Request thumbnailRequest = obtainRequest(target, thumbSizeMultiplier, getThumbnailPriority(), coordinator);
        coordinator.setRequests(fullRequest, thumbnailRequest);
        return coordinator;
    } else {
        // Base case: no thumbnail.
        return obtainRequest(target, sizeMultiplier, priority, parentCoordinator);
    }
}

private Request obtainRequest(Target<TranscodeType> target, float sizeMultiplier, Priority priority,
        RequestCoordinator requestCoordinator) {
    return GenericRequest.obtain(
            loadProvider,
            model,
            signature,
            context,
            priority,
            target,
            sizeMultiplier,
            placeholderDrawable,
            placeholderId,
            errorPlaceholder,
            errorId,
            fallbackDrawable,
            fallbackResource,
            requestListener,
            requestCoordinator,
            glide.getEngine(),
            transformation,
            transcodeClass,
            isCacheable,
            animationFactory,
            overrideWidth,
            overrideHeight,
            diskCacheStrategy);
}
```
>这个方法有很多我们都没必要去看的，我们只需要知道前面这些都是和缩略图有关就对了（根据参数命名大致能猜得出来），每个if-else中都会去调用obtainRequest()方法，所以我们重点看下面这个方法，这个方法很短，直接调用GenericRequest.obtain()方法并返回一个Request子类对象，这个方法所传的参数大多数都是我们调用Glide的时候所配置的，比如占位图，错误图，优先级等等，因此我们大致可以知道，这里是组装我们传的那些参数用来创建一个Request请求对象。

创建好Request之后，下一个操作就是调用runRequest方法执行该request：

```
RequestTracker类：

public void runRequest(Request request) {
    requests.add(request);
    if (!isPaused) {
        request.begin();
    } else {
        pendingRequests.add(request);
    }
}
```
如果Glide当前不是处于暂停状态，那么就会调用request的begin方法，前面有说道request是一个接口，那么我们只要找到实现它的地方就好了，在GenericRequest类中实现了这个begin方法：

```
GenericRequest类：

@Override
public void begin() {
    startTime = LogTime.getLogTime();
    if (model == null) {
        onException(null);
        return;
    }

    status = Status.WAITING_FOR_SIZE;
    if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
        onSizeReady(overrideWidth, overrideHeight);
    } else {
        target.getSize(this);
    }

    if (!isComplete() && !isFailed() && canNotifyStatusChanged()) {
        target.onLoadStarted(getPlaceholderDrawable());
    }
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logV("finished run method in " + LogTime.getElapsedMillis(startTime));
    }
}
```
>这个方法首先会判断你调load方法时传进来的参数是否为空，如果为空就会去调用onException方法，这个方法用来加载错误图片，这里就不进去看了，有兴趣的可以跟进去看一下，然后一系列判断之后会调用target.onLoadStarted()方法，这时图片加载开始了，调用那个方法时会将占位图传过去，在图片加载过程中会先显示占位图。然后来看看加载图片的地方，这里要分两种情况：第一，onSizeReady()，这个方法是用户设置了图片的宽高之后会调用的；第二，target.getSize(),用户没有设置宽高会调用。但是在第二种方法中最终也会调用onSizeReady()方法，这里就不走这个流程了，我们直接看onSizeReady()方法

```
GenericRequest类：

@Override
public void onSizeReady(int width, int height) {
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logV("Got onSizeReady in " + LogTime.getElapsedMillis(startTime));
    }
    if (status != Status.WAITING_FOR_SIZE) {
        return;
    }
    status = Status.RUNNING;

    width = Math.round(sizeMultiplier * width);
    height = Math.round(sizeMultiplier * height);

    ModelLoader<A, T> modelLoader = loadProvider.getModelLoader();
    final DataFetcher<T> dataFetcher = modelLoader.getResourceFetcher(model, width, height);

    if (dataFetcher == null) {
        onException(new Exception("Failed to load model: \'" + model + "\'"));
        return;
    }
    ResourceTranscoder<Z, R> transcoder = loadProvider.getTranscoder();
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logV("finished setup for calling load in " + LogTime.getElapsedMillis(startTime));
    }
    loadedFromMemoryCache = true;
    loadStatus = engine.load(signature, width, height, dataFetcher, loadProvider, transformation, transcoder,
            priority, isMemoryCacheable, diskCacheStrategy, this);
    loadedFromMemoryCache = resource != null;
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logV("finished onSizeReady in " + LogTime.getElapsedMillis(startTime));
    }
}
```
>这个方法的内容也不是很多，一眼看下来我们大致就能知道engin.load()方法是一个比较重要的方法，没错（整个方法里面就调了这一个方法，瞎子都能看出来，），那我们就进到engin.load()方法中看看吧：

```
Engine类：

public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
        DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
        Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
    Util.assertMainThread();
    long startTime = LogTime.getLogTime();

    final String id = fetcher.getId();
    EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
            loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
            transcoder, loadProvider.getSourceEncoder());

    EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
    if (cached != null) {
        cb.onResourceReady(cached);
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Loaded resource from cache", startTime, key);
        }
        return null;
    }

    EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
    if (active != null) {
        cb.onResourceReady(active);
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Loaded resource from active resources", startTime, key);
        }
        return null;
    }

    EngineJob current = jobs.get(key);
    if (current != null) {
        current.addCallback(cb);
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Added to existing load", startTime, key);
        }
        return new LoadStatus(cb, current);
    }

    EngineJob engineJob = engineJobFactory.build(key, isMemoryCacheable);
    DecodeJob<T, Z, R> decodeJob = new DecodeJob<T, Z, R>(key, width, height, fetcher, loadProvider, transformation,
            transcoder, diskCacheProvider, diskCacheStrategy, priority);
    EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
    jobs.put(key, engineJob);
    engineJob.addCallback(cb);
    engineJob.start(runnable);

    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Started new load", startTime, key);
    }
    return new LoadStatus(cb, engineJob);
}
```
>这里前面两个if是判断缓存中有没有（loadFromCache方法和loadFromActiveResources方法），如果有就从缓存中去拿，如果没有的话，那么先会从缓存中拿EngineJob对象，如果缓存中没有的话，那就会通过engineJobFactory工厂去生成一个EngineJob对象（这里用到了工厂设计模式），然后把生成的EngineJob对象添加到缓存中。最后再用生成的EngineJob对象和DecodeJob对象去生成一个EngineRunnable对象（创建了一个线程），然后给EngineJob对象添加了一个回调并且启动了线程，剩下的活都交给线程去做了，所以下面我们进到线程中看看

```
EngineRunnable类：

@Override
public void run() {
    if (isCancelled) {
        return;
    }

    Exception exception = null;
    Resource<?> resource = null;
    try {
        resource = decode();
    } catch (Exception e) {
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            Log.v(TAG, "Exception decoding", e);
        }
        exception = e;
    }

    if (isCancelled) {
        if (resource != null) {
            resource.recycle();
        }
        return;
    }

    if (resource == null) {
        onLoadFailed(exception);
    } else {
        onLoadComplete(resource);
    }
}
```
>线程的话我们直接看run方法就好了，这里有两个对象，Resource和Exception，能看出来一个是成功的时候生成的图片资源，另一个是失败时候生成的异常信息（不成功，便成仁），我们可以看到调用了decode()方法来生成Resource，那我们就来看看decode()方法

```
EngineRunnable类：

private Resource<?> decode() throws Exception {
    if (isDecodingFromCache()) {
        return decodeFromCache();
    } else {
        return decodeFromSource();
    }
}

private Resource<?> decodeFromSource() throws Exception {
    return decodeJob.decodeFromSource();
}
```
>最后会调用decodeJob的decodeFromSource()方法

```
DecodeJob类：

public Resource<Z> decodeFromSource() throws Exception {
    Resource<T> decoded = decodeSource();
    return transformEncodeAndTranscode(decoded);
}

private Resource<T> decodeSource() throws Exception {
    Resource<T> decoded = null;
    try {
        long startTime = LogTime.getLogTime();
        final A data = fetcher.loadData(priority);
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Fetched data", startTime);
        }
        if (isCancelled) {
            return null;
        }
        decoded = decodeFromSourceData(data);
    } finally {
        fetcher.cleanup();
    }
    return decoded;
}

private Resource<Z> transformEncodeAndTranscode(Resource<T> decoded) {
    long startTime = LogTime.getLogTime();
    Resource<T> transformed = transform(decoded);
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Transformed resource from source", startTime);
    }

    writeTransformedToCache(transformed);

    startTime = LogTime.getLogTime();
    Resource<Z> result = transcode(transformed);
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Transcoded transformed from source", startTime);
    }
    return result;
}

```
>首先通过decodeSource()方法来获取一个Resource对象，然后再通过调用transformEncodeAndTranscode()方法来处理这个Resource对象，我们可以看到decodeSource()方法中调用了一个fetcher.loadData(priority)方法，这个fetcher是我们在前面onSizeReady方法中一步一步传进来的（可以试着往回走走看），然后最终

#### 未完待续。。。。