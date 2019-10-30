---
title: RxJava系列 - Obervables之间的区别
copyright: true
date: 2019-06-08 22:45:10
tags:
- Observable
categories:
- RxJava系列
---

### 1. 被观察者之间之间的区别

| 类型      |    描述  |   
| :--------: | :-----------------|
| Observable<泛型>|  能发送0个或n个onNext事件，最终以onComplete或onError终止事件。 |  
| Flowable<泛型>| 能发送0个或n个onNext事件，最终以onComplete或onError事件终止，并且支持背压，可以控制数据源发送的速度。 |
| Single<泛型>| 只能发送单个onSuccess或onError事件同时终止事件。 |
| Maybe<泛型>|  能发送0个或1个数据，要么是onSuccess，要么是onError，同时终止事件，不会同时执行onComplete。|
| Completable|  从来不发送数据，只处理onError或onComplete事件同时终止事件，不会同时执行onError和onComplete。|


<!-- more -->

**1. Observable<泛型>**
> 1. 可以发送0或n个数据，只要发送了onComplete或onError事件，就会停止发送数据。
```java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("A");
        emitter.onNext("B");
        //发送异常不会导致奔溃，下游会在onError中收到异常消息
        emitter.onError(new Throwable("Error"));
        emitter.onComplete();
    }
}).subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onNext(String s) {
        Log.d(TAG, "rxObservable onNext ： " + s);
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "rxObservable onError ： " + e.toString());
    }
    @Override
    public void onComplete() {
        Log.d(TAG, "rxObservable onComplete ： ");
    }
});
```

**2. Flowable<泛型>**
> 1. 可以发送0或n个数据，只要发送了onComplete或onError事件，就会停止发送数据。
> 2. 可以在观察者的onSubscribe方法中通过`Subscription.request(n)`来设置请求发送事件的数量。
> 3. 可以通过使用背压(BackpressureStrategy)来解决上游发送事件的速度和下游处理事件的速度不同导致的问题。
```java
Flowable.create(new FlowableOnSubscribe<String>() {
    @Override
    public void subscribe(FlowableEmitter<String> emitter) throws Exception {
        emitter.onNext("A");
        emitter.onNext("B");
        //发送异常不会导致奔溃，下游会在onError中收到异常消息
        emitter.onError(new Throwable("Error"));
        emitter.onComplete();
    }
}, BackpressureStrategy.BUFFER)
        .subscribe(new FlowableSubscriber<String>() {
            @Override
            public void onSubscribe(Subscription s) {
	            //设置请求发送事件的数量
                s.request(Long.MAX_VALUE);
            }
            @Override
            public void onNext(String s) {
                Log.d(TAG, "rxFlowable onNext ： "+s);
            }
            @Override
            public void onError(Throwable t) {
                Log.d(TAG, "rxFlowable onError ： "+t.toString());
            }
            @Override
            public void onComplete() {
                Log.d(TAG, "rxFlowable onComplete ： ");
            }
        });
```

**3. Single<泛型>**
> 1. 只能发送通过onSuccess或onError发送单个事件，即使有多个事件，也只会发送第一个，并且发送之后订阅终止。
> 
> 2. onSuccess和onError不能同时发送时，否则会出现如下异常，导致程序奔溃：`The exception could not be delivered to the consumer because it has already canceled/disposed the flow or the exception has nowhere to go to begin with.`
```java
Single.create(new SingleOnSubscribe<String>() {
    @Override
    public void subscribe(SingleEmitter<String> emitter) throws Exception {
	    //下游之后收到第一个onSuccess发送的数据
        emitter.onSuccess("Success");
        emitter.onSuccess("Success2");
        //emitter.onError(new Throwable("Error"));

    }
}).subscribe(new SingleObserver<String>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onSuccess(String s) {
        Log.d(TAG, "rxSingle onSuccess ： " + s);
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "rxSingle onError ： " + e.toString());
    }
});
```

**4. Maybe<泛型>**
> 1. 通过onSuccess发送多个事件时，下游只会收到第一个事件，后面的事件全部被抛弃，并且不会同时发送onComplete事件。
> 
> 2. onSuccess和onError不能同时发送时，否则会出现如下异常，导致程序奔溃：`The exception could not be delivered to the consumer because it has already canceled/disposed the flow or the exception has nowhere to go to begin with.`
```java
Maybe.create(new MaybeOnSubscribe<String>() {
    @Override
    public void subscribe(MaybeEmitter<String> emitter) throws Exception {
        emitter.onSuccess("Success");
        emitter.onSuccess("Success2");
		//emitter.onError(new Throwable("Error"));
        emitter.onComplete();
    }
}).subscribe(new MaybeObserver<String>() {
    @Override
    public void onSubscribe(Disposable d) {
    }

    @Override
    public void onSuccess(String s) {
        Log.d(TAG, "rxMaybe onSuccess ： " + s);
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "rxMaybe onError ： " + e.toString());
    }
    @Override
    public void onComplete() {
        Log.d(TAG, "rxMaybe onComplete ： ");
    }
});
```

**5. Completable**
> 1. 只能处理onError或者onComplete发送的事件，并且不会同时发送，一个事件发送之后，订阅关系终止。
```java
Completable.create(new CompletableOnSubscribe() {
    @Override
    public void subscribe(CompletableEmitter emitter) throws Exception {
        emitter.onError(new Throwable("Error"));
        emitter.onComplete();
    }
    }).subscribe(new CompletableObserver() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onComplete() {
        Log.d(TAG, "rxCompletable onComplete ： ");
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "rxCompletable onError ： " + e.toString());
    }
});
```
---

### 2. 观察者之间的区别

| 类型      |    方法  |   
| :--------: | :--------|
|Observer<泛型>|onSubscribe(Disposable d)、onNext(T t)、onError(Throwable e)、onComplete()|
|FlowableSubscriber<泛型>|onSubscribe(Subscription s)、onNext(T t)、onError(Throwable e)、onComplete()，继承自Subscriber|
|SingleObserver<泛型>|onSubscribe(Disposable d)、onSuccess(T t)、onError(Throwable e)|
|MaybeObserver<泛型>|onSubscribe(Disposable d)、onSuccess(T t)、onError(Throwable e)、onComplete()|
|CompletableObserver|onSubscribe(Disposable d)、onError(Throwable e)、onComplete()|

#### 1. 方法说明
**1. onSubscribe(Disposable d)、 onSubscribe(Subscription s)**
> 观察者和被观察者建立订阅关系时会最先调用该方法。
> 
> **Disposable.dispose()：**表示主动断开上下游的订阅关系。
> **Disposable.isDisposed()：**判断当前上下游之间的订阅关系是否已经断开。
> 
> **Subscription .request(Long.MAX_VALUE)：**设置观察者能处理事件的数量，如果设置为0，下游将收不到任何onNext事件，即使上游发送了很多。
> **Subscription .cancel()：**表示主动断开上下游的订阅关系。

**2. onNext(T t)、onSuccess(T t)、onError(Throwable e)**
> 处理上游对应onNext、onSuccess、onError事件发送的结果。

---

### 3. 发射器Emitter之间的区别

| 类型      | 继承 |    方法 |
| :--------: | :--------:|:--------|
|Emitter|父类|onNext(T t)、onError(Throwable e)、onComplete()|
|ObservableEmitter<泛型>|是|setDisposable(Disposable d)、setCancellable(Cancellable c)、isDisposed()、tryOnError(Throwable t)|
|FlowableEmitter<泛型>|是|setDisposable(Disposable d)、setCancellable(Cancellable c)、isCancelled()、tryOnError(Throwable t)、long requested()|
|SingleEmitter<泛型>|否|setDisposable(Disposable d)、setCancellable(Cancellable c)、isDisposed()、tryOnError(Throwable t)、 onSuccess(T t)、onError(Throwable t)|
|MaybeEmitter<泛型>|否|setDisposable(Disposable d)、setCancellable(Cancellable c)、isDisposed()、tryOnError(Throwable t)、 onSuccess(T t)、onError(Throwable t)、onComplete()|
|CompletableEmitter|否|setDisposable(Disposable d)、setCancellable(Cancellable c)、isDisposed()、tryOnError(Throwable t)、onError(Throwable t)、onComplete()|

#### 1. 方法说明
**1. Emitter的onNext(T t)、onError(Throwable e)、onComplete()**
> ObservableEmitter和FlowableEmitter都是继承自Emitter，通过以上方法进行数据的发送。

**2. isDisposed()、isCancelled()**
> 都是用于判断当前上下游之间是否已经取消订阅关系，isCancelle()是FlowableEmitter独有的。

**3. FlowableEmitter 的 long requested()**
> 用于获取下游设置的事件处理数量。
> 
> **注意：**
> **1. 如果异步订阅，即上下游不在同一个线程，上游emitter.requested()获取数量默认是128(RxJava内部设置的默认值就是128)，并不是下游Subscription.request(n)设置的数量。**
> 
> **2. 如果是同步订阅，上游获取的emitter.requested()等于下游Subscription.request(n)设置的数量，并且上游每发送一个事件、下游接收到一个事件，emitter.requested()获取的数据都会同步更新。**

---

### 4. 背压(BackpressureStrategy)策略之间的区别
> **背压是基于异步线程的。**

| 类型      |     描述|
| :-------- | :--------|
|MISSING|相当于没有使用背压策略。当上游发送事件数量大于缓存区大小时(默认128)，将会提示异常并在下游onError事件中接收到异常提醒：io.reactivex.exceptions.MissingBackpressureException: Queue is full?!可以在后面继续使用类似于onBackpressureBuffer()、onBackpressureLatest()、onBackpressureDrop()的策略。|
|ERROR|当上游发送事件数量大于缓存区大小时(默认128)，将会抛出异常并在下游onError事件中接收到异常：io.reactivex.exceptions.MissingBackpressureException: create: could not emit value due to lack of requests|
|BUFFER	|相当于将缓存区设置为无限大，不管上游发送多少数据都不会抛出异常，但是要注意可能会因为事件数量过多导致OOM|
|DROP|将超过默认缓存区(128)大小的事件直接抛弃，比如上游发送了160个事件，最终保存的只有1-128,129往后的都被抛弃了。|
|LATEST|将超过默认缓存区(128)大小的事件直接抛弃，但是会保存最后一个事件，比如上游发送了160个事件，最终保存的只有1-128 + 160,129-159都被抛弃了。|

**1. MISSING - - 相当于没有使用背压策略**
> 比如：上游一起发送了160个事件，下游每次接收70个事件，当上游发送事件数量大于128时，直接抛出`io.reactivex.exceptions.MissingBackpressureException: Queue is full?!`异常，回调到下游的onError，下游将接收不到前面发送的其他数据。
> 
> **输出结果：**
> 上游发送事件：0 -- 159 共计160个事件，超出缓存区大小，抛出异常。
> ![miss.png](https://i.loli.net/2019/06/08/5cfbcd1fc505d32980.png)



**2. ERROR**
> 比如：上游一起发送了160个事件，下游每次接收70个事件，当上游发送事件数量大于128时，直接抛出`io.reactivex.exceptions.MissingBackpressureException: create: could not emit value due to lack of requests`异常，回调到下游的onError，下游将接收不到前面发送的其他数据。
> 
> **输出结果：**
> 上游发送事件：0 -- 159 共计160个事件，超出缓存区大小，抛出异常。
> ![err.png](https://i.loli.net/2019/06/08/5cfbcd68364ef96978.png)


**3. BUFFER - -  onBackpressureBuffer()**
> 比如：上游一起发送了160个事件，下游每次接收70个事件，到第三次接收时，160个事件全部都被下游接收完成。
> 
> **输出结果：**
> 上游发送事件：0 -- 159 共计160个事件，
> 第一次请求70个事件：输出0 -- 69，
> 第二次请求70个事件：输出 70 -- 139，
> 第二次请求70个事件：输出 140 -- 159，onComplete结束订阅关系。
> ![buffer.png](https://i.loli.net/2019/06/08/5cfbcd1e4e42472009.png)


**4. DROP - - onBackpressureDrop()**
> 比如：上游一起发送了160个事件，下游每次接收70个事件，会发现第二次接收之后总共只有128个事件，即使再次要求接收事件，也不会收到任何数据，这说明上游发送的160个事件中，超过了128的事件都被直接抛弃了。
> 
> **输出结果：**
> 上游发送事件：0 -- 159 共计160个事件，
> 第一次请求70个事件：输出0 -- 69，
> 第二次请求70个事件：输出 70 -- 127 ， onComplete结束订阅关系。
> ![drop.png](https://i.loli.net/2019/06/08/5cfbcd1eb85e074384.png)



**5. LATEST - - onBackpressureLatest()**
> 比如：上游一起发送了160个事件，下游每次接收70个事件，会发现第二次接收之后总共只有129个事件，即使再次要求接收事件，也不会收到任何数据，这说明上游发送的160个事件中，第1-128和第160个事件被保留了，其余的129-159之间的事件都被抛弃了。
> 
> **输出结果：**
> 上游发送事件：0 -- 159 共计160个事件，
> 第一次请求70个事件：输出0 -- 69，
> 第二次请求70个事件：输出 70 -- 127、159共计129个事件， onComplete结束订阅关系。
> ![latest.png](https://i.loli.net/2019/06/08/5cfbcd1f72b5748052.png)



**6. 示例**
> **上游：**一次性发送160个事件。 **下游：**每次请求70个事件。
```java
private static Subscription subscription;
//点击事件，每次请求发送70个事件
public static void requestRx() {
    subscription.request(70);
}
public static void rxFlowable() {
    Flowable.create(new FlowableOnSubscribe<Integer>() {
        @Override
        public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
	        //上游一次性发送160个事件
            while (a < 160) {
                Log.d(TAG, "下游要求事件数量 : " + emitter.requested());
                Log.d(TAG, "上游发送数据 : " + a);
                emitter.onNext(a);
                a++;
            }
            emitter.onComplete();
        }
        //设置不同的背压处理方式
    }, BackpressureStrategy.DROP)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new FlowableSubscriber<Integer>() {
                @Override
                public void onSubscribe(Subscription s) {
                //外部定义Subscription，方便请求事件
                    subscription = s;
                }
                @Override
                public void onNext(Integer s) {
                    Log.d(TAG, "下游rxFlowable onNext ： " + s);
                }
                @Override
                public void onError(Throwable t) {
                    a = 0;
                    Log.d(TAG, "下游rxFlowable onError ： " + t.toString());
                }
                @Override
                public void onComplete() {
                    a = 0;
                    Log.d(TAG, "下游rxFlowable onComplete ： ");
                }
            });
    
}
```

---

### 5. 线程调度

| 类型      |     描述 |
| :-------- | :--------|
|Schedulers.io()| 用于IO密集型操作，比如网络请求、读取SD卡文件、查询数据库等，具有线程缓存机制，在此调度器接收到任务后，先检查线程池中是否有空闲的线程，如果有则复用，如果没有则创建新的线程，并加入到线程池，如果每次都没有空闲的线程，则会无上限的创建线程。|
|Schedulers.computation()|用于CPU密集型计算任务，即不会被IO等操作限制性能的耗时操作，比如xml、json文件解析、Bitmap图像压缩等，具有固定的线程池，大小为CPU的核数。不可用于IO操作，因为IO操作的等待时间会浪费CPU。|
|Schedulers.newThread()|在每执行一个任务时创建一个新的线程，不具有缓存机制。新创建一个线程比复用一个线程更耗时，虽然使用io()的地方都可以使用newThread()，但是newThread()的效率没有io()高。|
|Schedulers.single()|拥有一个线程单例，所有的任务都在这一个线程中执行，如果该线程中有任务在执行，其他任务将会按照FIFO的原则依次执行。|
|Schedulers.trampoline()|在当前线程立即执行任务，如果当前线程有正在执行的任务，则将其暂停，直到新插入的任务执行完毕，再继续执行之前未完成的任务。|
|AndroidSchedulers.mainThread()|在Android UI线程中执行任务，专为Android开发定制。|

**1. 线程切换**
> **subscribeOn(@NonNull Scheduler scheduler)**
> 事件产生：指定上游Observable发送数据、订阅时所处的线程，多次调用只有第一次生效。
> 
> **observeOn(@NonNull Scheduler scheduler)**
> 事件消费：指定下游Observer接受数据时所处的线程，多次调用都会生效，最终以最后一次调用为主。

**2. doOnSubscribe(Consumer<? super Subscription> onSubscribe)**
>  doOnSubscribe()与onStart()类似，在subscribe()调用之后，事件发送之前被调用，但是doOnSubscribe()可以指定线程。
>  
>  默认情况下，doOnSubscribe发生在subscribe()所在线程，而如果在 doOnSubscribe() 之后有 subscribeOn() 的话，它将执行在离它最近的 subscribeOn() 所指定的线程。
>  
>  多个doOnSubscribe()&subscribeOn()组合在一起同时调用时，越靠后的越先被调用，但是如果是doOnSubscribe()&doOnSubscribe&subscribeOn()的方式调用，则越靠近subscribeOn()的越先调用

```java
Observable.create(onSubscribe)
    .subscribeOn(Schedulers.io())
    .doOnSubscribe(new Action0() {
        @Override
        public void call() {
	        //如果订阅开始时需要loading
            progressBar.setVisibility(View.VISIBLE); // 需要在主线程执行
        }
    })
    .subscribeOn(AndroidSchedulers.mainThread()) // 指定主线程
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(subscriber);
```
