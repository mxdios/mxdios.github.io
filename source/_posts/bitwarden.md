---
title: 宝塔独立部署Bitwarden密码管理
date: 2021-11-03 14:30:50
tags: [Bitwarden]
categories: [工具]
toc: true
---

之前一直用1Password管理密码，但是没买订阅服务，相当于手机的本地密码存储。浏览器上Chrome和钥匙串混用，特别不方便。终于发现了[Bitwarden](https://bitwarden.com/)。

<!--more-->

这里我部署的是[Vaultwarden](https://rs.bitwarden.in/)，也是原来的bitwarden_rs。因为Vaultwarden对配置要求低，同样开源，支持docker部署，而且使用的客户端、浏览器插件都是Bitwarden官方的。

## 安装镜像

### 安装Docker管理器

在宝塔的“软件商店”面板里直接搜索“Docker管理器”，安装即可

### 安装Vaultwarden镜像

镜像名称：`vaultwarden/server`

![](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/WX20211103-101352@2x.png)

镜像安装完成之后，需要创建Docker容器

### 创建容器

![创建容器](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/image-20211103102621111.png)

- 镜像：选择刚才安装的镜像`vaultwarden/server:latest`
- 绑定IP：默认`0.0.0.0`
- 端口映射：`80`（容器端口固定），`5555`（服务器端口可自定义，需要在宝塔面板和服务器开放该端口），写完端口映射，点 `+`
- 目录映射：容器目录：`/www/wwwroot/pwd`（/www/wwwroot/是站点的固定目录，pwd可自定义，注意后面创建站点的目录也要是此目录），服务器目录固定：`/data`，写完目录映射，点 `+`
- 内存配额：可按照自己的服务器配置分配，Vaultwarden对内存要求很低

填写完成后点击提交，并修改容器名称。

### 命令行创建镜像和容器

需要已安装Docker管理器

```dockerfile
# 拉取镜像
docker pull vaultwarden/server:latest
# 创建容器
docker run -d --name vaultwarden -v /www/wwwroot/pwd/:/data/ -p 5555:80 vaultwarden/server:latest
```

## 创建站点

![创建站点](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/image-20211103113428280.png)

- 根目录要和`目录映射`中的`容器目录`一致！
- PHP版本纯静态即可

站点创建之后，添加`Let's Encrypt`SSL证书，可以用https访问。

## 添加反代

给站点添加反向代理

目标URL：`http://127.0.0.1:5555`，端口要和设置的服务器端口一致。

![添加反代](https://imagedb-1257991841.cos.ap-beijing.myqcloud.com/image-20211103120910132.png)

至此，Vaultwarden部署完毕，可通过域名访问。

## 关闭注册

关闭注册之前，一定要自己先注册个账号！：）

1. 在Docker管理器中先停掉vaultwarden容器

2. 删除容器，删除容器不会删除你的数据

3. 使用如下命令重新创建容器

   ```dockerfile
   # 创建容器
   docker run -d --name vaultwarden -e SIGNUPS_ALLOWED=false -v /www/wwwroot/pwd/:/data/ -p 5555:80 vaultwarden/server:latest
   ```

4. 用命令行关闭容器，再重启

   ```dockerfile
   # 关闭容器
   docker stop vaultwarden
   # 启动容器
   docker start vaultwarden
   ```

   这一步非常重要，有时候会出现关闭注册之后，之前注册的账号也无法登录，使用命令行进行容器重启操作，可解决该问题。

## 更新

使用命令行完成更新操作，完整命令如下：

```dockerfile
# 拉取最新镜像
docker pull vaultwarden/server:latest
# 停止旧容器
docker stop vaultwarden
# 删除旧容器
docker rm vaultwarden
# 创建新容器
docker run -d --name vaultwarden -e SIGNUPS_ALLOWED=false -v /www/wwwroot/pwd/:/data/ -p 5555:80 vaultwarden/server:latest
# 关闭容器
docker stop vaultwarden
# 启动容器
docker start vaultwarden
```
