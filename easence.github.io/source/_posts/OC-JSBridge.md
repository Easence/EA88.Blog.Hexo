---
title: WebViewJavascriptBridge原理分析
categories:
tags: 
- Web
- JavaScript
- JSBridge

---

#前言
本文主要研读了iOS端的[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)源码，并将其流程进行梳理，分为iOS端调用JS端，以及JS端调用iOS端。

# iOS端调用JS端
## 主流程
1. JS端调用`WebViewJavascriptBridge`的`registerHandler(handlerName, handler)`注册相应的回调，并存到一个`messageHandlers`的数组当中。
2. JS端通过将一个隐藏的iframe的src设置为：`https://__bridge_loaded__`，来触发OC的WebView的回调`webView: shouldStartLoadWithRequest: navigationType:`，此时会判断该请求的url，如果是上面的的`https://__bridge_loaded__`，则将`WebViewJavascriptBridge`的JS代码注入（通过调用UIWebView的`stringByEvaluatingJavaScriptFromString:`或者WKWebview的`stringByEvaluatingJavaScriptFromString:completionHandler:`的方法）到网页中。
3. iOS端调用`callHandler:handlerName data: responseCallback:`，并将其参数封装成一个WVJBMessage（NSDictionary），格式如下：

	```
	message:
	{
	    callbackId = "objc_cb_6"; //6是_uniqueId从0，以1为步长累加上来的
	    data =     {
	        greetingFromObjC = "Hi there, JS!";
	    };
	    handlerName = testJavascriptHandler; //在js端注册的名字，其对应的是一个方法。
	}
	```
	- 向Web端注入JSBridge的JS代码之前先将message保存在一个`startupMessageQueue（NSMutableArray）`中，待调用`injectJavascriptFile`向Web端注入JSBridge的JS的时候，递归的执行`_dispatchMessage:`。
	- iOS端调用方法`_dispatchMessage:`中将message序列化成一个json字符串`messageJSON`。
4. iOS端接下来会回调UIWebView的`stringByEvaluatingJavaScriptFromString:`或者WKWebview的`stringByEvaluatingJavaScriptFromString:completionHandler:`方法执行JS端的代码：`WebViewJavascriptBridge._handleMessageFromObjC('messageJSON')`
5. JS端执行`WebViewJavascriptBridge._handleMessageFromObjC()`,从数组`messageHandlers`根据上面所说的`handlerName`取出相应的方法执行，如果iOS端有传入`callbackId`，则将其变成`responseId`构造一个`message`，并写入在`sendMessageQueue`中，接着将`iframe`的`src`设置为：`https://__wvjb_queue_message__`，这将触发UIWebView的`webView:shouldStartLoadWithRequest:navigationType:`或者WKWebView的`webView:decidePolicyForNavigationAction:decisionHandler:`回调方法。
6. iOS端在调用判断请求是否是`https://__wvjb_queue_message__`，如果是，则调用`flushMessageQueue`，从JS端中的`sendMessageQueue`取出`message（json格式）`，并将其转换成本地的`WVJBMessage（NSDictionary）`，并通过它的`responseId`从本地的`responseCallbacks`取出回调方法进行执行。

# JS端调用iOS端
## 主流程
1. iOS端在WebView加载前，调用`registerHandler:handler:`方法注册回调。这些回调会以注册时传入的方法名为key，存入名为`messageHandlers（NSMutableDictionary）`中。
2. JS端调用`setupWebViewJavascriptBridge:`
 发起一个`https://__bridge_loaded__`请求（通过设置`iframe`的`src`实现）触发iOS端的UIWebView的`webView:shouldStartLoadWithRequest:navigationType:`或者WKWebView的`webView:decidePolicyForNavigationAction:decisionHandler:`回调方法，将JSBridge的JS代码注入到网页中。
3. JS端调用`bridge.callHandler(handlerName, data, responseCallback)`，然后将各个参数封装成一个message，其中会生成一个`callbackId`来标记这个`responseCallback`，存入`sendMessageQueue`中，接着设置`iframe`的`src`为`https://__wvjb_queue_message__`，依次触发UIWebView的`webView:shouldStartLoadWithRequest:navigationType:`或者WKWebView的`webView:decidePolicyForNavigationAction:decisionHandler:`回调方法。
4. iOS端在调用判断请求是否是`https://__wvjb_queue_message__`，如果是，则调用`flushMessageQueue`，从JS端中的`sendMessageQueue`取出`message（json格式）`，并将其转换成本地的`WVJBMessage（NSDictionary）`，并通过它的`handlerName`从本地的`messageHandlers`取出回调方法进行执行。
5. iOS端接着并将JS端传入的`callbackId`变成`responseId`构造一个`message`,iOS端调用方法`_dispatchMessage:`中将message序列化成一个json字符串`messageJSON`。
6. JS端执行`WebViewJavascriptBridge._handleMessageFromObjC()`,根据`responseId`从数组`responseCallbacks`中取出相应的回调执行。

