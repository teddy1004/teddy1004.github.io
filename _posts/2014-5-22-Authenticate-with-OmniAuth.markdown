---
layout: post
title: "使用 Facebook 账号登录网站"
date: 2014-05-22 09:44:33
---
随着各种产品的不断出现，大家注册的各种账号也越来越多，如果能用一个账号来登录不同的网站那么将带来很大的便利。Facebook 做为社交网站的 No.1，用它的账号来做登录验证是个不错的选择。
实现这个功能，我们需要用到 OmniAuth 这个 gem，先说说 OmniAuth 是什么:

> OmniAuth is a library that standardizes multi-provider authentication for 
> web applications. It was created to be powerful, secure, and flexible. Any
> developer can create strategies for OmniAuth that can authenticate users
> via disparate systems. OmniAuth strategies have been created for 
> everything from Facebook to LDAP.

简单的说，OmniAuth 就是一个 Library，在我们使用各种不同的 provider 来进行 Authentication 的过程中，它为我们提供了很大的帮助，大大的简化了这一过程。

使用 OmniAuth，首先在 Gemfile 中加入它，由于现在 OmniAuth 为不同的 provider 进行了分类，而我们这次要使用 Facebook 账号来登录，因此在 Gemfile 中加入

{% highlight ruby %}
gem 'omniauth-facebook'
{% endhighlight %}

然后执行 bundle，接着我们要在 Rails 项目中对 OmniAuth 进行初始化，在 initializers 文件夹中创建新文件 omniauth.rb

#### config/initializers/omniauth.rb
{% highlight ruby %}
Rails.application.config.middleware.use OmniAuth::Builder do  
  provider :facebook, ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
end
{% endhighlight %}

这个文件是用来配置 OmniAuth 支持的不同的 Provider 的，我们可以根据自己的需要加入不同的 provider，比如 Twitter啦，GitHub啦，像国内的 Weibo，QQ 也是有相应的 gem 的。ENV['FACEBOOK_KEY'] 和 ENV['FACEBOOK_SECRET'] 这两个是需要我们自己配置的，我们在 Facebook 注册开发者建立自己的应用后，有一个对应的 APP ID 以及 APP SECRET，这两个就是我们要的啦。

现在我们去 developers.facebook.com 注册自己的 app，`梯子自备`，完成之后把 APP ID 和 APP SECRET 复制下来即可。记得为 app 添加一个 web app 的类型，然后设置它的 url，这里由于我们是在本机测试，所以设置为 http://localhost:3000/

下来继续回到 Rails 中进行设置，我们在登录的页面加入 Login with Facebook 这个链接，url 为 /auth/facebook。好了下来就进入浏览器中点击这个链接，我们的应用会跳转到 Facebook 的页面，我们输入自己的账号密码登录后，给予相应的权限。接着又转跳回我们的应用，BOOM! 出错了，`no route matches: auth/facebook/callback`。当我们的应用在 Facebook 那边授权成功之后，会执行一个回调函数，让我们对拿到的用户的相关数据进行操作。而这个回调函数是要由我们来写的，所以下来首先在 routes.rb 中加入对这条路由的处理:

#### config/routes.rb
{% highlight ruby %}
get 'auth/:provider/callback', to: 'omniauths#create'
{% endhighlight %}

我们设置的路由没有直接用 facebook 而是设为 :provider，这是因为后面我们可能会加入不同的第三方验证。后面的 to 就是处理这个 callback 的 controller 了。下来我们在 Rails 中加入这个 controller：

{% highlight ruby %}
rails g controller omniauths create
{% endhighlight %}

#### app/controllers/omniauths_controller.rb
{% highlight ruby %}
class OmniauthsController < ApplicationController
  def create
    raise env['omniauth.auth']
  end
end
{% endhighlight %}

这里我们并没有进行过多的处理，而是先将 OmniAuth 取回的数据输出了，接下来可以根据自己的需求取相应的内容。比如邮箱、uid、用户名之类的，接着用它来创建新用户即可。以后在用 Facebook 账号登录过程中，我们先会查询是否有该邮箱用户存在，并且 uid 也相同，如果存在的话则直接登录该用户即可。

### 小插曲
由于自己的项目中，最初是使用自己写的用户验证，使用了 Rails 自己提供的 `has_secure_password`。由于 has_secure_password 的存在，会每次验证 password_digest 这个属性。而在使用 Facebook 账号登录的过程中，由于并没有密码，因此创建的时候会报错。在查看文档之后发现，has_secure_passowrd 可以传入一个参数让它不对 password 进行验证，但是如果通过我们自己的验证系统注册的话这个验证又是却是需要的，最终我对它们进行了如下处理：

#### app/models/user.rb
{% highlight ruby %}
class User < ActiveRecord::Base
    has_secure_password(validations: false)

    validates :password, length: {minimum: 6},
                         confirmation: true,
                         on: :create,
                         if: :no_provider?

    private
    def no_provider?
        provider.blank?
    end
end
{% endhighlight %}

这样我们就可以只在创建用户的时，同时这个创建的过程中 provider 属性为空（说明是使用我们自己的验证系统注册）的情况下，才去对 password 进行验证。