---
title: Retrofit2.0系列 - 简单封装请求(3)
copyright: true
date: 2019-10-30 16:25:56
tags:
	封装请求
categories:
	Retrofit2.0系列
---


- **Retrofit 2.0系列 - 玩转注解(1)**
- **Retrofit 2.0系列 - 全面解析拦截器(2)**
- **Retrofit2.0系列 - 简单封装请求(3)**

---

#### 1. 请求接口的定义
**使用天气接口进行调试，定义接口时要注意几点：**
- 使用GET或者POST方式请求时，注解使用的是不同的
- GET形式的接口：weather_mini?city={city}，注意接口中参数的传递方式
- Call<>函数中的参数与实际调用的要保持一致，默认的ResponseBody返回的是json形式
```java
public interface GApiServices {
    /**
     * 错误方式
     * weather_mini?city={city}
     * 请求参数在？号后面，此种方式需用@Query，否则抛出如下异常，并且参数不需要在@GET()里面声明
     * java.lang.IllegalArgumentException: URL query string "city={city}" must not have replace block. For dynamic query parameters use @Query.
     * 
     */
    @GET("weather_mini?city={city}")
    Call<ResponseBody> queryCityWeather1(@Query("city") String city);


    /**
     *  完整的请求地址
     *  http://wthrcdn.etouch.cn/weather_mini?city=%E4%B8%8A%E6%B5%B7
     */
    @GET("weather_mini")
    Call<WeatherBean> queryCityWeather2(@Query("city") String city);
 }
```

<!-- more -->

---

#### 2.请求的统一封装
**关于请求封装的几点说明：**
- 请求使用单例的形式
- 提供了可替换域名的接口
- 加入了自定义的请求头拦截器
- 加入了okhttp中的日志拦截器
```java
public class RetrofitManager {
    private final static String BASE_URL = "http://wthrcdn.etouch.cn/";
    private static RetrofitManager manager;
    private static Retrofit retrofit;
    private static OkHttpClient client;

    public static RetrofitManager getInstance() {
        if (manager == null) {
            synchronized (RetrofitManager.class) {
                if (manager == null) {
                    manager = new RetrofitManager();
                }
            }
        }
        return manager;
    }

	//不可替换域名
    public GApiServices getApiServices(Context context) {
        return getApiServices(context, BASE_URL);
    }

	//可替换域名
    public GApiServices getApiServices(Context context, String url) {
        if(!url.equals(BASE_URL)){
            retrofit = null;
        }
        if (retrofit == null) {
            retrofit = new Retrofit.Builder().baseUrl(url).addConverterFactory(GsonConverterFactory.create(new
                    GsonBuilder().setDateFormat("yyyy-MM-dd hh:mm:ss").create())).client(getClient(context)).build();
        }
        return retrofit.create(GApiServices.class);
    }


    private static OkHttpClient getClient(Context context) {
        if (client == null) {
            client = new OkHttpClient.Builder().
		           //请求头加入的位置会影响日志拦截器，如果请求头在前，日志的request是可以拦截到头信息，否则拦截不到
                    addInterceptor(new HeadersInterceptor())
                    //日志拦截
                    .addInterceptor(new HttpLoggingInterceptor()
                    .setLevel(HttpLoggingInterceptor.Level.BODY))
                    //时间可以定义为常量
                    .connectTimeout(30, TimeUnit.SECONDS)
                    .readTimeout(30, TimeUnit.SECONDS)
                    .writeTimeout(30, TimeUnit.SECONDS)
                    .retryOnConnectionFailure(true)
                    .build();
        }
        return client;
    }
}
```

---
#### 3.返回数据的统一格式
**既然是统一的请求封装，返回的数据也需要统一格式，这里还是以天气接口返回数据作为示例**
- 天气接口返回的json数据(省略部分数据)
```java
{
    "data":{
        "yesterday":{
            "date":"12日星期四",
            "high":"高温 14℃",
            "fx":"北风",
            "low":"低温 6℃",
            "fl":"<![CDATA[<3级]]>",
            "type":"小雨"
        }
    },
    "status":1000,
    "desc":"OK"
}
```
- 定义基础的bean，不管哪个接口都会返回**`status、desc、data数据`**等固定的字段
- 使用T泛型，子类只需要关心data数据中的具体字段
```java
public class BaseBean<T> implements Serializable{
    private int status;
    private String desc;
    private T data;

    public T getData() {
        return data;
    }

    public int getStatus() {
        return status;
    }

    public String getDesc() {
        return desc;
    }
}
```
- 具体的实现类，继承自BaseBean，然后根据返回数据中的data数据定义字段
```java
public class WeatherBean2 extends BaseBean<WeatherBean2> {
    private YesterdayBean yesterday;
    private String city;

    public YesterdayBean getYesterday() {
        return yesterday;
    }

    public String getCity() {
        return city;
    }

    public static class YesterdayBean {
	    //省略具体的字段.......
    }
}
```
- 实际调用接口请求数据,处理回调
```java
RetrofitManager.getInstance().getApiServices(this).queryCityWeather2("上海").enqueue(new Callback<WeatherBean2>() {
           @Override
           public void onResponse(Call<WeatherBean2> call, Response<WeatherBean2> response) {
               WeatherBean2 bean = response.body();
               Log.i("wd_re", "getStatus = " + bean.getStatus());
               Log.i("wd_re", "getDesc = " + bean.getDesc());
               
               WeatherBean2.YesterdayBean weatherBean2 =  bean.getData().getYesterday();
               Log.i("wd_re", "日期 = " + weatherBean2.getDate());
           }
           @Override
           public void onFailure(Call<WeatherBean2> call, Throwable t) {
           }
    });
```

---
#### 4.自定义回调函数Callback，简化返回操作
看到上面的请求结果，成功之后还是在**`onResponse`**中做的处理，还是太过复杂，不太符合我们的需求。
- 需求：成功进入onSuccess回调，失败进入onFailure回调，并且单个接口不关心怎么进入，只需要处理对应的回调逻辑

**改进方式：**
- 自定义callback函数，统一处理返回结果
- 回调函数中返回相应的code以及返回结果的对象
```java
public abstract class BaseCallback<T> implements Callback<T> {
    @Override
    public void onResponse(Call<T> call, Response<T> response) {
        try {
            if (response.isSuccessful()) {//[200..300)
                BaseBean<T> baseBean = (BaseBean<T>) response.body();
                T bean = baseBean.getData();
                onSuccess(response.code(), bean);
            } else {
                onFailure(response.code(), response.message());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onFailure(Call<T> call, Throwable t) {
        call.cancel();
        onNetFailure(t);
    }

	//定义抽象方法，回调中根据情况选择执行
    public abstract void onSuccess(int code, T response);

    public abstract void onFailure(int code, String errMsg);

    public abstract void onNetFailure(Throwable e);
}
```
- 实际调用接口请求数据，处理回调
```java
 RetrofitManager.getInstance().getApiServices(this).queryCityWeather2("上海")
 .enqueue(new BaseCallback<WeatherBean2>(){
           @Override
           public void onSuccess(int code, WeatherBean2 response) {
	           //根据定义的对象直接获取参数
               Log.i("wd_re", "getYesterday = " + response.getYesterday().toString());
               Log.i("wd_re", "getCity = " + response.getCity());
           }
           
           @Override
           public void onFailure(int code, String errMsg) {
           }
           
           @Override
           public void onNetFailure(Throwable e) {
           }
       });
```


---
#### 5.存在问题以及需要改进之处
- 这里定义的接口都在**`GApiServices`** ，假如某一天接口需要写到另一个**`ApiServices`** 中，那么**`getApiServices`**接口就不能适应修改；
- 假如返回数据**`response.body()`**返回的数据解析异常，应该提前检测并做好相应的处理；
- 一般返回数据的接口中都是**`"detail":{ ... }`**，加入某个接口返回的是数组：**`"detail":[]`**；
- 提前对服务端返回结果根据不同的code状态进行判断，减少每次请求时都要判断是否成功、失败；


