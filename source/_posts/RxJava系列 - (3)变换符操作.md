---
title: RxJava系列 - (3)变换符操作
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

前面我们获取到了当前的天气，然后这一次的请求就算结束了。
但是当我们在实际操作中，往往有很多嵌套的操作，比如注册、登录逻辑，完成了上一次的请求紧接着根据返回值做下一次请求。
这里我们还是以天气为例，请求获取第一个城市的天气之后，再去请求获取第二个城市的天气。
#### 1.嵌套请求的一般写法
在一个请求成功的回调中调用另一个请求，从而达到嵌套请求的效果。
需求：在方法**`startRx8()`**中获取上海的天气，成功之后在**`onComplete()`**中调用**`startRx9()`**获取北京的天气。
```java
//方法startRx8()与startRx9()写法完全一致，只是一个请求城市为上海，一个为北京
public void startRx8(final Context context) {
RetrofitManager
    .getInstance()
    .getApiServices(context).queryCityWeatherRX1("上海")
    .subscribeOn(Schedulers.newThread())//io线程发起网络请求
    .observeOn(AndroidSchedulers.mainThread())//主线程处理结果
    .subscribe(new Observer<WeatherBean>() {
        @Override
        public void onSubscribe(Disposable d) {}
        @Override
        public void onNext(WeatherBean weatherBean2) {}
        @Override
        public void onError(Throwable e) {}
        @Override
        public void onComplete() {
            Log.i(TAG, "请求成功");
            startRx9(context);
        }
 });
}
```
很显然，在之前我们这样的请求方式可以说很常见，但是当我们学会使用RxJava中的变换操作符之后，将会发生巨大的变化。

---
#### 2.Map变换操作符
map是RxJava中最简单的一个变换操作符，它的作用是：**对上游发送的每一个事件应用一个函数Function，使得每个事件都能按照这个指定的函数去进行对象的变化**，用水管，模型来看就是：
![map](https://i.loli.net/2019/10/30/VkwHvUZsEX5mjJi.png)

map中的Function函数将上游的圆形事件变化为矩形事件，所以下游接收时获得的就是矩形事件，有点偷梁换柱的意思。
用简单的代码示例来看：
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
            //原始数据为int类型的100、200
                e.onNext(100);
                e.onNext(200);
            }
        }).map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) throws Exception {
            //Function中变换数据为String类型
                return "Change int to string : "+integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
            //请求完成之后收到结果
                Log.i(TAG, "请求结束 ： "+s);
            }
        });

//日志结果
I/RxJava: 请求结束 ： Change int to string : 100
I/RxJava: 请求结束 ： Change int to string : 200
```
从日志结果我们可以看到，本来是Integer类型的数据，通过map转换之后，编程了String类型的。map可以将上游发送的事件转换成任意的类型，只要你喜欢。

---
#### 3.FlatMap变换操作符
flatMap是一个相比map更加强大的转换操作符，如果map是一对一，那么flatMap就是多对更多。
**1.如何理解**
在[给初学者的RxJava2.0教程(三)](http://www.jianshu.com/p/128e662906af)的文章中是这样描述的：**FlatMap将一个发送事件的上游Observable变换为多个发送事件的Observables，然后将它们发射的事件合并后放进一个单独的Observable里.**

**2.自我理解**
可是经过自己的琢磨，我理解的就有别于上面的描述：**FlatMap将上游Observable发送的N个事件，按照Function中指定的数目M，将上游事件一一拆分为M个子事件，然后无序的整合到新的Observable中继续发送**

**3.水管模型解释**
虽然描述理解不同，但是管道这个模型图还是理解的很到位的。
![Alt text](https://i.loli.net/2019/10/30/RZtuVk7rMwyi3GH.png)

注意上游事件的颜色、颜色、颜色！！！
上游发送了1、2、3三个圆形事件，中间的flatMap是将一个圆形事件转换成一个三角形事件和一个矩形事件并组合成新的上游Observable。
再来分解看下转换的流程：
![Alt text](https://i.loli.net/2019/10/30/X71eYfZiJQSvwOE.png)

上游每发送一个事件, flatMap都将创建一个新的水管, 然后发送转换之后的新的事件, 下游接收到的就是这些新的水管发送的数据. 这里需要注意的是：**flatMap并不保证事件的顺序, 也就是图中所看到的, 并不是事件1就在事件2的前面**.

**4.代码中看流程**
还不能理解的我们就从代码中再琢磨一下。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                e.onNext(100);
                e.onNext(200);
                e.onNext(300);
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(Integer integer) throws Exception {
                final List<String> list = new ArrayList<>();
                //这里决定事件拆分个数 
                for (int i = 0; i < 2; i++) {
                    list.add("Change int to string : " + integer);
                }
                //生成新的上游Observable对象
                return  Observable.fromIterable(list);
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.i(TAG, "请求结束 ： "+s);
            }
        });

//日志结果
I/RxJava: 请求结束 ： Change int to string : 100
I/RxJava: 请求结束 ： Change int to string : 100
I/RxJava: 请求结束 ： Change int to string : 200
I/RxJava: 请求结束 ： Change int to string : 200
I/RxJava: 请求结束 ： Change int to string : 300
I/RxJava: 请求结束 ： Change int to string : 300
```
我们上面说flatMap并不保证事件顺序，但是从日志中看到的却是100、200、300这样按照顺序来的，明显和我们的预期不符啊。
不用着急，本质上flatMap是无序的，只是上面的日志恰好按顺序打印了，我们可以通过**`Observable.fromIterable(list).delay(1000, TimeUnit.MILLISECONDS);`**方法，延时一点可以比较有效的看到无序的事件发送结果(可是我还是没看到无序的日志，不知道什么gui)。

---
#### 4. concatMap变换操作符
concatMap和flatMap的不同就在于：**concatMap是有序的、有序的、有序的**，不管是否延时，都是有序的。
我们可以copy上面的同样的代码，将flatMap替换为concatMap，得到一个有序的日志结果。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                e.onNext(100);
                e.onNext(200);
                e.onNext(300);
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(Integer integer) throws Exception {
                final List<String> list = new ArrayList<>();
                //这里决定事件拆分个数 
                for (int i = 0; i < 2; i++) {
                    list.add("Change int to string : " + integer);
                }
                //生成新的上游Observable对象
                return  Observable.fromIterable(list);
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.i(TAG, "请求结束 ： "+s);
            }
        });

//日志结果
I/RxJava: 请求结束 ： Change int to string : 100
I/RxJava: 请求结束 ： Change int to string : 100
I/RxJava: 请求结束 ： Change int to string : 200
I/RxJava: 请求结束 ： Change int to string : 200
I/RxJava: 请求结束 ： Change int to string : 300
I/RxJava: 请求结束 ： Change int to string : 300
```

---
#### 5. 实际操作
还是上面的请求：获取到上海的天气之后，紧接着获取北京的天气情况。
```java
RetrofitManager.getInstance().getApiServices(context).queryCityWeatherRX1("上海")
                .subscribeOn(Schedulers.newThread())//io线程发起网络请求
        .observeOn(AndroidSchedulers.mainThread())//主线程处理结果
        .doOnNext(new Consumer<WeatherBean>() {
            @Override
            public void accept(WeatherBean weatherBean) throws Exception {
                Log.i(TAG, "上海getCity = " + weatherBean.getData().getCity());
            }
        })
        .observeOn(Schedulers.io())
        .flatMap(new Function<WeatherBean, ObservableSource<WeatherBean>>() {
            @Override
         public ObservableSource<WeatherBean> apply(WeatherBean weatherBean) throws Exception {
                Log.i(TAG, "转换中-------------------------------");
                Log.i(TAG, "getCity = " + weatherBean.getData().getCity());
                Log.i(TAG, "转换中-------------------------------");
      return RetrofitManager.getInstance().getApiServices(context).queryCityWeatherRX1("北京");
            }
        })
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Consumer<WeatherBean>() {
            @Override
            public void accept(WeatherBean weatherBean) throws Exception {
                Log.i(TAG, "北京getCity = " + weatherBean.getData().getCity());
            }
        });

//日志结果
I/RxJava: 上海getCity = 上海
I/RxJava: 转换中-------------------------------
I/RxJava: getCity = 上海
I/RxJava: 转换中-------------------------------
I/RxJava: 北京getCity = 北京
```
代码中我们利用`flatMap`进行转换，也获取到了第一次请求到的城市-上海的天气情况，我们可以在这里面做一些必要的事情，然后紧接着进行第二次的请求，获取城市-北京的天气情况，最后在`subscribe`中获取返回数据。

