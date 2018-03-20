---
title: RxSwift基础
categories: 开发日志
tags: Swift
---

## 写在开始

本文的写作目标受众是刚接触RxSwift、以及还在探索RxSwift的读者。接下来会从4个方面做介绍：为什么用、怎么理解，基本概念及其使用、用的时候注意什么。

## 为什么用

RxSwift是MVVM模式的最佳伴侣之一。

项目开发采用的基础模式是MVVM，其中M（Model）与V（View）之间往往需要数据的双向绑定，即，M（Model）的变动能及时通知到（View），V（View）的变动能及时更新到M（Model）。如此一来，我们可能会想到KVO，但是如果采用KVO,会发现业务逻辑很大一部分就是在C（Controller），这就变成MVC模式了，而非在MVVM模式。

幸好，有了响应式编程框架，如：RxSwift、ReactiveCocoa等，能很好的实现M（Model）与V（View）之间往往需要数据的双向绑定。

至于，为什么使用RxSwift，而不采用ReactiveCocoa，网上应该很多资料介绍，在此就不做过多的阐述，但我认为都是实现Reactive的框架，主要思想、接口应该基本一致，可能会有一些特性差异（比如冷热信号）。当初决定使用RxSwift的原因有两点：

1. 之前没有使用过RxSwift，想尝试一下。
2. RxSwift社区很活跃。

接下来就介绍RxSwift的几个基本概念。

## 怎么理解

我们在理解RxSwift的时候，可以与监听者模式联想在一起。一个信号量（Observable）可以当做被监听者，订阅者（Observer，在RxSwift中，我们可以认为就是一个闭包。实际上是一个Observer对象分装了这个闭包）则是监听者。一个信号量可以被多个订阅者订阅。当信号量维护的值有变化时，订阅者能收到相应的通知。

## 基本概念及其使用

在我们打算深层次探索一个开源框架的时候，可能最好的方式就是先熟悉它的基本概念、模仿Demo去使用、然后配合调试去阅读源码。本文要做的主要事情就是介绍它的基本概念、以及一些简单使用。后续，再去分析它的源码实现。

### ObservableType和ObserverType

- `ObservableType`是一个`Protocol`，所有的被监听者都实现该协议。最重要的是，定义了最基本的订阅方法`func subscribe<O: ObserverType>(_ observer: O) -> Disposable where O.E == E`，以及`public func asObservable() -> Observable<E>`的实现。

- `ObserverType`是一个`Protocol`，所有的监听者都实现该协议。定义了最基本的收到通知后的处理方法`func on(_ event: Event<E>)`，以及`public func onNext(_ element: E)`，`public func onCompleted()`，`public func onError(_ error: Swift.Error)`实现基本的。

### Observable和Observer

- `Observable`是一个`class`，是`ObservableType`的最基本的实现，同时，也是一个泛型，，这样我们就可以把Swift中的所有类型都包装成一个被监听者类型。一般其子类都会维护一个`Observer`的list。

- 以`Observer`为后缀的类，都是`ObserverType`的子类，当调用`Observable`的`subscribe`方法都会显式或隐式创建一个以`Observer`为后缀的的类型，添加到`Observable`的监听者列表里面。

- `Observable`的发送的消息有三类：Next、Error、Completed。后两种会结束整个消息流。对应的`Observer`处理方式也有三种：onNext、onError、onCompled。

### Disposable
`Disposable`是一个`Protocol`，定义了一个`func dispose()`方法，调用`Observable`的`subscribe`方法返回的类型，其作用就是给调用者机会从`Observable`的`Observer`列表中将该`Observer`移除。

> 一般使用过程中，我们都是在使用`Observable`类型时候，在后面加上`takeUntil(self.rx.deallocated)`或者`disposed(by:disposeBag)`来达到释放`Observer`d的目的。


### SubjectType

`SubjectType`是一个`Protocol`，继承自`ObservableType`，但它也可以转换成`Observer`，可以简单的认为它既是被监听者，也是监听者。两个角色可以随意转换。他的具体实现有（但不限于）：

- `PublishSubject`：订阅者会错过之前发送过的消息。
- `ReplaySubject`：订阅者不会错过之前发送过的消息，可以设置不想错过的消息数目。
- `BehaviorSubject`：被监听者会保留最后一条消息，新来的订阅者首先会收到之前保留的这条消息。

> 具体解释可以参考[这里](http://reactivex.io/documentation/subject.html)

### RxSwift独有的特性
RxSwift除了响应式编程的一些通用功能，自己还拥有一些独有的特性，这`Single`，`Completable `，`Maybe`几个特性，都是`PrimitiveSequence`的一个别名，可以认为是对`Observable`的一个封装，其实现如下：

```
public struct PrimitiveSequence<Trait, Element> {
    let source: Observable<Element>

    init(raw: Observable<Element>) {
        self.source = raw
    }
}
```
#### Single
顾名思义，`Single`跟不普通的`Observable`的不同在于，只会发送一个元素或者一个Error。常用的场景是HTTP请求，往往只返回一个Response或者Error。`Single`的定义如下：

```
/// Sequence containing exactly 1 element
public enum SingleTrait { }
/// Represents a push style sequence containing 1 element.
public typealias Single<Element> = PrimitiveSequence<SingleTrait, Element>

public enum SingleEvent<Element> {
    /// One and only sequence element is produced. (underlying observable sequence emits: `.next(Element)`, `.completed`)
    case success(Element)
    
    /// Sequence terminated with an error. (underlying observable sequence emits: `.error(Error)`)
    case error(Swift.Error)
}
```
由上面的实现，我们可以看到`Single`只是收发的事件有两种：一个是`.success(Element)`，另一个就是`.error(Swift.Error)`。

#### Completable
`Completable`跟不普通的`Observable`的不同在于，只会发送一个Completed或者一个Error。当我们不需要关心一个数据流的具体元素是什么，只关心完成与否的时候，就可以用`Completable`。其定义如下：

```
/// Sequence containing 0 elements
public enum CompletableTrait { }
/// Represents a push style sequence containing 0 elements.
public typealias Completable = PrimitiveSequence<CompletableTrait, Swift.Never>

public enum CompletableEvent {
    /// Sequence terminated with an error. (underlying observable sequence emits: `.error(Error)`)
    case error(Swift.Error)
    
    /// Sequence completed successfully.
    case completed
}
```

#### Maybe
`Maybe`是`Single`和`Completable`的结合，会发送一个元素或者一个Completed或者一个Error，三选一。其定义如下：

```
/// Sequence containing 0 or 1 elements
public enum MaybeTrait { }
/// Represents a push style sequence containing 0 or 1 element.
public typealias Maybe<Element> = PrimitiveSequence<MaybeTrait, Element>

public enum MaybeEvent<Element> {
    /// One and only sequence element is produced. (underlying observable sequence emits: `.next(Element)`, `.completed`)
    case success(Element)
    
    /// Sequence terminated with an error. (underlying observable sequence emits: `.error(Error)`)
    case error(Swift.Error)
    
    /// Sequence completed successfully.
    case completed
}
```

#### Driver
可以认为他是专门为数据模型驱动UI而订制的。它主要是为了解决下面3个问题：

- 当我们将获取数据的信号bind到某个UI元素的时候，如果获取数据信号发送的是一个Error消息，那么UI元素将无法刷新。Driver初始化的时候可以设置一个：当发生Error的时候的默认值。
- UI刷新必须在主线程。Driver默认在主线程刷新。
- 能重放消息，比如：一个页面有两个UI需要用到获取数据的信号，而这个信号是对一个网络请求的封装。那么每次我们订阅那个信号的时候，都会发一次网络请求。而用Driver可以重放最近一次的结果，那么，第二次订阅就不会再触发网络请求了。

## 用的时候注意什么

说起循环引用，可能大家都不陌生，自OC进入ARC时代，就一直是内存泄漏的罪魁祸首。

而，采用了RxSwift，一不小心就会导致内存泄漏。因为我们才使用MVVM模式的时候，往往是ViewController里面引用ViewModel，然后在RxSwift的闭包中使用到`Self.xxx`这样的调用，这就容易引起循环引用，从而导致内存泄漏，因此在使用RxSwift的时候应该处处留心，应该时刻谨记以下两点：

1. 必要的时候在闭包的参数前加上`[weak self]`这样的声明。如果不确定，该不该加，那就统一加上，一般不会出现问题，因为大部分情况下业务逻辑都是帮随着View出现而调用，View的消失而取消。
2. 在每个信号订阅后，及时加上`takeUntil(self.rx.deallocated)`或者`disposed(by:disposeBag)`。

## 最后

RxSwift是一个比较庞大的库，上面只做了些基本的分享，如果要熟悉所有的API，需要不断阅读文档、反复使用、阅读源码。如果研究透了它，我相信不仅能让我们的开发效率提高，还能加深我们自身对Swift这门语言的理解。一起加油吧！

