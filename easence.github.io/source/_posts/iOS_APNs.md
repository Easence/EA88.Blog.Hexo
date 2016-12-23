---
title: iOS开发之新版APNs搭建必备知识
date: 2016-12-20 18:31:37
categories:
 - Apple Development
 - iOS开发笔记
tags: 
- 推送
- APNs
---

*本文的大部分内容是对苹果关于APNs官方文档的翻译以及整理。*

## 一、设备token和消息的生命周期
关于设备token以及推送消息的生命周期需要注意下面几点：

- Token会在iOS系统更新或者设备数据、设置被擦除的时候改变。
- 当设备离线的时候，APNS会将消息数据存储一段时间，等设备上线后重发。如果设备在离线期间，向APNS发送了多条推送消息，APNS将会丢弃掉前面的一些消息，只保留后面的消息，要是设备长时间离线，则会将所有的消息丢弃掉。
- 可以通过设置http/2头中的`apns-collapse-id`键值对来合并消息，比如：`apns-collapse-id : 2`，那么value为2的消息将被APNS合并成一条消息推送给设备。

## 二、Provider（后台）与APNs的交互

Provider（即，APP的后台）与APNS有两种安全的交互方式，都必须采用TLS以保证可靠性，更详细的内容继续往下看。
> `TLS`的简单理解是，为了保证数据传输安全，在HTTP层与TCP层之间插入的一个安全校验层，它所做的事情简单来说就是：**“通过CA申请的证书验证client与server是可靠的之后，通过相应的公私钥加密一个`协商的公钥`，之后的真实数据传输就使用这个公钥进行加密，以保证数据的安全可靠性”**，关于TLS的更详细的介绍，可以参考[这篇文章](https://segmentfault.com/a/1190000002554673)

### 基于Token的方式（Token-Based Provider-to-APNs Trust）

#### 1、流程概述
这种方式适合在基于HTTP/2协议的Provider使用，它与APNs之间的连接通过[JWT（JSON web tokens）](https://tools.ietf.org/html/rfc7519)来验证。在这种方式下不需要使用证书+私钥的方式来建立可靠连接。Provider只需要提供一个一对公私钥（私钥给APNs保存，公钥Provider自己保存）,并使用其中的私钥生成并加密JWT Token，每次向APNs请求推送的时候带上这个Token即可。

具体步骤如下：

1. Provider通过`TLS`向APNs发起请求。
2. APNs返回一个证书给Provider。
3. Provier验证这个证书。通过后，发送push数据并带上JWT token。
4. APNs验证token，并返回请求的结果。

![Establishing and using token-based connection trust between a provider and APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_provider_ct_2x.png)


> 建立TLS连接必须要有一个`GeoTrust Global CA root certificate`，在macOS中，这个证书已经安装在keychain中，如果是其他操作系统则可以在[GeoTrust Root Certificates website](https://www.geotrust.com/resources/root-certificates/)下载。

#### 2、Provider Authentication Tokens
关于JWT（JSON Web Token）的详细资料可以通过[这里](https://tools.ietf.org/html/rfc7519)了解。同时也可以从[这里](https://jwt.io)找到一些现成可用的库。

下面对JWT进行详细的介绍，一个JWT实际上是一个JSON对象，它的头部必须包含：

- 用以加密token的加密算法(**alg**) ，比如：ES256。
- 10个字符长度的标识符(**kid**)，（登入苹果开发者账号后，进入到**Certificates, Identifiers & Profiles**，然后点击**APNs Auth Key**，最后在右侧找到**Apple Push Notification Authentication Key (Sandbox & Production)**选项，点击创建后可以创建一个p8文件。）

同时他的claims payload部分必须包含：

- issuer(**iss**) registered claim key，其值就是10个字符长的Team ID。
- issued at (**iat**) registered claim key，其值是一个秒级的UTC时间戳。

比如：

```
{
    "alg": "ES256",
    "kid": "ABC123DEFG"
}
{
    "iss": "DEF123GHIJ",
    "iat": 1437179036
 }
```
创建完这个token后，必须使用自己的私钥对其进行加密，然后再采用基于P-256曲线和SHA-256哈希算法的椭圆曲线数字签名算法（ECDSA）进行签名，并将`alg`键的值设置为`ES256`。(注意：APNs只支持**ES256**签名的JWT，否则会返回`InvalidProviderToken(403)`错误)

为了保证安全，APNs要求定期更新token，时间间隔为1小时，如果APNs发现当前的时间戳与`iat`值中的时间戳相比，大于一个小时，那么APNs会拒绝推送消息，并返回`ExpiredProviderToken (403)`错误。

### 基于证书的方式（Certificate-based connection trust)

#### 流程概述

这种方式是指Provider可以采用一个唯一的证书以及一个加密的私钥来与APNs交互，其中证书是由苹果产生的（通过苹果账号登录到[developer account](https://developer.apple.com/account/#/welcome)配置创建）。整个交互过程如下：

1. Provider通过`TLS`向APNs请求连接。
2. APNs向Provider返回一个APNs证书。
3. Provider验证这个APNs证书，并将从苹果官网获取的证书返回给APNs。
4. APNs验证通过后，这个链接就算是建立了。

![Establishing certificate-based connection trust between a provider and APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_provider_ct_certificate_2x.png)

#### APNs Provider Certificates
创建步骤可以参考[Configure push notifications](https://help.apple.com/xcode/mac/8.1/index.html?localePath=en.lproj#/dev11b059073)中的Generate a universal APNs client SSL certificate章节。

## 三、APNs连接

### 连接的管理
苹果的两个APNs server分别为：

- **Development server:** `api.development.push.apple.com:443`
- **Production server:** `api.push.apple.com:443`

要与APNs交互要求server必须支持1.2及上版本的TLS协议。通过上面的介绍我们已经知道，server跟APNs交互有两种方式：基于JWT的方式以及基于证书的方式。为了保证高质量的使用APNs应该注意如下几点：

- 对于基于JWT的方式，应该定时更新token，token的有效期为1小时。
- 对于基于JWT的方式，能每次请求都创建新的token，尽量在一小时内使用同一个token。
- 不能频繁的建立、关闭连接，否则APNs会把这当做是黑客攻击，拒绝访问。应该尽量将连接保活，直到你认为这个连接接下来会长时间不使用为止。
- 当需要发送大量的推送数据的时候，可以同时创建多个连接，以改善性能。
- 当吊销证书或者token时，应该关闭所有相关的连接。

### HTTP/2的请求与响应
详情可参见苹果官方文档[HTTP/2 Request to APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW2)。这里介绍了接口的**请求参数**、**返回结果**，**错误码**以及**示例代码**。下面仅截取了其中的例子，以加深对APNs的使用的理解：

- 基于证书的方式的request：

	```
	HEADERS
	  - END_STREAM
	  + END_HEADERS
	  :method = POST
	  :scheme = https
	  :path = /3/device/00fc13adff785122b4ad28809a3420982341241421348097878e577c991de8f0
	  host = api.development.push.apple.com
	  apns-id = eabeae54-14a8-11e5-b60b-1697f925ec7b //可以不填，如果不填APNs会自己创建一个UUID并在response中返回
	  apns-expiration = 0
	  apns-priority = 10
	DATA
	  + END_STREAM
	    { "aps" : { "alert" : "Hello" } }
	```
	
- 基于token方式的request：

	```
	HEADERS
	  - END_STREAM
	  + END_HEADERS
	  :method = POST
	  :scheme = https
	  :path = /3/device/00fc13adff785122b4ad28809a3420982341241421348097878e577c991de8f0
	  host = api.development.push.apple.com
	  authorization = bearer eyAia2lkIjogIjhZTDNHM1JSWDciIH0.eyAiaXNzIjogIkM4Nk5WOUpYM0QiLCAiaWF0I
	 jogIjE0NTkxNDM1ODA2NTAiIH0.MEYCIQDzqyahmH1rz1s-LFNkylXEa2lZ_aOCX4daxxTZkVEGzwIhALvkClnx5m5eAT6
	 Lxw7LZtEQcH6JENhJTMArwLf3sXwi
	  apns-id = eabeae54-14a8-11e5-b60b-1697f925ec7b
	  apns-expiration = 0
	  apns-priority = 10
	  apns-topic = <MyAppTopic>
	DATA
	  + END_STREAM
	    { "aps" : { "alert" : "Hello" } }

	```
	
- 失败后的response:

	```
	HEADERS
	  - END_STREAM
	  + END_HEADERS
	  :status = 400
	  content-type = application/json
	    apns-id: <a_UUID>
	DATA
	  + END_STREAM
	  { "reason" : "BadDeviceToken" }
	```
	
- 成功后的response：

	```
	HEADERS
	  - END_STREAM
	  + END_HEADERS
		apns-id = eabeae54-14a8-11e5-b60b-1697f925ec7b
		:status = 200
	``` 

## 四、设备token的生成与分发
在app启动的时候，必须向iOS系统注册远程推送，成功后，苹果将会返回一个设备token给app，此时app就可以将这个token上报给自己的后台。

如果有必要产生一个新的token，APNs会使用设备证书生成一个token（其中包含了一个设备ID），并使用token key加密后返回给设备。同时设备会将这个token以`NSData`对象的形式返回给app，app获取到该token之后应该将其发送到自己后台，后台之后就可以通过这个token来发送推送数据。过程如下图：

![Managing the device token](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_generation_2x.png)

## 最后
通过苹果的官方文档我们可以知道provider与APNs的交互过程中，需要注意一下几点：

- 推荐使用`HTTP/2`协议。
- 必须加入`TLS`层。
- 基于JWT的方式，token的最大有效期为1小时，并且不能频繁更换token。
- 不能频繁创建、关闭连接，应该尽量少开连接，如果过于频繁，APNs将把其当做是黑客攻击，但是如果数据量大，可以同时多个连接向APNs发送消息。
- 吊销token或者证书的时候，应该及时关闭老的连接。

## 参考文献
1. [APNs Overview](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)
2. [Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW2)




