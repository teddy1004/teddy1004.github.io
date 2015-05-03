---
layout: post
title: "tap in Ruby"
date: 2015-5-3 15:40:33
---
在看一些 Ruby 的开源项目的代码中经常见到 `tap` 方法的出现，但是感觉又不是有很明确的用意，花了点时间了解了一下这个方法，发现是一个非常好用的方法。

我们在 Ruby 中常常会写到类似这样的代码

```ruby
ary = [1, 2, 3]
ary.each do |ele|
  # do something with ele
end
```

而如果我们想对 `ary` 这个对象执行一些操作的话，那该怎么办呢？这时就是 `tap` 派上用场的时候。`ary.tap` 就可以进入 `ary` 这个对象。这个例子可能看起来觉得并没有凸显 `tap` 的用途。但是设想我们对一个对象执行一系列链式的方法，而我们又想知道调用链中每一步这个对象的结果，这个时候 `tap` 的作用就很明显了。

还有一种情况就是我们可以用 `tap` 来对一个对象进行一些很方便的操作。假设我们有 `{ one: { two: { three: {} } } }` 这样一个 Hash。我们想把它变成这样 `{ one: { two: { three: { first: 'Hello', second: 'world' } } } }`。

看看常规的写法和用了 `tap` 之后差别是不是更优雅呢

```ruby
# without tap
hash[:one][:two][:three][:first] = 'hello'
hash[:one][:two][:three][:second] = 'world'

# with elegant tap
hash[:one][:two][three].tap do |three|
  three[:first] = 'hello'
  three[:second] = 'world'
end
```

