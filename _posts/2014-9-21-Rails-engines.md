---
layout: post
title: "Rails Engines and other extensions"
date: 2014-9-21 22:05:21
---
今天来说说我们在拓展一个 Rails app 中常用到的几种形式。通常我们会在 `Gemfile` 中引入一些我们需要的 gem，然后这些 gem 会提供相应的功能来为我们的开发提供便利。

但是在 Rails 官方的 guides 中还看到了 `Engines` 和 `Plugins` 这两个概念，它们是用来做什么的呢？

### Rails Engines

Engines 可以认为是一个小型的 `Rails application` 被嵌入到它的 `Host application` 来提供一些功能。事实上，一个完整的 Rails application 可以说是一个加强版的 engine，`Rails::Application` 类中很多行为都是继承自 `Rails::Engine` 的。像我们比较熟悉的 `Devise` 之类的 gem 其实就是一个 Rails engine。

我们在终端下使用 `rails plugin new blorgh --mountable` 来创建一个 `Rails engine`，`mountable` 选项表示创建一个可嵌入的、命名空间独立的 `engine`。

`engine` 的使用也很简单，在我们的主 app 中我们只需要在 Gemfile 中加入相应的 gem 就可以了，比如上面创建的那个 engine 就是 `gem 'blorgh', path: "path/to/blorgh"`。

下面是一个 Rails engine 的结构:

```bash
.
├── Gemfile
├── Gemfile.lock
├── MIT-LICENSE
├── README.rdoc
├── Rakefile
├── app
│   ├── assets
│   ├── controllers
│   ├── helpers
│   ├── mailers
│   ├── models
│   └── views
├── bin
│   └── rails
├── blorgh.gemspec
├── config
│   └── routes.rb
├── lib
│   ├── blorgh
│   ├── blorgh.rb
│   └── tasks
└── test
    ├── blorgh_test.rb
        ├── dummy
            ├── integration
                └── test_helper.rb
```

整体看起来和一个 `Rails application` 基本一致，有没有注意到 `test/dummy` 这个目录，它里面的结构是这样的:

```bash
.
├── README.rdoc
├── Rakefile
├── app
├── bin
├── config
├── config.ru
├── db
├── lib
├── log
├── public
└── tmp
```

这根本就是一个 `Rails application` 的结构吗，其实它是 Rails 提供的很贴心的功能，为我们展示了一个引入了我们自己的 engine 的示例。

__test/dummy/config/routes.rb__

```ruby
Rails.application.routes.draw do

  mount Blorgh::Engine => "/blorgh"
end
```

所以我们该怎么用也就一目了然了。

### Plugin

首先来看看 `plugin` 的创建命令 `rails plugin new plugin_name`，是不是和 `Rails engine` 很像？事实上，我们可以这样认为 `An engine can be a plugin, and a plugin can be an engine`。两者的区别更多在 `aim` 上，`plugin` 更倾向于对 `Rails` 自身的一种拓展，可以让我们在 `Rails` 上添加一些功能而不影响到原有的代码。因此，它的结构也相对更简单一些:

```bash
.
├── Gemfile
├── Gemfile.lock
├── MIT-LICENSE
├── README.rdoc
├── Rakefile
├── lib
├── test
└── yaffle.gemspec
```

### 我们该如何选择？

* 如果我们需要为 Rails 添加一些通用的功能，那么 `plugin` 跟为合适
* 如果我们创建的是一个 `Ruby` 的类库，与 `Rails` 无关，在 `Sinatra` 等等其他之类任意的框架都可以使用，比如一个 `Markdown` 的解析器，那么创建一个 `Ruby gem` 更为合适
* 如果我们要创建一个很多 `Rails application` 通用的模块，比如一个注册验证的模块，这时候就是 `Rails Engines` 出场的时候了 (以前居然都是把需要的代码在不同项目间复制来复制去，笑死我吧！！！)
