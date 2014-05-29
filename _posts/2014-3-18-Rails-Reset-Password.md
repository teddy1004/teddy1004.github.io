---
layout: post
title: "Reset password in Rails"
date: 2014-3-18 17:14:30
---
之前这些功能都是通过 Devise 来实现的，今天刚好有空，希望自己来做一个密码重置的功能。大概思路其实很简单，包括了以下几步:

* 在登录页面添加一个“忘记密码”的选项，用户点击该页面后会跳转到重置密码的页面
* 在重置密码页面中，用户需要输入注册时使用的邮箱，然后点击重置
* 接着用户会收到重置的邮件，里面包含了重置密码的链接
* 进入该链接后重新输入密码即可更改密码

接着我们来看实现过程中的几个关键步骤，首先是增加重置密码的 Controller
```bash
rails g controller password_resets
```

同时我们要在用户的 model 中加入两个新的 column，password_reset_token 以及 password_reset_sent_at ，这两个分别表示用户重置的 token 以及重置的时间（用来在一段时间后将其过期）。

接着就是为用户发送重置密码的邮件了，我们创建一个新的 mailer ，将其命名为 user_mailer ，并为其添加一个 password_reset 方法
```ruby
def password_reset(user)
    @user = user # 这里创建 @user 是为给用户发出的邮件的模版中使用
    mail :to => user.email, :subject => "Password Reset"
end
```

在 User model 中定义一个 send_password_reset 方法
```ruby
def send_password_reset
    self.update_attribute :password_reset_token, token_generator # 这里 token_generator 方法需要自己定义
    UserMailer.password_reset(self).deliver # 发送密码重置邮件
end
```

在 PasswordResetsController 的 create 方法中如下定义
```ruby
def create
    user = User.find_by_email(params[:email])
    user.send_password_reset if user
    redirect_to root_path, :success => "Password reset email sent"
end
```

在给用户的 email 中，我们大概写入以下内容
```html
To reset your password, click the URL below.

<%= edit_password_reset_url(@user.token) %>

If you did not request your password to be reset, just ignore this email and your password will continue to stay the same.
```

用户最终收到的邮件大致如下
```html
To reset your password, click the URL below.

http://localhost:4000/password_resets/O2eyoAcgio9a4IP7ujmhrQ/edit

If you did not request your password to be reset, just ignore this email and your password will continue to stay the same.
```

我们再在 PasswordResetsController 的 Update 方法中进行处理即可，至此用户重置密码的功能基本实现。但是这里我们会有一个问题，当用户点击重置密码的时候，由于后端需要执行发邮件的操作，而这又是一个耗时的过程，导致用户的进程堵塞，提交请求非常缓慢。因此，我们可以应该将这些耗时的任务添加到后台来做，这次我用到的是一个叫做 Resque 的 gem

> Resque (pronounced like "rescue") is a Redis-backed library for creating
> background jobs, placing those jobs on multiple queues, and processing them
> later.

通过使用 Resque，可以使用户在提交重置密码请求后，服务器立刻响应，将耗时的任务放到后台去做，这样无疑提升了用户体验。我们来看看大致的实现过程，首先是在 Gemfile 中添加 Resque。
```ruby
# Use Resque for putting time-consuming tasks to background
gem 'resque'
```

然后编写一个 resque 的 Rakefile，下面是一个基本功能的配置
```ruby
require 'resque/tasks'

task 'resque:setup' => :environment
```

接着就是创建 workers 了，参考 RailsCasts 中关于 Resque 的介绍，创建 app/workers/mail_deliver.rb 文件，每一个 worker 中会有一个 perform 类方法

> Background jobs can be any Ruby class or module that responds to perform.
> Your existing classes can easily be converted to background jobs or you can
> create new classes specifically to do work. Or, you can do both.

user.rb 中 send_password_reset 方法也有一定的变化:
```ruby
def send_password_reset
    # ...
    # Instead of send email directly, we shoud send email in the background
    # process
    Resque.enque(MailDeliver, id)
end
```
上面代码唯一变动的就是原本直接使用 ActionMailer 发送邮件，而现在将这个过程发到了后台的队列中。注意这里我们是仅仅传入了 user 的 ID，因为我们传入的任何数据进入队列之后会被转为 JSON 格式存入 Redis 数据库中，因此我们不应该直接传入类似于 user 这种复杂的 ActiveRecord 对象。传入 ID 之后我们可以再在 worker 中根据 ID 来获取具体的对象。

mail_deliver.rb 内容很简单:
```ruby
class MailDeliver
    @queue = :mail_queue
    def self.perform(user_id)
        user = User.find(user_id)
        UserMailer.password_reset(user).deliver
    end
end
```
通过把发送邮件的功能加入到后台队列之后，我们会发现用户点击重置密码之后很快就会转跳到新的页面而不需要像以前那样长时间的等待。
