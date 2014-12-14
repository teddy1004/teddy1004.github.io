---
layout: post
title: "hasOwnProperty() method in JavaScript"
date: 2014-12-14 16:30:41
---
由于最近工作中写 JavaScript 的时间比较多，以前觉得这个语言挺恶心的。写多了发现它的一些 best practice 后慢慢觉得如果我们在写代码中尽量使用这个语言的 `good part` 而摒弃一些 `bad part` 的话，JavaScript 的代码也能变得漂亮优美起来。所以先从读一些优秀的 JavaScript 开源代码做起，最近在读代码中经常见到对一个 Object 像下面那样循环

```javascript
var man = {
  hands: 2,
  legs: 2,
  heads: 1
}

for (var key in man) {
  if (man.hasOwnProperty(key)) {
    // do something about man[key]...
  }
}
```

一直对为什么要使用 `hasOwnProperty` 方法来判断 key 是否存在感到疑惑。后来再查询了一些资料之后，才知道这个还是和 JavaScript 的 `prototype` 有关系。假如我们在代码中的某个地方加入了这样一段 `Object.prototype.clone = function () { ... };`。

这时我们在上述代码中打印一下相应的 property 会发现是如下的情况:

```javascript
// with hasOwnProperty
for (var i in man) {
  if (man.hasOwnProperty(i)) {
    console.log(i, ":", man[i]);
  }
}

/*
result in the console
hands:2
legs:2
heads:1
*/

// without hasOwnProperty
for (var i in man) {
  console.log(i, ":", man[i]);
}

/*
result in the console
hands:2
legs:2
heads:1
clone:function()
*/
```

看到了区别了吧，这也就是 `hasOwnProperty` 的使用场景。所以它并不一定总是必须的，但是如果当你对对象的原型链并不清楚的时候，使用 `hasOwnProperty` check 一下是很有必要的，这样可以避免很多预料外的 bug。
