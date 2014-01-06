---
layout: post
title: "为 Jekyll 添加 CoffeeScript 和 Sass 的支持"
date: 2014-1-6 00:16:30
---
Jekyll 用来写博客很爽，可是写习惯了 CoffeeScript 和 Sass 的话，该如何让 Jekyll 支持它呢。其实很简单，实现方式和 Rails 里面差不多。
首先是建立一个 Gemfile ，引入需要的 gems

{% highlight ruby %}
gem "jekyll"
gem "sass"
gem "coffee-script"
# rake后面要用到
gem "rake"
{% endhighlight %}

下面就要用到之前导入的 rake 这个 gems 了，rake 就不用介绍啦～ Ruby Make

{% highlight ruby %}
desc "compile and run the site"
task :default do
    pids = [
        spawn("jekyll server -w"),
        spawn("scss --watch _assets:stylesheets"),
        spawn("coffee -b -w -o javascripts -c _assets")
    ]

    trap :INT do
        Process.kill :INT, *pids
        exit 1
    end

    loop do
        sleep 1
    end
end
{% endhighlight %}

简单的说 Rakefile 中的代码

* 首先来说说 spawn，spawn 会创建一个子进程然后返回该进程的 pid，因此 pids 中保存了 3 个子进程的 pid。
* trap 这里用到 Unix 信号，trap :INT 即监控 Interrupt 信号，当收到 :INT 信号后，杀死所有 pids 中的3个进程(Process.kill :INT, *pids)然后退出。
* 最后一段代码的目的是要使主进程一直处于休眠状态呢，为什么要这样做呢？因为当子进程还未执行完自己的任务时主进程就退出的话，子进程就会变为 Orphan Process (孤儿进程)，这样会不便于我们监视进程的运行状态