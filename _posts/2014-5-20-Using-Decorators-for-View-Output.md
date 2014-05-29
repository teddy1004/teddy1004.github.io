---
layout: post
title: "Using decorators for view output"
date: 2014-5-20 13:28:24
---
在处理 Views 的过程中，我们时常会遇到需要写一些 Helper Method 的情况。通常的做法是我们把它放在 app/helpers 这个目录下，大部分时间这是没什么问题的，但是有时候我们也会遇到一些特殊的情况。

先来看一个简单的 Helper Method 的例子

#### app/views/posts/show.html.erb
```html
<span><%= publication_date @post %></span>
```

#### app/helpers/posts_helper.rb
```ruby
module PostsHelper
    def publication_date(post)
        post.created_at.strftime '%Y-%m-%d'
    end
end
```

这是我们在 Rails 中常用的 view helper，但是它会有两个问题

* 污染了全局的命名空间，而本身的出发点其实是针对专门的一个 model 的
* 必须穿雨一个对象，在这个对象上来进行进一步操作

那么来看看下面这段代码
#### app/views/users/show.html.erb
```html
<span><%= publication_date @user %></span>
```

这段代码有问题么？虽然产生出的结果是错误的，但是却是允许这样用的。因此如果我们的 View Helpers 增多之后，以上的两个问题就会给我们带来不少的困惑。在查了一些资料之后，觉得使用 decorators 来处理这个问题是个不错的办法
#### app/decorators/post_decorator.rb
```ruby
class PostDecorator
    attr_reader :post

    def initialize(post)
        @post = post
    end

    def publication_date
        post.created_at.strftime '%Y-%m-%d'
    end

    def method_missing(method_name, *args, &block)
        post.send(method_name, *args, &block)
    end

    def respond_to_missing?(method_name, include_private = false)
        post.respond_to?(method_name, include_private) || super
    end
end
```

在 Controller 中我们需要多做一下处理
#### app/controllers/posts_controller.rb
```ruby
class PostsController < ApplicationController
    def show
        post = Post.find(params[:id])
        @post_decorator = PostDecorator.new(post)
    end
end
```

这样，我们在视图中就可以直接这样写了
#### app/views/posts/show.html.erb
```html
<span><%= @post_decorator.publication_date %></span>
```

这样处理起来最初会显得麻烦一些，但是随着 Views Helpers 的增多，还是能为我们叫减小不少的麻烦。
