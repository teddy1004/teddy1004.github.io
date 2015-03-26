---
layout: post
title: "修改 Ember locationType 时候引发的一个问题"
date: 2015-3-26
---
在公司的一个 web 项目的部署中，由于考虑到服务器的限制以及部署的简易性。将 locationType 为 `hash`。之前在后端转跳的链接是 `app/?token=xxx`

然后在前端通过 `beforeModel` 中的 `transition.queryParams.token` 来设置 token 参数，在将 locationType 改为 hash 之后出现了无法获取 queryParams (queryParams.token is undefined) 的情况。

### 解决
将转跳的链接配置为 `app/#/?token=xxx`。之前有些想当然了，因为既然将 locationType 配置为 hash 那么首页的链接也就应该是 `app/#/` 而不再是 `app/` ，在将跳转的 url 更改之后，可以再次获取到 token 了。

解决办法其实很简单，不过着实一直是没注意到这里，因为是代码出了 bug，以后要细心呐。