---
title: 跨域导致iframe无法使用Cookie
date: 2021-09-18 14:30:50
tags: [web]
categories: [web]
toc: true
---

在工作中，有一个项目需要在我们的平台内嵌其他平台，用户信息需要共享。在使用iframe内嵌时，发现内嵌的页面用户校验失败。两个平台系统没有问题，单独访问时都可以。

测试时Chrome有问题，使用Firefox可以，Firefox更新之后有同样问题。那就是浏览器的问题了。

<!--more-->

## 问题原因

Chrome 80版本默认屏蔽了第三方Cookie，SameSite属性设置为Lax，防止跨域传送Cookie，防止 CSRF 攻击和用户追踪。[SameSite属性介绍](http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)

在接口请求响应头`Set-Cookie`有个提示：`此Set-Cookie标头未指定"SameSite"属性,它默认为"SameSite=Lax,"因为它来自跨站点响应，而不是对顶级导航的响应，必须为此此Set-Cookie设置"SameSite=None"才能实现跨站点使用`

当设置`SameSite=None`时，必须同时设置`Secure`属性才有效，而且必须通过https发送cookie。

## 项目背景

我们的A平台，集成了B平台模块，A平台在iframe嵌套B平台时通过`url`将用户`token`传给B，B有用户`token`后，有自成一套的用户逻辑。

B平台使用`session`进行用户鉴权，会产生一条session id：`PHPSESSID`，这个会存储在cookie中，每次接口请求时由此鉴权。

`PHPSESSID`这条cookie是后端php项目自动生成的，这就很难改它的`SameSite`和`Secure`

## 解决方法

**先说方案：直接问题没解决，从根本解决了，B平台取消session鉴权方案，采用和A平台一致的token鉴权，真釜底抽薪**

**另一种方案：两平台放到同一域下，不跨域自然没有该问题了。**

网上有很多临时解决办法，比如修改`chrome://flags/`里的配置，手动设置这条cookie的`SameSite`和`Secure`，这些在开发时可作为临时手段，生产环境就扯淡了。

生产环境需要通过代码修改`SameSite`和`Secure`，我是个php纯小白，来解决这个问题：

### 修改服务端源码

在php服务端代码里设置：

```php
if (isset($_COOKIE["PHPSESSID"])) {
   setcookie("PHPSESSID", $_COOKIE["PHPSESSID"], ['samesite' => 'None', 'secure' => true]);
}
```

这个方式是可以将PHPSESSID这条cookie的SameSite设置为None，Secure设置为true。之后iframe嵌套也完全可以发送cookie。

前提是这段代码加到哪里？必须在`$_COOKIE["PHPSESSID"]`有值时才能修改属性。PHPSESSID是自动产生的，说是在`session_start();`方法调用后创建，但我加在这后面也完全无效。

### 设置header

在根目录的`index.php`文件中设置header

```php
header('Set-Cookie:'.'PHPSESSID='.session_id().';Path=/;SameSite=None;Secure=true;');
```

和上面同样的问题，`session_id()`根本不能获取到实际的sessionid。

### 设置session属性

```php
ini_set('session.cookie_secure', 'On');
ini_set('session.cookie_samesite', 'None');
```

这个方法在php的`index.php`文件中添加过，在php的配置文件`php.ini`中也添加过：

```php
session.cookie_samesite = "None"
session.cookie_secure = On
```

### 设置Nginx配置项

```nginx
proxy_cookie_path / "/; httponly; secure; SameSite=None";
```

在nginx配置项里增加这条配置，还是没能解决。

### 设置响应头

```js
response.setHeader('Set-Cookie', 'HttpOnly;Secure;SameSite=None')
```

这个没有试验。

## 参考文章

[chrome浏览器跨域Cookie的SameSite问题导致访问iframe内嵌页面异常](https://blog.csdn.net/yhyc812/article/details/108623844)

