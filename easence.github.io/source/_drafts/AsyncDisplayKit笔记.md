---
title: AsyncDisplayKit笔记
categories: 
- Apple Development
- iOS开发笔记
tags: UI优化
---

## 开始
众所周知，`UIKit`它的基础UI单元是`UIView`，类似的`AsyncDisplayKit`的基础单元的是`ASDisplayNode`，`ASDisplayNode`是`UIView`的抽象，不同的是`ASDisplayNode `是线程安全的，可以在在主线程之外使用。

要让app能流畅运行就需要保证app的渲染帧数在60f/s，这也是iOS开发的黄金法则。因此需要将图片解码、文字计算、渲染等耗时的任务从主线程中剥离出去。而`AsyncDisplayKit`做到了这一点。下面就介绍`ASDK`(`ASDK`是`AsyncDisplayKit`的缩写)中的具体模块。

## Nodes
`node`对应于`UIKit`中的`UIView`，可以说是对`UIView`的封装，`node`的大部分属性方法命名都跟`UIView`保持一致，如果需要访问`node`对应的`UIView`，或者`CALayer`，可以直接使用`node.view`或者`node.layer`，但是要注意在主线程使用它们。

`ASDK`提供了不同类型的node，它们的层级如下图：

![node-hierarchy](http://asyncdisplaykit.org/static/images/node-hierarchy.png)

## Node Containers
如果想在现有的工程中使用`ASDK`，不能直接将`node`加到现有的view层级中，这样会导致渲染后直接被刷新到屏幕中。

而是，应该是将node加到的[`node Containers`](http://asyncdisplaykit.org/docs/containers-overview.html)中，这些Container就像是UIkit与ASDK之间的桥梁，它们负责管理它们的孩子node的状态，在适当的时机有效的更新孩子node的数据以及渲染UI。

## Layout Engine
ASDK的布局引擎是基于`CSS FlexBox`模型构建的，每一个`node`都会有一个与之对应的`ASLayoutSpec`来管理它的布局。`CSS FlexBox`相关的学习资料可以在参考如下几个网站：

- [ASStackLayout Game](http://nguyenhuy.github.io/froggy-asdk-layout/)
- [Visual Guide to CSS3 Flexbox](https://demos.scotch.io/visual-guide-to-css3-flexbox-flexbox-playground/demos/)
- [FlexBox Patterns](http://www.flexboxpatterns.com/home)


