---
title: RxJava系列 - (5)Backpressure & Flowable
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

### 1. Backpressure 

**Backpressure**叫做什么背压式，我还以为是按摩呢。
先不说背压什么的，我们看下由**zip操作符**延伸来的问题

- **zip延伸问题**
我们学习zip的时候，可以看到他可以将多个上游发送的事件组合起来，然后在形成新的事件发送给下游。那这样就会产生各种各样的问题，比如：**上游水管A发送事件的速度很快，而水管B发送事件的速度很慢，**假如水管A发送了100个事件，而水管B才发送了1个事件，组合之后，水管A还剩下了99个已发送的事件，**那这些事件去哪里呢？不可能平白消失吧？**

- **水缸存储事件**
当然不会消失，zip给每一个上游的水管都准备了一个大水缸，用来储存这些还来不及进行组合的事件。我们用水管图表示就是这样的：
![Alt text](https://i.loli.net/2019/10/30/SuBF4zjgNWxq9HA.png)

上面那个蓝色的正方形就是水缸，上游水管发送的事件都放到里面，然后水缸之间的事件一一组合，当某个水缸空了的时候就处于等待的状态。

这个水缸有个特点：**事件是按照顺序保存的，遵循先进先出的原则，和队列是一样的，Zip的内部源码就是如此实现的。**

- **水缸漏水**
再大的水缸也会有装满的时候。
上游如果无限制的发送事件，水缸里总会被事件填满知道溢出，我们可以从这个教程中看到漏水事件是什么个情况：
[给初学者的RxJava2.0教程(五)](http://www.jianshu.com/p/0f2d6c2387c9)

---
### 2. 使用Observable解决上下游流速不均问题
先看一个例子：无限循环发送事件，导致内存溢出。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {  //无限循环发送事件
                    emitter.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io())
               .observeOn(AndroidSchedulers.mainThread())
               .subscribe(new Consumer<Integer>() {
     @Override
     public void accept(Integer integer) throws Exception {
     Log.d(TAG, "" + integer);
   }
});
```
- **方案1：使用filter进行过滤，减少下游接收事件数量**
上面的例子中，上游无限发送事件，下游不管是不是自己需要的，统统的装进了自己的水缸，导致内存溢出了，解决这种情况的一种方式就是：只选择符合自己条件的事件装进水缸，其他的事件忽略不计。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io())
                .filter(new Predicate<Integer>() {
                    @Override
                    public boolean test(Integer integer) throws Exception {
                        return integer % 10 == 0;//只要能被10整除的事件
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.d(TAG, "" + integer);
                    }
                });
```
这里使用了filter函数过滤了不能被10整除的事件，乍一看好像满足了我们的需求。这种通过过滤减少放进水缸数量的方式虽然满足少部分情景，但存在一个问题:**丢失了大部分的事件**，那你也许会像，下游不行，那从上游入手也可以呀！

- **方案2：减缓上游发送事件的速度**
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                    Thread.sleep(2000);  //每次发送完事件延时2秒
                }
            }
      }).subscribeOn(Schedulers.io())
           .observeOn(AndroidSchedulers.mainThread())
           .subscribe(new Consumer<Integer>() {
          @Override
          public void accept(Integer integer) throws Exception {
          Log.d(TAG, "" + integer);
    }
});
```
这里每发送一个事件，延时2s再发送下一个事件，虽然数量没变，但是给了下游足够的时间来从水缸中取出事件进行处理，不至于导致水缸爆满的情况。

---

### 3. 使用Flowable解决上下游流速不均问题
- **首先了解一下Observable和Flowable使用时的区别**
> Observable  对应  Observer ，  Flowable  对应  Subscriber

```java
Flowable<Integer> upstream = Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "emit 1");
                emitter.onNext(1);
                Log.d(TAG, "emit 2");
                emitter.onNext(2);
                Log.d(TAG, "emit 3");
                emitter.onNext(3);
                Log.d(TAG, "emit complete");
                emitter.onComplete();
            }
        }, BackpressureStrategy.ERROR); //增加了一个参数

        Subscriber<Integer> downstream = new Subscriber<Integer>() {

            @Override
            public void onSubscribe(Subscription s) {
                Log.d(TAG, "onSubscribe");
                s.request(Long.MAX_VALUE);  //注意这句代码
            }

            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "onNext: " + integer);
            }
            @Override
            public void onError(Throwable t) {
                 Log.w(TAG, "onError: ", t);
            }
            @Override
            public void onComplete() {
                Log.d(TAG, "onComplete");
            }
        };
        upstream.subscribe(downstream);
```
上面的代码创建了一个上游Flowable和一个下游Subscriber，与Observable的不同之处在于Flowable多了一个参数：**BackpressureStrategy.ERROR**，这个参数指的是：**FLowable选择的背压策略，这里使用ERROR策略，即当上下游流速不均时，直接抛出MissingBackpressureException异常**，其他策略后面继续讲解。

另一个区别是：下游的**onSubscribe**方法中，传递的不再是`Disposable`，而是`Subscription`，而两者的区别在于：
> 1. Disposbale.dispose()方法切断水管，Subscription.cancel()切断水管；
> 2. Subscription新增一个方法 void request(long n)，该方法表示的是：`下游每次能处理多少个事件`

- **void request(long n)的简单说明和重要性**
request(long n)方法表示的是下游每次能处理多少个事件，不设置是表示没有处理能力，不管是同步订阅还是异步订阅，request(long n)的效力都一样，下面分几种情况看下request(long n)对事件的影响。
> `Flowable采用的是响应式拉取来处理事件，下游通过调用request()方法来设置从上游拉取事件的数量，没有处理能力时就不去拉取事件进行处理。`
> 
> **1. 不调用request(long n)方法**
> ```java
> new Subscriber<Integer>() {
            @Override
            public void onSubscribe(Subscription s) {}
> ```
> `1. 同步订阅：下游没有处理能力，不能处理上游发送的任何事件，此时会直接抛出MissingBackpressureException异常；`
> 原因：上游发送了事件，下游又没有能力处理，那总不能一直等待，如果是在主线程那就会卡主页面，所以就直接抛出异常，提醒我们要进行request的调用。
> 
> `2. 异步订阅：下游没有处理能力，不能处理上游发送的任何事件，但是上游会正常发送完不超过128个事件，也不会抛出异常，如果超过128个，则直接抛出MissingBackpressureException异常。`
> 原因：上游能发送不超过128个事件却不抛出异常是因为Flowable中默认设置水缸大小为128，当上下游不在同一个线程中时，上游发送的事件会先保存到水缸中，下游虽然没有调用request方法，但是并不影响默认水缸存储事件，但是下游不处理事件式，超过了128个事件，水缸已经满了就会抛出异常。

### 4. 背压策略
上面说的只是在Flowable使用`BackpressureStrategy.ERROR`策略时出现的情况，其实BackpressureStrategy还有其他几种情况可供选择，具体请查看：RxJava-Observables之间的区别：4. 背压(BackpressureStrategy)策略之间的区别



