---
title: macOS应用上架被拒
date: 2017-07-27 16:26:58
tags: [macOS]
categories: [macOS开发]
toc: true
---

这是我开发的第一个macOS客户端，名字为[速记](http://markmiao.com/2017/07/26/stenonote/)。是用swift3.0，面向Google开发，磕磕绊绊的写完这一两个界面，相比预想功能大概实现了50%，也算完整，没有完成到预想效果，便按耐不住上传App Store，日后迭代呗。

<!--more-->

结果昨天上传，今天早上就被苹果爸爸拒了。一看被拒理由，哐哐哐列了四五条之多，一下子有点懵。详细看过之后，发现这些问题还是蛮不错的，都是我没注意到的，特此记录下来：

## 问题1：应用程序的完整性

问题描述：

```
Specifically, the app shows no response when click on the menubar extra icon. 
```

这个问题拒的我完全没脾气，因为点击一个按钮没有任何反应，苹果据此认为该应用没做完，我竟毫无反驳之力。

情况是这样的：这个`extra icon`，在有选中内容时，点击会弹出NSPopover，未选中内容时，点击没有任何反应，所以程序不完整。记住即便是空内容也要做提示处理！

## 问题2：应用程序名称不统一

问题描述：

```
We noticed that your app name to be displayed on the App Store does not sufficiently match the name of the app displayed when installed on macOS.

iTunes Connect Name: 速记
App Name when Installed: StenoNote
App Name when Launched: 速记
App Name in About/Hide/Quit Menu: 速记
```

在iOS开发中，工程名和应用名是完全不同的，应用名可以在`Info.plist`里的`Bundle display name`中自定义。但是macOS不同，即便是在`Bundle display name`自定义了名字，在Dock上鼠标悬浮到应用上时和Launchpad里应用图标下显示的名字仍然是工程名。还有在Mac左上角苹果图标旁边的应用名也是工程名，即便是你在xib或storyboard里改了这里的文字，这里依旧不会变。

我并不知道如何修改这些名字，后来在`Info.plist`里修改了`Bundle name`，Mac左上角苹果图标旁边的应用名改变了，但是Dock上和Launchpad里面的没有变化。提交应用之后，这也成了被拒理由之一。

解决办法当然是统一应用名称：

在`TARGETS -> Build Settings -> Product Name`这里面自定义应用名称。

## 问题3：权限配置问题

问题描述：

```
Your app uses one or more entitlements which do not have matching functionality within the app. Apps should have only the minimum set of entitlements necessary for the app to function properly. Please remove all entitlements that are not needed by your app and submit an updated binary for review, including the following:

com.apple.security.files.user-selected.read-only	
```

macOS客户端要想上架App Store，必须开启`App Sandbox`功能。这里面牵扯到一些权限配置：网络访问、硬件资源、联系人、定位、日历，还有文件和目录的访问。应用中不需要的权限一定不要打开，不然就会以上述理由被拒。

我不知道什么时候开启了`User Selected File`为`Read Only`，被拒了，马上改为`None`。

## 问题4：黑暗模式

问题描述：

```
The user interface of your app is not consistent with the macOS Human Interface Guidelines. Specifically:

We found that when Dark Mode is enabled, the menu bar extra icons aren't visible.
```

这个问题略惊，之前恍惚听过黑暗模式，从来没用过。启动黑暗模式是在：系统偏好设置 -> 通用 -> 勾选使用暗色菜单和Dock

![设置黑暗模式](http://oalg33nuc.bkt.clouddn.com/2017-07-27-17-20-20.png)

黑暗模式下Dock和上方工具条都变为黑色半透明，工具条上的图标齐刷刷变为白色，而我的应用图标不见了...不见了...见了...了...

在stackoverflow找到一个问题解答：[How to detect dark mode in Yosemite to change the status bar menu icon](https://stackoverflow.com/questions/25379525/how-to-detect-dark-mode-in-yosemite-to-change-the-status-bar-menu-icon)。

设置分布式观察者，获取Mac模式变化：

```Swift
DistributedNotificationCenter.default().addObserver(self, selector: #selector(changeStatusBarImage(not:)), name: NSNotification.Name(rawValue: "AppleInterfaceThemeChangedNotification"), object: nil)
```

每次更改模式都会调用通知方法：

```Swift
func changeStatusBarImage(not: Notification) {
		print("change")
}
```

还可以使用下面方法获取当前Mac的模式：

```Swift
let dict = UserDefaults.standard.persistentDomain(forName: UserDefaults.globalDomain)
let style = dict?["AppleInterfaceStyle"]
print(style)//是暗黑模式下style打印Optional(Dark)，普通模式下打印nil
```

stackoverflow问题解答中有一个外链，说明了如果不是根据模式切换去更换复杂图片的话，仅是白变黑，黑变白，仅需要如下几行代码设置`NSStatusBar`的图片即可：

```Swift
let image:NSImage = NSImage(named: "图片名称")!
image.isTemplate = true
item.button?.image = image
```

## 问题5：无菜单重新打开主窗口

问题描述：

```
The user interface of your app is not consistent with the macOS Human Interface Guidelines.

Specifically, we found that when the user closes the main application window there is no menu item to re-open it.
```

对于这个问题我是存在异议的，在点击关闭按钮后主窗口退出，点击Dock上的应用图标是可以唤起应用主窗口的。但是文中强调`menu item`，难道必须右键菜单里需要加上打开客户端主窗口的操作？

暂且在右键菜单里添加了打开客户端主窗口的功能：

```Swift
func applicationDockMenu(_ sender: NSApplication) -> NSMenu? {
   let menu = NSMenu()
   let menuItem = NSMenuItem(title: "打开速记", action: #selector(openNoteViewController), keyEquivalent: "O")
   menu.addItem(menuItem)
   return menu
}
```

写这篇文章的时候，应用经过上述修改再一次提交审核，等待审核中，期待审核结果……

----

后面的审核历程颇有些戏剧性，当晚审核被拒，被拒原因是上面的`问题1`和`问题5`，说是在macOS10.12.6系统版本下出现的问题，这是当前macOS最新系统版本，我也是在此系统版本下开发并修改上述问题的。

本着苹果爸爸不欺我的崇拜之心，经过五分钟的测试+Google，最终还是没发现问题所在。十分钟后我反馈了我的疑问，第二天苹果给我的回复是：我们重新测试了你的应用，发现在macOS10.12.6上没有问题，问题是在macOS10.10.5上发现的，你的应用最低系统支持是10.10，所以要解决10.10.5上的问题。

有理有据，合情合理。

10.10.5上有问题，那就不支持10.10了，最低系统版本改为10.11，上架成功。


