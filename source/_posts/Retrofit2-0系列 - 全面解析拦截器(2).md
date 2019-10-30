---
title: Retrofit2.0系列 - 全面解析拦截器(2)
copyright: true
date: 2019-10-30 16:22:11
tags:
	拦截器
categories:
	Retrofit2.0系列
---

- **Retrofit 2.0系列 - 玩转注解(1)**
- **Retrofit 2.0系列 - 全面解析拦截器(2)**
- **Retrofit2.0系列 - 简单封装请求(3)**

---
### 1. 拦截器(Interceptors)定义
- **拦截器是一种可以用来转换、重试、重写请求的机制**
- **拦截器被定义为一个接口，其中就包含一个`intercept(Chain chain)`方法和一个接口`interface Chain`，使用时重点在于`request`和`proceed(Request request)`两个方法**
	```java
	public interface Interceptor {
	  Response intercept(Chain chain) throws IOException;
	
	  interface Chain {
	    Request request();
	
	    Response proceed(Request request) throws IOException;
	
	    /**
	     * Returns the connection the request will be executed on. This is only available in the chains
	     * of network interceptors; for application interceptors this is always null.
	     */
	    @Nullable Connection connection();
	  }
	}
	```

---

<!-- more -->

### 2. 拦截器分类
**`应用拦截器(Application Interceptors)、网络拦截器(NetWork Interceptors)`**

#### 2.1 应用拦截器(Application Interceptors)
- **主要用途：主要用于查看请求信息和返回信息** 
- **添加方式：** 
	```java
	//1.在OkHttpClient.Builder中添加
	new OkHttpClient.Builder()
	    .addInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY));
	
	//2.在OkHttpClient中添加
	client.interceptors().add(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY));
	```

#### 2.2 网络拦截器(NetWork Interceptors)
- **主要用途：不仅用于查看请求信息和返回信息，还可以添加、删除、替换请求头信息、改变请求携带的实体** 
- **添加方式：** 
	```java
	//1.在OkHttpClient.Builder中添加
	new OkHttpClient.Builder()
	    .addNetworkInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY));
	
	//2.在OkHttpClient中添加
	client.networkInterceptors().add(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY));
	```

#### 2.3 区别与选择
 **从输出的日志来看，网络拦截器会包含信息更多，如`Accept-Encoding: gzip`、`Host`等**
- **应用拦截器**

        1. 不必要担心响应和重定向之间的中间响应。
        2. 通常只调用一次，即使HTTP响应是通过缓存提供的。
        3.  遵从应用层的最初目的。与OkHttp的注入头部无关，如If-None-Match。
        4. 允许短路而且不调用Chain.proceed()。
        5. 允许重试和多次调用Chain.proceed()。

- **网络拦截器**

        1. 允许像重定向和重试一样操作中间响应。
        2. 网络发生短路时不调用缓存响应。
        3. 在数据被传递到网络时观察数据。
        4. 有权获得装载请求的连接。
- **[Github关于拦截器的说明，并且包含拦截器示例、重写请求、重写响应示例](https://github.com/square/okhttp/wiki/Interceptors)**

---
### 3. 不同类型的拦截器
#### 3.1 日志拦截器
**HttpLoggingInterceptor中提供了四种类型的log日志以供使用，我们也可自己定义自己的log日志拦截器**

```java 
public enum Level {
	/** No logs. */
    NONE,
    /**
     * Logs request and response lines.
     *
     * <p>Example:
     * <pre>{@code
     * --> POST /greeting http/1.1 (3-byte body)
     *
     * <-- 200 OK (22ms, 6-byte body)
     * }</pre>
     */
    BASIC,
    /**
     * Logs request and response lines and their respective headers.
     *
     * <p>Example:
     * <pre>{@code
     * --> POST /greeting http/1.1
     * Host: example.com
     * Content-Type: plain/text
     * Content-Length: 3
     * --> END POST
     *
     * <-- 200 OK (22ms)
     * Content-Type: plain/text
     * Content-Length: 6
     * <-- END HTTP
     * }</pre>
     */
    HEADERS,
    /**
     * Logs request and response lines and their respective headers and bodies (if present).
     *
     * <p>Example:
     * <pre>{@code
     * --> POST /greeting http/1.1
     * Host: example.com
     * Content-Type: plain/text
     * Content-Length: 3
     *
     * Hi?
     * --> END POST
     *
     * <-- 200 OK (22ms)
     * Content-Type: plain/text
     * Content-Length: 6
     *
     * Hello!
     * <-- END HTTP
     * }</pre>
     */
    BODY
  }
```
---
#### 3.2 自定义日志拦截器
**这里主要是根据自己的需求重新组织`logForRequest`和`logForResponse`中日志信息的组成，可以输出想要的请求头信息、请求往返时间、返回参数信息等。**

```java 
   @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        //自定义请求是日志输出格式
        logRequest(request);
        //关键点：让请求继续执行
        Response response = chain.proceed(request);
        //自定义返回时日志输出格式
        logResponse(response);
        return response;
    }
```

**这里主要看一下自定义的返回时日志的输出`logResponse(response)`，里面有一个坑需要注意一下**
- **此方法只是简单的输出URl、头信息、返回body等；**
- **注意不要跳坑：okhttp中`response.body().string()`只能有效的调用一次，否则会出现`java.lang.IllegalStateException: closed`异常**

```java
private void logResponse(Response response) {
        try {
            StringBuffer var0 = new StringBuffer("Response-Start : ");
            //url
            var0.append("url : " + response.request().url()).append("\n");
            //Headers
            Headers var1 = response.headers();
            for (int i = 0, count = var1.size(); i < count; i++) {
                String name = var1.name(i);
                // Skip headers from the request body as they are explicitly logged above.
                if (!"Content-Type".equalsIgnoreCase(name) && !"Content-Length".equalsIgnoreCase(name)) {
                    var0.append(name + ": " + var1.value(i)).append("\n");
                }
            }
            //response.body().string()
            //重点注意:这里有个坑，okhttp中response.body().string()只能有效调用一次，当调用之后，response中的流会被关闭，
            //从而出现异常：java.lang.IllegalStateException: closed
            //buffer.append("body : "+response.body().string()).append("\n");

            //response body
            ResponseBody var2 = response.body();
            if (null != var2) {
                BufferedSource source = var2.source();
                source.request(Long.MAX_VALUE);
                Buffer var3 = source.buffer();
                Charset var4 = Charset.forName("UTF8");
                MediaType contentType = var2.contentType();
                if (contentType != null) {
                    var4 = contentType.charset(Charset.forName("UTF8"));
                }

                if(isPlaintext(var3) && var2.contentLength() != 0L){
                    String var5 = var3.clone().readString(var4);
                    var0.append(var5).append("\n");
                }
            }
            var0.append("Response-End");
            Log.i("Logging", var0.toString());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static boolean isPlaintext(Buffer buffer) {
        try {
            Buffer e = new Buffer();
            long byteCount = buffer.size() < 64L?buffer.size():64L;
            buffer.copyTo(e, 0L, byteCount);

            for(int i = 0; i < 16 && !e.exhausted(); ++i) {
                int codePoint = e.readUtf8CodePoint();
                if(Character.isISOControl(codePoint) && !Character.isWhitespace(codePoint)) {
                    return false;
                }
            }

            return true;
        } catch (EOFException var6) {
            return false;
        }
    }
```
- **最终输出日志如下，这个都是按照我们自己所设定的输出的日志**
```
Response-Start : url : https://api.sdk.mobilega.com/v1/wdgame/auth/
Server: nginx
Date: Wed, 02 Aug 2017 03:17:02 GMT
Connection: keep-alive
Set-Cookie: NSC_KJFMBO-TEL-XFC-TTM=ffffffff091c2fdc45525d5f4f58455e445a4a423660;expires=Wed, 02-Aug-2017 03:18:54 GMT;path=/;secure;httponly
{"token":"490054868|***|1501643822|24|1|fff88b5e7a10d6845fc593e9266fb3b8".......}
Response-End
```

---
#### 3.3 缓存拦截器 
 **1. 简要说明`Cache-control`**
`Cache-control` 是由服务器返回的 Response 中添加的头信息，它的目的是告诉客户端是要从本地读取缓存还是直接从服务器摘取消息，它有不同的值，每一个值有不同的作用。

- **public (仅为响应标头)**    
> 响应：告知任何途径的缓存者,可以无条件的缓存该响应。

- **private (仅为响应标头)**    
> 响应：告知缓存者(一般是指用户代理、常见浏览器的本地缓存；用户也是指系统用户。但也许不应排除某些网关可以识别每个终端用户的情况)，只针对单个用户缓存响应， 且可以具体指定某个字段，如`private—"username"`，则响应头中名为username的标头内容，不会被共享缓存。

- **no-cache**
> 请求：告知缓存者必须原原本本的转发原始请求，并告知任何缓存者别直接拿你缓存的副本糊弄人，你需要去转发我的请求，并验证你的缓存(如果有的话)。
> 对应名词：端对端重载
> 
> 响应：允许缓存者缓存副本，那么其实际价值是：总是强制缓存者校验缓存的新鲜度，一旦确认新鲜，则可以使用缓存副本作为响应。 no-cache还可以指定某个包含字段，比如一个典型应用：`no-cache=Set-Cookie`。 这样做的结果：就是告知缓存者对于`Set-Cookie`字段，你不要使用缓存内容，而是使用新的，其他内容则可以使用缓存。

- **no-store**
> 请求：告知请求和响应都禁止被缓存。(也许是出于隐私考虑)    
> 
> 响应：同上

- **max-age**
> 请求：强制响应缓存者，根据该值校验新鲜性，即自身的Age值与请求时间做比较，如果超出`max-age`值则强制去服务器端验证，以确保返回一个新鲜的响应，其功能本质上与传统的Expires类似，但区别在于Expires是根据某个特定日期值做比较，一但缓存者自身的时间不准确，则结果可能就是错误的，而`max-age`显然无此问题，` max-age`的优先级也是高于Expires的。    
> 
> 响应：同上类似，只不过发出方不一样

- **max-stale**
> 请求：意思是允许缓存者发送一个过期不超过指定秒数的、陈旧的缓存。
>     
> 响应：同上

- **must-revalidate (仅为响应标头)**    
> 响应：意思是如果缓存过了新鲜期，则必须重新验证，而不是试图返回一个不在新鲜期的缓存。
> 与`no-cache`的区别在于：no-cache完全无视新鲜期的概念，总是强制重新验证，理论上`must-revalidate`更节省流量，但相比no-cache可能并不总是那么精准，因为即使缓存者认为是新鲜的，也不能保证服务器端没有做过更新。如果缓存者是一个缓存代理服务器，如果其试图重新验证时，无法连接上原始服务器，则也不允许返回一个不新鲜的、缓存中的副本，而是必须返回一个504 Gateway timeout。

- **proxy-revalidate (仅为响应标头)**    
> 响应：限制上与`must-revalidate`类似；
> 区别在于受体的范围：proxy-revalidate是要排除掉用户代理的缓存的，即其规则并不应用于用户代理的本地缓存上。

- **min-fresh (仅为请求标头)**    
> 请求：告知缓存者如果当前时间加上min-fresh的值，超了该缓存的过期时间则要给一个新的。

- **only-if-cached (仅为请求标头)**    
> 请求：告知缓存者希望内容来自缓存，且并不关心被缓存响,是否是新鲜的。

- **s-maxage (仅为响应标头)**    
> 响应：与max-age的唯一区别是：s-maxage仅仅应用于共享缓存，而不应用于用户代理的本地缓存、针对单用户的缓存，另外s-maxage的优先级要高于max-age。

- **cache-extension** 
> cache-extension是一个泛化的代称，它指所有自定义或者说扩展的指令，客户端和服务器端都可以自定义扩展Cache-Control相关的指令。
> 实际上我们可以这样设置： `Cache-Control:max-age=300, custom-directive = xxx, public`， 这样我们就定义了一个被统称为cache-extension的扩展指令，该指令如果对应的客户端或服务器端不认识就会忽略掉。

---
**2. 实现缓存拦截器**
- **2.1 缓存拦截器的实现大致可以分为以下几步：**
	- **设置缓存路径以及大小**
	- **创建 OKhttpClient，并添加拦截器**
	- **创建retrofit，并添加client**

- **2.2 定义缓存拦截器**
```java
/**
 * 1. request拦截：如果没有网络就从缓存中获取
 * 2. response拦截：如果有网络就从网络获取，并设置缓存超时时间；
 *                  如果没有网络就设置缓存时间
 */
public class CacheInterceptor  implements Interceptor{
    @Override
    public Response intercept(Chain chain) throws IOException {
        //request - 没有网络
        Request request  = chain.request();
        if(!NetUtils.hasNetwork()){
            request = request.newBuilder()
                    .cacheControl(CacheControl.FORCE_CACHE)
                    .build();
        }

        //response - 有网络
        Response response = chain.proceed(request);
        if(NetUtils.hasNetwork()){
            //设置缓存超时时间为1小时
            int maxAge = 60 * 60;
            //从请求头获取缓存设置
            //String cacheControl = request.cacheControl().toString();
            return  response.newBuilder()
                    .header("Cache-Control","public,max-age="+maxAge)
                    //.header("Cache-control",cacheControl)
                    .removeHeader("Pragma")
                    .build();
        }else{
            //设置超时时间为1天
            int maxStale = 60 * 60 * 24;
            return response.newBuilder()
                    .header("Cache-Control","public,only-if-cached,max-stale="+maxStale)
                    .removeHeader("Pragma")
                    .build();
        }
    }

	//没有代码逻辑，只作为演示
    static  class NetUtils{
        public static boolean hasNetwork(){
            return true;
        }
    }
}
```

- **2.3 定义实现接口**
```java
//可以在接口中增加请求头，设置缓存相关
public interface ApiService {
    @Headers("Cache-Control:public,max-age=3600")
    @POST
    Call<ResponseBody> queryData(@Field("id") String id);
}
```

- **2.4 简单实现**
```java
//设置缓存路径
File cacheFile = new File(this.getCacheDir(),"cacheFile");
//设置缓存为10M
Cache cache = new Cache(cacheFile,10 * 1024 * 1024);

//创建OKHttpClient，添加拦截器
OkHttpClient client = new OkHttpClient.Builder()
        .addNetworkInterceptor(new CacheInterceptor())  
        .cache(cache)
        .build();

//创建retrofit
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("")
        .addConverterFactory(GsonConverterFactory.create(new Gson()))
        .client(client)
        .build();

//请求接口
ApiService apiService = retrofit.create(ApiService.class);
apiService.queryData("1");
```
---
**3. 参考文章**
[1. Web缓存相关知识点](http://www.cnblogs.com/_franky/archive/2011/11/23/2260109.html)
[2. Retrofit2,okhttp3 通过拦截器（Interceptor）两种缓存（离线和在线）](http://blog.csdn.net/u010286855/article/details/52608485)
[3. Retrofit 2: Enabling Offline Cache](https://caster.io/episodes/retrofit-2-offline-cache/)

---
#### 3.4 拦截器中统一添加请求头
**在请求接口时都会添加请求头信息，对于公共的请求头我们可以统一添加，不要重复造轮子，但是也要结合实际，因为并不是所有的请求头信息都会是相同的，这时候我们便可以在接口定义时增加特殊的请求头信息。**
-	**统一添加公共请求头**
```java
 @Override
    public Response intercept(Chain chain) throws IOException {
        //添加统一的请求头，注意不要盲目照搬，
        //这里添加的header与在接口中传入的header即使同名，也不会覆盖，而是并行存在。
        Request request = chain.request().newBuilder()
                .addHeader("Client-Info","Android/7.0")
                .addHeader("Game_ID","5015")
                .addHeader("Accpet","application/json")
                .build();
        return chain.proceed(request);
    }
```
- **如果有个接口需要单独增加一个头信息，如：auth，可以在接口定义时传入或者在接口调用时传入**
```java
    //需要特别增加的都可以依次加入，选择哪个方式都可以
    @FormUrlEncoded
    @Headers("auth:URGLDSUWPWEWP")
    @POST
    Call<ResponseBody> request18(@Field("id") String id);
    
	@FormUrlEncoded
    @POST
    Call<ResponseBody> request18(@Header("auth") String auth,@FieldMap Map<String,String> map);

	@FormUrlEncoded
    @POST
    Call<ResponseBody> request18(@HeaderMap Map<String,String> headerMap,@FieldMap Map<String,String> map);
```
- **不同的方式加入的请求头并不会覆盖，即使是同名的。不过需要注意的是：同名参数具体解析谁需要服务端决定**
```java
--> POST https://api.sdk.game.com/v1/ztgame/login http/1.1
//统一添加的请求头
08-03 10:56:06.542 27587-28341/? D/OkHttp: Content-Type: application/x-www-form-urlencoded
08-03 10:56:06.543 27587-28341/? D/OkHttp: Content-Length: 38
08-03 10:56:06.543 27587-28341/? D/OkHttp: GAME-ID: 5012
08-03 10:56:06.543 27587-28341/? D/OkHttp: Accept: application/json
08-03 10:56:06.543 27587-28341/? D/OkHttp: Client-Info: ***
08-03 10:56:06.543 27587-28341/? D/OkHttp: Authentication: ****

//拦截器添加的请求头
08-03 10:56:06.543 27587-28341/? D/OkHttp: Client-Info: Android/7.0
08-03 10:56:06.543 27587-28341/? D/OkHttp: GAME-ID: 5015
08-03 10:56:06.543 27587-28341/? D/OkHttp: Accpet: application/json

//接口定义单独传入的请求头
08-03 10:56:06.543 27587-28341/? D/OkHttp: auth: YUYUYU

08-03 10:56:06.544 27587-28341/? D/OkHttp: --> END POST (38-byte body)
```
---

#### 好文推荐
[1. Retrofit — Basic Authentication on Android](https://futurestud.io/tutorials/android-basic-authentication-with-retrofit)