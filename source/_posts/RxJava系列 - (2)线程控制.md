---
title: RxJava系列 - (2)线程控制
copyright: true
date: 2019-10-30 16:32:00
tags:
categories:
	RxJava系列
---


[RxJava系列 - (1)初步认识](https://www.syncxiao.com/2019/10/30/RxJava%E7%B3%BB%E5%88%97%20-%20&#40;1&#41;%E5%88%9D%E6%AD%A5%E8%AE%A4%E8%AF%86/)
[RxJava系列 - (2)线程控制](https://www.syncxiao.com/2019/10/30/RxJava%E7%B3%BB%E5%88%97%20-%20&#40;2&#41;%E7%BA%BF%E7%A8%8B%E6%8E%A7%E5%88%B6/)
[RxJava系列 - (3)变换符操作](https://www.syncxiao.com/2019/10/30/RxJava%E7%B3%BB%E5%88%97%20-%20&#40;3&#41;%E5%8F%98%E6%8D%A2%E7%AC%A6%E6%93%8D%E4%BD%9C/)
[RxJava系列 - (4)zip操作符](https://www.syncxiao.com/2019/10/30/RxJava%E7%B3%BB%E5%88%97%20-%20&#40;4&#41;zip%E6%93%8D%E4%BD%9C%E7%AC%A6/)
[RxJava系列 - (5)Backpressure & Flowable](https://www.syncxiao.com/2019/10/30/RxJava%E7%B3%BB%E5%88%97%20-%20&#40;5&#41;Backpressure%20&%20Flowable/)
[RxJava系列 - (6)操作符说明](https://www.syncxiao.com/2019/04/20/RxJava%E7%B3%BB%E5%88%97%20-%20&#40;6&#41;%E6%93%8D%E4%BD%9C%E7%AC%A6%E8%AF%B4%E6%98%8E/)
[RxJava系列 - (7)Observables之间的区别 ](https://www.syncxiao.com/2019/06/08/RxJava%E7%B3%BB%E5%88%97%20-%20&#40;7&#41;Observables%E4%B9%8B%E9%97%B4%E7%9A%84%E5%8C%BA%E5%88%AB/)

---


RxJava还提供了强大的线程控制模型，我们可以根据不同的需求指定相应的线程进行控制。

#### 1.统一线程执行任务
![水管模式1](https://i.loli.net/2019/10/30/Olc23vMm19T7Uwy.png)

我们还是以水管作为我们的模型，一般在未指定线程的情况下，上游和下游都是处在同一个线程中(Android中默认为主线程)，即上游在哪个线程中发送事件，下游就在哪个线程中接收事件。

- **Android默认主线程**
我们举个例子说明上面的默认线程：主线程。假设在`onCreate`中有如下操作：上游创建Observable，下游接收事件，分别通过日志查看上游、下游所处的线程。
```java
//上游、下游事件
 Observable<String> o1 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                e.onNext("Thread Test");
                Log.d(TAG, "subscribe-onNext所在线程 ：" + Thread.currentThread().getName());
            }
        });


Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String o) throws Exception {
                Log.d(TAG, "consumer-accept接收数据 : " +o+";  所在thread : "+ Thread.currentThread().getName());
            }
        };

o1.subscribe(consumer);


//日志结果
D/RxJava: consumer-accept接收数据 : Thread Test;  所在thread : main	//上游
D/RxJava: subscribe-onNext所在线程 ：main  //下游
```
可以看到上游`Observable`在主线程中发送了事件，下游`Consumer`同样在主线程接收事件，从水管模型来说就是上游、下游是在同一根水管中流动着。

---

<!-- more -->

#### 2.切换线程执行任务
显然上面的模式不太符合我们实际的需求，比如：我们需要在子线程中进行耗时操作，然后回到主线程中更新UI，使用水管来描述就是如下图：
![水管模式2](https://i.loli.net/2019/10/30/VXDs6rZCQuUS4tl.png)

图中我们用`黄色表示子线程，蓝色表示主线程`，即上游在子线程发送事件，下游在主线程接收事件，线程的控制就交给RxJava来进行调度。
```java
//上游、下游事件，事件的创建方式不变
 Observable<String> o1 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                e.onNext("Thread Test");
                Log.d(TAG, "subscribe-onNext所在线程 ：" + Thread.currentThread().getName());
            }
        });

Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String o) throws Exception {
                Log.d(TAG, "consumer-accept接收数据 : " +o+";  所在thread : "+ Thread.currentThread().getName());
            }
        };
        
//事件调用
//subscribeOn一般指定上游,observeOn一般指定下游
o1.subscribeOn(Schedulers.newThread())//子线程
        .observeOn(AndroidSchedulers.mainThread())//主线程
        .subscribe(consumer);

//日志结果
RxJava: consumer-accept接收数据 : Thread Test;  所在thread : main  //主线程接收事件
D/RxJava: subscribe-onNext所在线程 ：RxNewThreadScheduler-49  //子线程发送事件
```
事件的创建方式没有改变，只是在调用事件时增加了两句代码：
`.subscribeOn(Schedulers.newThread())
 .observeOn(AndroidSchedulers.mainThread())`
 通过日志就可看到，上游发送事件是在名为：`RxNewThreadScheduler-49 `的线程中，而下游接收事件是在`main`即主线程中。

一般来说可以这么理解：`subscribeOn()`方法用于指定上游发送事件的线程，`observeOn()`方法用于指定下游接收事件的线程。

自然我们就会产生疑问：**多次调用线程切换会是什么结果呢？是遵循第一次的还是遵循最后调用的呢?**
- 多次指定上游的线程只有第一次指定的有效, 也就是说多次调用subscribeOn() 只有第一次的有效, 其余的会被忽略.
- 多次指定下游的线程是可以的, 也就是说每调用一次observeOn() , 下游的线程就会切换一次.

线程的创建不变，我们看下调用时多次切换线程的结果：
```java
发送事件时先指定子线程，然后切换到io线程；接收事件时先指定主线程，然后切换到子线程，再切换到熊线程。

//上游多次指定线程只有第一次会生效
o1.subscribeOn(Schedulers.newThread())
        .subscribeOn(Schedulers.io())
        //下游多次指定线程，每次都会生效
        .observeOn(AndroidSchedulers.mainThread())
        .observeOn(Schedulers.newThread())
        .observeOn(Schedulers.io())
        .subscribe(consumer);

//日志结果
D/RxJava: subscribe-onNext所在线程 ：RxNewThreadScheduler-1
D/RxJava: consumer-accept接收数据 : Thread Test;  所在thread : RxCachedThreadScheduler-2
```
从日志来看，发送事件是在`RxNewThreadScheduler-1线程`，接收事件是在`RxCachedThreadScheduler-2线程`。其中**`CachedThread表示的是IO线程池中的一个`**，我们上游、下游最后切换到的都是`Schedulers.io()`，但是只有下游处于IO线程中，所以验证了我们上面的结论：**多次切换线程，上游subscribeOn()只有第一次有效，下游observeOn()每次都会生效，切换线程。**

除了上面的形式之外,我们还可以在调用`doOnNext()`之前指定线程，达到切换线程的效果。
```java
//上游多次指定线程只有第一次会生效
o1.subscribeOn(Schedulers.newThread())
        .subscribeOn(Schedulers.io())
        //下游多次指定线程，每次都会生效
        .observeOn(AndroidSchedulers.mainThread())
        .observeOn(Schedulers.io())
        .doOnNext(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
            //IO线程
            Log.d(TAG, "IOThread : doOnNext-accept :"+Thread.currentThread().getName());
            }
        })
        .observeOn(AndroidSchedulers.mainThread())
        .doOnNext(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
            //主线程
            Log.d(TAG, "MainThread : doOnNext-accept :"+Thread.currentThread().getName());
            }
        })
        .observeOn(Schedulers.newThread())
        //子线程
        .subscribe(consumer);


//日志结果
D/RxJava: subscribe-onNext所在线程 ：RxNewThreadScheduler-1
D/RxJava: IOThread : doOnNext-accept :RxCachedThreadScheduler-2
D/RxJava: MainThread : doOnNext-accept :main
D/RxJava: consumer-accept接收数据 : Thread Test;  所在thread : RxNewThreadScheduler-2
```
可以看到在执行`doOnNext()`方法之前分别指定了IO线程和AndroidMain线程，日志结果也明确的反应出了两个方法在不同线程中执行的结果。


---
#### 3.RxJava提供的多种线程
在RxJava中, 已经内置了很多线程选项供我们选择, 例如有：
- Schedulers.io() 代表io操作的线程, 通常用于网络,读写文件等io密集型的操作
- Schedulers.computation() 代表CPU计算密集型的操作, 例如需要大量计算的操作
- Schedulers.newThread() 代表一个常规的新线程
- AndroidSchedulers.mainThread() 代表Android的主线程

这些内置的Scheduler已经足够满足我们开发的需求, 因此我们应该使用内置的这些选项, 在RxJava内部使用的是线程池来维护这些线程, 所有效率也比较高.

---
#### 4.简单实例：获取天气
我们以获取天气作为实例，封装请求相对简单，只作为示例，不做深究。
- **接口的定义**
```java
//定义接口，根据城市名称获取对应天气，切换这里的Bean对象要与返回的Json对象对应，因为我们是通过Gson来进行对象转化
@GET("weather_mini")
Observable<WeatherBean> queryCityWeatherRX1(@Query("city") String city);
```
- **请求示例与数据解析**
```java
//获取上海的天气，注意线程之间的切换，主线程中做耗时操作会抛出异常
RetrofitManager.getInstance().getApiServices(this).queryCityWeatherRX1("上海")
               .subscribeOn(Schedulers.io()) //io线程发起网络请求
               .observeOn(AndroidSchedulers.mainThread()) //主线程处理结果
               .subscribe(new Observer<WeatherBean>() {
                   @Override
                   public void onSubscribe(Disposable d) {
                   }
                   @Override
                   public void onNext(WeatherBean weatherBean2) {
                   Log.i("wd_re", "getYesterday = " + weatherBean2.getData().getYesterday());
                   Log.i("wd_re", "getCity = " + weatherBean2.getData().getCity());
                   }
                   @Override
                   public void onError(Throwable e) {
                       Log.i("wd_re", "请求失败 : "+e.toString());
                   }
                   @Override
                   public void onComplete() {
                       Log.i("wd_re", "请求成功");
                   }
               });

//请求结果
getYesterday = YesterdayBean{date='13日星期一', high='高温 19℃', fx='东南风', low='低温 14℃', fl='<![CDATA[<3级]]>', type='小雨'}
getCity = 上海
请求成功
```
- **线程问题**
如果我们做请求网络等耗时操作时，直接在主线程中进行，即请求时去除**`.subscribeOn(Schedulers.io()`**，
会出现如下错误：**`android.os.NetworkOnMainThreadException`**。
```java
//出现线程错误 : android.os.NetworkOnMainThreadException
 RetrofitManager.getInstance().getApiServices(this).queryCityWeatherRX1("上海")
                    //.subscribeOn(Schedulers.io())//io线程发起网络请求
                    .observeOn(AndroidSchedulers.mainThread())//主线程处理结果
                    .subscribe()
```

---
#### 4.重点推荐：RxJava2.0的系列文章
这是一个关于RxJava2.0的系列教程，很有学习的价值，我的例子也是参考而来，重点推荐大家看下。
[给初学者的RxJava2.0教程](http://www.jianshu.com/p/464fa025229e)






