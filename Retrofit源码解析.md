## Retrofit源码解析
###1、定义
Retrofit是基于OkHttp的一个Resultful设计风格的Http网络请求框架。确切的来说，Retrofit只负责网络请求接口的封装（包括请求参数，头部等信息），本质上还是由OkHttp来处理网络请求；在服务端但会数据之后，OkHttp将原始的结果交给Retrofit，Retrofit根据用户的需求对结果进行解析。

### 2、过程
(1)、通过解析网络请求接口的注解配置网络请求参数

(2)、通过动态代理生成网络请求对象

(3)、通过网络请求适配器（CallAdapter）将网络请求对象进行平台适配

(4)、通过网络请求执行器（Call）发送网络请求

(5)、通过数据转换器（Converter）解析服务器返回的数据

(6)、通过回调执行器（CallBackExecutor）切换线程

(7)、用户在主线程处理数据

### 3、设计模式

(1)、Builder模式：

```
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://xxx.xxx.xx")
                .client(new OkHttpClient())
                .addConverterFactory(GsonConverterFactory.create())
                .build();
```

(2)、动态代理设计模式

```
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
}
```

(3)、工厂设计模式

简单工厂，抽象工厂，方法工厂

Converter.Factory——————>抽象工厂

GsonConverterFactory.create()——————>方法工厂

### 4、Retrofit源码解析

**(1)、Retrofit的实例是使用建造者模式通过Builder类进行创建的。建造者模式：将一个复杂对象的构建与表示分离，使得用户在不知道对象的创建细节情况下就可以直接创建复杂的对象。**

```
Retrofit(okhttp3.Call.Factory callFactory, HttpUrl baseUrl,
      List<Converter.Factory> converterFactories, List<CallAdapter.Factory> adapterFactories,
      @Nullable Executor callbackExecutor, boolean validateEagerly) {
    this.callFactory = callFactory;
    this.baseUrl = baseUrl;
    this.converterFactories = unmodifiableList(converterFactories); // Defensive copy at call site.
    this.adapterFactories = unmodifiableList(adapterFactories); // Defensive copy at call site.
    this.callbackExecutor = callbackExecutor;
    this.validateEagerly = validateEagerly;
}
```
首先创建一个Retrofit对象我们需要配置好以下变量：

* `baseUrl`：网络请求的url地址
* `callFactory`：网络请求工厂
* `adapterFactories`：网络请求适配器工厂的集合
* `converterFactories`：数据转换器工厂的集合
* `callbackExecutor`：回调方法执行器
* `validateEagerly`：是否提前对业务接口中的注解进行验证转换的标志位

上面的factories工厂就是设计模式中的工厂设计模式：将类实例化的操作与使用对象的操作分开，使得使用者不用知道具体参数就可以实例化所需要的产品类。

>这里详细介绍一下CallAdapterFactory：这个factory负责生产CallAdapter，CallAdapter是网络请求执行器Call的适配器，它的作用是将默认的网络请求执行器OkHttpCall（实现了Call接口）转换成适合被不同平台来调用的网络请求执行器形式。

**(2)、Builder**

```
public static final class Builder {
    private final Platform platform;
    private @Nullable okhttp3.Call.Factory callFactory;
    private HttpUrl baseUrl;
    private final List<Converter.Factory> converterFactories = new ArrayList<>();
    private final List<CallAdapter.Factory> adapterFactories = new ArrayList<>();
    private @Nullable Executor callbackExecutor;
    private boolean validateEagerly;

    Builder(Platform platform) {
      this.platform = platform;
      // Add the built-in converter factory first. This prevents overriding its behavior but also
      // ensures correct behavior when using converters that consume all types.
      converterFactories.add(new BuiltInConverters());
    }

    public Builder() {
      this(Platform.get());
    }

    Builder(Retrofit retrofit) {
      platform = Platform.get();
      callFactory = retrofit.callFactory;
      baseUrl = retrofit.baseUrl;
      converterFactories.addAll(retrofit.converterFactories);
      adapterFactories.addAll(retrofit.adapterFactories);
      // Remove the default, platform-aware call adapter added by build().
      adapterFactories.remove(adapterFactories.size() - 1);
      callbackExecutor = retrofit.callbackExecutor;
      validateEagerly = retrofit.validateEagerly;
    }
}
```
从Builder里面的变量我们可以看出，Retrofit中的成员变量都是由Builder进行设置的。
>首先，通过Platform.get()拿到平台信息(是Android还是Java)，如果是Android平台的话，会创建默认的网络请求适配器工厂，该默认工厂生产的 adapter 会使得Call在异步调用时在指定的 Executor 上执行回调；然后还返回一个默认的回调方法执行器，用来切换线程（子->>主线程），并在主线程（UI线程）中执行回调方法。

**总结：**Builder类给Retrofit设置了一些参数(调用build()方法时)，然后设置了默认的

* 平台类型对象：Android
* 网络请求适配器工厂：CallAdapterFactory
* 数据转换器工厂：converterFactory
* 回调执行器：callbackExecutor

**(3)、baseUrl("http://xxx.xxx.xxx")**

```
public Builder baseUrl(String baseUrl) {
      checkNotNull(baseUrl, "baseUrl == null");
      HttpUrl httpUrl = HttpUrl.parse(baseUrl);
      if (httpUrl == null) {
        throw new IllegalArgumentException("Illegal URL: " + baseUrl);
      }
      return baseUrl(httpUrl);
}

public Builder baseUrl(HttpUrl baseUrl) {
      checkNotNull(baseUrl, "baseUrl == null");
      List<String> pathSegments = baseUrl.pathSegments();
      if (!"".equals(pathSegments.get(pathSegments.size() - 1))) {
        throw new IllegalArgumentException("baseUrl must end in /: " + baseUrl);
      }
      this.baseUrl = baseUrl;
      return this;
}
```
>第一个方法是把String类型的Url转换成OkHttp用的HttpUrl，然后第二个方法是把Url参数分割成一个List集合，然后检测list最后一个是不是以“/”结尾。

**(4)、addConverterFactory(GsonConverterFactory.create())**

```
public Builder addConverterFactory(Converter.Factory factory) {
      converterFactories.add(checkNotNull(factory, "factory == null"));
      return this;
}
```
>将创建好的Converter.Factory添加到converterFactories集合中。默认retrofit使用Gson解析，若使用其他解析方式，可以自定义解析器，但是必须继承Converter.Factory

**(5)、build()过程**

```
public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }

      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
      adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
          callbackExecutor, validateEagerly);
}
```
> 在build()方法中分别
>
> * 配置了网络请求执行器（callFactory）
> * 配置了回调方法执行器（callbackExecutor）
> * 配置了网络请求适配器工厂（callAdapterFactory）
> * 配置了数据转换器工厂（converterFactory）

最终会返回一个Retrofit对象，并传入上述配置好的变量。

**总结：Retrofit使用建造者模式通过Builder类创建了一个Retrofit实例，具体创建细节是配置了：**

* 平台类型对象（Platform - Android）
* 网络请求url地址（baseUrl）
* 网络请求工厂（callFactory，默认使用OkHttpCall）
* 网络请求适配器工厂集合（adapterFactories，本质是配置了网络请求适配器工厂- 默认是ExecutorCallAdapterFactory）
* 数据转换器工厂的集合（converterFactories，本质是配置了数据转换器工厂）
* 回调方法执行器（callbackExecutor，默认回调方法执行器作用是切换线程(子线程 - 主线程)）

**由于使用了Builder模式，所以开发者不需要关心配置细节就可以创建Retrofit实例了。在创建Retrofit对象时，你也可以通过更多灵活的方式去处理你的需求，如使用不同的Converter、使用不同的CallAdapter，这也就提供了你使用RxJava来调用Retrofit的可能。**

**(6)、创建网络请求接口实例**

```
FRApi api = retrofit.create(FRApi.class);


public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
}
```
>首先通过`Utils.validateServiceInterface(service)`来判断入参是否是一个接口。然后根据判断`validateEagerly`是否需要提前验证，`eagerlyValidateMethods(service)`方法的作用是给接口中每个方法的注解进行解析并得到一个ServiceMethod对象，以Method为键将该对象存入LinkedHashMap集合中。最后返回一个通过动态代理创建的网络请求接口实例，该动态代理是为了拿到网络请求接口实例上的所有注解。

**动态代理设计模式：**是Java提供的一套动态代理机制，将代理类的实现交给`InvocationHandler`类作为具体的实现，每次调用网络请求接口中的方法时，都要在`InvocationHandler`类的invoke()方法中被拦截一次，然后让代理类去完成相应的操作。


#### 未完待续。。。。