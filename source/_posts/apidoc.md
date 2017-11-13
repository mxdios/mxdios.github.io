---
title: 用apiDoc写接口文档
date: 2017-11-13 17:44:16
tags: tags: [文档]
categories: [HTML5]
toc: true
---

作为移动端开发，接口文档看得多，写得少。最近对已开发完成的app进行接口文档整理，发现了apiDoc这款神器。见识到接口文档也可以写的这么高大上，之前用markdown写弱爆了，用word的自杀吧。

<!--more-->

### 安装


在安装`npm`的基础上，执行命令安装apiDoc

```
npm install apidoc -g
```

安装完成，Mac上便具备apiDoc环境，可以开始写文档了。

### 创建文档

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
    "title": "接口文档说明",
    "filename": "header.md"
  },
  "footer": {
    "title": "其他内容",
    "filename": "footer.md"
  },
  "template": {
  	 "withCompare": true,
  	 "withGenerator": true
  }
}
```

字段解释：

- `version`：文档版本号
- `url`：域名的主地址，接口中不变的那一块
- `header`、`footer`：页眉、页脚，可以不写。里面分别跟着页眉页脚的标题和markdown文件路径，markdown文件和`apidoc.json`同级目录
- `withCompare`：版本比较，默认开启，`withGenerator`：生成器，页面底部apiDoc的标注，默认开启



