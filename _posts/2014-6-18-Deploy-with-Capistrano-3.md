---
layout: post
title: "使用 Capistrano 3 部署 Rails 应用"
date: 2014-6-18 10:20:20
---
之前跑应用的时候一直是自己手动一步一步搭起来的，之前看到论坛上有好多帖子谈到自动化部署的。一直打算要在自己的项目种来实验一下的，结果一直。。。懒，今天终于下定决心，开始研究 Capistrano 的文档，开始体验一下自动化部署的。
好多人觉得部署这种事情，就一步一步跑起每个模块就好了，为什么要用这种也算挺重量级的框架来部署呢。以前我也是这么认为，但是发现随着应用的复杂度的提升，用到的东西越来越多，部署确实是一个比较大的工程了。而且自动化部署的好处是只需第一次配置好，今后的所有的部署只需根据现在的部署文件简单的修改一下就好了。反观手动部署的话，每次都要重复同样的一堆指令，开始还好，久了就真的难以忍受了。现在开始进入正题。

### The Stack

* Nginx 作为 web server
* Unicorn 作为 app server

### 步骤
#### 在 Gemfile 中添加 gems

```ruby
gem 'capistrano', '~> 3.2.0'

# rails plugins
gem 'capistrano-rails', '~> 1.1.0'

gem 'capistrano-bundler'
gem 'capistrano-rvm'

# Use unicorn as app server
gem 'unicorn'
```
接着执行`bundle install`来安装所需的 gem 包。

#### 在项目中倒入 Capistrano 的配置文件

在终端执行命令`bundle exec cap install`，接着项目中就会生成以下的文件：

```bash
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
                └── tasks
```
