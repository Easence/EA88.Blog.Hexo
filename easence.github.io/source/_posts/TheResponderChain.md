---
title: iOS的事件处理顺序
categories: 
- Apple Development
- iOS开发笔记
tags: 
- 事件响应链
date: 2016-10-27 17:23:51
---


通常我们开发的app的UI元素都是有一个个view对堆叠起来的，也就是说我们点击屏幕的某一个区域的时候，会触动所有包含这个区域的view。那么这个触摸事件该由哪个View来处理，或者我们想更改iOS默认的处理方式又该如何下手，我们就需要先了解iOS中的事件是如何分发以及如何响应的。接下来介绍的是从官方文档精选出来的一部内容并加以了说明。

## 事件分发
当用户触发一个事件，UIKit会创建一个对应的对象来表示该事件（比如：触摸事件：UIEvent），并把该事件对象传递给app的事件队列里面，然后由UIApplication的单列从队首取出一个事件，通常会分发到UIWindow，再由UIWindow传递到下一个responder，直到找到一个能相应该事件的responder为止(对于触摸事件，通常就是hit-test View)。例如：

![Hit-testing returns the subview that was touched][1]

假设点击了View E。那么这个触摸事件传递的顺序是这样子的：

1. 由于触摸的区域在最外层的A之内，所以会检查它的子view B跟C。
2. 由于触摸区域在C内，因此会检查它的子view D和E。
3. 最后由于该触摸区域在E内，并且E是最顶层的View，因此它会成为hit-test view。

## 事件响应链
事件响应的顺序跟事件分发的顺序是相反的。通常事件首先都是由first responder处理的，如果它不想处理，则交由它的nextResponder处理，此时该responder就变成了first responder，反复如此，直到有一个responder愿意处理，如果到了最后都没有responder处理，则丢弃该事件。UIApplication, UIViewController以及UIView都是responder（都是UIResponder的子类）。

![The responder chain on iOS][2]

由上图可以看出，事件处理的的优先级从高到低，大概是这样子的：

**view** -> **view'superView** -> ... -> **viewController** -> **window** -> **application**

## 最后
至此，我们已经知道了iOS是如何处理事件，我们大致可以将这个过程分为两部分：事件分发以及事件响应，这两个过程的顺序是相反的。对于一个app来说，事件分发，就是从application开始找到一个最上面的view或者controller来处理事件。如果最上面的view不愿意处理，就进入事件响应的流程，会将这个事件一直往下抛，直到找到一个愿意处理的view或者controller，如果大家都不愿处理，就丢弃这个事件。

## 参考文档
[Event Delivery: The Responder Chain][3]

---

[1]: https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/hit_testing_2x.png

[2]: https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/iOS_responder_chain_2x.png

[3]: https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html
