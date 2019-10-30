---
title: RxJava系列 - 操作符说明
copyright: true
date: 2019-04-20 10:49:07
tags:
- 操作符
categories:
- RxJava系列

---

### 1. 创建操作符
**1. create(ObservableOnSubscribe<T> source)**
> 完整的创建一个Observable或Flowable。

```java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {

    }
});


Flowable.create(new FlowableOnSubscribe<String>() {
    @Override
    public void subscribe(FlowableEmitter<String> emitter) throws Exception {

    }
}, BackpressureStrategy.BUFFER);
```

<!-- more -->

**2. just(T item1, T item2, T item3, T item4, T item5, T item6, T item7)**
> 快速创建一个Observable对象，直接发送事件给下游，最多只能发送10个事件。


```java
Observable.just(1,2,3,4,5,6,7,8,9,10);

Observable.just("A","B","C")
```

**3. fromArray(T... items)**
> 快速创建一个Observable对象，将传入的数据直接发送给下游。
> 
> **注意：传入的如果和list集合，最终返回给下游的也是一个list集合，原样传给了下游。**


```java
//也可以直接发送
Observable.fromArray(1, 2, 3).subscribe();
//数组
Integer[] items = { 0, 1, 2, 3, 4 };
Observable.fromArray(items).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG,"数据 : "+integer);
    }
});

//如果是list，下游收到的也是list
Observable.fromArray(list).subscribe(new Consumer<List<Integer>>() {
    @Override
    public void accept(List<Integer> integers) throws Exception {
        Log.d(TAG,"数据 : "+integers);
    }
});
```

**4. fromIterable(Iterable<? extends T> source)**
> 快速创建一个Observable对象，将传入的list集合数据发送给下游。
> 
> **注意：传入的数据只能是Iterable对象的数据，不能直接传入数组或者数字。**


```java
Observable.fromIterable(list).subscribe(new Consumer<Integer>() {
        @Override
        public void accept(Integer integer) throws Exception {
            Log.d(TAG,"数据 : "+integer);
        }
    });
```

**5. empty()、error(final Throwable exception) 、never()**
> **1. empty()：** 该方法创建的被观察者对象发送事件的特点：仅发送Complete事件，直接通知完成， 即观察者接收后会直接调用onCompleted。
> **Observable observable1=Observable.empty();** 
> 
> **2. error()：** 该方法创建的被观察者对象发送事件的特点：仅发送Error事件，直接通知异常(可以自定义异常)，即观察者接收后会直接调用onError。
> **Observable observable2=Observable.error(new RuntimeException())**
> 
> **3. never()：**该方法创建的被观察者对象发送事件的特点：不发送任何事件，即观察者接收后什么都不调用。
> **Observable observable3=Observable.never();**

**6. defer(Callable<? extends ObservableSource<? extends T>> supplier)**
> 延迟创建Observable，即直到有观察者订阅时才动态创建一个新的Observable，并发送最新的数据。
> 
> 比如：需要定时执行或周期性执行某些操作，还要确保数据的最新的，就可以使用defer。


```java
//比如已经定义变量a
Observable observable = Observable.defer(new Callable<ObservableSource<?>>() {
       @Override
       public ObservableSource<?> call() throws Exception {
           return Observable.just(a);
       }
   });
//然后变量a被重新赋值
a = 10;
observable.subscribe(new Consumer<Integer>() {
        @Override
        public void accept(Integer o) throws Exception {
            Log.d(TAG,"数据 : "+o);
        }
    });

//输出结果：10
```

**7. timer(long delay, TimeUnit unit, Scheduler scheduler)**
> 创建一个Observable对象，并在指定的延迟时间之后发送一个数值0(Long类型)给下游，本质上就是：延迟指定时间后调用onNext(0)。
> 
> //参数delay表示延迟时间，参数unit表示时间单位
> **timer(long delay, TimeUnit unit)**
> //参数scheduler表示线程调度
> **timer(long delay, TimeUnit unit, Scheduler scheduler)**

```java
Observable.timer(3,TimeUnit.SECONDS)
   .subscribe(new Observer<Long>() {
       @Override
       public void onNext(Long aLong) {
           Log.d(TAG,"onNext 收到数据 ： "+aLong);
       }

       @Override
       public void onComplete() {
           Log.d(TAG,"onComplete------");
       }
   });

//输出结果：onNext中收到数据0，然后onComplete被回调。
```

**8. interval(long initialDelay, long period, TimeUnit unit, Scheduler scheduler)**
> 创建一个Observable对象，每隔指定的时间发送一个事件，事件序列由0开始，每次递增1，无限递增。
> 
> //参数initialDelay表示初次发送事件的延时时间，参数period表示之后发送事件的间隔时间，参数unit表示时间单位，schedule表示线程调度
> **interval(long initialDelay, long period, TimeUnit unit, Scheduler scheduler)**

```java
Observable.interval(5, 1, TimeUnit.SECONDS, Schedulers.io())
    .subscribe(new Consumer<Long>() {
        @Override
        public void accept(Long aLong) throws Exception {
            Log.d(TAG,"下游收到数据 : "+aLong);
        }
    });

//输出结果：0 1 2 3....,第一次输出0时延迟了5s，以后每隔1s产生一个事件。
```

**9. intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit, Scheduler scheduler)**
> 创建一个Observable，每隔指定的时间，从指定位置开始，发送指定数量的事件，每次递增1。
> 
> //参数start：事件发送开始的位置
> //参数count：发送事件的总数
> //参数initialDelay：初次发送事件的延迟时间
> //参数period：后续发送事件的间隔时间
> **intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit)**

```java
//从2开始，总计发送10个事件，首次延迟2s，之后间隔1s发送一个事件
Observable.intervalRange(2,10,2, 1, TimeUnit.SECONDS)
    .subscribe(new Consumer<Long>() {
        @Override
        public void accept(Long aLong) throws Exception {
            Log.d(TAG,"下游收到数据 : "+aLong);
        }
    });

//输出结果：2 3 4 ....  11，从2开始总共发送10个事件
```

**10. range(final int start, final int count)、rangeLong(long start, long count)**
> 创建一个Observable对象，发送一个事件序列，序列每次递增1，可指定开始位置和发送事件总数，没有延迟。
> 
> rangeLong支持Long类型

```java
Observable.range(2,10).subscribe(new Consumer<Integer>() {
     @Override
     public void accept(Integer integer) throws Exception {
          Log.d(TAG,"下游收到数据 : "+integer);
     }
});

//输出结果：2 3 4 ... 11，从2开始，发送10个事件。
```

---
### 2. 变换操作符
**1. map(Function<? super T, ? extends R> mapper)**
> 将上游Observable发送的每一个事件通过指定函数，转变为任意类型的事件。
> 比如：将Integer类型的数据转换为String类型数据


```java
Observable.just(1,2,3,4)
        .map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) throws Exception {
                return "变换 "+integer;
            }
        }).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.d(TAG,"下游收到数据 : "+s);
    }
});

//输出结果：变换1 变换2 变换3 变换4，将数值1234前面增加了文字变成String类型。
```

**2.1  flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper)\ concatMap**

**2.2  flatMap(Function<? super T, ? extends ObservableSource<? extends U>> mapper,BiFunction<? super T, ? super U, ? extends R> resultSelector)**

**2.3  concatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper, int prefetch)**

> flatMap\concatMap 使用一个指定的函数对原始Observable发射的每一项数据之行相应的变换操作，这个函数返回一个本身也发射数据的Observable，然后FlatMap合并这些Observables发射的数据，最后将合并后的结果当做它自己的数据序列发射。
> 
> 比如：上游O1发送了事件1,2,3，然后经过flatMap\concatMap操作符可以将：
> ----->事件1变换成有Oa发送的1a、1b事件，
> ----->事件2变换成由Ob发送的2a、2b事件，
> ----->事件3变换成有Oc发送的3a、3b事件，
> 然后flatMap\concatMap还会将上面的6个事件合并，当做原始Observable的事件发送给下游。
> 
> **注意：**
> 1. flatMap\concatMap变换后下游收到的事件数量 = 变换后事件数量的总和，与原始Observable发送的事件数量无关。
> 
> 2. **flatMap的变换是无序的，**比如原始Observable发送数据顺序为：1,2,3，经过flatMap转换之后下游收到的事件顺序可以是：1a、3a、2a、2b、1b、3b。
> 
> 3. **concatMap的变换是有序的，严格按照原始Observable的事件发送顺序，**比如原始Observable发送数据顺序为：1,2,3，经过flatMap转换之后下游收到的事件顺序一定是：1a、1b、2a、2b、3a、3b。

```java
//上游发送了一批学生的信息，要求下游不进行遍历就可以收到每个学生的每一门课程的名称.
Observable.create(new ObservableOnSubscribe<Student>() {
        @Override
        public void subscribe(ObservableEmitter<Student> emitter) throws Exception {
        //发送每一个学生信息给下游(List数据此处未展示)
            emitter.onNext(getStudents().get(0));
            emitter.onNext(getStudents().get(1));
            emitter.onNext(getStudents().get(2));
        }
    })
    //flatMap无序
    .flatMap(new Function<Student, ObservableSource<Classes>>() {
    //concatMap有序
    //.concatMap(new Function<Student, ObservableSource<Classes>>() {
        @Override
        public ObservableSource<Classes> apply(Student student) throws Exception {
        //将学生信息转换为课程信息，获取每一个学生课程信息的集合，然后fromIterable会逐个将课程名称发送给下游。
            return Observable.fromIterable(student.className);
        }
    }).subscribe(new Consumer<Classes>() {
        @Override
        public void accept(Classes classes) throws Exception {
        //下游直接获取到的就是每个学生每一门课程名称，不需要再次遍历。
            Log.d(TAG,"下游收到数据 ： "+classes.cName);
        }
    });

//输出结果：
下游收到数据 ： 学生1--课程1
下游收到数据 ： 学生1--课程2
下游收到数据 ： 学生1--课程3
下游收到数据 ： 学生2--课程1
下游收到数据 ： 学生2--课程2
下游收到数据 ： 学生2--课程3
下游收到数据 ： 学生3--课程1
下游收到数据 ： 学生3--课程2
下游收到数据 ： 学生3--课程3
```
**3. buffer(long timespan, TimeUnit unit, Scheduler scheduler)  、 buffer(ObservableSource<B> boundary, Callable<U> bufferSupplier)**


> 定期从Observable中获取指定数量的事件，先存到缓存区，然后再组装成一个集合发送给下游。
> 
> //参数count表示缓存区大小，当只有一个参数时，默认skip=count
> **buffer(int count)**
> //参数skip表示获取新事件的数量。
> **buffer(int count, int skip)**
> 
> **注意：**
> 1. 如果缓存区小于新事件获取数量，就会存在因为取到的事件无法缓存，导致部分事件无法发送到下游的情况。
> 比如：原始Observable有1、2、3、4、5、6、7、8事件，缓存区大小count=2，每次取新事件skip=3，那么下游收到的数据为：1、2、4、5、7、8。


```java
Observable
    .just(1, 2, 3, 4, 5, 6, 7, 8)
    .buffer(2, 3)
    .subscribe(new Observer<List<Integer>>() {
        @Override
        public void onNext(List<Integer> integers) {
            Log.d(TAG, " 事件数量 = " + integers.size());
            for (Integer value : integers) {
                Log.d(TAG, " 接收事件 = " + value);
            }
        }
    });

//输出结果：
事件数量 = 6
接收到的数据只有：1 2 4 5 7 8
```

---
### 3. 组合、合并操作符
**1. concat(Iterable<? extends ObservableSource<? extends T>> sources) 、 concat(ObservableSource<? extends ObservableSource<? extends T>> sources) 、  concatArray(ObservableSource<? extends T>... sources)**

> 组合多个被观察者Observable一起发送数据，合并后按之前合并顺序串行发送事件，concat组合数量≤4，concatArray组合数量>4.


```java
 Observable.concat(
	Observable.just(1, 2, 3),
    Observable.just(4, 5, 6),
    Observable.just(7, 8, 9),
    Observable.just(10, 11, 12))

//输出结果：1 2 3 4 5 6 7 8 9 ....
```

**2. merge(Iterable<? extends ObservableSource<? extends T>> sources) 、 merge(ObservableSource<? extends ObservableSource<? extends T>> sources) 、  mergeArray(ObservableSource<? extends T>... sources)**

> 组合多个被观察者Observable一起发送数据，合并后按之前合并顺序并行发送事件，即同时发送事件，merge组合数量≤4，margeArray组合数量>4.


```java
Observable.merge(
	//参数：开始位置、发送事件总数、第一次发送延迟时间、每次发送间隔时间
    Observable.intervalRange(0, 3, 1, 1, TimeUnit.SECONDS), 
    Observable.intervalRange(2, 3, 1, 1, TimeUnit.SECONDS));

//输出结果：0 2 -- 1 3 -- 2 4
```

**3.1  concatDelayError(Iterable<? extends ObservableSource<? extends T>> sources)  **

**3.2  mergeDelayError(Iterable<? extends ObservableSource<? extends T>> sources) 、 mergeDelayError(ObservableSource<? extends ObservableSource<? extends T>> sources)** 

> **使用情况：**当使用concat、merge操作符时，如果组合中某个observable发送了error事件，则会立即终止其他observable发送其他事件。
> 
> **解决方案：** 若希望onError事件推迟到其他所有observable发送事件结束之后才触发，可以使用concatDelayError \ mergeDelayError操作符。
> 
> **案例：**比如observable1和observable2，当o1发送了error事件时，o1就停止发送其他事件了，那么此时o2也将不再发送其他事件，如果使用concatDelayError \ mergeDelayError则可以发送完所有事件之后，再触发error事件。
> 
> **疑问：**如果有observable同时发送了onError和onComplete事件，那么使用了concatDelayError \ mergeDelayError后，还会触发之前的oEerror事件吗？
> **答：**onComplete和onError事件不会同时发送，所以发送了complete就不会有error，反之也一样。

```java
Observable.concatArrayDelayError(
    Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
    emitter.onNext(1);
    emitter.onNext(2);
    emitter.onNext(3);
    // 发送Error事件，因为使用了concatDelayError，所以第2个Observable将会发送事件，等发送完毕后，再发送错误事件
    emitter.onError(new NullPointerException());
    emitter.onComplete();
    }
    }),Observable.just(4, 5, 6))

//输出结果：1 2 3 4 5 6 onError事件错误信息
```

**4.1  zip(Iterable<? extends ObservableSource<? extends T>> sources, Function<? super Object[], ? extends R> zipper)**

**4.2  zip(ObservableSource<? extends T1> source1, ObservableSource<? extends T2> source2,BiFunction<? super T1, ? super T2, ? extends R> zipper)**
> 组合多个Observable，但是严格按照每个Observable发送的事件序列进行位置合并，即A1-B1,A2-B2,A3-B3...，并且合并后事件的数量是根据Observable中事件数量最少的来决定的。
> 
> 下游收到的事件数量和上游众多Observable中事件数量最少的那个一致，虽然多余的事件也会发送，但是下游不会收到。

```java
Observable o1 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "o1 发送事件 1");
                emitter.onNext(1);
                Log.d(TAG, "o1 发送事件 2");
                emitter.onNext(2);
                Log.d(TAG, "o1 发送事件 3");
                emitter.onNext(3);
                Log.d(TAG,"o1 发送事件 onComplete");
                 emitter.onComplete();
            }
        }).subscribeOn(Schedulers.io());

        Observable o2 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                Log.d(TAG, "o2 发送事件 A");
                emitter.onNext("A");
                Log.d(TAG, "o2 发送事件 B");
                emitter.onNext("B");
                Log.d(TAG, "o2 发送事件 C");
                emitter.onNext("C");
                Log.d(TAG, "o2 发送事件 D");
                emitter.onNext("D");
                Log.d(TAG, "o2 发送事件 onComplete");
                emitter.onComplete();
            }
        }).subscribeOn(Schedulers.newThread());

        Observable.zip(o1, o2, new BiFunction<Integer, String, String>() {
            @Override
            public String apply(Integer integer, String s) throws Exception {
                Log.d(TAG, "zip 开始进行合并-------------");
                return integer + s;
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
            }
            @Override
            public void onNext(String s) {
                Log.d(TAG, "下游收到 zip 合并结果  s = " + s);
            }
            @Override
            public void onError(Throwable e) {
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "下游结束-------------");
            }
        });

//输出结果：1A 2B 3C； 但是事件D也会发送，不过下游不会收到。
```

**5.1  combineLatest(ObservableSource<? extends T1> source1, ObservableSource<? extends T2> source2,BiFunction<? super T1, ? super T2, ? extends R> combiner) **

**5.2  combineLatestDelayError(Function<? super Object[], ? extends R> combiner,int bufferSize, ObservableSource<? extends T>... sources)**
> 假设上游有n个Observable，将前面n-1个Observable发送的最后一个数据，与第n个Observable的每一个数据进行相应的函数结合(可以相加、相乘等)，然后将结合的数据发送给下游。
> 
> 比如：上游的Observable有A(1,2,3)，B(5,6,7,8)，C(A、B、C、D)，然后结合的方式是相加，那么结合的过程就是：A中的最后一个数据A(3)和B中的最后一个数据B(8)相加，然后与C中的每一个数据进行相加。
> 
> 下游收到的事件数量取决于最后一个Observable中事件的数量。

```java
Observable.combineLatest(
    Observable.just(1, 2, 3),
    Observable.just(5, 6, 7, 8),
    Observable.just("A", "B", "C", "D"),
    new Function3<Integer, Integer, String, String>() {
    @Override
    public String apply(Integer integer, Integer aLong, String s) throws Exception {
		Log.d(TAG, "合并事件  integer = " + integer + ",  aLong = " + aLong + ",   s = " + s);
		return integer + aLong + s;
	}
    }).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.d(TAG, "下游收到合并结果 = " + s);
    }
    });

//输出结果：11A 11B 11C 11D; O1和O2的最后一个数据相加：3+8=11，然后再与O3的每个数据相加。
```

**6. reduce(BiFunction<T, T, T> reducer) **
> 将Observable中需要发送的事件全部聚合成一个事件，然后在发送出去。
> 比如：Observable中与事件a=1、b=2、c=3、d=4，那么聚合过程就是：a与b进行聚合，然后在与c进行聚合，然后在与d进行聚合，将得到的最终结果发送给下游。
> 
> 如果是聚合函数是相乘，那么就是：1 * 2，然后2 * 3，然后6 * 4，最终得到数据24发送给下游。


```java
Observable.just(1,2,3,4)
    .reduce(new BiFunction<Integer, Integer, Integer>() {
    @Override
    public Integer apply(Integer integer, Integer integer2) throws Exception {
    //integer表示合并之后的结果，integer2表示发送过来的新数据
        return integer * integer2;
    }
    }).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG,"下游 ："+integer);
        }
    });

//输出结果：24；这里做的是乘法运算：1*2=2,2*3=6,6*4=24这样一个过程，使用前面两个数据的结果，在与后一个数据进行计算，以此类推。 
```

**7. collect(Callable<? extends U> initialValueSupplier, BiConsumer<? super U, ? super T> collector)**
> 将Observable发送的数据整合到一个数据结构中。
> 比如：Observable发送数据：1,2,3,4，通过collect操作符整合，发送给下游的可以是一个List<>集合,集合中包含以上数据。


```java
Observable.just(1,2,3,4)
		  .collect(new Callable<ArrayList<Integer>>() {
            @Override
            public ArrayList<Integer> call() throws Exception {
            //第一步：创建一个数据结构容器，用于收集Observable发送的数据
                return new ArrayList<>();
            }
        }, new BiConsumer<ArrayList<Integer>, Integer>() {
            @Override
            public void accept(ArrayList<Integer> integers, Integer integer) throws Exception {
            //参数integers是前面创建的数据结构，参数integer是发送过来的数据，这里可以增加条件，收集需要的数据
                if(integer % 2 == 0)  integers.add(integer);
            }
        }).subscribe(new Consumer<ArrayList<Integer>>() {
            @Override
            public void accept(ArrayList<Integer> integers) throws Exception {
                for (int data : integers){
                    Log.d(TAG,"下游收到的数据包含 ： "+data);
                }
            }
        });
```

**8. startWith(Iterable<? extends T> items) 、 startWith(ObservableSource<? extends T> other) 、  startWithArray(T... items)**
> 在一个Observable发送数据之前，追加发送一些数据或者一个新的Observable，追加的数据在原有数据之前，比如原来是：1,2,3，追加数据是：4,5,6，那么下游收到数据的顺序是：4,5,6,1,2,3。
> 
> 追加一个数据使用startWith(2)，追加多个数据使用startWithArray(2,3,4)，两者可以同时使用，遵循后追加的数据在前的原则。
> 
> 需要追加新的Observable只能使用startWith操作符。


```java
Observable.just(1, 2, 3)
    .startWith(Observable.just(8, 9))
    .startWithArray(4, 5, 6)
    .subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "下游收到的数据包含 ： " + integer);
        }
    });
//输出结果：4 5 6 8 9 1 2 3；后追加的在前面输出。
```

**9. count()**
> 统计被观察者发送事件的数量


```java
Observable.just(1, 2, 3, 4)
          .startWithArray(5,6,7)
          .count()
          .subscribe(new Consumer<Long>() {
              @Override
              public void accept(Long aLong) throws Exception {
                  Log.e(TAG, "发送的事件数量 =  "+aLong);
              }
          });
          
//此处会统计两次发送的事件
发送的事件数量 = 7
```

---
### 4. 功能性操作符
**1. delay**
> **使得被观察者延迟一段时间再发送事件。**
> 
> // 1. 参数delay是延迟时间；参数unit是时间单位
> **delay(long delay,TimeUnit unit)**
> 
> // 2. 参数scheduler是线程调度器
> **delay(long delay,TimeUnit unit,mScheduler scheduler)**
> 
> // 3.参数delayError是是否延迟发送错误消息
> // 如果delayError = true，表示会正常发送完前面的事件，然后在发送错误消息
> // 如果delayError = false，表示直接发送错误消息，前面的事件都不发送了
> **delay(long delay,TimeUnit unit,boolean delayError)**
> **delay(long delay,TimeUnit unit,mScheduler scheduler,boolean delayError):**

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onError(new NullPointerException());
                emitter.onNext(3);

            }
        }).delay(2,TimeUnit.SECONDS,false)
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(Disposable d) {

                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.d(TAG,"onNext下游收到事件 ： "+integer);
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG,"onError下游收到事件 ： "+e.toString());
                    }

                    @Override
                    public void onComplete() {

                    }
                });
    }

//日志信息
true ： 
onNext下游收到事件 ： 1
onNext下游收到事件 ： 2
onError下游收到事件 ： java.lang.NullPointerException

false：
onError下游收到事件 ： java.lang.NullPointerException
```

**2. do操作符**
> do操作符只是一种总称，包含doOnComplete 、doOnNext、 doAfterNext 、doOnEach等等，主要是在订阅的整个生命周期的不同阶段进行不同的操作。


```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
                e.onError(new Throwable("发生错误了"));
            }
        }).doOnEach(new Consumer<Notification<Integer>>() {
            // 1. 当Observable每发送1次数据事件就会调用1次
            @Override
            public void accept(Notification<Integer> integerNotification) throws Exception {
                Log.d(TAG, "doOnEach: " + integerNotification.getValue());
            }
        }).doOnNext(new Consumer<Integer>() {
            // 2. 执行Next事件前调用
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "doOnNext: " + integer);
            }
        }).doAfterNext(new Consumer<Integer>() {
            // 3. 执行Next事件后调用
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "doAfterNext: " + integer);
            }
        }).doOnComplete(new Action() {
            // 4. Observable正常发送事件完毕后调用
            @Override
            public void run() throws Exception {
                Log.e(TAG, "doOnComplete: ");
            }
        }).doOnError(new Consumer<Throwable>() {
            // 5. Observable发送错误事件时调用
            @Override
            public void accept(Throwable throwable) throws Exception {
                Log.d(TAG, "doOnError: " + throwable.getMessage());
            }
        }).doOnSubscribe(new Consumer<Disposable>() {
            // 6. 观察者订阅时调用
            @Override
            public void accept(@NonNull Disposable disposable) throws Exception {
                Log.e(TAG, "doOnSubscribe: ");
            }
        }).doAfterTerminate(new Action() {
            // 7. Observable发送事件完毕后调用，无论正常发送完毕 / 异常终止
            @Override
            public void run() throws Exception {
                Log.e(TAG, "doAfterTerminate: ");
            }
        }).doFinally(new Action() {
            // 8. 最后执行
            @Override
            public void run() throws Exception {
                Log.e(TAG, "doFinally: ");
            }
        }).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "接收到了事件" + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        });
    }
```

**3. onErrorReturn(Function<? super Throwable, ? extends T> valueSupplier)**
> 当遇到错误或者异常时，可以捕获在它之前的错误，并发送一个自己定义的事件给下游，然后正常终止，不会因为前面的错误而出现异常。
> 同时onErrorReturn操作符重新发送的事件，下游也是在onNext回调中收到，不是在onError中。


```java
Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("AA");
                emitter.onNext("BB");
                emitter.onError(new Throwable("上游突然发生了错误"));
            }
        }).onErrorReturn(new Function<Throwable, String>() {
            @Override
            public String apply(Throwable throwable) throws Exception {
                return "我知道上游发生了错误，没关系，我来拯救。";
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
            }
            @Override
            public void onNext(String s) {
                Log.d(TAG,"下游onNext收到数据 : "+s);
            }
            @Override
            public void onError(Throwable e) {
                Log.d(TAG,"下游onError收到数据 : "+e.toString());
            }
            @Override
            public void onComplete() {
            }
        });
    }
```

**4. onErrorResumeNext(final ObservableSource<? extends T> next) 、 onErrorResumeNext(Function<? super Throwable, ? extends ObservableSource<? extends T>> resumeFunction)**
> 当遇到错误时重新定义一个新的Observable，然后使用新的Observable继续发送数据，发生错误之前正确发送的数据还是会发送给下游。
> 
> **注意：onErrorResumeNext可以捕获Throwable或者Exception类型的错误。**


```java
Observable.create(...)
//主要代码，演示捕获异常
		.onErrorResumeNext(new Function<Throwable, ObservableSource<? extends String>>() {
            @Override
            public ObservableSource<? extends String> apply(Throwable throwable) throws Exception {
            //重新定义新的Observable，发送事件
                return Observable.just("A","B","C");
            }
        })
```

**5. onExceptionResumeNext(final ObservableSource<? extends T> next)**
> 当遇到错误时Exception类型的错误时，可以将其捕获，但是获取不到具体的错误信息，然后通过`new ObservableSource`获得一个观察者Observer对象，指定回调到下游的onNext、onError或者onComplete中。
> 
> **注意：onExceptionResumeNext无法捕获到Throwable类型的错误，错误信息会直接回调到下游的onError中。**


```java
Observable.create(...)
//主要代码，演示捕获异常
	.onExceptionResumeNext(new ObservableSource<String>() {
            @Override
            public void subscribe(Observer<? super String> observer) {
                observer.onNext("上游出现异常，我捕获到了。");
            }
        });
```

**6. retry**
> **当发生错误时，让Observable重新发送数据，从第一个数据开始发送。**
> 
> **1. retry()**
> 出现错误时，让被观察者重新发送数据，若一直错误，则一直重新发送，直到成功了才停止，否则是死循环。
> 
> **2. retry(long times)**
> 出现错误时，让被观察者重新发送数据，参数表示重试次数，若重试结束还没有成功，就回调到下游的onError中。

> **3. retry(Predicate<? super Throwable> predicate)**
> 出现错误后，判断是否需要重新发送数据，如果需要重新发送，但是一直是错误，就一直循环。
> ```java
> Observable.create(...)
> .retry(new Predicate<Throwable>() {
            @Override
            public boolean test(Throwable throwable) throws Exception {
                Log.d(TAG,"我获取到了错误信息 ： "+throwable.toString());
                //true : 表示重试；false：不重试，就直接回调到下游的onError中。
                return true;
            }
        })
> ```
> 
> **4. retry(BiPredicate<? super Integer, ? super Throwable> predicate)**
> 出现错误后，判断是否需要重新发送数据，增加了重试次数，可以进行相关判断，如果在重试n次之后还是错误，就回调到下游的onError中结束。
> ```java
> Observable.create(...)
> .retry(new BiPredicate<Integer, Throwable>() {
            @Override
            public boolean test(Integer integer, Throwable throwable) throws Exception {
                Log.d(TAG,"重试次数 = "+integer);
                Log.d(TAG,"错误消息 = "+throwable.toString());
                if(integer == 5){
                    return false;
                }
                return true;
            }
        })
> ```
> **5. retry(long times, Predicate<? super Throwable> predicate)**
> 出现错误后，判断是否需要重新发送数据，如果需要则会根据前面第一个参数进行n次重试，如果之后还是错误，则回调到下游的onError中结束。

**7. retryUntil(final BooleanSupplier stop)**
> **上游错误后，判断是否需要重新发送数据。**
> 
> **true：**不重试，拦截错误返回给下游的onError。
> **false：**重试，如果一直失败就会一直重试。


```java
Observable.create(...)
.retryUntil(new BooleanSupplier() {
            @Override
            public boolean getAsBoolean() throws Exception {
                Log.d(TAG,"我拦截上游错误----");
                //true：拦截错误回调给下游的onError， false：重试
                return true;
            }
        })
```

**8. retryWhen(final Function<? super Observable<Throwable>, ? extends ObservableSource<?>> handler)**
> 当出现错误时，将发生的错误(比如Throwable错误)传递给一个新的Observable，并由新的这个Observable来决定是否需要原始的Observable重新发送事件。
> **注意：**
> 1. 当新的Observable返回onNext事件时，会进行重试，如果原始的Observable一直失败，就一直重试，如果成功了，新的Observable发送的事件不会发送到下游。
> 
> 2. 当新的Observable返回error(比如Observable.error(new Thrwoable("error")))事件时，不进行重试，而是直接回调到下游的onError事件中。


```java
Observable.create(...)
.retryWhen(new Function<Observable<Throwable>, ObservableSource<?>>() {
    @Override
    public ObservableSource<?> apply(Observable<Throwable> throwableObservable) throws Exception {
        return throwableObservable.flatMap(new Function<Throwable, ObservableSource<?>>() {
            @Override
            public ObservableSource<?> apply(Throwable throwable) throws Exception {
            //直接返回error则不会进行重试，而是回调到下游的onError中。
                return Observable.error(new Throwable("retryWhen 中终止错误，不重试。"));
                //如果是重新发送事件，则会进行重试，原始的Observable一直失败就一直重试，如果成功了，这里发送的事件是不会发送到下游的。
				return Observable.just("A","B");
                }
            });
        }
    })
```

**9. repeat() 、repeat(long times)**
> 无条件的重复发送Observable的事件，可以传递参数控制重复次数，如果遇到错误，就不会重复发送事件了。
> 
> **注意：是重复发送正确的事件，是不是遇到错误进行重试。**

**10. repeatWhen(final Function<? super Observable<Object>, ? extends ObservableSource<?>> handler)**
> **有条件的重复发送Observable的事件。**
> 
> **原理：**repeatWhen操作符是将上游Observable停止发送事件的标识(比如onError和onComplete都标识停止发送事件)转换成一个Object类型的数据，传递给一个新的Observable，然后由新的Observable来决定是否重复发送数据。
> 
> **条件：**
> **1. 不重复发送：**当新的Observable返回onComplete或onError事件时，则不重复发送原始Observable的数据。
> 
> **2. 重复发送：**当新的Observable发送正常事件，比如onNext事件时，则重复发送原始的Observable的数据，新的Observable数据不会发送给下游，只是一个标识而已。
> 
> ```java
> Observable.create(...)
> .repeatWhen(new Function<Observable<Object>, ObservableSource<?>>() {
    @Override
    public ObservableSource<?> apply(Observable<Object> objectObservable) throws Exception {
        return objectObservable.flatMap(new Function<Object, ObservableSource<?>>() {
            @Override
            public ObservableSource<?> apply(Object o) throws Exception {
                Log.d(TAG,"我来决定是否重复--");
                //1. 只是发送了一个onConplete事件，下游并不会收到onComple回调，所以不会进行重复发送。
                //return Observable.empty();
                //2. 发送了error事件，不会进行重复发送，并且error会回调到下游的onError中。
                return Observable.error(new Throwable("我要终止"));
                //3. 发送的是onNext事件，所以会重复发送原始的Observable的事件，并一直重复。
                //return Observable.just(1);
            }
        });
    }
})
> ```


---
### 5. 过滤操作符
**1. filter(Predicate<? super T> predicate)**
> 根据特定的条件过滤上游Observable发送的事件。


```java
//过滤比3小的事件
Observable.just(1,2,3,4,5,6)
        .filter(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
            //根据test()的返回值进行过滤，true：过滤，false：不过滤。
                return integer >=3;
            }
        }).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG,"下游收到事件 : "+integer);
    }
});

//输出结果：3 4 5 6，此处不小于3的事件就被过滤了。
```

**2. ofType(final Class<?> clazz)**
> 根据指定的数据类型进行过滤，比如：Integer、String等，符合条件的会发送给下游，不符合条件的就直接抛弃。


```java
Observable.just(1,"rxjava",3,"android",5)
          //将String类型的数据发送给下游，其他类型的都抛弃
            .ofType(String.class).subscribe(new Consumer<String>() {
        @Override
        public void accept(String s) throws Exception {
            Log.d(TAG,"下游收到事件 : "+s);
        }
    });

//输出结果：rxjava android，整型事件都被过滤了。
```

**3. skipLast(long time, TimeUnit unit, Scheduler scheduler)、skipLast(long time, TimeUnit unit, Scheduler scheduler)**
> 从前面、后面跳过某个发送的事件。
> 
> //从前面、后面跳过指定时间内发送的数据。
> **skip(long time, TimeUnit unit)**
> **skipLast(long time, TimeUnit unit)**


```java
//根据数量进行筛选
Observable.just(1, "rxjava", 3, "android", 5, 6, 7)
        //跳过前面发送的两个事件
        .skip(2)
        //跳过后面发送的两个事件
        .skipLast(2)
        .subscribe(new Consumer<Serializable>() {
            @Override
            public void accept(Serializable serializable) throws Exception {
                Log.d(TAG, "下游收到事件 : " + serializable.toString());
            }
        });
//输出结果：3 android 5，注意这里有Integer和String类型的数据，所以下游进行了序列化。

//根据时间进行筛选
//这里是：从0开始，发送8个事件，第1个时间延时0s，以后每隔1s发送一个事件
Observable.intervalRange(0,8,0,1,TimeUnit.SECONDS)
        //跳过前2s发送的事件，不论多少个事件都跳过
        .skip(2,TimeUnit.SECONDS)
        //跳过最后2s发送的事件，不论多少个事件都跳过
        .skipLast(2,TimeUnit.SECONDS)
        .subscribe(new Consumer<Serializable>() {
            @Override
            public void accept(Serializable serializable) throws Exception {
                Log.d(TAG, "下游收到事件2 : " + serializable.toString());
            }
        });
//输出结果：2 3 4 5 ，原本应该是：0 1 2 3 4 5 6 7
```

**4. distinct() 、 distinct(Function<? super T, K> keySelector)、distinctUntilChange() 、 distinctUntilChanged(Function<? super T, K> keySelector)**
> **distinct：**过滤事件序列中重复的事件，即前面出现过的，全部过滤掉。
> 
> **distinctUntilChange：**过滤事件序列中连续重复的事件，即使前面有出现过也没关系，只要不是连续重复的。

```java
//1. 过滤重复数据，只要出现过的就过滤掉
Observable.just(1,2,2,4,5,5,5,1,2,1)
        .distinct()
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "下游收到事件: " + integer);
            }
        });
//输出结果：1 2 4 5

//2. 过滤掉连续重复的数据
Observable.just(1,2,2,4,5,5,5,1,2,1)
        .distinctUntilChanged()
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "下游收到事件2 : " + integer);
            }
        });
//输出结果：1 2 4 5 1 2 1

//3. 过滤集合中重复的数据或者连续重复的数据
//简化Students设置的过程，最终返回一个包含20条数据的集合。
for (int i = 1; i < 20; i++) {
    Student student = new Student();
    if(i % 2 == 0){
        student.setScode("2");
    }else if(i%3 == 0){
        student.setScode("3");
    }else{
        student.setScode("1");
    }
}
//从集合中根据scode进行重复、连续重复过滤
Observable.fromIterable(getStudents())
        .distinct(new Function<Student, String>() {
            @Override
            public String apply(Student student) throws Exception {
                return student.getScode();
            }
        }).flatMap(new Function<Student, ObservableSource<String>>() {
    @Override
    public ObservableSource<String> apply(Student student) throws Exception {
    //使用flatMap进行变换，下游直接拿到需要的数据
        return Observable.just(student.name+" - "+student.scode);
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.d(TAG, "下游收到事件3: " + s);
    }
});
//输出结果：
//distinct:学生-1-1,学生-2-2, 学生-3-3，一起只有3条数据。
//distinceUntilChanged:学生-1-1 ... 学生-19-1,一起20条数据全部输出，因为按照上面的方式设置的集合，不存在连续重复。
```

**5. take(long time, TimeUnit unit, Scheduler scheduler)、takeLast(long count, long time, TimeUnit unit, Scheduler scheduler)**
> 指定下游只能接受前n个、后n个数据，上游有多少个数据还是会正常发送，只是在中间被从前、后取出n个发送给下游。
> 
> **takeLast取事件时上游一定要调用onComplete，确认事件发送完毕，否则取不到事件。**

```java
//去最后发送的两个事件
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        Log.d(TAG, "上游发送事件 ： 1");
        emitter.onNext(1);
        Log.d(TAG, "上游发送事件 ： 2");
        emitter.onNext(2);
        Log.d(TAG, "上游发送事件 ： 3");
        emitter.onNext(3);
        Log.d(TAG, "上游发送事件 ： 4");
        emitter.onNext(4);
        Log.d(TAG, "上游发送事件 ： 5");
        emitter.onNext(5);
        //一定记得发送onComplete事件，否则从后取事件时会认为上游事件没有发送完毕，取不到事件。
        emitter.onComplete();
    }
}).takeLast(2).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "下游事件 ："+integer);
    }
});

//输出结果：4 5
```

**6. throttleFirst(long skipDuration, TimeUnit unit, Scheduler scheduler)、throttleLast(long intervalDuration, TimeUnit unit, Scheduler scheduler)**
> 在某段时间内，只发送该段时间内的第一次、最后一次的事件。
> **注意：**
> 1. 使用throttleFirst时，onComplete或者onError方法始终会发送，即使在该段时间内已经发送了其他事件。
> 2. 使用throttleLast时，如果onComplete或者onError在间隔事件内，和前一个事件同时发送时，前一个事件将不被发送，只发送onComplete或onError会。
> 
> **应用场景：**发送短信验证码时，在几秒钟之内连续点击多次，只响应第一次点击事件，发送一个短信验证码。
> 
> **比如：**每间隔0.5s发送1 2 3 4 5 6 7 8等事件，而throttleFirst设置的间隔时间是1s，那么最终发送的事件就会是：1 3 5 7。
> 
> **盗图说明：**
> ![Alt text](https://i.loli.net/2019/04/20/5cba8c9cced2a.png)


```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        Thread.sleep(500);

        emitter.onNext(2);
        Thread.sleep(500);

        emitter.onNext(3);
        Thread.sleep(500);

        emitter.onNext(4);
        Thread.sleep(500);

        emitter.onNext(5);
        Thread.sleep(500);

		//注意：这里事件6将不会发送，取而代之的是onComplete事件
        emitter.onNext(6);             
        emitter.onComplete();
    }
}).throttleLast(1,TimeUnit.SECONDS)
        .subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
            }
            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "onNext下游事件 ："+integer);
            }
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "onError下游事件 ：");
            }
            @Override
            public void onComplete() {
                Log.d(TAG, "onComplete下游事件 ：");
            }
       });
//输出结果：
throttleFirst：1 3 5 onComplete，最后的onComplete依然会发送，即使在间隔时间内有其他事件发送了。
throttleLast：2 4 onComplete，事件6将不会发送。
```

**7. sample(long period, TimeUnit unit, Scheduler scheduler) 、 sample(ObservableSource<?> sampler)**
> 在某段时间内，只发送该时间内最后一次事件，与上面的throttleLast用法一致，其实throttleLast内部调用的就是sample。

**8. throttleWithTimeout(long timeout, TimeUnit unit, Scheduler scheduler)、 debounce(long timeout, TimeUnit unit, Scheduler scheduler)**
> 发送数据事件时，如果两个事件之间的时间间隔 < 指定时间，则抛弃前一次发送的事件，直到指定时间内再没有新的数据事件发送时，才会发送最后一次的事件。
> 
> 比如：指定间隔事件是2s，然后开始发送A，等待1s后，发送B，然后等待1s后，发送C，然后等待2s发送D，此时下游收到数据：C和D。
> ------ > 因为A-B之间间隔不足2s，事件A被抛弃；
> ------ > 而B-C之间间隔也不足2s，事件B被抛弃；
> ------ > 最后C-D之间间隔了2s，所以事件C被保留发送给下游，事件D之后没有发送其他事件了，那么事件D也会发送给下游。
> 
> **注意：**
> **1. onComplete和onError事件不受限制，即使和前一个事件的间隔小于指定时间，同样会发送给下游。**
> 
> **2. debounce(Function<? super T, ? extends ObservableSource<?>>  debounceSelector) 还可以重新定义一个Observable，对发送的数据进行判断**


```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        Thread.sleep(1000);
//1和2之间间隔小于2s，事件1被抛弃
        emitter.onNext(2);
        Thread.sleep(2000);
//2和3之间间隔刚好2s，事件2被保留，事件3之后只有onComplete事件，没有其他事件，所以事件3也被保留发送。
        emitter.onNext(3);
        Thread.sleep(500);
        emitter.onComplete();
    }
}).throttleWithTimeout(2, TimeUnit.SECONDS)
        .subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
            }
            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "onNext下游事件 ：" + integer);
            }
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "onError下游事件 ：");
            }
            @Override
            public void onComplete() {
                Log.d(TAG, "onComplete下游事件 ：");
            }
        });
//输出结果：2 3 onComplete，onComplete不管间隔时间是多少都会发送。
```
**9. firstElement() 、 lastElement()**
> 仅选取第一个、最后一个事件发送给下游。
> 包含onComplete时。两者都不会发送个下游，但如果是onError时，lastElement会发送给下游，并且不发送其他事件。
> 
> **注意：firstElement 、 lastElement返回类型是Maybe，所以下游使用的是new MaybeObserver。**


```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        Thread.sleep(1000);

        emitter.onNext(2);
        Thread.sleep(2000);

        emitter.onNext(3);
        Thread.sleep(500);
        emitter.onComplete();
    }
}).lastElement().subscribe(new MaybeObserver<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onSuccess(Integer integer) {
        Log.d(TAG,"数据 : "+integer);
    }
    @Override
    public void onError(Throwable e) {
    }
    @Override
    public void onComplete() {
        Log.d(TAG,"数据onComplete ");
    }
});

//输出结果：
//firstElement：1
//lastElement：3
```

**10. elementAt(long index) 、elementAt(long index, T defaultItem)**
> 接收指定索引位置的事件，允许越界，也不会抛出异常，越界时可以指定默认值。即使有onComplete事件也不会发送给下游。
> 
> //参数index表示索引位置，索引从0开始。
> **Maybe<T> elementAt(long index)**
> //参数defaultItem表示越界后设置的默认值。
> **Single<T> elementAt(long index, T defaultItem)**

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onNext(3);
        emitter.onNext(4);
        emitter.onComplete();
    }
}).elementAt(8,99).subscribe(new SingleObserver<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onSuccess(Integer integer) {
        Log.d(TAG,"数据 : "+integer);
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG,"onError ");
    }
});
//输出结果：99，越界之后输出指定的默认值。
```

**11. elementAtOrError(long index)**
> 接收指定索引位置的事件，如果越界会抛出异常。
> 
> //参数index表示索引位置
> **Single<T> elementAtOrError(long index)**
> 注意：
> 1. 如果下游订阅时使用的是：subscribe(new SingleObserver<>())，如果越界了会回调到下游的onError方法中，并不会奔溃。
> 2. 如果下游订阅时使用的是：subscribe(new Consumer<>())，如果越界了，会导致程序奔溃。


```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onNext(3);
        emitter.onNext(4);
        emitter.onComplete();
    }
}).elementAtOrError(8).subscribe(new SingleObserver<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onSuccess(Integer integer) {
        Log.d(TAG,"发生onSuccess");
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG,"发生error : "+e.toString());
    }
});

//输出结果：抛出异常，回调到onError中。
```


---
### 6. 条件、布尔操作符
**1. all(Predicate<? super T> predicate)**
> 判断发送的每个数据是否满足条件，满足返回true，不满足返回false。
> 
> **注意：返回的不是发送的数据具体的值，并且会逐一判断每个发送的数据。**


```java
Observable.just(3,4,5,1)
        .all(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer>2;
            }
        }).subscribe(new Consumer<Boolean>() {
    @Override
    public void accept(Boolean aBoolean) throws Exception {
        Log.d(TAG,"满足条件: "+aBoolean);
    }
});

//输出结果:前面三个数都>2,但是1<2,所以最终结果是:false
```

**2. takeWhile(Predicate<? super T> predicate)**
> 判断发送的每个数据是否满足特定条件，如果满足就发送，不满足就不发送给下游。
> 
> **注意：发送的是满足条件的数据的值**

```java
Observable.just(3,4,5,1)
        .takeWhile(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer>2;
            }
        }).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG,"收到数据 :"+integer);
    }
});

//输出结果：3 4 5
```

**3. skipWhile(Predicate<? super T> predicate)**
> 判断发送的每个数据是否满足特定的条件，如果满足(true)就不发送给下游，直到不满足(false)时，才把开始把后续的所有数据都发给下游，不管条件是否满足。(**一次满足即可**)
> **注意：一定是返回false时，才开始发送数据，之前的不会发送。**

```java
Observable.just(3,4,5,1,0,5,6)
        .skipWhile(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer>2;
            }
        }).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG,"收到数据 :"+integer);
    }
});

//输出结果：1 0 6 5，从1开始返回false的，后续就直接发送，不管判断条件。
```

**4. takeUntil(ObservableSource<?> other)  、 takeUntil(Predicate<? super T> stopPredicate)**
> 判断发送的每个数据是否满足特定条件，不满足时发送数据，直到满足条件时停止发送，满足条件的这一个数据还是会发送的，后续的就不发了。
> 
> **注意：满足条件时才停止，不满足时却是发送数据的。**


```java
Observable.just(-1,1,0,2,6,1)
        .takeUntil(new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) throws Exception {
                return integer>2;
            }
        }).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG,"收到数据 :"+integer);
    }
});

//输出结果：-1 1 0 2 6，正好6满足条件，后面的1就不发送了。
```
**5. skipUntil(ObservableSource<U> other)**
> skipUntil传入的是Observable，当传入的Observable开始发送数据时，原始的Observable才开始发送数据，相当于新的Observable是原始Observable发送数据的开关。
> 
> **注意：当新传入的Observable开始发送数据时，原始的Observable先于该时间点发送的数据，将不会发送给下游。**


```java
//每隔1s发送一个数据，数据从0开始发送，每次递增1
Observable.interval(1,TimeUnit.SECONDS)
		//延时5s开始发送数据
        .skipUntil(Observable.timer(5,TimeUnit.SECONDS))
        .subscribe(new Observer<Long>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG,"onSubscribe---------");
    }
    @Override
    public void onNext(Long value) {
        Log.d(TAG,"下游收到数据 :"+value);
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG,"onError :"+e.toString());
    }
    @Override
    public void onComplete() {
        Log.d(TAG,"onComplete---------");
    }
});

//输出结果：4 5 6 7 8....，前面已经发送的数据(0 1 2 3)不会发给下游
//如果上游是一下子发送完了数据，这里新传入的Observable延时了5s，那么下游就收不到任何数据了，直接回调到onComplete。
```

**6. sequenceEqual(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2)**
> 判断两个Observable发送的数据是否完全相同，相同返回true，不相同返回false。
> 
> **数据相同，顺序不同返回也是false。**


```java
Observable.sequenceEqual(
    Observable.just(1,2,3),
    Observable.just(2,1,3))
    .subscribe(new Consumer<Boolean>() {
        @Override
        public void accept(Boolean aBoolean) throws Exception {
            Log.d(TAG,"输出结果:"+aBoolean);
        }
    });
//输出结果:false,数据顺序不一致，也是false。
```

**7. contains(final Object element) **
> 发送的数据中是否包含指定的数据，包含返回true，不包含返回false。
> 
> **内部实现：any(Predicate<? super T> predicate)**


```java
Observable.just(1,2,3,4)
    .contains(3)
    .subscribe(new Consumer<Boolean>() {
        @Override
        public void accept(Boolean aBoolean) throws Exception {
            Log.d(TAG,"输出结果:"+aBoolean);
        }
    });
//输出结果：true
```

**8. isEmpty()**
> 判断发送的数据是否为空，若为空，返回true，不为空，返回false。

```java
Observable.just("A","B","","C")
    .isEmpty()
    .subscribe(new Consumer<Boolean>() {
        @Override
        public void accept(Boolean aBoolean) throws Exception {
            Log.d(TAG,"输出结果:"+aBoolean);
        }
    });
//输出结果:false,即使传的是""也不会认为是空的
```

**9. amb(Iterable<? extends ObservableSource<? extends T>> sources) **
>  当需要发送多个Observable的数据时，只发送最先发送了数据的Observable的数据，其余Observable的数据抛弃。

```java
List<ObservableSource<Integer>> list = new ArrayList<>();
list.add(Observable.just(1,2,3).delay(1,TimeUnit.SECONDS));
list.add(Observable.just(4,5,6));
Observable.amb(list).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
   Log.d(TAG,"收到数据 :"+integer);
    }
});

//输出结果：4 5 6，因为第一个Observable发送数据延迟了1s，所以第二个会先发送数据，amb()操作符会使得第一个Observable的数据被抛弃
```

**10. defaultIfEmpty(T defaultItem)**
> 在不发送任何有效事件(即onNext事件)、只发送onComplete事件的前提下，发送一个指定的默认值。


```java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        //emitter.onNext("AA");
        emitter.onComplete();

    }
}).defaultIfEmpty("BB")
        .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(String s) {
                Log.d(TAG,"onNext 数据 ： "+s);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG,"onError 数据 ： "+e.toString());
            }

            @Override
            public void onComplete() {
                Log.d(TAG,"onComplete");
            }
        });
//输出结果：BB onComplete，如果是上游发送了onNext事件，就不会发送BB了。
```




