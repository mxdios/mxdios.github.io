---
title: macOS开发笔记(2)
date: 2020-04-29 06:05:19
tags: [macOS]
categories: [macOS开发]
toc: true
---

## NSToolbar

常用`NSToolbar`开发偏好设置界面，在xib中可以直接拖`NSToolbar`到`Window`中，可以自定义图片文字。

<!-- more -->

勾选`Behavior`中的`Selectable`，`NSToolbarItem`是可选中样式的。

设置某个`NSToolbarItem`默认选中，需要先设置默认选中的`NSToolbarItem`的`Identifier`，使用下面代码完成默认选中：

```Swift
toolbar.selectedItemIdentifier = NSToolbarItem.Identifier(rawValue: "id")
```

## Window按钮

控制window左上角关闭、最小化、最大化按钮的是否可用，以及显示隐藏。

可以在xib中勾选`Controls`中的`Close`、`Minimize`、`Resize`，控制按钮是否可用。或者使用代码：

```Swift
self.window?.standardWindowButton(.closeButton)?.isEnabled = false
```

使用代码可以隐藏按钮：

```Swift
self.window?.standardWindowButton(.miniaturizeButton)?.isHidden = true
self.window?.standardWindowButton(.zoomButton)?.isHidden = true
```

## NSButton

设置按钮为蓝色背景，需要设置按钮的`Key Equivalent`为回车键

## Label

macOS中`Label`继承自`NSTextField`，有两种：`Label` 和`Wrapping Label`

`Wrapping Label`为多行Label，无需设置Line Break为`Character Wrap`

使用`Label` 设置Line Break也无法折行显示。

当`Label`仅作为文字显示时，鼠标移上不变光标显示，需要设置Action为`Sent On End Editing`

## NSTableCellView

移除默认的Cell点击变蓝，设置NSTableView的`selectionHighlightStyle` :

```swift
self.tableView.selectionHighlightStyle = .none
```

NSTableCellView根据存放的Label内容自适应高度，OSX 10.13以后，在xib中设置正确的约束，设置Row自适应高度即可：

```swift
self.tableView.usesAutomaticRowHeights = true
```

无需指定默认高度`rowHeight`，亦无需实现row高度的代理方法`func tableView(_ tableView: NSTableView, heightOfRow row: Int) -> CGFloat`

[基于视图的NSTableView，其行具有动态高度](https://www.itranslater.com/qa/details/2325926279662535680)

## NSTextField

限制输入框`NSTextField`的输入内容，仅允许输入数字，需要自定义`Formatter`

```swift
import Cocoa

class OnlyIntegerValueFormatter: NumberFormatter {
    override func isPartialStringValid(_ partialString: String, newEditingString newString: AutoreleasingUnsafeMutablePointer<NSString?>?, errorDescription error: AutoreleasingUnsafeMutablePointer<NSString?>?) -> Bool {
        if partialString.isEmpty {
            return true
        }
        // 禁止输入非数字 && 首位不能为0 && 最多输入3位
        return Int(partialString) != nil && Int(partialString)! > 0 && partialString.count < 4
      }
}
```

