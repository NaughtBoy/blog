---
title: 使用RxJava实现EventBus——RxBus（翻译）
date: 2016-06-16 17:35:44
tags: RxJava
categories: Android
---


> 原文：http://nerds.weddingpartyapp.com/tech/2014/12/24/implementing-an-event-bus-with-rxjava-rxbus/

**（译者注：文中的EventBus多指事件总线这种设计模式，而非[``EventBus``](https://github.com/greenrobot/EventBus)这个具体的类库。） **

这一篇文章有三个部分:
- 快速了解什么是EventBus
- 使用RxJava实现EventBus
- 对RxBus的一些想法

<!-- more -->

“RxBus”不是一种类库。使用RxJava实现EventBus是非常简单的，而且不需要担心会让你的类库变大。

## 第一部分：什么是Event Bus

让我们来讨论下两种相似的东西：观察者模式和Pub-Sub模式。
（译者注：Pub-Sub，即pubish-subscribe，发布-订阅）

### 观察者模式
这种模式的意思是：你的一个类或者主要对象（Observable）将一个相关的信息通知（Event）给其他所有感兴趣的类或对象（Observer）。

### Pub-Sub模式
该模式的目标和观察者模式的目的是一样的，即：你想让其它的一些类知道或了解到某些事件的发生。
观察者模式和Pub-Sub模式有一个重要的区别：Pub-Sub模式的关注点在于把广播发出去，而观察者模式并不在乎消息是发到哪里去了，只是在乎消息是否已经发出去了，换句话说，被观察者（ 或发布者）不需要知道谁是观察者（或订阅者）。

### 为什么要匿名？
计算机编程里面一个好词汇叫“解耦”。你应该在你的程序设计里面尽可能地保持低耦合度。
通常情况下，你希望发布者（Publisher）都能知道每一个需要通知的订阅者（ subscriber），这样才能在事件或者消息准备好了的时候准确通知到这些订阅者。当在EventBus 里面，发布者并不需要知道每一个需要通知的订阅者，这种独立性有助于解耦，因为发布者和订阅者不需要通过逻辑编码来建立它们的关系。
话句话说，在你编写代码的每一个地方尽量有意识地解耦。

### 怎么匿名？
那么，在Pub-Sub模式里面自然就出现了这么一个问题：怎么实现发表者（publisher）和订阅者（ Subscriber）之间的匿名？一个简单的实现方法就是创建一个中间人（ middleman）并让这个中间人负责所有的通信。而EventBus就是中间人的一种。
在Android里面，通常使用两个EventBus的类库：Square 的[``Otto``](http://square.github.io/otto/) 和 Green Robot 的[``EventBus``](https://github.com/greenrobot/EventBus)。网上有大量的关于如何把它们应用到你的App里面的文章。

## 使用RxJava实现EventBus
我以前发表过一篇有关 [RxJava for Android](https://github.com/kaushikgopal/RxJava-Android-Samples)的文章,我会继续在那篇文章写完具体的实现。以下是实现中相关的一部分：
```
// 这就是那个中间类
public class RxBus {

    private final Subject<Object, Object> _bus = new SerializedSubject<>(PublishSubject.create());

 public void send(Object o) {
        _bus.onNext(o);
  }

    public Observable<Object> toObserverable() {
        return _bus;
  }
}```

现在，你有了一个可以用的EventBus了。
下面是发送一个事件的代码：
```
@OnClick(R.id.btn_demo_rxbus_tap)
public void onTapButtonClicked() {

    _rxBus.send(new TapEvent());
}```

下面是你在``fragment``、``services``等需要监听这些事件的地方的代码：
```
// 这个_rxBus 对象实例是那个发布事件的对象
_rxBus.toObserverable().subscribe(new Action1<Object>() {
  @Override
  public void call(Object event) {
    if(event instanceof TapEvent) {
      _showTapText();
    }else if(event instanceof SomeOtherEvent) {
      _doSomethingElse();
    }
  }
});```

在这个例子里，我们在顶部的绿色``fragment``发布了一个事件，并在通过总线在底部蓝色的``fragment``进行监听。

[![demo](http://nerds.weddingpartyapp.com/images/posts/rxbus_simple.gif "demo")](http://nerds.weddingpartyapp.com/images/posts/rxbus_simple.gif "demo")

## 对RxBus的一些想法
###  Dead Events（死亡事件：没有订阅者的事件）
有某些情况下，知道当前有没有观察者（Observers）在监听总线（Bus）是很有用的。比如，在你[用EventBus处理你的消息推送](https://markhudnall.com/2013/11/13/gcm-foreground-and-background/)，而且你不想App在前台的时候发送一个 push notification（推送通知），那么监听一个 [Dead Event](https://github.com/square/otto/blob/master/otto/src/main/java/com/squareup/otto/DeadEvent.java)就很重要了。
又比如我们最近发布的 Wedding Party（译者注：原文，目测是一款App），我们增加了“消息”的功能。如果用户打开App（至少有一个以上对bus的监听者），我们不会发送push notification。但如果App在后台的话，我们会发送一个能让他们知道聊天信息的push notification。要是一个事件被发送到Event Bus中，如果当前没有监听者，就会返回一个Dead Event。如果我们得到一个返回的Dead Event的话就会发送一个 Push Notification。
你该怎么用 RxBus实现Dead Event呢？
其实很简单，``Subject``类中有一个很有用的``hasObservers()``方法就可以实现了。这个方法是在[1.x的版本被加到RxJava](https://github.com/ReactiveX/RxJava/pull/1802)的，所以至少在当前最后的版本（0.23.0）的RxAndroid中可以找到这个方法。
### 那么我应该使用RxBus而不是 Otto/Event Bus 吗？
如果你只是先在你的Android App里面简单地应用Event Bus，你最好还是使用Otto（更推荐）或者[Event Bus](https://github.com/greenrobot/EventBus)。Otto 拥有更加简洁的由注解驱动的Api，使用起来会更加地方便。
如果你熟悉Rx，而且已经在你的项目里面使用了RxJava，而且又想摆脱多余的库，那么你一定要试试RxBus。

> 译者注：英文水平有限，恳请看到该文章的诸位帮忙校对下，看看哪里出错了。