---
title: RxJava系列 - 初步认识(1)
copyright: true
date: 2019-10-30 16:32:00
tags:
categories:
	RxJava系列
---

- **RxJava系列 - 初步认识(1)**
- **RxJava系列 - 线程控制(2)**
- **RxJava系列 - 变换符操作(3)**
- **RxJava系列 - zip操作符(4)**
- **RxJava系列 - Backpressure & Flowable(5)**

---

#### 1.基本概念
- **RxJava的观察者模式**
> RxJava有四个基本概念：**Observable** (可观察者，即被观察者)、**Observer** (观察者)、**subscribe** (订阅)、事件。
> **Observable** 和 **Observer** 通过 **subscribe()**方法实现订阅，从而 **Observable** 可以在需要的时候发出事件了通知**Observer**。

- **回调事件 : onCompleted()、onError()**
与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 **onNext()** （相当于 onClick() / onEvent()）之外，还定义了两个特殊的事件：**onCompleted()** 和 **onError()**。
>**onCompleted() : **:事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 **onNext()** 发出时，需要触发 **onCompleted()** 方法作为标志。
> 
>**onError() : ** 事件队列异常。在事件处理过程中出异常时，**onError()** 会被触发，同时队列自动终止，不允许再有事件发出。
> 
>在一个正确运行的事件序列中, **onCompleted()** 和 **onError()** 有且只有一个，并且是事件序列中的最后一个。需要注意的是，**onCompleted()** 和 **onError()** 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。


---

<!-- more -->

#### 2.基本实现
基于以上的概念， RxJava 的基本实现主要有三点：

- **创建Observer**
Observer 即观察者，它决定事件触发的时候将有怎样的行为。 RxJava 中的 Observer 接口的实现方式：
```java
Observer<String> observer = new Observer<String>() {
        @Override
        public void onSubscribe(@NonNull Disposable d) {

        }

        @Override
        public void onNext(@NonNull String s) {

        }

        @Override
        public void onError(@NonNull Throwable e) {

        }

        @Override
        public void onComplete() {

        }
    };
```

	除了 Observer 接口之外，RxJava 还内置了一个实现了 Observer 的抽象类：Subscriber。 Subscriber 对 Observer 接口进行了一些扩展，但他们的基本使用方式是完全一样的：
```java
Subscriber<String> subscriber = new Subscriber<String>() {
        @Override
        public void onSubscribe(Subscription s) {

        }

        @Override
        public void onNext(String s) {

        }

        @Override
        public void onError(Throwable t) {

        }

        @Override
        public void onComplete() {

        }
    };
```

- **创建Observable**
**Observable** 即被观察者，它决定什么时候触发事件以及触发怎样的事件。 RxJava 使用 **create()** 方法来创建一个 **Observable** ，并为它定义事件触发规则：
```java
Observable observable = Observable.create(new ObservableOnSubscribe() {
        @Override
        public void subscribe(@NonNull ObservableEmitter e) throws Exception {
            e.onNext("");
            e.onNext("");
            e.onComplete();
        }
    });
```
除了最基本的创建事件的方法**create()**之外，RxJava还提供了一些快捷的方法来创建事件队列，例如：
1. **just(T...) **将传入的参数一次发送出去
```java
 /**
 * 将会依次调用：onNext("A");onNext("B");onNext("C");
 */
 Observable<String> observable1 = Observable.just("A","B","C");
```
2. **fromArray(T) / Observable.fromIterable()** 将传入的数组或 Iterable 拆分成具体对象后，依次发送出去
```
 /**
 * 将会依次调用：onNext("A");onNext("B");onNext("C");
 */
private String[] s = {"A","B","C"};
Observable<String> observable2 = Observable.fromArray(s);

//list集合中迭代处理
List<String> list = new LinkedList<String>();
Observable<String> observable3 = Observable.fromIterable(list);
```

- **Subscribe订阅**
创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了。代码形式很简单：
```java
observable.subscribe(observer);
```

---
#### 3.简单易懂的实现方式
上面的概念太难理解了，这里我们给个生动的比喻，然后再来一个通俗易懂的示例加深理解。

##### 1. 图示与代码
- **图示  ： 借用水管的概念来替代观察者和被观察者，以及事件的流向来阐述RxJava的原理**
> ![水管事件流向图](https://i.loli.net/2019/10/30/PDmV4urInpeTUci.png)
> 
> 我们依据上图的水管来描述一下事件：
> 1. 上面的水管为事件产生的水管，我们称之为**上游**，下面的水管为事件接收的水管，我们称之为**下游**
> 2. 上游、下游通过一定的方式连接，上游按照**1、2、3的顺序**发送事件，下游按照**1、2、3的顺序**接收事件(和官网的相反)
> 3. 上游、下游分别对应RxJava中的**Observable**和**Observer**,他们之间的连接就对应着**subscribe()**

-  **代码 ： 表示RxJava的关系如下：**
```java
//1. 创建一个水管上游
    Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
            e.onNext("第一个事件");
            e.onNext("第二个事件");
            e.onNext("第三个事件");
            //事件结束
            e.onComplete();
        }
    });

 //2. 创建一个下游
    Observer<String> observer = new Observer<String>() {
        @Override
        public void onSubscribe(@NonNull Disposable d) {
            Log.d(TAG,"onSubscribe Disposable : "+d);
        }

        @Override
        public void onNext(@NonNull String s) {
            Log.d(TAG,"onNext  s :"+s);
        }

        @Override
        public void onError(@NonNull Throwable e) {
            Log.d(TAG,"onError");
        }

        @Override
        public void onComplete() {
            Log.d(TAG,"onComplete --- 事件结束");
        }
    };

 //3. 建立连接
    public void startRx(){
    //只有上游和下游建立了连接之后才会开始发送事件，即调用了subscribe()方法
        observable.subscribe(observer);
    }

//4. 运行结果
	D/RxJava: onSubscribe Disposable : null
	D/RxJava: onNext  s :第一个事件
	D/RxJava: onNext  s :第二个事件
	D/RxJava: onNext  s :第三个事件
	D/RxJava: onComplete --- 事件结束
```
是不是觉得和其他的调用方式没什么区别呢？那我们来看下更灵活的调用方式。

---
##### 2. 链式调度
- **RxJava的链式调度**
```java
Observable.create(new ObservableOnSubscribe<String>(){
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("链式调度1");
                e.onNext("链式调度2");
                e.onNext("链式调度3");
                e.onComplete();
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d(TAG,"onSubscribe Disposable : "+d);
            }

            @Override
            public void onNext(@NonNull String s) {
                Log.d(TAG,"onNext  s : "+s);
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.d(TAG,"onError");
            }

            @Override
            public void onComplete() {
                Log.d(TAG,"onComplete");
            }
        });

//运行结果
	D/RxJava: onSubscribe Disposable : null
	D/RxJava: onNext  s : 链式调度1
	D/RxJava: onNext  s : 链式调度2
	D/RxJava: onNext  s : 链式调度3
	D/RxJava: onComplete
```
从链式调度来看简洁明了，比较符合现在的编码习惯，我们来看下这一连串的代码中比较陌生的部分：
**`ObservableEmitter`** 和 **`Disposable `**.

##### 3.陌生对象
- **(1) ObservableEmitter**
> Emitter是发射的意思，所以来看这个应该就是指的：**事件发出**，他有三种不同的事件类型：**onNext(T value)、onComplete()、onError(Throwable error)**，分别指的就是next事件、complete事件和error事件。
> 
> 这些事件并不是毫无规则、随意发射的，需要满足一定的规则：
> -  上游可以发送无限个onNext，下游可以接收无限个onNext。
> - 当上游发送了一个onComplete事件后，上游处于onComplete之后的事件还会继续发送；而下游在接收到onComplete事件之后，将不再继续接收其他事件。
> - 当上游发送了一个onError事件后，上游处于onError之后的事件还会继续发送；而下游在接收到onError事件之后，将不再继续接收其他事件。
> - 上游可以不发送onComplete 或 onError 事件
> - **重点：onComplete事件和onError事件必须唯一且互斥，**即不能发送多个onComplete事件，也不能发送多个onError事件，也不能先发一个onComplete事件，再发送一个onError事件，反之亦然。
> 
> > 注: 关于onComplete和onError唯一并且互斥这一点, 是需要自行在代码中进行控制, 如果你的代码逻辑中违背了这个规则, 并不一定会导致程序崩溃. 比如发送多个onComplete是可以正常运行的, 依然是收到第一个onComplete就不再接收了, 但若是发送多个onError, 则收到第二个onError事件会导致程序会崩溃.

- **事件发送规则示意图** 

 **onNext事件:**
 ![onNext事件](https://i.loli.net/2019/10/30/zctx2MJFVdAYXvB.png)
 
 **onComplete事件:**
 ![onComplete事件](https://i.loli.net/2019/10/30/Cti3mXU572hcSOK.png) 
 
 **onError事件:**
 ![onError事件](https://i.loli.net/2019/10/30/3z4BvWkJE5YhSdZ.png)

- **(2) Disposable**
> Disposable意思为一次性，放入到RxJava中来理解这个的话，可以是当调用了**dispose()**方法之后，上游与下游之间的连接就中断了，从而导致下游收不到事件，**但是并不会阻止上游继续发送剩余的事件**。

	下面我们就来看下上游持续发送事件，下游在收到**1**的标识之后中断水管间的连接：
```java
Observable.create(new ObservableOnSubscribe<String>(){
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("链式调度1");
                e.onNext("链式调度2");
                Log.d(TAG,"subscribe  下面要终止事件了。。。");
                e.onNext("1");
                Log.d(TAG,"subscribe  上面已经终止事件，但是我还要继续。。。");
                e.onNext("链式调度3");
                e.onComplete();
            }
        }).subscribe(new Observer<String>() {
            private Disposable disposable;
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d(TAG,"onSubscribe Disposable : "+d);
                disposable = d;

            }

            @Override
            public void onNext(@NonNull String s) {
                Log.d(TAG,"onNext  s : "+s);
                if(s.equals("1")){
                    disposable.dispose();
                }
                Log.d(TAG,"onNext  disposable.isDisposed : "+disposable.isDisposed());
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.d(TAG,"onError");
            }

            @Override
            public void onComplete() {
                Log.d(TAG,"onComplete");
            }
        });
    }

//运行结果
D/RxJava: onSubscribe Disposable : null
D/RxJava: onNext  s : 链式调度1
D/RxJava: onNext  disposable.isDisposed : false
D/RxJava: onNext  s : 链式调度2
D/RxJava: onNext  disposable.isDisposed : false
D/RxJava: subscribe  下面要终止事件了。。。
D/RxJava: onNext  s : 1
D/RxJava: onNext  disposable.isDisposed : true
D/RxJava: subscribe  上面已经终止事件，但是我还要继续。。。
```
> 从上面的日志我们可以看到，在收到标识为**1**后，我们中断了水管，isDisposed 这个标志的返回值编变成了：**disposable.isDisposed : true**，表明事件已经被中断，下游将不再接收任何事件。
> 
> 但是我们可以看到这句：**上面已经终止事件，但是我还要继续。。。**，在水管中断了之后，上游还是继续发送了剩余的事件，从而可以看出：**调用 disposable.dispose()方法后，上游会继续发送完剩余的事件，但是下游并不会接收中断后的事件。**
> 
> 这个以水管来说就是：一根正在流水的管子，从中间堵住，上面的水还是会继续的向下流，只不过是堵起来了；但是下游不会有水流下来了。


##### 4.subscribe的重载方法
```java
public final void subscribe(Observer<? super T> observer) {}
 
public final Disposable subscribe() {}

public final Disposable subscribe(Consumer<? super T> onNext) {}

public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {} 

public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete) {}

public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete, Consumer<? super Disposable> onSubscribe) {}
```

- (1) 带有`Observer`参数的`subscribe()`使用方法时可以参考上面的处理；
- (2) 不带任何参数的`subscribe()`方法，表示我不关心上游的任何事件，不管是数据还是什么都不会去理会；
- (3) 带有一个参数的`subscribe()`方法，表示下游只关心上游的onNext事件，其他事件就当做没看见，随便你们在处理；


---


#### 4.重点推荐：RxJava2.0的系列文章
这是一个关于RxJava2.0的系列教程，很有学习的价值，我的例子也是参考而来，重点推荐大家看下。
[给初学者的RxJava2.0教程](http://www.jianshu.com/p/464fa025229e)
