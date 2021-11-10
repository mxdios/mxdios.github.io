---
title: 博客中集成Microsoft Clarity
date: 2021-11-10 14:30:50
tags: [hexo]
categories: [工具]
toc: true
---

[Microsoft Clarity](https://clarity.microsoft.com/)是微软出的用户体验优化工具，可以监控用户在网页中的交互行为。开源免费，国内可用。

<!--more-->

## 创建项目

在[Microsoft Clarity](https://clarity.microsoft.com/)上登录，创建项目，提供要监控的网页地址即可。创建完成之后，Microsoft Clarity会分配集成代码和集成方法。官方列出了一些第三方平台集成方法，还可以手动安装追踪代码。

以手动为例，代码块如下：

```javascript
<script type="text/javascript">
    (function(c,l,a,r,i,t,y){
        c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
        t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
        y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window, document, "clarity", "script", "xxxxxxxxxx");
</script>
```

代码块中的`xxxxxxxxxx`便是分配的网站ID。

## 在博客中集成

hexo的博客主题中很少有集成Microsoft Clarity的，我其中一个博客用的`hexo`+`hexo-theme-matery`，下面以`hexo-theme-matery`为例集成Microsoft Clarity。

这个和百度统计一样，都是将代码块放到网站的`<head>`中。我修改了`hexo-theme-matery`的源码，只需要在配置文件`_config.yml`开启Microsoft Clarity，写入分配的网站id即可。

在`hexo-theme-matery/layout/_partial`目录下新建文件`microsoft-clarity.ejs`，文件中写入代码：

```ejs
<!-- Microsoft Clarity -->
<% if (theme.microsoftClarity.enable) { %>
<script type="text/javascript">
    (function (c, l, a, r, i, t, y) {
        c[a] = c[a] || function () { (c[a].q = c[a].q || []).push(arguments) };
        t = l.createElement(r); t.async = 1; t.src = "https://www.clarity.ms/tag/" + i;
        y = l.getElementsByTagName(r)[0]; y.parentNode.insertBefore(t, y);
    })(window, document, "clarity", "script", "<%- theme.microsoftClarity.id %>");
</script>
<% } %>
```

在`hexo-theme-matery/layout.ejs`文件中<body>标签中添加集成代码：

```ejs
<%- partial('_partial/microsoft-clarity') %>
```

在配置文件`_config.yml`中添加配置项：

```yaml
# 添加 Microsoft Clarity 分析配置
microsoftClarity:
  enable: true # 开启Microsoft Clarity
  id: xxxxxxxxxx # 网站id
```

至此，Microsoft Clarity在hexo博客中集成完毕，重新部署之后，便可在Microsoft Clarity中查看用户行为。

## NexT主题中集成

在`next/layout/_third-party/analytics/`目录下新建文件`microsoft-clarity.swig`，文件中写入代码：

```ejs
{%- if theme.microsoft_clarity %}
  <script type="text/javascript">
    (function(c,l,a,r,i,t,y){
        c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
        t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
        y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window, document, "clarity", "script", "{{ theme.microsoft_clarity }}");
  </script>
{%- endif %}
```

在`next/layout/_third-party/analytics/index.swig`文件中添加如下代码：

```ejs
{% include 'microsoft-clarity.swig' %}
```

在配置文件`_config.yml`中添加配置项：

```yaml
# Microsoft Clarity
microsoft_clarity: xxxxxxxxxx # 网站id
```

注意：配置项`microsoft_clarity`必须用`_`连接，如果使用`-`写成`microsoft-clarity`不生效。
