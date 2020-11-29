---
title: framework合并问题
date: 2020-11-29 17:21:55
tags: [静态库]
categories: [iOS开发]
toc: true
---

上一篇关于静态库的文章：[制作.a和.framework静态库](https://markmiao.com/2016/11/14/%E5%88%B6%E4%BD%9C-a%E5%92%8C-framework%E9%9D%99%E6%80%81%E5%BA%93/)。文章很老了，有许多问题，最近用Xcode12打包framework，遇到了一些问题，记录一下。
<!--more-->

打包静态库常用脚本配置的方式，在Run Script添加脚本。编译之后，真机和模拟器的framework会完成合并。但Xcode升级之后脚本失效了，则分别打包，手动完成合并。

在编译时，要选中包的TARGET，分别在真机和模拟器下完成编译。在Products里面Show in Finder便能看到完成打包的framework文件。

xxx.framework文件夹中xxx文件，没有后缀名且体积最大，这就是静态库文件。

查看静态文件支持架构：

```sh
lipo -info 静态库xxx文件路径
```

真机：arm64 armv7， 模拟器：arm64 x86_64 i386 

静态文件合并命令：

```shell
lipo -create 真机xxx文件路径 模拟器xxx文件路径 -output 新文件路径
```

Xcode升级12之后，报错了：

```
have the same architectures (arm64) and can't be in the same fat output file
```

因为两个静态文件都支持`arm64`，无法合并。

可以移除模拟器静态文件的`arm64`架构：

```sh
lipo 模拟器静态文件xxx -remove arm64 -output 移除后模拟器静态文件xxx名
```

移除之后，重新查看支持架构，只有`armv7`，重新合并就没有问题了。

网上对于该问题的一些解决办法：

[iOS 14, lipo error while creating library for both device and simulator](https://stackoverflow.com/questions/64022291/ios-14-lipo-error-while-creating-library-for-both-device-and-simulator)

[XCode12 模拟器静态库支持arm64架构引发的系列问题](https://blog.csdn.net/huawt520/article/details/109305833)

