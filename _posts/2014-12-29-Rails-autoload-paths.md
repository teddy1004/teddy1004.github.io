---
layout: post
title: "Rails autoload_paths"
date: 2014-12-29 14:30:31
---
之前的博客中提过 Ruby 中的常量查找以及 `$LOAD_PATH`。而在 Rails 中，又将这个步骤更加简化了。先来看一段 Rails 程序中常见的代码。在 `app/controllers/` 中有一个 `posts_controller.rb` 文件

```ruby
class PostsController < ApplicationController
  def index
    @posts = Post.all
  end
end
```

这段代码初看很简单，其实背后 Rails 帮我们做了很多事。比如 `ApplicationController` 和 `Post` 这两个常量是哪里来的？为什么我们不需要去 require 它就可以直接使用了呢？

这就是因为 Rails 中有一个类似于 `$LOAD_PATH` 的东西去查找这些变量，它就是 `autoload_paths`。在 `config/application.rb` 文件中我们对它进行配置，`config.autoload_paths += [...]`。默认的情况下，Rails 的 autoload_paths 包括了以下目录

```bash
$ bin/rails r 'puts ActiveSupport::Dependencies.autoload_paths'
.../app/assets
.../app/controllers
.../app/helpers
.../app/mailers
.../app/models
.../app/controllers/concerns
.../app/models/concerns
.../test/mailers/previews
```

那么下来来解决我们的问题，`ApplicationController` 和 `Post` 这两个东西到底是怎么来的。

### ApplicationController
为了加载 `ApplicationController`，Rails 会去遍历 `autoload_paths`。因此首先会去找 `app/assets/application_controller.rb`，不存在。再去找 `app/controllers/application_controller.rb`。bingo！接着 Rails 会去为我们加载这个文件。

### Post
查找 post.rb 的过程会麻烦一些，因为是在 `PostsController` 中调用它。因此首先会把下面的目录遍历一次

```bash
app/assets/posts_controller/post
app/controllers/posts_controller/post
app/helpers/posts_controller/post
...
test/mailers/previews/posts_controller/post
```

查找无果，接着会去它的父命名空间去查找，在这里来说就是 `top level` 了。

```bash
app/assets/post.rb
app/controllers/post.rb
app/helpers/post.rb
app/mailers/post.rb
app/models/post.rb
```

最后我们会找到我们需要的 `app/models/post.rb`。