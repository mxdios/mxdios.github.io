---
title: macOS开发笔记(1)
date: 2017-12-07 16:05:19
tags: [macOS]
categories: [macOS开发]
toc: true
---

旧坑不填，喜开新坑。这两天又计划开发一款macOS应用，之前开发的[速记](http://markmiao.com/2017/07/26/stenonote/)完全是练手，反正现在是不想填坑了，索性再开一个。实际做一个项目，是学习的最佳途径。本文记录了我在开发中遇到的各种问题，以及找到的解决办法，权当以后查阅的笔记。如果能给某位朋友带来帮助，是我的荣幸。如果发现问题，敬请评论吐槽。

<!-- more -->

开发环境：**Xcode 9.1 + Swift4**

## StatusBar

### 设置按钮

在`applicationDidFinishLaunching`方法中调用下面代码，设置StatusBar按钮

```Swift
func createButtonStatusBar() {
   let statusBar = NSStatusBar.system
   let item = statusBar.statusItem(withLength: NSStatusItem.squareLength)
   item.button?.target = self
   item.button?.action = #selector(itemActionShowView(_:))
   let image:NSImage = NSImage(named: NSImage.Name(rawValue: "mremindbaricon"))!
   image.isTemplate = true
   item.button?.image = image
   statusItem = item;
}
@objc func itemActionShowView(_ sender: NSStatusBarButton) {
   print("点击")
}
```

`itemActionShowView`是实现的点击方法，Swift4.0之后，必须增加`@objc`修饰符，是因为Swift4.0之后去掉了Swift3.x对隐式类型推断的特性，必须手动管理`@objc`，保证oc和Swift代码能互相调用。

### 设置菜单

在`Main.storyboard`的`Application Scene`中拖入`NSMenu`，将`NSMenu`拖线到`AppDelegate`中。如同上面设置statusBar按钮一样，不需要设置按钮的`target`和`action`，需要设置`menu`。

```Swift
@IBOutlet weak var toolMenu: NSMenu!

...

item.menu = toolMenu
```

![添加NSMenu](http://oalg33nuc.bkt.clouddn.com/2017-12-04-15-48-40.png)

添加`NSMenu`的点击事件，直接拖线`action`即可。

## Dock和Window

### 隐藏Dock上的应用图标

开发statusBar应用，不需要弹出Window界面，可以在`Main.storyboard`中的`Window Controller Scene`中取消`Is Initial Controller`的勾选。

不需要在Dock上出现应用图标，`Info.plist`中添加Key - Value：`Application is agent (UIElement)` - `YES`


### Window居中

```Swift
window.center()//居中
```

### Window的层级关系

Window有一个`level`属性，标识Window的层级关系。设置Window的`level`：

```Swift
self.window?.level = .floating //Window设置为浮动的
```

还有好多其他层级关系，诸如`normal`、`submenu`、`tornOffMenu`......

### 启动Window

弹出一个Window页面，并将其设置为`keyWindow`（第一响应者的Window窗口）：

```Swift
//将addWindowController.window启动，并设置为keyWindow
addWindowController.window!.makeKeyAndOrderFront(self)
```

有时候该方法只能将Window显示出来，并不能设置为keyWindow，如果是`normal`层级的Window，Window都不能覆盖其他应用窗口，只会在其他应用窗口下面显示。特别是statusBar应用，在其他应用窗口为`keyWindow`时，点击工具条上的statusBar，启动Window，无法完成`keyWindow`设置。

需要调用下面代码，将其他应用都取消`keyWindow`

```Swift
NSApp.activate(ignoringOtherApps: true)
```

## NSTextField

### 输入时隐藏蓝色边框

`xib`中在`Focus Ring`中选择`None`

![隐藏蓝色边框](http://oalg33nuc.bkt.clouddn.com/2017-12-05-10-45-36.png)

或者在代码中执行：

```Swift
textField.focusRingType = .none
```

### 文本框第一响应者

```Swift
self.window?.makeFirstResponder(textField)//设置为第一响应者
self.window?.makeFirstResponder(nil) //取消第一响应者
```

## 获取版本号

### macOS版本号

macOS10.10及以上，使用下面的方法：

```Swift
let version = ProcessInfo.processInfo.operatingSystemVersion
print("\(version.majorVersion).\(version.minorVersion).\(version.patchVersion)")
```

还有如下方式（已弃用）

```Swift
#import <CoreServices/CoreServices.h>

SInt32 major, minor, bugfix;
Gestalt(gestaltSystemVersionMajor, &major);
Gestalt(gestaltSystemVersionMinor, &minor);
Gestalt(gestaltSystemVersionBugFix, &bugfix);
print("\(major).\(minor).\(bugfix)")
```

其他方式请见：[查找Mac OS X版本号](https://stackoverflow.com/questions/6492038/find-mac-os-x-version-number-in-objective-c)

### app版本号

```Swift
let version = Bundle.main.object(forInfoDictionaryKey: "CFBundleShortVersionString")
let bundle = Bundle.main.object(forInfoDictionaryKey: "CFBundleVersion")
```

## 设置快捷键和关闭退出应用

### 快捷键

xib或storyboard设置快捷键很方便，直接在`Key Equivalend`输入快捷键即可。

![设置快捷键](http://oalg33nuc.bkt.clouddn.com/2017-12-07-15-42-28.png)

### 退出应用

从`action`拖线到`Application`，选择`stop:`方法即可：

![拖线到Application](http://oalg33nuc.bkt.clouddn.com/WX20171207-154727.png)

![选择stop:方法](http://oalg33nuc.bkt.clouddn.com/2017-12-07-15-48-21.png)

或者拖线到`First Responder`，选择`terminate:`方法

![拖线到First Responder](http://oalg33nuc.bkt.clouddn.com/WX20171207-155548.png)

或者调用代码：

```Swift
NSApplication.shared().terminate(self)
```

### 关闭应用

拖线到`First Responder`，选择`performClose: `方法

## 其他操作

### 发送邮件

使用`NSSharingService`调用macOS邮件应用，发送邮件。

```Swift
let emailService = NSSharingService(named: NSSharingService.Name.composeEmail)!
emailService.recipients = ["i@markmiao.com"]
emailService.subject = "邮件内容"
if emailService.canPerform(withItems: ["邮件内容"]) {
  emailService.perform(withItems: ["邮件内容"])
} else {
  print("没配置邮件账户")
}
```

### 在浏览器中打开网页

```Swift
NSWorkspace.shared.open(URL(string: "http://markmiao.com")!)
```


