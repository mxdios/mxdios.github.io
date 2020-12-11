---
title: Vue与App交互
date: 2020-12-11 14:31:49
tags: [vue]
categories: [iOS开发]
toc: true
---

app开发经常会内嵌h5页面，原生app与h5页面相互调用，完成数据传递。一套h5页面能多端运行，实现跨平台，这就是常见的Hybrid app（混合app）

<!--more-->

## WKWebView

### 通过 url 向 vue 传参

在原生WKWebView中加载URL时，以拼接方式向h5页面传参：

```swift
let urlStr = "https://markmiao.com/app/request?data=123"
let request = URLRequest(url: URL(string: urlStr)!)
webView.load(request)
```

vue中接收传递数据：

```vue
this.data = this.$route.query.data
```

也可以通过`&`符号拼接多个参数传递：

```swift
let urlStr = "https://markmiao.com/app/request?data=123&index=1"
```

URL是有长度限制的，如果要传递的数据很多，可以采用`evaluateJavaScript`，在原生中调用vue中的js方法传递数据。

### evaluateJavaScript 向 vue 传参

`evaluateJavaScript`接收的是一个字符串方法名，传参时需要将json对象的参数转为json字符串，用单引号包裹放到字符串方法中：

```swift
let funStr = "vueFunctionStr('\(jsonStr)')" //vueFunctionStr是vue中的方法名，jsonStr是json字符串的传参
self.webView.evaluateJavaScript(funStr) { (any, error) in
    if error != nil {
        XDLog("调用失败 = \(error.debugDescription)")
    }
}
```

vue的方法接收数据：

```js
mounted () { // 在window中注册方法
  window.vueFunctionStr = this.vueFunctionStr
}

vueFunctionStr (jsonStr) {
  let jsonDict = JSON.parse(jsonStr)
  console.log(jsonDict)
},
```

### vue 向 WKWebView传参

h5也可以通过调用原生方法传参，`WKWebView`需要添加方法监听，`vueFunction`是要监听的方法名。

在 `WKScriptMessageHandler`方法中接收传参，`message.name`是方法名，`message.body`是传递参数：

```swift
// 添加监听
override func viewWillAppear(_ animated: Bool) {
		super.viewWillAppear(animated)
		self.webView.configuration.userContentController.add(self, name: "vueFunction")
}
// 移除监听
override func viewWillDisappear(_ animated: Bool) {
		super.viewWillDisappear(animated)
    self.webView.configuration.userContentController.removeScriptMessageHandler(forName: "vueFunction")
}

extension WebViewController: WKScriptMessageHandler, WKNavigationDelegate {
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        XDLog("vue调用原生 = \(message.name), \(message.body)")
    }
  
	  // 获取h5网页title
  	func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        webView.evaluateJavaScript("document.title") { (title, error) in
            if let titleStr = title as? String {
                self.navigationItem.title = titleStr
            } else {
                self.navigationItem.title = ""
            }
        }
    }
}
```

在vue中通过`window.webkit`调用原生方法，传参：

```js
if (window.webkit) {
	// vueFunction 是在WKWebView中监听的方法名，postMessage后面是要传递的json对象参数
  window.webkit.messageHandlers.vueFunction.postMessage({
    data: 123
  })
}
```

## uni-app

uni-app是一款跨平台框架，可以以一套代码编译成iOS/Android/各种小程序，一套代码多端运行。

uni-app完全是vue的开发方式，API高度类似小程序，相比原生性能和拓展性都是问题。框架本身也存在一些bug，好在官方迭代很快。

### 通过 url 向 vue 传参

uni-app有`web-view`组件，用来加载网页，可以通过`src`的URL地址向vue传参：

```vue
<web-view src="https://markmiao.com/app/request?data=123"></web-view>
```

vue接收参数和上面一样，也是通过`this.$route.query.data`

### vue 向 uni-app 传参

参考文章：[在web-view加载的本地及远程HTML中调用uni的API及网页和vue页面通讯](https://ask.dcloud.net.cn/article/35083)

vue 向 uni-app 传参需要引入uni-app提供的sdk，vue中只需要在`public/index.html`文件中引入即可：

```html
<script type="text/javascript" src="https://js.cdn.aliyun.dcloud.net.cn/dev/uni-app/uni.webview.1.5.1.js?rev=1.0.0"></script>
```

如果要兼容小程序，还需要引入微信js-sdk，如果引入需要在前面：

```html
<script type="text/javascript" src="//res.wx.qq.com/open/js/jweixin-1.4.0.js"></script>  
```

在vue中调用`uni.postMessage`方法向uni-app传参：

```js
uni.postMessage({
  data: {
    to: 'appFunction', //方法名
    data: { //传递参数
      data: 1
    }
  }
})
```

在uni-app中的 `getMessage`方法中得到传参：

```js
getMessage: function (e) {
    var dataDict = e.detail.data[0]
    console.log("方法名 = " + dataDict.to + "传参 = " + dataDict.data)
}
```