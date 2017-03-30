---
layout:     post
title:      "Android RxJava + Retrofit 实现网络请求的轮询"
subtitle:   "Android RxJava + Retrofit Leaning"
date:       2017-03-30
author:     "Zhongshuzhi"
header-img: "img/post-bg-rwd.jpg"
tags:
    - Android
---

# Android RxJava + Retrofit 实现网络请求的轮询

[zhongshuzhi的博客]:https://zhongshuzhi.github.io
欢迎访问[zhongshuzhi的博客][]

***
[TOC]

***
## *轮询部分的代码*
这里给出轮询请求的实际代码，结合了*Retrofit*与*Rxjava*。

``` java
//在不用再轮询时请将实例化的Subscription unsubscribe掉
Subscription netWorkSubscription = ApiManage.getInstence().getNetApiService().queryFromServer()
.repeatWhen(new Func1<Observable<? extends Void>, Observable<?>>() {
    @Override
    public Observable<?> call(Observable<? extends Void> observable) {
        //此处设置请求的发送间隔
        return observable.delay(2, TimeUnit.SECONDS);
    }
})
.takeUntil(new Func1<SmartResult<Integer>, Boolean>() {
    @Override
    public Boolean call(SmartResult<Integer> integerSmartResult) {
        //返回你所设的条件是否为真
        return integerSmartResult.getData() == 1;
    }
})
.filter(new Func1<SmartResult<Integer>, Boolean>() {
    @Override
    public Boolean call(SmartResult<Integer> integerSmartResult) {
        //返回你所设的条件是否为真
        return integerSmartResult.getData() == 1;
    }
})
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(new Observer<SmartResult<Integer>>() {
    @Override
    public void onCompleted() {
        //请求成功结束
    }

    @Override
    public void onError(Throwable e) {
        //请求失败
        Log.d("HttpRequest" , e.toString());
    }

    @Override
    public void onNext(SmartResult<Integer> integerSmartResult) {
        //收到请求结果后的逻辑
    }
});
```
***
*PS:SmartResult类是用于网络请求中的一个辅助类，包含code,msg,data三个变量，code存储请求状态，msg存储请求信息，data存储泛型的数据类型。*

``` java
 public class SmartResult<T> {
 
   private int code;
   private String msg;
   private T data;
   
   public SmartResult(int code, String msg, T data) {
       this.code = code;
       this.msg = msg;
       this.data = data;
   }
}
```
***

该次轮询请求是向服务器请求类型为SmartResult<Integer\>的Json数据，当SmartResult.getData()!=1时，以2秒为间隔向服务器再次请求数据，知道SmartResult所携带的Data数据为1时停止轮询，执行onNext中的逻辑。

## 关于*RxJava*

*RxJava*是*ReactiveX*在JVM上的一个实现。使用`Observable`对象来实现异步与基于事件的程序。

什么是[ReactiveX](https://github.com/mcxiaoke/RxDocs/blob/master/Intro.md)?

### *实现轮询功能的关键操作符*

* `repeatWhen`
`repeatWhen`操作符不是缓存和重放原始`Observable`的数据序列，而是有条件的重新订阅和发射原来的`Observable`。
这里我们通过在返回的`Observable`对象中调用`obeservable.delay(int span,int unit)`函数来实现轮询间隔的设置。
关于`repeatWhen`操作符的详情请看[这里](https://github.com/mcxiaoke/RxDocs/blob/master/operators/Repeat.md)。
* `takeUntil`
`takeUntil`操作符订阅并开始发射原始`Observable`，它还监视你提供的第二个`Observable`。如果第二个`Observable`发射了一项数据或者发射了一个终止通知， `takeUntil`返回的`Observable`会停止发射原始`Observable`并终止。此处我们用其来终止轮询。`takeUntil`操作符的详情请看[这里](https://github.com/mcxiaoke/RxDocs/blob/master/operators/Conditional.md#TakeUntil)。
* `filter`
`filter`操作符使用你指定的一个谓词函数测试数据项，只有通过测试的数据才会被发射。不满足测试条件的`Observable`对象则被过滤掉。`filter`操作符的详情请看[这里](https://github.com/mcxiaoke/RxDocs/blob/master/operators/Filter.md)。

## 关于*Retrofit*

*A type-safe HTTP client for Android and Java.* *Retrofit*采用注解的方式对Android的网络请求进行了封装，同时对*RxJava* 有着良好的支持。示例代码如下：

``` java
public interface NetApi {
	@Post("xxx/queryFromServer")
	Observable<SmartResult<Integer>> queryFromServer();
}
```
***

```java
public class ApiManage{

    NetApi netApi;

    public NetApi getNetApiService() {
        if (netApi == null) {
            synchronized (this) {
                if (netApi == null) {
                    netApi = new Retrofit.Builder()
                            .baseUrl("http://xxx/xxxx")
                            .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                            .addConverterFactory(GsonConverterFactory.create())
                            .build().create(NetApi.class);
                }
            }
        }
        return netApi;
    }
}
```
