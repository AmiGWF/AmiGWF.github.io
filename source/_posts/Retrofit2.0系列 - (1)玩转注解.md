---
title: Retrofit2.0系列 - (1)玩转注解
copyright: true
date: 2019-10-30 15:41:23
tags:
	注解
categories:
	Retrofit2.0系列
---

[Retrofit 2.0系列-玩转注解](https://www.syncxiao.com/2019/10/30/Retrofit2.0%E7%B3%BB%E5%88%97%20-%20%E7%8E%A9%E8%BD%AC%E6%B3%A8%E8%A7%A3(1)/)


- **[Retrofit 2.0系列 - 全面解析拦截器(2)](https://www.syncxiao.com/2019/10/30/Retrofit2-0%E7%B3%BB%E5%88%97%20-%20%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90%E6%8B%A6%E6%88%AA%E5%99%A8(2)/)**
- **[Retrofit2.0系列 - 简单封装请求(3)](https://www.syncxiao.com/2019/10/30/Retrofit2-0%E7%B3%BB%E5%88%97-%E7%AE%80%E5%8D%95%E5%B0%81%E8%A3%85%E8%AF%B7%E6%B1%82(3)/)**

---


#### 1. 方法注解：
**Retrofit 2.0 支持以下方法注解：`@GET、@POST、@PUT、@DELETE、@PATH、@HEAD、@OPTIONS、@HTTP`，这和Http中涉及到的请求方法基本保持一致，但是多了一个：`@HTTP`的方法注解。**
```java
	/**
	 * 自定义HTTP请求方法
     * @HTTP ： 方法注解，可以替代HTTP中其他的方法
     * method ：指定请求的方法，如GET、POST
     * path ： 接口路径
     * hasBody ： 是否有请求体
     */
    @HTTP(method = "GET",path = "wdgame/auth/{id}",hasBody = false)
    Call<ResponseBody> requestIn17(@Path("id")String id);
```

<!-- more -->

#### 2. 标记注解：
- **`@FormUrlEncoded`：用于`POST`请求时以表单方式提交**

- **`@Multipart`:用于文件上传**
 
- **`@Streaming`:用于大文件的下载**

#### 3. 参数注解：
 **以下请求链接中BASE_URL为：https://api.sdk.com/v2/**

 - **`@Path`：URL占位符，用于请求链接的?号之前，可支持多个@Path连续使用，但是参数个数固定**
```java
	 /**
     * 访问链接：https://api.sdk.com/v2/wdgame/{auth}?name=wd
     * 
     * https://api.sdk.com/v2/wdgame/UEGLAUWGJAJG?name=wd
     */
    @GET("wdgame/{auth}")
    Call<ResponseBody> requestIn1(@Path("auth") String auth);
    

	/**
     * 访问链接：https://api.sdk.com/v2/wdgame/{auth}/{id}
     * 
     * https://api.sdk.com/v2/wdgame/IDFAJGADKHGA/642232884
     */
    @GET("wdgame/{auth}/{id}")
    Call<ResponseBody> requestIn2(@Path("auth") String auth, @Path("id") String id);
```

- **`@Query & @QueryMap`：用于请求链接的?号之后，可支持多个@Query连续使用，但是参数个数固定，@QueryMap参数不固定**
```java
	/**
     * 访问链接：https://api.sdk.com/v2/wdgame/auth/?authId=??&id=??
     * 
     * https://api.sdk.com/v2/wdgame/auth/?authId=YWKFIADHGFAL&id=623042744
     */
    @GET("wdgame/auth")
    Call<ResponseBody> requestIn3(@Query("authId") String authId, @Query("id") String id);


    /**
     * 访问链接：https://api.sdk.com/v2/wdgame/auth/?authId=??&id=??
     * 
     * https://api.sdk.com/v2/wdgame/auth/?authId=YWKFIADHGFAL&id=623042744
     * 
     * 参数以Map的形式传入
     * map.put("authId","QWEDOWUEHGSL")
     * map.put("id","232372794")
     */
    @GET("wdgame/auth")
    Call<ResponseBody> requestIn4(@QueryMap Map<String, String> map);
```

- **`@Field & @FieldMap`：配合`@FormUrlEncoded`使用，用于POST请求时简单的键值对参数的传递，支持Map形式传递参数**
```java
	/**
     * 访问链接：https://api.sdk.com/v2/wdgame/auth
     * 表单形式提交
     * 参数：account、password
     */
    @FormUrlEncoded
    @POST("wdgame/auth")
    Call<ResponseBody> requestIn5(@Field("account") String account, @Field("password") String password);

    /**
     * 访问链接：https://api.sdk.com/v2/wdgame/auth
     * 表单形式提交
     * 参数：account、password
     * 
     * 参数以Map的形式传入
     * map.put("account","wd")
     * map.put("password","123456")
     */
    @FormUrlEncoded
    @POST("wdgame/auth")
    Call<ResponseBody> requestIn6(@FieldMap Map<String, String> map);

	/**
     * 访问链接：https://api.sdk.com/v2/wdgame/auth
     * 传递数组参数
     *
     * 网上说可以这样传递，但是多个数组会有问题，暂未考证
     */
    @POST("wdgame/auth")
    Call<ResponseBody> requestIn8(@Field("account[]") String... account);
```

- **`@Body`：将实例对象转化为json格式数据传递，可以不需要配合`@FormUrlEncoded`**
```java
	/**
     * 访问链接：https://api.sdk.com/v2/wdgame/auth
     * 以对象的形式传递json数据
     *
     * AccountBean.Account bean = new AccountBean.Account();
     * bean.setAccount("wd");
     * bean.setUid(1246);
     * requestIn7(bean)
     */
    @POST("wdgame/auth")
    Call<ResponseBody> requestIn7(@Body AccountBean.Account bean);
```

- **`@Part & @PartMap & @Multipart`：用于POST文件上传，需要配合`@Multipart`使用，多用于字节流的传输**
```java
	/**
     * 单文件上传
     * 访问链接：https://api.sdk.com/v2/wdgame/upload
     *
     * 未验证
     */
    @Multipart
    @POST("wdgame/upload")
    Call<ResponseBody> requestIn9(@Part("desc") ResponseBody desc ,@Part MultipartBody.Part file);

    /**
     * 多文件上传
     * 访问链接：https://api.sdk.com/v2/wdgame/upload
     *
     * 未验证
     */
    @Multipart
    @POST("wdgame/upload")
    Call<ResponseBody> requestIn10(@Part("desc") ResponseBody desc ,@PartMap Map<String,ResponseBody> files);
```

-  **`@Header & @Headers`:请求头的单独设置或以参数的形式传入，适合于特殊情况，不使用统一的请求头，一个是修饰参数、一个是修饰方法**
```java
	/**
     * 动态设置单个请求头
     */
    @GET("wdgame/auth")
    Call<ResponseBody> requestIn11(@Header("X-Client-Info")String info,@Query("authId") String authId, @Query("id") String id);

    /**
     * 动态设置多个请求头
     */
    @GET("wdgame/auth")
    Call<ResponseBody> requestIn12(@HeaderMap Map<String,String> map,@Query("authId") String authId, @Query("id") String id);

    /**
     * 静态设置多个请求头
     */
    @Headers({"X-Client-Info:******","Accept:******"})
    @GET("wdgame/auth")
    Call<ResponseBody> requestIn13(@Query("authId") String authId, @Query("id") String id);
```

- **`@Url`：Url以参数的形式传入，用于特殊的需要单独构造的Url，并且参数、请求头也都可以单独传递**
```java
	/**
     * 请求链接、参数、请求头全部分散传入
     * https://api.sdk.com/v2/wdgame/auth?auth=DGSDJLWOURNGA
     * 如果参数在？之前，此种形式存在问题
     */
    @GET
    Call<ResponseBody> requestIn14(@Url String url, @QueryMap Map<String, Object> paramMap, @HeaderMap Map<String, Object> headsMap);


    /**
     * 请求链接、参数、请求头全部分散传入
     * https://api.sdk.com/v2/wdgame/auth
     */
    @POST
    @FormUrlEncoded
    Call<ResponseBody> requestIn15(@Url String url, @FieldMap Map<String, Object> paramMap, @HeaderMap Map<String, Object> headsMap);
```


- **`@Streaming`：用于大文件的下载**
```java
	/**
     * 用于下载大文件
     */
    @Streaming
    @GET
    Call<ResponseBody> requestIn16(@Url String url);
```





