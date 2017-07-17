---
title: Xcode9主要的新特性
---

# 前言
本文主要列出了Xcode9新加的3个主要功能，这些功能不仅能提高开发效率，同时也能增加代码的鲁棒性。

# 无线连接开发设备

- 进入Window > Devices and Simulators。
- 连接你的设备（比如iPhone）。
- 勾上“Connect via network”。
- 连接成功后，设备右侧会有一个图标。
- 也可以指定ip，右键的设备，选择“Connect via ip address”

![](https://help.apple.com/xcode/mac/9.0/en.lproj/Art/debug_network_iPhone_connect.png)
![](https://help.apple.com/xcode/mac/9.0/en.lproj/Art/debug_network_iPhone_connected.png)

# git flow
### GitHub支持。
- 直接从Xcode中搜索GitHub项目并clone。
![](http://uploadimg.szrd.phiwifi.com/jx6dix3ud8.png)

- GitHub官网上可以直接用Xcode打开。
![](http://uploadimg.szrd.phiwifi.com/inpbt9spcu.png)

### 新加入的Source Control navigator
- 更加方便的查看分支以及提交信息。
- 右键可在直接从某个分支创建子分支。
![](http://uploadimg.szrd.phiwifi.com/pqrpnreup9.png)

- 可在某个commit上创建tag。
![](http://uploadimg.szrd.phiwifi.com/x46dwmh11y.png)

- tag可以分组，便于管理。
![](http://uploadimg.szrd.phiwifi.com/inpbt9spcu.png)

# 调试相关
### Xcode Runtime Tools
- **Main Thread Checker**：在运行时检查该在主线程运行的代码是否正确的执行。比如：UI的操作需要在主线程执行，如果不在主线程执行会报相应的错误。在scheme中开启/关闭（默认开启），如下图：
![](http://uploadimg.szrd.phiwifi.com/9uph4p57wu.png)
![](http://uploadimg.szrd.phiwifi.com/rbt3u8ypfv.png)

- **Address Sanitizer**：检查内存是否是否超出了作用域，当代码使用了超出作用域的内存，会报相应错误。在scheme中开启/关闭（默认关闭），如下图：
![](http://uploadimg.szrd.phiwifi.com/4c6xcaymt1.png)
![](http://uploadimg.szrd.phiwifi.com/atvdwob1xd.png)

- **Undefined Behavior Sanitizer**：值得如潜在的整型溢出、数据越界等，下图列出了一些属于Undefined Behavior的异常。
![](http://uploadimg.szrd.phiwifi.com/eau1t0aq5e.png)
![](http://uploadimg.szrd.phiwifi.com/cztgud12pu.png)

---
