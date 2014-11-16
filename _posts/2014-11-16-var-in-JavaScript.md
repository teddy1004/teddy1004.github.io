---
layout: post
title: "JavaScript 中坑坑的 var"
date: 2014-11-16 15:40:33
---
这周在开发过程中，感觉写的 JS 的代码要比 Ruby 多好多。顺便就翻了翻以前写 JS 时候纪录的很多资料，今天就来和大家分享一下这个虽然能做很多事情但是坑也很多的 JS 的其中一坑 - `var`。

说 var 之前要先先说说 JavaScript 中的全局变量。通常来说，我们在 JavaScript 中要尽量减少全局变量的使用，因为这可能会造成变量名冲突。避免全局变量的最好的办法就是使用 var 关键字来声明变量。

在 JavaScript 中

* 你可以使用一个甚至没有声明的对象的变量
* 所有未声明的变量都会成为全局对象的一个属性 (就像一个声明了的全局变量一样)

```javascript
function sum(a, b) {
  result = x + y;
  return result;
}
```

result 在没有被声明的情况下就被使用了，这个代码可以工作，但是在调用了这个函数之后就会多出一个名为 result 的全局变量，这很有可能会导致问题。

解决的方法就是使用 var

```javascript
function sum(a, b) {
  var result = a + b;
  return result
}
```

其次，对于全局变量来说，使用 var 声明和没有使用 var 声明也是有区别的。

* 使用 var 声明创建的全局变量不能被删除
* 未使用 var 声明创建的全局变量可以被删除

因为没有使用 var 声明生成的全局变量不是真正的变量，他们__只是全局对象的`属性`__。属性可以通过 delete 删除，但是变量不行。

__预料之外的 var__
看看下面的代码

```javascript
myName = "global";

function func() {
  alert(myName);
  var myName = "local";
  alert(myName);
}

func();
```

不知道大家对这段代码运行时 alert 的期待是什么呢？就我而言，我期待的是第一次是 _global_，第二次是 _local_。但实际上第一次是 _undefined_，第二次是 _local_。

因为在 JavaScript 中，只要变量在同一个作用域，那么就认为是声明了的。`就算是在 var 语句之前使用也一样`。