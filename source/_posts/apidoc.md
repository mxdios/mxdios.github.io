---
title: 用apiDoc写接口文档
date: 2017-11-13 17:44:16
tags: [文档]
categories: [web]
toc: true
---

作为移动端开发，接口文档看得多，写得少。最近对已开发完成的app进行接口文档整理，发现了[apiDoc](http://apidocjs.com/)这款神器。见识到接口文档也可以写的这么高大上，之前用markdown写弱爆了，用word的自杀吧。

<!--more-->

## 安装

在安装`npm`的基础上，执行命令安装apiDoc

```
npm install apidoc -g
```

安装完成，Mac上便具备apiDoc环境，可以开始写文档了。用apiDoc写接口文档就是写注释，支持各种语言的注释，比如：C#、Go、Java、JavaScript、PHP等等。

## 创建配置文件

在任意位置创建存放文档的文件夹，在该文件夹内创建文件`apidoc.json`，这是apiDoc的配置文件。内容示例如下：

```
{
  "name": "文档名称",
  "version": "0.1.0",
  "description": "文档描述",
  "title": "文档网页标题",
  "url" : "https://api.github.com/v1",
  "sampleUrl": "https://api.github.com/v1",
  "header": {
    "title": "页眉",
    "filename": "header.md"
  },
  "footer": {
    "title": "页脚",
    "filename": "footer.md"
  },
  "template": {
  	 "withCompare": true,
  	 "withGenerator": true
  }
}
```

字段解释：

- `version`：文档版本号。
- `url`：域名的主地址，接口中不变的那一块。
- `sampleUrl`：测试API方法接口，如果设置，在每个接口下都有一个测试接口的表单。如果不想显示测试表单，需要在接口文档中增加`@apiSampleRequest off`。
- `header`、`footer`：页眉、页脚，可以不写。里面分别跟着页眉页脚的标题和markdown文件路径，markdown文件和`apidoc.json`同级目录。
- `withCompare`：版本比较，默认开启，`withGenerator`：生成器，页面底部显示apiDoc，默认开启。

## 创建接口源文档

我是以`js`文件写的接口文档，在配置文件`apidoc.json`统计目录下创建接口文件`接口.js`，文件名称自定义。

apiDoc最大的亮点是可以版本比较，接口更改过，可以通过接口右侧的版本号选择进行变化比较。

![版本比较](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/2017-11-14-09-36-28.png)

这需要在源文档中保留每个历史版本，所以每个接口都可能对应N个历史版本，所以建议一个接口用一个文件。

### 内容示例

创建一个`编辑资料.js`文件，并输入以下内容：

```
/**
@api {POST} users/userinfo.asp [编辑资料]
@apiVersion 0.1.0
@apiName userinfo
@apiGroup users
@apiSampleRequest off
@apiDescription 编辑用户资料

@apiParam {String} userid 用户id
@apiParam {String} username 用户名
@apiParam {String} usersex 性别
@apiParam {String} token token

@apiSuccess {Object}   data  对象数据
@apiSuccess {String}   data.userheadimg     头像
@apiSuccess {String}   data.username     用户名
@apiSuccess {String}   data.userphone     手机号
@apiSuccess {String}   data.usersex     性别
@apiSuccess {String}   errorcode  错误码
@apiSuccess {String}   errormessage  信息

@apiExample 返回示例
HTTP/1.0 0 ok
{
    data =     {
        userheadimg = "http://xxxxxxx/api/user/2017/11/13/5EC93450A72741FA92AB4E08D4A96710.jpg";
        username = markmiao;
        userphone = 13000000000;
        usersex = "男";
    };
    errorcode = 0;
    errormessage = "success";
}
*/
```

![部署完成后的显示样式](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/2017-11-14-09-52-10.png)

### 字段解释

文档中所有内容是包含在`/**   */`注释中的，用apiDoc写接口文档就是写注释。

`@api {POST} users/userinfo.asp [编辑资料]` 

- 必需，`{}`内是接口请求方式，GET、POST、PUT等，后面的是接口地址改变部分，会与`apidoc.json`中的`"url"`组成完整接口地址显示。`[]`内是接口标题。

`@apiVersion 0.1.0`

- 接口版本号，如果接口改变了，需要将上面内容复制一份，修改版本号，修改接口内容，就可以在文档中进行版本内容比对。

`@apiName userinfo`

- 接口名称，不会显示在文档中，是每个接口的唯一标识，以此来区别接口。

`@apiGroup users`

- 接口组名，接口可以分组存放，将一类接口放到一组总，组名相同的会被归到一组。

`@apiSampleRequest off`

- 隐藏测试表单

`@apiDescription 编辑用户资料`

- 接口描述

`@apiParam {String} userid 用户id`

- 接口请求参数，`{}`内是参数类型，后面跟着参数字段名和字段描述。

`@apiSuccess {Object}   data  对象数据`

- 接口返回值，`{}`内是返回值类型，后面跟着返回值字段名和字段描述。要想出现返回值表格中缩进的样式，以`data.userheadimg`这种形式写，`userheadimg`会在`data`下面缩进，表示属于`data`对象中的元素。

`@apiExample 返回示例`

- 接口返回的数据结构示例

## 生成接口文档

`cd`到接口文件夹，使用命令：

```
apidoc -i . -o doc/
```

意思是使用当前文件夹(`. 代表当前文件夹`)下的所有文件，生成apiDoc文档放到`doc/`文件夹下。

命令执行完毕，在`doc/`目录下点击`index.html`，便可本地浏览用apiDoc写成的接口文档。

