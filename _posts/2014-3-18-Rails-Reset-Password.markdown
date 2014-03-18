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
{% highlight ruby %}
rails g controller password_resets
{% endhighlight %}

同时我们要在用户的 model 中加入两个新的 column，password_reset_token 以及 password_reset_sent_at ，这两个分别表示用户重置的 token 以及重置的时间（用来在一段时间后将其过期）。

接着就是为用户发送重置密码的邮件了，我们创建一个新的 mailer ，将其命名为 user_mailer ，并为其添加一个 password_reset 方法
{% highlight ruby %}
def password_reset(user)
    @user = user # 这里创建 @user 是为给用户发出的邮件的模版中使用
    mail :to => user.email, :subject => "Password Reset"
end
{% endhighlight %}

在 User model 中定义一个 send_password_reset 方法
{% highlight ruby %}
def send_password_reset
    self.update_attribute :password_reset_token, token_generator # 这里 token_generator 方法需要自己定义
    UserMailer.password_reset(self).deliver # 发送密码重置邮件
end
{% endhighlight %}

在 PasswordResetsController 的 create 方法中如下定义
{% highlight ruby %}
def create
    user = User.find_by_email(params[:email])
    user.send_password_reset if user
    redirect_to root_path, :success => "Password reset email sent"
end
{% endhighlight %}

在给用户的 email 中，我们大概写入以下内容
{% highlight html %}
To reset your password, click the URL below.

<%= edit_password_reset_url(@user.token) %>

If you did not request your password to be reset, just ignore this email and your password will continue to stay the same.
{% endhighlight %}

用户最终收到的邮件大致如下
{% highlight html %}
To reset your password, click the URL below.

http://localhost:4000/password_resets/O2eyoAcgio9a4IP7ujmhrQ/edit

If you did not request your password to be reset, just ignore this email and your password will continue to stay the same.
{% endhighlight %}

我们再在 PasswordResetsController 的 Update 方法中进行处理即可，至此用户重置密码的功能基本实现。但是这里我们会有一个问题，当用户点击重置密码的时候，由于后端需要执行发邮件的操作，而这又是一个耗时的过程，导致用户的进程堵塞，提交请求非常缓慢。因此，我们可以应该将这些耗时的任务添加到后台来做，这次我用到的是一个叫做 Resque 的 gem

> Resque (pronounced like "rescue") is a Redis-backed library for creating 
> background jobs, placing those jobs on multiple queues, and processing them 
> later.

通过使用 Resque，可以使用户在提交重置密码请求后，服务器立刻响应，将耗时的任务放到后台去做，这样无疑提升了用户体验。我们来看看大致的实现过程，首先是在 Gemfile 中添加 Resque。
{% highlight ruby %}
# Use Resque for putting time-consuming tasks to background
gem 'resque'
{% endhighlight %}

然后编写一个 resque 的 Rakefile，下面是一个基本功能的配置
{% highlight ruby %}
require 'resque/tasks'

task 'resque:setup' => :environment
{% endhighlight %}

接着就是创建 workers 了，参考 RailsCasts 中关于 Resque 的介绍，创建 app/workers/mail_deliver.rb 文件，每一个 worker 中会有一个 perform 类方法

> Background jobs can be any Ruby class or module that responds to perform.
> Your existing classes can easily be converted to background jobs or you can
> create new classes specifically to do work. Or, you can do both.

mail_deliver.rb 内容很简单，
{% highlight ruby %}
class MailDeliver
    @queue = :mail_queue
    def self.perform(user)
        UserMailer.password_reset(mailer).deliver
    end
end
{% endhighlight %}

可以看到，其实只是将原本在主线程上执行的任务放到了后台的进程中来执行，很简单吧。
具体更详细的资料参考 RailsCasts #271 Resque 以及 #274 Remember Me & Reset Password，里面有更详细的讲解。
