---
layout: post
title: "AngularJS Directive for Gravatar"
date: 2014-1-2 15:40:33
---
目前公司在进行的项目中需要加入对 Gravatar 的支持，之前是通过 Ruby on Rails 在后端写一个 helper 方法来实现。而在查阅了一些资料之后，发现这在前端通过 AngularJS 来实现也是很容易的——通过 AngularJS 的一个核心功能 Directive。
首先来看看 Directive，什么是 Directive 呢？官方文档里面这样说到：

> At a high level, directives are markers on a DOM element (such as 
> an attribute, element name, or CSS class) that tell AngularJS's 	HTML 
> compiler ($compile) to attach a specified behavior to that 	DOM element or 
> even transform the DOM element and its 	children.

在我的理解来看，Directive 的实现可以自己定义一些有特殊功能的 template 加入到 HTML 中。显然，我们需要的 Gravatar 功能就是 Directive 派上用场的时候了。

**avatar.coffee**:
{% highlight coffeescript %}
app.directive 'avatar', (md5) ->  
  return {
    restrict: 'E',
    replace: true,
    template: '<img src="https://secure.gravatar.com/avatar/{{gravatar_id}}.png?s={{size}}"/>'
    link: (scope, element, attrs) ->
      scope.size = attrs.size
      scope.gravatar_id = md5.createHash(attrs.email)
  }
{% endhighlight %}

**HTML**:
{% highlight HTML %}
<avatar size="40" email="..."></avatar>
{% endhighlight %}

简单的说一下代码, avatar.coffee 中，首先要引入 md5 模块。因为 Gravatar 的地址是由用户的 email 进行 md5 加密生成的字符串和其他部分组成的。再来说说 Directive 中几个指令的含义:
* restrict 表示对 Directive 形式的限制，E 表示 element, 还有其他的如 A,C 等等，具体查文档～
* replace 表示是否用生成的模版替换 Directive，针对本代码来说若为 true，则 avatar 会被替换掉，false 的话则会将 template 生成的代码加入到 avatar 中
* link 待续...
* 
* 
* link 表示 directive 要执行的方法