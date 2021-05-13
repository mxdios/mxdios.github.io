---
title: iOS开发笔记四
date: 2021-05-13 14:31:49
tags: [iOS]
categories: [iOS开发]
toc: true
---

## MJRefresh跳动bug

iOS11之后，UITableView在刷新列表时`contentOffset`会发生变化，MJRefresh监听了`contentOffset`变化，会导致上拉加载更多组件闪动出现。我升级到最新版本`v3.5.0`，问题依旧存在，等官方修复。暂时解决办法：

<!--more-->

```swift
tableView.estimatedRowHeight = 0
tableView.estimatedSectionHeaderHeight = 0
tableView.estimatedSectionFooterHeight = 0
```

## 系统拍照裁剪不可拖动

我们在使用`UIImagePickerController`做照片选择器时，`sourceType = .camera`去拍照，设置`allowsEditing = true`拍完照的时候，可拖动照片裁剪为正方形，再使用裁剪后的照片。

官方bug，裁剪时无法拖动。这个bug从iOS6到现在的iOS14一直存在，不知道apple啥时候能解决。

问题讨论见：[UIImagePicker allowsEditing stuck in center](https://stackoverflow.com/questions/12630155/uiimagepicker-allowsediting-stuck-in-center)

解决方法：通过`UIImagePickerController`的拓展，重置`UIScrollView`的`contentOffset` 

```swift
/// 修复拍照切割时不可拖动到边的系统bug
extension UIImagePickerController {
    func fixCannotMoveEditingBox() {
        if let cropView = cropView,
           let scrollView = scrollView,
           scrollView.contentOffset.y == 0 {
            let top = cropView.frame.minY //+ self.view.safeAreaInsets.top
            let bottom = scrollView.frame.height - cropView.frame.height - top
            scrollView.contentInset = UIEdgeInsets(top: top, left: 0, bottom: bottom, right: 0)
            
            var offset: CGFloat = 0
            if scrollView.contentSize.height > scrollView.contentSize.width {
                offset = 0.5 * (scrollView.contentSize.height - scrollView.contentSize.width)
            }
            scrollView.contentOffset = CGPoint(x: 0, y: -top + offset)
        }
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) { [weak self] in
            self?.fixCannotMoveEditingBox()
        }
    }
    
    var cropView: UIView? {
        return findCropView(from: self.view)
    }
    
    var scrollView: UIScrollView? {
        return findScrollView(from: self.view)
    }
    func findCropView(from view: UIView) -> UIView? {
        let width = UIScreen.main.bounds.width
        let size = view.bounds.size
        if width == size.height, width == size.height {
            return view
        }
        for view in view.subviews {
            if let cropView = findCropView(from: view) {
                return cropView
            }
        }
        return nil
    }
    func findScrollView(from view: UIView) -> UIScrollView? {
        if let scrollView = view as? UIScrollView {
            return scrollView
        }
        for view in view.subviews {
            if let scrollView = findScrollView(from: view) {
                return scrollView
            }
        }
        return nil
    }
}
```

在创建`UIImagePickerController`时，调用拓展方法：

```swift
let imagePicker = UIImagePickerController()
imagePicker.sourceType = .camera
imagePicker.fixCannotMoveEditingBox()
imagePicker.allowsEditing = true
present(imagePicker, animated: true, completion: nil)
```



