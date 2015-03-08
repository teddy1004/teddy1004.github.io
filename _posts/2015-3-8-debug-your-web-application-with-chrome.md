---
layout: post
title: "Debug your web app with Chrome"
date: 2015-3-9 15:40:30
---
今天看奇总写了关于使用 Chrome 来调试 Android app 的分享，觉得必须得写一篇关于使用 Chrome
来调试 web app 的分享了。因为相比于 iOS/Android app 开发，web 开发实在是比较缺乏优秀的 IDE，
很难想象如果没有像 Chrome Dev Tools，调试将是这么痛苦的一个过程。今天就来分享一下我在开发的
过程中常用到的一些不错的特性。

### 告别 console.log
在以前没有使用 Dev Tools 的时候，我经常为了想知道一个变量在代码运行到某一块的时候值是多少，然后
就直接在代码里面加入个 `console.log()` 将那个变量的值打印出来。这是一个并不总是可行的笨办法(好
像听起来有点绕)

* 首先我们得去修改代码，调试完了还要记得把它删掉
* 其次这并不一定能起到我们预期的用处，也许在 console 里面输出的结果是正确的，但是我们无法看到
代码的调用堆栈，也许随着代码的深入运行，我们原本认为正确的值已经变了

综上，用 `console.log` 来调试既麻烦而且往往也只能看到比较表面的东西，我们想更深入的看代码的运行
情况就得借助断点了。

![](/images/chrome-dev-1.png)

如上图，我们可以在 Chrome Dev Tools 里面找到我们的源文件，可以用 `cmd+p` 迅速唤出我们需要 debug
 的文件，可以点击行号加上断点。然后 reload，当代码运行到断点的地方浏览器就会停下来，我们就可以随意
 查看变量、`step into`, `step over` 了。

### Conditional Breakpoint
![](/images/chrome-dev-1.png)

有时我们并不需要代码总是运行到断点的地方就停住，而是希望当它满足某个条件的时候才会停住。这时我们可以
使用 `conditional breakpoint`。当我们传入的表达式为 true 的时候，代码才会暂停。

除此之外，我们还可以加入 `DOM Breakpoints`、`XHR Breakpoints`，`Event Listener Breakpoints`，为我们调试提供了极大的便利。

### Profiler
现在 web 开发对体验也是要求越来越高，利用 Chrome Dev Tools 我们可以方便的为我们的应用
做 profile，了解它的内从占用、CPU 占用，是否有内存泄漏等。

除此之外，我们可以查看页面每个部分加载渲染耗费的时间，为我们优化应用提供了一个很好的指标。

除了以上一些基本功能之外，我们还可以利用插件来调试一些框架独有的东西。比如 Ember.js 有 `Ember `
`Ember inspector`，Angular.js 有 `AngularJS Batarang`。甚至还可以用 `Node inspector` 来调试 Node.js 的应用 (这个真的是十分好用！)。
