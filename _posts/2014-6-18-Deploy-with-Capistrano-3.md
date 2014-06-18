---
layout: post
title: "使用 Capistrano 3 部署 Rails 应用"
date: 2014-6-18 10:20:20
---
之前跑应用的时候一直是自己手动一步一步搭起来的，之前看到论坛上有好多帖子谈到自动化部署的。一直打算要在自己的项目种来实验一下的，结果一直。。。懒，今天终于下定决心，开始研究 Capistrano 的文档，开始体验一下自动化部署的。
好多人觉得部署这种事情，就一步一步跑起每个模块就好了，为什么要用这种也算挺重量级的框架来部署呢。以前我也是这么认为，但是发现随着应用的复杂度的提升，用到的东西越来越多，部署确实是一个比较大的工程了。而且自动化部署的好处是只需第一次配置好，今后的所有的部署只需根据现在的部署文件简单的修改一下就好了。反观手动部署的话，每次都要重复同样的一堆指令，开始还好，久了就真的难以忍受了。现在开始进入正题。

#### The Stack

* Nginx 作为 web server
* Unicorn 作为 app server

#### 步骤
##### 在 Gemfile 中添加 gems

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

##### 在项目中倒入 Capistrano 的配置文件

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

##### 修改 Capfile 配置文件

```ruby
Dir.glob('lib/capistrano/tasks/*.cap').each { |r| import r }
Dir.glob('lib/capistrano/**/*.rb').each { |r| import r }
```

这些是我们在后面的配置中要自己写一些 tasks，默认情况下导入的是`lib/capistrano/tasks/*.rake`文件。

同时还要加入下面的一些插件：

```ruby
require 'capistrano/bundler'
require 'capistrano/rvm'
require 'capistrano/rails/migrations'
```

第一个插件是保证我们在部署的时候，`gems`也会自动的安装，`capistrano/rvm`是用来为我们控制 Ruby 的版本的，这样可以保证部署时使用的 Ruby 的版本是我们所需要的。`migrations`是保证在进行部署的时候，`migrations`这些操作也会自动进行。

##### 部署的配置信息

在将配置信息之前，先要说说 Capistrano 中 `stage` 这个概念。个人感觉它和 Rails 中的运行环境类似。默认拥有两个 Stage，`staging`，`production`。

我们在`config/deploy.rb`中定义公用的变量，比如一些公用的`task`，还有`scm`，`git_repo`之类的变量。而和不同运行环境相关的变量则放到相应的 Stage 配置文件中，如`config/deploy/production.rb`

###### config/deploy.rb

```ruby
# config valid only for Capistrano 3.2
lock '3.2.1'

set :application, 'deploy_with_capistrano'
set :deploy_user, 'teddy'

# setup repo details
set :scm, :git
set :repo_url, 'git@github.com:teddy1004/deploy_with_capistrano.git'

# setup rvm.
set :rvm_type, :user
set :rvm_ruby_version, '2.0.0-p353'

# how many old releases do we want to keep
set :keep_releases, 5

# Rails 中一些公用的配置文件（包含重要信息），同时我们要将这些文件添加到 .gitignore 中
set :linked_files, %w{config/database.yml config/secrets.yml}

# dirs we want symlinking to shared
set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

# which config files should be copied by deploy:setup_config
set(:config_files, %w(  
  nginx.conf
  database.yml.example
  secrets.yml.example
  unicorn.rb
  unicorn_init.sh
))

# which config files should be made executable after copying
# by deploy:setup_config
set(:executable_config_files, %w(
  unicorn_init.sh
))

# files which need to be symlinked to other parts of the
# filesystem. For example nginx virtualhosts, log rotation
# init scripts etc.
# The full_app_name variable isn't available at this point
# so we use a custom template {{}} tag and then add it at 
# run time
set(:symlinks, [
  {
    source: "nginx.conf",
    link: "/etc/nginx/sites-enabled/{{full_app_name}}"
  },

  {
    source: "unicorn_init.sh",
    link: "/etc/init.d/unicorn_{{full_app_name}}"
  }
])

namespace :deploy do
  # 在本地编译静态文件然后上传
  after 'deploy:symlink:shared', 'deploy:compile_assets_locally'
  after :finishing, 'deploy:cleanup'

  # 在配置之前，删除 nginx sites-enabled 目录下 default 文件，因为
  # 它有可能和我们的配置信息发生冲突
  before 'deploy:setup_config', 'nginx:remove_default_vhost'

  # 当配置完成之后，重启 Nginx 服务器
  after 'deploy:setup_config', 'nginx:reload'

  # As of Capistrano 3.2, the `deploy:restart` task is not called
  # automatically.
  after 'deploy:publishing', 'deploy:restart'
end
```
这里面有两个比较重要的概念，`linked_files`，`linked_dirs`

解释他们之前，先得说 Capistrano 中`shared`目录这个概念。下面是一个多次 deploy 之后服务器生成的目录结构：

```bash
.
├── current -> /home/user/apps/appname/releases/20140618072703
├── releases
│   ├── 20140618072703
│
├── repo
│   ├── branches
│   ├── config
│   ├── description
│   ├── FETCH_HEAD
│   ├── HEAD
│   ├── hooks
│   ├── info
│   ├── objects
│   ├── packed-refs
│   └── refs
├── revisions.log
└── shared
    ├── bin
    ├── bundle
    ├── config
    ├── log
    ├── public
    ├── tmp
    └── vendor
```
`shared`目录就是用来保存项目中共享的一些内容，它们不是随着每次新的发布而同时发生变动的，比如数据库的配置文件，还有`session`信息等。`linked_dirs`和`linked_files`就是将制定的文件、目录链接到`shared`目录中。

###### linked_dirs
它是将项目制定的目录链接到`shared`目录中

###### linked_files
这个刚好和`linked_dirs`相反，它是将`shared`目录中的文件链接到当前发布的项目中。如果`shared`目录中没有我们指定的文件，那么在部署的过程中会报错。我们经常会将`database.yml`，`secrets.yml`这些存有敏感信息放在这个目录中，同时将项目中这些文件加入到`.gitignore`文件中。

##### 修改 Production 环境下的配置信息
接着我们来在`config/deploy/production.erb`文件下配置`remote_server`的具体信息

```ruby
# Capistrano 中 stage 默认包括了三种环境, 也是 test, development, production
set :stage, :production
set :branch, "master"

set :full_app_name, "#{fetch(:application)}_#{fetch(:stage)}"
set :server_name, "www.example.com"

server 'www.example.com', user: 'deploy', roles: %w{web app db}, primary: true

set :deploy_to, "/home/#{fetch(:deploy_user)}/apps/#{fetch(:full_app_name)}"
set :rails_env, :production
set :unicorn_worker_count, 5
set :enable_ssl, false
```

这里有疑问的地方就是关于 Capistrano 中`roles`这个变量，我还不是很了解，但是目前按照默认配置的都是可以工作的。当我们执行`cap stage_name task_name`命令的时候，Capistrano 会按路径`config/deploy/stage_name.rb`找到这个文件，并且在`deploy.rb`__之后加载这个文件__

##### 向服务器上传 config 文件

运行`cap production deploy:setup_config`指令，这一步是将部署所需的配置文件上传到服务器端。注意`deploy:setup_config`这些都是我们自己写的`task`，在目录`lib/capistrano`目录下。

##### 部署

```bash
cap proudction deploy
```

第一次部署会慢一些，因为要安装相应的 gems，以后就会快很多了。

#### 总结
这只是对 Capistrano 的一次大概的探索，还有很多概念没有理解透彻。这些自动化的工具都是需要一定的学习成本的，所以经常会懒得学，宁可一行一行敲命令（应该不止我一个人吧）。但是只要花一些时间去学习、使用，就会发现能极大的提升效率。所以在以后的项目中，要坚持使用自动化部署工具来部署，好像有个叫`Mina`的也挺火，先把 Capistrano 玩熟了就去试试那个。