---
title: RxJava系列 - (4)zip操作符
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
#### 1.zip操作符说明
RxJava中的操作符多种多样，这里介绍的是名为：zip的操作符。
**`zip`**可以通过一个函数Function将多个Observable发送的事件按顺序组合到一起，然后再依次发送这些组合事件，并且发送组合事件的个数与组合前的Observable中发送事件最少的一致。

通过管道模型图来理解下：
![Alt text](https://i.loli.net/2019/10/30/uePrYAc5Dg6psxJ.png)

上图中与以往不同之处在于我们上游存在两个管道。
管道1发送圆形事件，管道2发送三角形事件，通过**`zip操作符`**的组合后变为了矩形事件。
为了加强理解，我们再看下分解动作：
![Alt text](https://i.loli.net/2019/10/30/lbFCeW4hcPdwzGI.png)

从分解动作中，我们获取到以下几个重要信息：
- **组合过程是有序且一一对应的。**组合是分别从管道1、管道2中按顺序分别取出一个事件，组合成新的事件，所有上游事件只能使用一次并且严格遵守顺序。所以不会出现管道1中的事件1和管道2中的事件B进行组合的情况。
- **下游组合事件的最大数量由上游管道发送的最小数量决定。**很容易理解就是既然是组合事件，两个上游组合为一个上游且不重复，所以上游最少的事件数量就决定了下游组合事件的数量。

---


<!-- more -->

#### 2.zip操作符使用
按照上面的理解，我们撸一段代码看看什么情况。
```java
//上游1-发送了三个事件和complete
Observable<String> o1 = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> e) throws Exception {
        Log.i(TAG, "---------- o1 send 1 ----------");
        e.onNext("The One event 1");
        Log.i(TAG, "---------- o1 send 2 ----------");
        e.onNext("The One event 2");
        Log.i(TAG, "---------- o1 send 3 ----------");
        e.onNext("The One event 3");
        Log.i(TAG, "---------- o1 onComplete ----------");
        e.onComplete();
    }
});
//上游2-发送了两个事件和complete
Observable<Integer> o2 = Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        Log.i(TAG, "******* o2 send 1 *******");
        e.onNext(100);leep(2000);
        Log.i(TAG, "******** o2 send 2 ********");
        e.onNext(200);sleep(2000);
        Log.i(TAG, "******** o2 onComplete ********");
        e.onComplete();
    }
});
//zip合并事件，注意BiFunction函数中的参数要对应，前面的参数与上游Observable对应，最后一个参数与合并后返回的类型对应
Observable.zip(o1, o2, new BiFunction<String,Integer,String>() {
    @Override
    public String apply(String o, Integer o2) throws Exception {
        Log.i(TAG, "---------- zip ----------");
        return o+o2;
    }
}).subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onNext(String s) {
        Log.i(TAG, "---------- zip onNext ----------"+s);
    }
    @Override
    public void onError(Throwable e) {
    }
    @Override
    public void onComplete() {
        Log.i(TAG, "---------- zip complete ----------");
    }
});

//日志结果
I/RxJava: ---------- o1 send 1 ----------
I/RxJava: ---------- o1 send 2 ----------
I/RxJava: ---------- o1 send 3 ----------
I/RxJava: ---------- o1 onComplete ----------
I/RxJava: ******* o2 send 1 *******
I/RxJava: ---------- zip ----------
I/RxJava: ---------- zip onNext ----------The One event 1100
I/RxJava: ******** o2 send 2 ********
I/RxJava: ---------- zip ----------
I/RxJava: ---------- zip onNext ----------The One event 2200
I/RxJava: ******** o2 onComplete ********
I/RxJava: ---------- zip complete ----------
```
看起来不错，该执行的都执行了，该输出的日志信息看得也很清晰。
但是有没有发现，**为嘛o1发送完了之后，才轮到o2发送呢？说好的你一个，我一个，孕育新生命的呢？**

- **o1、o2都睡那么一小会看看**
```java
Observable<String> o1 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                Log.i(TAG, "---------- o1 send 1 ----------");
                e.onNext("The One event 1");
                SystemClock.sleep(2000);
                Log.i(TAG, "---------- o1 send 2 ----------");
                e.onNext("The One event 2");
                SystemClock.sleep(2000);
                Log.i(TAG, "---------- o1 send 3 ----------");
                e.onNext("The One event 3");
                SystemClock.sleep(2000);
                Log.i(TAG, "---------- o1 onComplete ----------");
                e.onComplete();
                SystemClock.sleep(2000);

            }
        });

        Observable<Integer> o2 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                Log.i(TAG, "******* o2 send 1 *******");
                e.onNext(100);
                SystemClock.sleep(2000);
                Log.i(TAG, "******** o2 send 2 ********");
                e.onNext(200);
                SystemClock.sleep(2000);
                Log.i(TAG, "******** o2 onComplete ********");
                e.onComplete();
                SystemClock.sleep(2000);

            }
        });

//日志结果
I/RxJava: ---------- o1 send 1 ----------
I/RxJava: ---------- o1 send 2 ----------
I/RxJava: ---------- o1 send 3 ----------
I/RxJava: ---------- o1 onComplete ----------
I/RxJava: ******* o2 send 1 *******
I/RxJava: ---------- zip ----------
I/RxJava: ---------- zip onNext ----------The One event 1100
I/RxJava: ******** o2 send 2 ********
I/RxJava: ---------- zip ----------
I/RxJava: ---------- zip onNext ----------The One event 2200
I/RxJava: ******** o2 onComplete ********
I/RxJava: ---------- zip complete ----------
```
看了这日志结果，此刻我的内心是奔溃的，睡了一下也还是o1先执行完成，才轮到o2。那我们想想还有什么可以改变的呢？
- **比如说切换一下o1、o2所在的线程**
```java
Observable<String> o1 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                Log.i(TAG, "---------- o1 send 1 ----------");
                e.onNext("The One event 1");
                SystemClock.sleep(2000);
                Log.i(TAG, "---------- o1 send 2 ----------");
                e.onNext("The One event 2");
                SystemClock.sleep(2000);
                Log.i(TAG, "---------- o1 send 3 ----------");
                e.onNext("The One event 3");
                SystemClock.sleep(2000);
                Log.i(TAG, "---------- o1 onComplete ----------");
                e.onComplete();
                SystemClock.sleep(2000);

            }
        }).subscribeOn(Schedulers.io());

        Observable<Integer> o2 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                Log.i(TAG, "******* o2 send 1 *******");
                e.onNext(100);
                SystemClock.sleep(2000);
                Log.i(TAG, "******** o2 send 2 ********");
                e.onNext(200);
                SystemClock.sleep(2000);
                Log.i(TAG, "******** o2 onComplete ********");
                e.onComplete();
                SystemClock.sleep(2000);

            }
        }).subscribeOn(Schedulers.io());

//日志结果
I/RxJava: ---------- o1 send 1 ----------
I/RxJava: ******* o2 send 1 *******
I/RxJava: ---------- zip ----------
I/RxJava: ---------- zip onNext ----------The One event 1100
I/RxJava: ---------- o1 send 2 ----------
I/RxJava: ******** o2 send 2 ********
I/RxJava: ---------- zip ----------
I/RxJava: ---------- zip onNext ----------The One event 2200
I/RxJava: ---------- o1 send 3 ----------
I/RxJava: ******** o2 onComplete ********
I/RxJava: ---------- zip complete ----------
I/RxJava: ---------- o1 onComplete ----------
```
这下看起来就很舒服了。
我们使得**o1、o2在不同的io线程中执行，**然后可以看到**o1发送事件之后，o2就跟着发送事件，然后zip执行操作，如此循环，直到某个管道事件发送结束**。

我们看到o1原本发送了**三个事件+complete**，o2原本发送了**两个事件+complete**，但是注意看，我们只输出了两个包含
**`--- zip onNext ---`**的日志，然后**o1发送事件3时，o2就直接onComplete了**，这就说明**o2的所有事件发送完了之后，不会再和o1的事件组合了，即使o1还会发送剩余的事件。**

- **Thread.sleep()对发送事件的影响**
在**给初学者的RxJava2.0教程(四)**中o1的剩余事件也没有发送了，而上面的日志中可以看到o1剩余的事件也发送了，即使没什么意义了。
这里的区别在于我使用的睡觉方式不同，使用的是：**SystemClock.sleep(2000)**，因为使用**Thread.sleep()**在这里会遇到一个问题：
```java
11-27 18:07:21.961 9942-10148/? E/AndroidRuntime: FATAL EXCEPTION: RxCachedThreadScheduler-1
 Process: com.wd.rm, PID: 9942
 io.reactivex.exceptions.UndeliverableException: java.lang.InterruptedException
     at io.reactivex.plugins.RxJavaPlugins.onError(RxJavaPlugins.java:349)
     at io.reactivex.internal.operators.observable.ObservableCreate$CreateEmitter.onError(ObservableCreate.java:74)
     at io.reactivex.internal.operators.observable.ObservableCreate.subscribeActual(ObservableCreate.java:43)
     at io.reactivex.Observable.subscribe(Observable.java:10910)
     at io.reactivex.internal.operators.observable.ObservableSubscribeOn$SubscribeTask.run(ObservableSubscribeOn.java:96)
     at io.reactivex.Scheduler$DisposeTask.run(Scheduler.java:452)
     at io.reactivex.internal.schedulers.ScheduledRunnable.run(ScheduledRunnable.java:61)
     at io.reactivex.internal.schedulers.ScheduledRunnable.call(ScheduledRunnable.java:52)
     at java.util.concurrent.FutureTask.run(FutureTask.java:237)
     at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:272)
     at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
     at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
     at java.lang.Thread.run(Thread.java:761)
  Caused by: java.lang.InterruptedException
     at java.lang.Thread.sleep(Native Method)
     at java.lang.Thread.sleep(Thread.java:371)
     at java.lang.Thread.sleep(Thread.java:313)
     at com.wd.ru.rx.RxJavaTest$23.subscribe(RxJavaTest.java:455)
     at io.reactivex.internal.operators.observable.ObservableCreate.subscribeActual(ObservableCreate.java:40)
     at io.reactivex.Observable.subscribe(Observable.java:10910) 
     at io.reactivex.internal.operators.observable.ObservableSubscribeOn$SubscribeTask.run(ObservableSubscribeOn.java:96) 
     at io.reactivex.Scheduler$DisposeTask.run(Scheduler.java:452) 
     at io.reactivex.internal.schedulers.ScheduledRunnable.run(ScheduledRunnable.java:61) 
     at io.reactivex.internal.schedulers.ScheduledRunnable.call(ScheduledRunnable.java:52) 
     at java.util.concurrent.FutureTask.run(FutureTask.java:237) 
     at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:272) 
     at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133) 
     at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607) 
     at java.lang.Thread.run(Thread.java:761) 
```
**`建议看下 给初学者的RxJava2.0教程(四) 评论，我说的也不清楚，一脸懵逼。。。`**

---
#### 3.实践
还是之前获取北京和上海的天气的例子，我们同时获取两个城市的天气，然后拿到两份数据进行其他操作。
```java
//上海
Observable<WeatherBean> observable1 = RetrofitManager.getInstance().getApiServices(context).queryCityWeatherRX1("上海").subscribeOn(Schedulers.io());

//北京
Observable<WeatherBean> observable2 = RetrofitManager.getInstance().getApiServices(context).queryCityWeatherRX1("北京").subscribeOn(Schedulers.io());

//zip操作
Observable.zip(observable1, observable2, new BiFunction<WeatherBean, WeatherBean, String>() {
       @Override
       public String apply(WeatherBean weatherBean, WeatherBean weatherBean2) throws Exception {
           String c1 = weatherBean.getData().getCity();
           String c2 = weatherBean2.getData().getCity();
           return c1+c2;
       }
        }).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.i(TAG, "获取最后的数据: "+s);
            }
        });

//日志结果
I/RxJava: 获取最后的数据: 上海北京
```
这个例子比较适合那种需要同时请求接口，但是互不依赖的两组数据，正好第三方都需要他们。






