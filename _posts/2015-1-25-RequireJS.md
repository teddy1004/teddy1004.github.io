---
layout: post
title: "前端模块化之 Require.js"
date: 2015-1-25 14:30:33
---
最近在研究前端的一些东西，感觉写前端遇到一个很困扰的问题就是代码模块化的问题。像 Rails 给我们了一个比较好的解决方案，在 Rails 中我们有一个 `application.js` 文件，这个文件负责引入相关的代码依赖，最后再编译这个文件，它里面引入的一堆 JS 文件就会被编译到一起。同时 Rails 中也把文件之间的依赖关系连同帮我们处理了，当我们用 `//= require xxx` 的时候，这个 require 的先后顺序是有影响的，所以我们可以根据自己的需要把代码拆分成相应的模块，然后根据它们之间的依赖关系按次序 `require`。

但是离开了 Rails 我们该怎么办呢? 最近在尝试拿 `Backbone.js` 写一些纯前端 MVC 的东西，Backbone 按照模块大致可以分为 `models`，`collections`，`views` 以及 `routers`。在最初的学习过程中，我将这些不同的模块全部都写在一个文件当中，这种方式在学习的过程中自然不会有什么问题。但是一旦代码量大了，我们如果还把这些代表不同逻辑的代码写在一起，那就会很混乱了。

理想中的感觉是像 iOS 开发中那样，每一个 `controller` 有自己对应的一个文件，每一个 `model` 对应的有自己的文件，然后它们中谁用到了对方我们可以直接 include 进来。这样做好拆分，才有种 Client-side MVC 的感觉吗。那到底有没有办法呢，答案当然是 `有`。而且还有不少解决方案，大致了解了几个之后，我比较喜欢的一个方案是 `Require.js`。

其实 Require.js 不仅仅是一个代码的模块化工具，利用好它还能有效的提升前端 JavaScript 文件的加载速度，今天这里我们主要说一下它用来做代码模块化的功能。

先来举个🌰看看具体的应用场景，假如我们有一个模块它可以提供 `加减乘除` 的功能，然后我们在另一个 JavaScript 文件种想用那个文件提供的这个加减乘除的办法，该怎么办？想了半天结果居然是没办法除非把它们写到一个文件里面，因为 JavaScript 中没有类似于 Ruby 中 require 的东西 (可别黑 Node.js，它们用 CommonJS 有一套很成熟的解决办法，这里说的是浏览器端的)。

在 Require.js 的帮助下，我们可以很轻松的做到，假设那个提供加减乘除的模块，我们为它创建一个 `math.js` 文件。

```javascript
define(function() {
    return {
      add: function() {
        // ...
      },

      sub: function() {
        // ...
      }

      // ...
    }
});
```

下来我们在别的文件中要使用它的话只需要这样写

```javascript
require(['math'], function(math) {
    math.add();
});
```

基本上就是我们想要的样子了，我把自己写的 Backbone.js 的东西也模块了一下，最后代码组织就成了下面这样了，一下清晰了很多吧 :)

```bash
.
├── app.js
├── collections
│   └── users.js
├── libs
│   ├── backbone.js
│   ├── jquery.js
│   └── underscore.js
├── main.js
├── models
│   └── user.js
├── require.js
├── router.js
├── templates
│   ├── edit.html
│   ├── list.html
│   └── user.html
├── text.js
└── views
    └── users
            ├── edit.js
            ├── list.js
            └── user.js
```
