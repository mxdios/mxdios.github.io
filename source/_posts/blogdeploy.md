---
title: Github Actions+宝塔部署hexo博客到云服务器
date: 2021-10-13 14:30:50
tags: [hexo]
categories: [工具]
toc: true
---

Github Actions是GitHub的持续集成服务，可以预设动作（如：main分支push）来触发一些自动化操作。可以用来自动化部署hexo博客。

<!--more-->

## 背景

[hexo](https://hexo.io/zh-cn/)是常用的一个博客框架，搭配GitHub Pages可以搭建博客。有个问题就是无论`用户站点`还是`项目站点`，git库必须是Public的，一个分支放博客站点，一个分支放博客源文件，源文件也开放多少有些不好。

当然也有人建两个库，Public的是站点库，Private是源文件的库。这种分库的也可以用Github Actions去自动部署。

还有使用GitHub Pages在国内访问慢是不可避免的问题，有人用github 和 gitee同时部署，国内外分别解析到不同站点。

本文不说博客搭建问题，主要介绍使用Github Actions实现push博客源文件自动化部署博客到云服务器。

## 云服务器建站

### 宝塔创建站点

![创建站点](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/image-20211014000310019.png)

使用宝塔创建站点，会设置好域名和网站目录，然后做好域名解析。（国内服务器域名需要备案）

在创建好的文件目录中会初始化一些文件，有一个`.user.ini`，是php的防跨站文件，需要在后面的`.yml`文件中忽略，否则在部署时会报错：

```
rsync: delete_file: unlink(.user.ini) failed: Operation not permitted (1)
```

### 设置ssh密钥

在宝塔的终端里，`cd ~/.ssh`，去到`.ssh`目录下，查看是否有`authorized_keys`、`id_rsa`、`id_rsa.pub`。如果没有，使用下面的命令创建：

````shell
ssh-keygen -m PEM -t rsa -b 4096
````

注意私钥需要`pem`格式，不然部署时会报错，问题说明见：[ssh-deploy/issues](https://github.com/easingthemes/ssh-deploy/issues/34)

创建完成之后，需要将公钥拷贝到`authorized_keys`中，使用命令：

```shell
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

至此，服务器中的设置完成。

## GitHub建库

在GitHub中创建hexo源文件库，可以是私有库。建完库之后，设置Secret

### 设置ACCESS_TOKEN

在项目的 Settings -> Secrets 中新建一个Sectet，设置`ACCESS_TOKEN`，名称可以自定义，注意在后面写`workflow`时文件名要对应。

`ACCESS_TOKEN`对应的值是上面在服务器中生成的私钥`id_rsa`，查看`id_rsa`命令：

```shell
cat ~/.ssh/id_rsa
```

### 设置REMOTE_HOST

`REMOTE_HOST`对应的值是服务器的ip地址

### 设置REMOTE_USER

`REMOTE_USER`对应的值是root

### 设置TARGET

`TARGET`对应的值是创建的站点根目录，如上图中`/www/wwwroot/xxx.com`

设置完成如下图：

![设置Secrets](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/image-20211014002208345.png)

这些值设置在Sectets中，相对来说比较安全。这些值设定之后无法再查看，只能更新或删除。

## 设置workflow

在项目的`Actions`中设置`workflow`，这是一个`.yml`文件。这个文件会被放在项目的`.github/workflow/xx.yml`，文件名可自定义。GitHub会自动执行这个目录下的`.yml`文件。

更详细的知识点请见：阮老师的[GitHub Actions 入门教程](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

下面是部署hexo博客到云服务器的完整`workflow`:

```shell
# main.yml
name: deploy to aliyun # 名称
on: # 触发条件，main分支有push操作时，触发这个自动化脚本执行
  push:
    branches:
      - main
jobs: # 任务集合
  build:
    runs-on: ubuntu-latest # 运行的虚拟机环境，使用最新版本的ubuntu
    steps: # 任务步骤
      # 切换分支
      - name: Checkout
        uses: actions/checkout@main
      # 下载 git submodule
      - uses: srt32/git-actions@v0.0.3
        with:
          args: git submodule update --init --recursive
      # 使用 node:10
      - name: use Node.js 10
        uses: actions/setup-node@v1
        with:
          node-version: 10
      # npm install
      - name: npm install
        run: |
          npm install -g hexo-cli
          npm install
        env:
          CI: true
      # build
      - name: hexo build
        run: |
          hexo clean
          hexo generate
        env:
          CI: true
          
      # Deploy
      - name: Deploy
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.ACCESS_TOKEN }} # 云服务器私钥
          ARGS: "-avzr --delete --exclude /.user.ini" # 忽略服务器上的.user.ini文件
          SOURCE: "public/" # hexo编译之后的博客文件目录
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }} # 云服务器ip地址
          REMOTE_USER: ${{ secrets.REMOTE_USER }} # 云服务器用户名
          TARGET: ${{ secrets.TARGET }} # 部署的云服务器目录
```

Deploy脚本详见：[ssh-deploy](https://github.com/easingthemes/ssh-deploy)

设置好Actions保存之后，会立即执行一次自动化部署，之后只需要写博客push到GitHub，即会自动部署。香的很！

![正确执行结果](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/image-20211014005840650.png)

## 参考文章

[使用GithubActions自动部署应用到自己的服务器（ECS）](https://cloud.tencent.com/developer/article/1720500)

[Github Actions实现CI/CD配置](https://segmentfault.com/a/1190000022990551)

