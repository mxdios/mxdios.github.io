---
title: Github Actions+宝塔部署hexo到阿里云
date: 2021-10-12 14:30:50
tags: [hexo]
categories: [web]
toc: true
---

Github Actions是GitHub的持续集成服务，可以预设动作（如：main分支push）来触发一些自动化操作。可以用来自动化部署hexo博客。

<!--more-->

背景

[hexo](https://hexo.io/zh-cn/)是常用的一个博客框架，搭配GitHub Pages可以搭建博客。有个问题就是无论`用户站点`还是`项目站点`，git库必须是Public的，一个分支放博客站点，一个分支放博客源文件，源文件也开放多少有些不好。

当然也有人建两个库，Public的是站点库，Private是源文件的库。这种分库的也可以用Github Actions去自动部署。

使用GitHub Pages在国内访问慢是不可避免的问题，有人用github 和 gitee同时部署，国内外分别解析到不同站点。

本文不说博客搭建问题，主要介绍使用Github Actions实现push博客源文件自动化部署博客到阿里云。



