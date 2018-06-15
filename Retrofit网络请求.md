# Retrofit

- A type-safe HTTP client for Android and Java

## 概述

- App应用程序通过Retrofit请求网络，实际上是使用Retrofit接口层封装请求参数，之后由OkHttp完成后续的请求操作
- 在服务端返回数据之后，OkHttp将原始的结果交给Retrofit，Retrofit根据用户的需求对结果进行解析
- Retrofit是将每个Http请求抽象成==Java接口==，用==注解==来描述网络请求的参数，用==注解==来配置网络请求的参数，内部实现原理是通过==动态代理==将接口的注解翻译成一个一个==Http请求==，最后通过==线程池==来实现Http请求
- Retrofit使用了大量的设计模式，将我们相关的网络请求模块完全的解耦，使网络请求更加简单流畅清晰。

### 例子

```
public interface GitHubService{
    @GET("users/{user}/repos")
    Call<List<Repo>> listRepos(@Path("user") String user);
}
```


```
public void netRequestWithRetrofit(){
    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("https://api.github.com/")//设置网络请求的url地址
            .addConverterFactory(GsonConverterFactory.create())//设置数据解析器
            .addCallAdapterFactory(RxJavaCallAdapterFactory.create())//支持RxJava平台
            .build();
    GitHubService service = retrofit.create(GitHubService.class);
    Call repos = (Call) service.getCall();
    repos.enqueue(new retrofit2.Callback()){
        public void onFailure(){
        }
        public void onResponse(){
        }
    }
}
```

### 实现Retrofit的基本步骤

- 1、添加Retrofit库的依赖，添加网络权限
- 2、创建 接收服务器返回数据 的Bean类
- 3、创建 用于描述网络请求 的接口
- 4、创建Retrofit的实例，使用的是Builder的构建者模式
- 5、创建 网络请求接口实例
- 6、发送网络请求（异步/同步）
- 7、处理服务器返回的数据

## Retrofit中涉及了代理的内容

### 代理

- 代理模式：为其他对象提供一种代理，用以控制对这个对象的访问
- 静态代理和动态代理

### 动态代理

- 动态代理：代理类在程序运行时创建的代理方式

#### 实现动态代理的方式

- 1、jdk动态代理
- 2、CGLIB

#### 总结

##### 1、运行期

- 动态代理是通过代理类与相关接口不直接发生联系的情况下，而在运行时实现我们的动态代理

##### 2、InvocationHandler接口和Proxy类

##### 3、动态代理与静态代理最大的不同

- 动态代理的代理类不需要手动生成，而该代理类是在运行期间动态生成的，不是Hashcode而是根据配置在运行期动态生成的
- 动态生成的代理类已经实现了代理对象的相关接口，只要提供给它一个InvocationHandler接口并实现其invoke()方法，其他的jdk就可以给我们实现相关操作

### Retrofit网络通信八步

- 1、创建Retrofit实例
- 2、定义一个网络请求接口并为接口中的方法添加注解
- 3、通过 动态代理 生成网络请求对象
- 4、通过 网络请求适配器 将 网络请求对象 进行平台适配
- 5、通过 网络请求执行器 发送网络请求
- 6、通过 数据转换器 解析数据
- 7、通过 回调执行器 切换线程
- 8、用户在主线程处理返回结果

### Retrofit七个成员变量

- Map<Method,ServiceMethod> serviceMethodCache:缓存网络请求相关的信息对象
- Factory callFactory:请求网络的OkHttp工厂，生成OkHttpClient
- HttpUrl baseUrl：网络请求的基地址
- List<retrofit2.Converter.Factory> converterFactories：数据转换器的工厂集合，生产数据转换器的工厂
- List<retrofit2.CallAdapter.Factory> adapterFactories：网络请求适配器的工厂集合，生产CallAdapter
- Executor callbackExecutor：执行回调请求
- boolean validateEagerly：标志位，表示是否需要立即解释接口中的方法

### Builder构建者模式&Builder内部类

#### Builder是一个Retrofit的静态内部类

#### Builder的成员变量

- Platform platform：表示Retrofit适配的平台
- Factory callFacto：请求网络的OkHttp工厂，生成OkHttpClient
- HttpUrl baseUrl：网络请求的地址
- List<retrofit2.Converter.Factory> converterFactories:数据转换器的工厂集合，数据转换器的作用就是将网络请求下来的response转换成所能用的java对象，默认情况下使用的是JsonConverterFactory
- List<retrofit2.CallAdapter.Factory> adapterFactories：网络请求适配器的工厂集合，把Call对象转换成我们能用的其他对象的请求
- Executor callbackExecutor：执行异步回调的
- boolean validateEagerly：标志位，表示是否需要立即解释接口中的方法，用在动态代理

#### Builder的无参构造函数

```
public Builder(){
    this(Platform.get());
}
```

##### 上面的Platform是一个单例的类

```

private static final Platform PLATFORM = findPlatform();
    static Platform get(){
        return PLATFORM;
}

//这个方法是判断支持哪一个平台
private static Platform findPlatform()
{
    try{
        Class.forName("android.os.Build");//利用反射
        if(VERSION.SDK_INT != 0){
            return new Platform.Android();
        }
    }
    ...
}

static class Android extends Platform{
    Android(){
    }
    
    //切换线程，从子线程切换到主线程，同时可以在主线程中实行回调方法
    public Executor defaultCallbackExecutor(){
        return new Platform.Android.MainThreadThreadExecutor();
    }
    
    //该Executor和主线程所绑定
    static class MainThreadExecutor implements Executor{
        private final Handler handler = new Handler(Looper.getMainLooper());
        
        MainThreadExecutor(){
        }    
        
        public void execute(Runnable r){
            this.handler.post(r);
        }
    
    //创建默认的网络请求适配器工厂
    Factory defaultCallAdapterFactory(Executor callbackExecutor){
        return new ExecutorCallAdapterFactory(callbackExecutor);
    }
}
```

#### Builder的有参构造函数

```
    Builder(Platform platform){
        this.converterFactories = new ArrayList();
        this.adapterFactories = new ArrayList();
        this.platform = platform;
        this.converterFactories.add(new BuiltInConverters());//非常重要的方法，添加convertFactory内置的数据转换器工厂，如果初始化converterFactory的时候没有添加，则会使用默认的数据转换器工厂BuiltInConverters()
    }
```

#### 总结

- 1、主要是配置平台类型对象Platform
- 2、主要是配置网络适配器的工厂和数据转换器的工厂

### baseurl / converter / calladapter

#### baseurl

- baseurl：将传入的String类型的url转换成OkHttp适合使用的url类型

#### converter

- addConverterFactory:将数据适配器加入到数据适配器工厂集合中

#### calladapter

- addCallAdapterFactory：将网络适配器加入到网络适配器工厂集合中


### build方法完成Retrofit对象创建流程

- build方法就是对Retrofit进行参数配置，然后创建Retrofit对象


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

## RxJavaCallAdapterFactory内部构造与工作原理

- RxJavaCallAdapterFactory 是继承 Factory的
- Factory是CallAdapter的一个静态内部类

```
public interface CallAdapter<T>{
    Type respnseType();//作用是返回Http返回解析后的类型，该类型是接口中定义的泛型实参例如下面的List<MyResponse>
    /*@GET(".../...")
    Call<List<MyResponse>> getCall();*/
    
    <R> T adapt(Call<R> var1);
    
    public abstract static class Factory{
        public Factory(){
        }
        
        //根据接口的返回类型，注解类型得到我们实际需要的adapter
        public abstract CallAdapter<?> get(Type var1,Annotation[] var2,Retrofit var3);
        
        protected static Type getParameterUpperBound(int index,ParameterizedType type)
        {
        return Utils.getParameterUpperBound(index,type);
        }
        
        //获取原始类型CallAdapter
        protected static Class<?> getRawType(Type type)
        {
        return Utils.getRawType(type);
        }
    }
}
```

### CallAdapter

- callAdapter：是将Retrofit中的泛型对象call<T>转换成Java对象，然后通过Java对象进行其他操作
- Call<?>：Retrofit中的Call<T>对象跟OkHttp中的Call<T>对象不一样，Retrofit中的是对OkHttp中的进行了封装
- Call<T>对象转换成java对象过程：创建Retrofit的Call<T>对象，通过Call对象发送Http请求，通过converter数据转换器，将请求回来的数据response转换成所需要的java对象

### RxJavaCallAdapterFactory工作过程

- 1、实现Factory：用来提供具体的适配逻辑
- 2、注册CallAdapter：将CallAdapter通过add方法添加到Retrofit当中
- 3、Factory.get()：获取CallAdapter
- 4、adapter方法：调用CallAdapter当中的adapter方法，通过这个方法将Call对象转换成每个平台所适用的类型

## 网络请求接口实例

- 首先需要创建一个接口类
```
public interface MyInterface{

    /*
    1、创建接口中接收网络请求数据的方法getCall()
    2、给方法添加返回值Call<>
    3、List<MyResponse>是我们需要解析数据的类型
    4、然后给方法添加注解 @GET表示请求网络数据的方法是GET
    5、@GET()中是相对地址
    */

    @GET(".../...")
    Call<List<MyResponse>> getCall();
}
```

- 调用Retrofit的create()方法，传入接口的Class对象

### create()方法

- 该方法是通过动态代理来解析接口中的方法参数注解
- 然后调用InvocationHandler()类中的invoke方法来做实际的操作

```
public <T> T create(final Class<T> service) {
        //对传进来的接口的字节码进行验证，方法里面实现的是抛出异常的操作
        Utils.validateServiceInterface(service);
        if (validateEagerly) {
            eagerlyValidateMethods(service);
        }
        
        //返回创建一个网络接口的动态代理对象，通过动态代理来创建网络接口实例
        //每当执行动态代理实例的方法时，就会调用InvocationHandler()中的invoke()方法
        return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[]{service},
                new InvocationHandler() {
                    private final Platform platform = Platform.get();

                    //该方法是真正解析接口注解操作的地方
                    @Override
                    public Object invoke(Object proxy, Method method, Object... args)
                            throws Throwable {
                        // If the method is a method from Object then defer to normal invocation.
                        if (method.getDeclaringClass() == Object.class) {
                            return method.invoke(this, args);
                        }
                        if (platform.isDefaultMethod(method)) {
                            return platform.invokeDefaultMethod(method, service, proxy, args);
                        }
                        //Retrofit最核心的三行代码
                        ServiceMethod serviceMethod = loadServiceMethod(method);
                        OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
                        return serviceMethod.callAdapter.adapt(okHttpCall);
                    }
                });
}

```

### ServiceMethod对象解析

```
//Retrofit最核心的三行代码
    ServiceMethod serviceMethod = loadServiceMethod(method);
    OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
    return serviceMethod.callAdapter.adapt(okHttpCall);

```

#### ServiceMethod

- 该类对应接口中的网络请求方法，例如上述接口中的getCall()方法
- 该类中有众多的网络请求配置的参数
- 因为Retrofit是使用OkHttp进行网络请求的
- 所以ServiceMethod最终还是交给OkHttp进行网络请求

##### ServiceMethod源码

```
final class ServiceMethod<R, T> {
  // Upper and lower characters, digits, underscores, and hyphens, starting with a character.
  static final String PARAM = "[a-zA-Z][a-zA-Z0-9_-]*";
  static final Pattern PARAM_URL_REGEX = Pattern.compile("\\{(" + PARAM + ")\\}");
  static final Pattern PARAM_NAME_REGEX = Pattern.compile(PARAM);

  final okhttp3.Call.Factory callFactory;
  final CallAdapter<R, T> callAdapter;

  private final HttpUrl baseUrl;
  //数据转换器，response的转换器，作用是把服务器返回的数据转换成所需要的JavaBean对象
  private final Converter<ResponseBody, R> responseConverter;
  private final String httpMethod;
  //网络请求的相对地址 网络请求的绝对地址是baseUrl+relativeUrl
  private final String relativeUrl;
  private final Headers headers;
  //网络请求的Http报文的body类型
  private final MediaType contentType;
  private final boolean hasBody;
  private final boolean isFormEncoded;
  private final boolean isMultipart;
  //接口方法参数的处理器，解析数据
  private final ParameterHandler<?>[] parameterHandlers;

}
```

##### ServiceMethod的内部类Builder

```
Builder(Retrofit retrofit, Method method) {
      this.retrofit = retrofit;
      this.method = method;
      //获取网络接口方法中的注解
      this.methodAnnotations = method.getAnnotations();
      //获取网络接口方法中的参数类型
      this.parameterTypes = method.getGenericParameterTypes();
      //获取网络接口方法中的注解中完整内容
      this.parameterAnnotationsArray = method.getParameterAnnotations();
    }
```

##### SerivceMethod的build()方法

```
public ServiceMethod build() {
      //通过调用createCallAdapter()方法获取callAdapter
      callAdapter = createCallAdapter();
      //通过调用callAdapter的responseType方法获取响应体类型
      responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      //通过createResponseConverter()方法获取响应转化器
      responseConverter = createResponseConverter();
      //遍历方法的注解
      for (Annotation annotation : methodAnnotations) {
        //解析方法注解
        parseMethodAnnotation(annotation);
      }
      //如果没有HTTP请求的方法，则抛异常
      if (httpMethod == null) {
        throw methodError("HTTP method annotation is required (e.g., @GET, @POST, etc.).");
      }
      //在上面遍历方法注解并解析方法注解的时候，hasBody，isMultipart，isFormEncoded等已经完成赋值
      //如果没有请求体
      if (!hasBody) {
         //但是 还是使用了@Multipart注解，使用了@Multipart 是一定有消息体的
        if (isMultipart) {
          throw methodError(
              "Multipart can only be specified on HTTP methods with request body (e.g., @POST).");
        }
         //但是 还是使用了 @FormEncoded，使用了@FormEncoded 是一定有消息体的
        if (isFormEncoded) {
          throw methodError("FormUrlEncoded can only be specified on HTTP methods with "
              + "request body (e.g., @POST).");
        }
      }
      //如果方法 入参的注解，它的长度也就是有多少个入参，遍历每个入参对应的注解
      int parameterCount = parameterAnnotationsArray.length;
      //创建对应的数量的参数处理类数组，这时候开始处理入参对应的注解了
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      //遍历入参的数量
      for (int p = 0; p < parameterCount; p++) {
         //首先获取入参的类型
        Type parameterType = parameterTypes[p];
         //如果是不能处理的类型，则抛异常
        if (Utils.hasUnresolvableType(parameterType)) {
          throw parameterError(p, "Parameter type must not include a type variable or wildcard: %s",
              parameterType);
        }
        //如果是可以处理的类型获取对应位置入参的注解
        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
        //如果对应入参的注解为null，则抛异常
        if (parameterAnnotations == null) {
          throw parameterError(p, "No Retrofit annotation found.");
        }
         //如果对应的入参注解不为null,则调用parseParameter方法获取ParameterHandler，cancel在这里方法里面创建ParameterHandler里面的静态内部类，后面咱们再仔细看下。
        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }
       //for循环遍历以后，如果relativeUrl还为null，同时没有使用！@url注解，这样的话我们就无法获取url地址了，所以要抛异常。
      if (relativeUrl == null && !gotUrl) {
        throw methodError("Missing either @%s URL or @Url parameter.", httpMethod);
      }
      //如果没有使用@FormEncoded注解、@Multipart和，同时HTTP请求方式也不需要响应提，但是却使用了@Body 注解，这不是自相矛盾，所以抛异常。
      if (!isFormEncoded && !isMultipart && !hasBody && gotBody) {
        throw methodError("Non-body HTTP method cannot contain @Body.");
      }
      //如果使用@FormEncoded注解，却没使用@Field注解，不符合规范，则抛异常
      if (isFormEncoded && !gotField) {
        throw methodError("Form-encoded method must contain at least one @Field.");
      }
      //如果使用 @Multipart 注解，却没有使用@Part 注解，抛异常
      if (isMultipart && !gotPart) {
        throw methodError("Multipart method must contain at least one @Part.");
      }
      //走到这里说明没有问题，则new一个ServiceMethod对象，入参是Builder自己，然后return这个ServiceMethod对象。
      return new ServiceMethod<>(this);
    }


```
- 1、通过createCallAdapter()方法取得callAdapter对象
- 2、通过callAdapter的responseType()方法获取响应对应的类型
- 3、通过createResponseConverter()来获取响应转化/解析器
- 4、遍历方法上的注解，并解析注解
- 5、判断是GET还是POST或者其他HTTP请求方式
- 6、异常逻辑判断，如果HTTP请求方式不需要消息体，但是却使用@FormEncoded注解、@Multipart，自相矛盾。
- 7、获取这个方法入参的个数
- 8、遍历这个方法的各个入参，首先判断是不是能处理的类型，如果是能处理的类型；然后获取这个入参对应的注解；最后调用parseParameter()来获取对应的parameterHandlers。
- 9、异常逻辑判断，比如如果没有相对路径还没有使用@Url，我们会无法得到具体的地址。
- 10、异常逻辑处理，既没有使用@FormUrlEncode 注解也没有使用@Multipart注解，且HTTP请求方式也需要使用请求，却使用@Body 注解，违反规范，抛异常
- 11、异常逻辑处理，使用@FormUrlEncoded注解，但是却没有使用@Field注解，这是违背Retrofit的规定的。
- 12、异常逻辑处理，使用了@Multipart 注解，却没有使用@Part注解，同样是违背Retrofit的规定的。
- 13、最后new了一个ServiceMethod对象，并return




##### loadServiceMethod


```
ServiceMethod loadServiceMethod(Method method){
    //缓存的是serviceMethod对象
    Map var3 = this.serviceMethodCache;
    //确保了serviceMethodCache这个对象线程安全同步
    synchronized(this.serviceMethodCache){
    //每次读取ServiceMethod时，都首先尝试从ServiceMethodCache中获取
        ServiceMethod result = (ServiceMethod)this.serviceMethodCache.get(method);
        if(result==null){
            result = (new retrofit2.ServiceMethod.Builder(this,method)).build();
            this.serviceMethodCache.put(method,result);
        
        }
        return result;
    }
}

```


### Retrofit请求

- 同步：OkHttpCall.execute()
- 异步：OkHttpCall.enqueue()


#### Retrofit同步请求

- 1、ParameterHandler：首先对网络请求接口中的方法和每个方法的参数利用ParameterHandler进行解析
- 2、ServiceMethod：根据ServiceMethod创建OkHttp的request对象，这里实现了缓存，调用的是ServiceMethodCache
- 3、OkHttp发送网络请求：通过ServiceMethod得到request对象，然后通过OkHttpCall的底层OkHttp库发送网络请求
- 4、converter：通过数据转换器converter对服务器返回的数据进行数据解析，默认是GSON
```
//核心方法
  @Override public Response<T> execute() throws IOException {
    okhttp3.Call call;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      if (creationFailure != null) {
        if (creationFailure instanceof IOException) {
          throw (IOException) creationFailure;
        } else {
          throw (RuntimeException) creationFailure;
        }
      }

      call = rawCall;
      if (call == null) {
        try {
           //将任务抛给OKHttp
          call = rawCall = createRawCall();
        } catch (IOException | RuntimeException e) {
          creationFailure = e;
          throw e;
        }
      }
    }

    if (canceled) {
      call.cancel();
    }
   
    //转换结果
    return parseResponse(call.execute());
  }

 //生成okhttp3.Call
  private okhttp3.Call createRawCall() throws IOException {
    //这个callFactory就是OKHttpClient
    okhttp3.Call call = callFactory.newCall(requestFactory.create(args));
    if (call == null) {
      throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
  }

 //将okhttp3.Response转换成Response
  Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
    ResponseBody rawBody = rawResponse.body();

    // Remove the body's source (the only stateful object) so we can pass the response along.
    rawResponse = rawResponse.newBuilder()
        .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
        .build();

    int code = rawResponse.code();
    if (code < 200 || code >= 300) {
      try {
        // Buffer the entire body to avoid future I/O.
        ResponseBody bufferedBody = Utils.buffer(rawBody);
        return Response.error(bufferedBody, rawResponse);
      } finally {
        rawBody.close();
      }
    }

}
```

- 我们调用了OKHttpCall.execute（），该方法生成一个okhttp3.Call将任务抛给OKHttp，完了调用parseResponse，
- 用Converter将okhttp3.Response转换成我们在范型中指定的类型Response<ResponseBody> response = call.execute()，我指定了okhttp3.ResonseBody。
- 然后返回结果。如果我在构造Retrofit时提供了GsonConverter，addConverterFactory(GsonConverterFactory.create())那么上面的T body = responseConverter.convert(catchingBody);
- responseConverter就是GsonConverter。



#### Retrofit异步请求


```
@Override public void enqueue(final Callback<T> callback) {
      if (callback == null) throw new NullPointerException("callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(final Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(call, new IOException("Canceled"));
              } else {
                callback.onResponse(call, response);
              }
            }
          });
        }

        @Override public void onFailure(final Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(call, t);
            }
          });
        }
      });
    }
```

## Retrofit框架设计模式

### 构建者模式

### 工厂模式

### 外观模式

### 策略模式

### 适配器模式

### 观察者模式

## 动态代理