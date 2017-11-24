---
title: RxSwift学习笔记(1)
date: 2017-11-24 14:32:44
tags: [swift, RxSwift]
categories: [iOS开发]
toc: true
---

[RxSwift](https://github.com/ReactiveX/RxSwift)，是Swift的函数响应式编程框架，以函数为工具，绑定数据联动，数据改变实时反映到结果呈现。这对我来说是一种全新的编程思想，我想去了解，学习。

本文记录了我学习RxSwift的历程，包含遇到的各种问题和我的一些理解。

<!--more-->

## 运行官方示例

从github上克隆RxSwift项目

```
git clone git@github.com:ReactiveX/RxSwift.git
```

双击打开`Rx.xcworkspace`，在工程中选择`RxExample-iOS`或`RxExample-macOS`，选择模拟器或真机，运行。

## 项目集成RxSwift

将RxSwift集成到项目中，推荐使用CocoaPods的方式，pod版本`1.3.1`亲测有效：

- 创建项目时勾选`Include Unit Tests`，包含单元测试。

- 创建`Podfile`文件，输入如下内容：

```
# Podfile
use_frameworks!

target '工程名字' do
    pod 'RxSwift',    '~> 4.0'
    pod 'RxCocoa',    '~> 4.0'
end

# RxTests 和 RxBlocking 将在单元/集成测试中起到重要作用
target '单元测试文件夹名字' do
    pod 'RxBlocking', '~> 4.0'
    pod 'RxTest',     '~> 4.0'
end
```

- 将工程名和单元测试名换成自己的，执行命令：

```
pod install
```

### 问题

问题1. 可能由于CocoaPods版本过低，pod失败，建议更新后重试，执行更新命令：

```
sudo gem install cocoapods
```

问题2. 

```
None of your spec sources contain a spec satisfying the dependency: `RxSwift (~> 4.0)`.
```

执行命令：

```
pod setup
```

## 实践

RxSwift的本质是观察者模式，时刻观察者一个序列，当序列达到预定条件，执行某种操作。比如钟表时间是一个序列，当到6点时就下班。时间是一个被观察的序列，6点是预定条件，下班是操作。

Rx中用`observable`表示变化序列，也就是被观察者。到达预定条件的操作用`subscribe`表示，被称为订阅。订阅完成之后，要对其进行清理，清理方式是丢掉处理袋`DisposeBag`中。

这是一个基本的Rx执行流程。

### 具体实现

用RxSwift写一个基本的按钮点击事件。在`ViewController`中引入头文件：

```Swift
import RxSwift
import RxCocoa
```

在`Main.storyboard`中拖入按钮，并连线到`ViewController`：

```Swift
@IBOutlet weak var button: UIButton!
```

创建全局变量处理袋：

```Swift
private let disposeBag = DisposeBag()
```

给按钮添加点击事件：

```Swift
button.rx.tap
	.subscribe(onNext: {
		print("点击")
	}).disposed(by: disposeBag)
```

完整代码：

```Swift
import UIKit
import RxSwift
import RxCocoa

class ViewController: UIViewController {
    @IBOutlet weak var button: UIButton!
    private let disposeBag = DisposeBag()
    override func viewDidLoad() {
        super.viewDidLoad()
        button.rx.tap
            .subscribe(onNext: {
                print("按钮点击")
            }).disposed(by: disposeBag)
    }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
```


## 其他

demo地址：[XDRxSwiftLearn](https://github.com/mxdios/XDRxSwiftLearn)

学习资料：

[靛青K神出品RxSwift 学习指导索引](http://t.swift.gg/d/2-rxswift)

[RxSwift中文本文档](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/)

[RxSwift入门](https://darkhandz.com/categories/iOS/Swift/)

