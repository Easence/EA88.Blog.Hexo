---
title: ffmpeg-iOS
categories:
tags:

---

## 关于FFMPEG的编译
### 编译FFMPEG
运行一下两个命令：

```
./configure
make
```
### 编译iOS使用版本

使用[FFmpeg-iOS-build-script](https://github.com/kewlbear/FFmpeg-iOS-build-script)这个shell脚本快速搞定。

### 编译/doc目录下的文档
可以将/doc目录下的texi编译成html，运行一下两个命令：

```
./configure
make doc
```

##依赖库
- **libz.tbd** : libz提供了一套与gzip有关的API
- **libbz2.tbd** : libbz2提供了一套与bzip2有关的API
- **libiconv.tbd** : 字符编码转换

