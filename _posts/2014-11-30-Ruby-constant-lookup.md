---
layout: post
title: "constant lookup in Ruby"
date: 2014-11-30 11:20:33
---
Ruby 中常量查找路径是新手接触 Ruby 时候常会遇到的一个问题，比如下面两种写法:

```ruby
# first
module A
  class B; end
end

# second
module A; end
class A::B; end
```
他们之间会有什么区别呢？

### Module.nesting
这是 Ruby 中很重要的一个概念，我们直接用代码说话吧

```ruby
module A
  module B; end
  module C
    module D
      p Module.nesting # => [A::C::D, A::C, A]
    end
  end
end
```

通过 `Module.nesting` 我们可以看到`常量的查找链`。因此继续上面的代码来说，我们可以在 `D` 这个 module 中调用查找到 `B module`。具体的一个过程是，首先会去尝试寻找 `A::C::D::B module`，不存在；再去寻找 `A::C::B module`，还是不存在；直到最后找到 `A::B module`，bingo! 这就是 Ruby 中常量寻找的一个大致的流程。

那么我们前面的那个问题的答案呢？接着上代码:

```ruby
C = "At the top level"

module A
  C = "In A"
end

module A
  module B
    p Module.nesting # => [A::B, A]
    p C              # => "In A"
  end
end

module A::B
  p Module.nesting # => [A::B]
  p C              # => "At the top level"
end
```

看到了吧，区别就在于 `Module.nesting` 这里，第二种写法直接进入的就是顶级作用域，当我们没有定义最外层的常量 `C` 的时候，就会出现错误。

```ruby
module A
  C = "In A"
end

module A::B
  p Module.nesting # => [A::B]
  p C # => NameError: uninitialized constant A::B::C
end
```

因此我们在写代码的时候要注意到这些问题，尤其是我们有时候会见到有的代码中这样写 `::A`。这就是为了主动声明常量的查找路径，以达到我们想要的结果。

突然想起了我前两天写代码中的一个小疑惑，继续上代码:

__client/base_controller.rb__
```ruby
module Client
  class BaseController < ApplicationController
    # ...
  end
end
```

```bash
.
├── application_controller.rb
├── client
│   └── base_controller.rb
```

代码中，`client/base_controller.rb` 是继承自 `application_controller.rb` 的，可是按照代码中的写法，`ApplicationController` 应该是在 `Client` 这个 namespace 下面的，为什么还是可以找到上一层目录中的 ｀application_controller.rb` 呢？按照 Ruby 的常量查找路径来说，在 BaseController 中的查找路径应该是 [Client::BaseController, Client]，是 Rails 又为我们做了一些事情吗？

是的! Rails 有自己的一套 `autoload system`，简而言之就是将 Ruby 内置的 `autoload system` 扩大增强了，所以能加载到这个文件，具体的内容在下一周再写更多。

### 参考

* [Rails autoloading — how it works, and when it doesn't](http://urbanautomaton.com/blog/2013/08/27/rails-autoloading-hell/)
* [Everything you ever wanted to know about constant lookup in Ruby](https://cirw.in/blog/constant-lookup/)