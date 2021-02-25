---
title: iOS导航栏一些问题
date: 2021-02-25 14:31:49
tags: [iOS]
categories: [iOS开发]
toc: true
---

## 禁止iOS14返回按钮菜单

iOS14之后，长按返回按钮会弹出可返回的控制器。iOS14，`UIBarButtonItem` 新增了一个`menu`类型，可新建子类，重写`menu`的set方法

<!--more-->

```swift
class BackBarButtonItem: UIBarButtonItem {
    @available(iOS 14.0, *)
    override var menu: UIMenu? {
        set {
            // Don't set the menu here
            // super.menu = menu
        }
        get {
            return super.menu
        }
    }
}
```

使用`BackBarButtonItem`创建`backBarButtonItem` ：

```swift
let backItem = BackBarButtonItem(title: "", style: .plain, target: nil, action: nil)
self.navigationItem.backBarButtonItem = backItem
```

参考链接：[ios - 是否可以在iOS 14+中禁用后退导航菜单？](https://www.coder.work/article/7148498)

## 拦截系统导航栏返回操作

swift通过分类实现：

```swift
/// 导航返回协议
@objc protocol NavigationProtocol {
    /// 导航将要返回方法
    /// - Returns: true: 返回上一界面， false: 禁止返回
    @objc optional func navigationShouldPopMethod() -> Bool
}
extension UIViewController: NavigationProtocol {
    func navigationShouldPopMethod() -> Bool {
        return true
    }
}
extension UINavigationController: UINavigationBarDelegate, UIGestureRecognizerDelegate {
    public func navigationBar(_ navigationBar: UINavigationBar, shouldPop item: UINavigationItem) -> Bool {
        if viewControllers.count < (navigationBar.items?.count)! {
            return true
        }
        var shouldPop = false
        let vc: UIViewController = topViewController!
        if vc.responds(to: #selector(navigationShouldPopMethod)) {
            shouldPop = vc.navigationShouldPopMethod()
        }
        if shouldPop {
            DispatchQueue.main.async {
                self.popViewController(animated: true)
            }
        } else {
            for subview in navigationBar.subviews {
                if 0.0 < subview.alpha && subview.alpha < 1.0 {
                    UIView.animate(withDuration: 0.25) {
                        subview.alpha = 1.0
                    }
                }
            }
        }
        return false
    }
		public func gestureRecognizerShouldBegin(_ gestureRecognizer: UIGestureRecognizer) -> Bool {
        if children.count == 1 {
            return false
        } else {
            if topViewController?.responds(to: #selector(navigationShouldPopMethod)) != nil {
                return topViewController!.navigationShouldPopMethod()
            }
            return true
        }
    }
}
```

在需要拦截返回的控制器中重写该方法：

```swift
override func navigationShouldPopMethod() -> Bool {
	return false //拦截返回
}
```

参考链接：[iOS 拦截系统导航栏返回按钮事件](https://www.cnblogs.com/hedongStudyRecord/p/11120890.html)

## UIBarButtonItem按钮图片颜色

```swift
let image = UIImage(named: "imagename")?.withRenderingMode(.alwaysOriginal) //设置图片mode
self.navigationItem.rightBarButtonItem = UIBarButtonItem(image: image, style: .plain, target: self, action: #selector(itemClick(item:)))
```

## 导航栏透明

```swift
self.navigationBar.setBackgroundImage(UIImage(), for: .default)
self.navigationBar.shadowImage = UIImage()
```

## 自定义导航栏按钮

```swift
let rightBtn = CustomButton(frame: CGRect(x: 0, y: 0, width: 30, height: 40))
rightBtn.addTarget(self, action: #selector(itemDeleteBtnClick), for: .touchUpInside)
self.navigationItem.rightBarButtonItem = UIBarButtonItem(customView: rightBtn)
```

