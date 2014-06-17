---
layout: post
title: "关于 Symbol#to_proc() 的一些事情"
date: 2014-6-15 11:26:30
---
今天在群里看到有朋友问到，像`["first", "second", "third"].map(&:capitalize)`这种看起来比较高端的用法，背后的实现过程到底是什么呢。记得以前恰好在 Ruby 元编程的书中看到过这个，这里就谈一下对它的理解吧。

首先来看一个简单的例子

```ruby
names = ['first', 'second', 'third']
names.map { |name| name.capitalize }
```

上面的代码我们一定很常见，它有什么问题呢？其实它没什么问题。。。哈哈，简单的来说就是感觉它不够 Ruby，Ruby 这样一个以简洁著称的语言居然只为了一个简单的函数调用写那么一长串 block，所以必然可以有更简单的办法。

那么要多简单呢？我们自然是希望 map 函数中的每一个元素直接调用 capitalize 就好啦，看起来就像这样 `names.map(capitalize)`。

接着上面的继续分析，首先我们直接使用 capitalize 这样会有一个问题，`undefined local variable or method 'capitalize'`，所以比较合适的方式是将它替换为 `Symbol` 即 `:capitalize`。我们希望 `:capitalize` 能实现像 block `{ |x| x.capitalize }`，因此我们首先要把这个 Symbol 转为一个 Proc。

因此我们需要进入 Symbol 类为它添加 to_proc 方法

```ruby
class Symbol
  def to_proc
    Proc.new { |x| x.send(self) }
  end
end
```

这时我们的函数就改造成这样了`names.map(:capitalize.to_proc)`，运行，还是会出错`wrong number of arguments (1 for 0)`，因为`map` 的参数需要为一个 block，这时我们可以使用`&`将它转化为block，所以最终的结果就是这样啦`names.map(&:capitalize.to_proc)`，在 Ruby 1.9 之后的版本中，to_proc 函数已经自己存在无需我们去定义了。因此可以简化为这样`names.map(&:capitalize)`，这就是我们最终看到的啦。

最后总结一下这个函数的一个完整实现过程其实就是`names.map(&(Proc.new { |x| x.send(:capitalize) }))`
