---
layout: post
title: "The Difference between instance_eval and instance_exec"
date: 2015-4-26 15:30:24
---
这周看到一篇博客讲了 Ruby 中 `instance_eval` 和 `instance_exec` 的区别，看完觉得收获很多，这里和大家分享一下，原文地址 [The difference between instance_eval and instance_exec](http://www.saturnflyer.com/blog/jim/2015/04/22/the-difference-between-instance_eval-and-instance_exec)。

在 Ruby 中我们通常用 `instance_eval` 来打开一个对象，比如下面这段代码

```ruby
class Klass
  def intialize
    @secret = 100
  end

  def secret
    @secret
  end
end
```

在上面的代码中当我们实例化 `Klass` 后，调用 secret 方法会输入 `@secret` 的值，也就是 100。由于没有提供 setter 方法，所以它的值是无法改变的，但是通过 `instance_eval` 可以实现我们的需求。

```ruby
k = Klass.new
k.instance_eval { @secret = 0 }
k.secret # => 0
```

在 FactoryGirl 中就有许多这样的用法，比如我们通常定义 factories 的时候

```ruby
FactoryGirl.define do
  factory :user do
    first_name "Teddy"
    last_name  "Wang"
  end
end
```

这里就是通过 `instance_eval` 来实现的

```ruby
def factory(name, &block)
  factory = Factory.new(name)
  factory.instance_eval(&block) if block_given?
end
```

Ruby 中还有一个类似功能的方法，叫 `instance_exec`。很多时候，两个看起来差不多，那么它们有什么不同呢？其实主要区别就是

* `instance_eval` 只能处理一个 block (或者 string)，如果想传更多的参数，那就...mengbi了
* `instance_exec` 则支持我们在传入 block 的同时，传入我们想要传入的参数

在写 factories 的时候，我们有时会这样定义

```ruby
FactoryGirl.define do
  factory :user do
    first_name "Teddy"
    last_name  "Wang"

    after(:create) do |user, evaluator|
      create_list(:post, evaluator.posts_count, user: user)
    end
  end
end
```

上面的代码中，`after(:create)` 是在 user 对象创建之后执行的，在接受 block 的同时，它还接受了两个参数，处理这种带参数的调用的时候我们就使用 `instance_exec` 来实现。
