---
layout: post
title: "关于 Rails 中 time_ago_in_words 的缓存"
date: 2014-6-21 23:30:33
---
昨天很 Happy 的把自己的个人项目部署起来了，结果后来发现有个地方出了点问题。就是在生产环境下，`views`中使用`time_ago_in_words`会有问题，由于页面是缓存到`redis`的。这就导致我们得到的类似`1分钟之前`这个结果始终都是不变的，而且还会引发`I18n`的问题。因为不同用户使用的语言设置可能不同，如果创建者使用的英文，那么`time_ago_in_words`在缓存中的就是英文，而中文环境的浏览者看到的也是中文了。虽然可以通过给`cache`中加入`locale`来区分不同语言环境保存不同的`cache`，但是时间显示错误的这个问题还是没法解决。

后来查阅了一些资料之后，发现`time_ago_in_words`也确实不是`best_practice`。除去`缓存`和`I18n`的问题之外，由于`time_ago_in_words`由于需要服务器来计算时间，所以也会影响性能(虽然应该不大)。因此，这种事情更适合交给前端来做。很多时候缓存都需要和`前端JS`来配合达到最好的效果。一方面我们尽可能多利用混存，这样能提高服务器的相应速度。另一方面，对于页面用户逻辑较多导致不同用户之间页面差别较大的情况，如果我们对每个用户的页面分别进行缓存，这样内存一定会爆掉。比较合适的方式就是我们对整个完整的页面进行混存，然后在前端根据不同用户的不同权限用`JS`来调整页面的显示。

回到`time_ago_in_words`的解决，这里比较好的一种方式是使用`time_tag`这个`view helper`，它返回的`html`类似这样

```html
<time datetime="2014-06-21T09:16:04Z" title="June 21, 2014 09:16"></time>
```

接着我们使用`jQuery`的一个叫做`timeago`的插件来计算距离现在的时间，我们可以自己写一个`view helper`

```ruby
def time_ago_tag(time)
    time_tag time, data: { behaviors: 'timeago'}
end
```

在`views`中，比如有一个`comment`，那么我们可以这样使用

```html
<%= time_ago_tag comment.created_at %>
```

生成的`html`就是下面这样

```html
<time data-behaviors="timeago" datetime="2014-06-21T09:16:04Z" title="June 21, 2014 09:16">about 7 hours ago</time>
```

这里我们用到了`data-behaviors`，这是`html 5`中新的一个属性，对我来说使用它的最大的好处就是`css selector`写起来会简单很多，比如我们的例子只需要这样

```javascript
$('[data-behaviors=~timeago]').timeago();
```

这样还有一个好处就是语义上更加明确，我们只需在命名时候取一个相符合的`behavior`，那么以后看起来就一目了然，相比`css selector`易读太多。

至此，我们就将时间的渲染交给了前端，解决了时间混存的问题。但是`I18n`依旧有问题，因为现在都是英文了。`jQuery.timeago`当然考虑到这个问题了，去他们的 Git Repo 里面能找到一堆国际化的文件了，我们只需要下载`jquery.timegago.zh-CN.js`到`vendor/assets/javascripts/locacles`就可以了。

###### jquery.timeago.zh-CN.js

```javascript
jQuery.timeago.settings.strings = {
    prefixAgo: null,
    prefixFromNow: "从现在开始",
    suffixAgo: "之前",
    suffixFromNow: null,
    seconds: "不到1分钟",
    minute: "大约1分钟",
    minutes: "%d分钟",
    hour: "大约1小时",
    hours: "大约%d小时",
    day: "1天",
    days: "%d天",
    month: "大约1个月",
    months: "%d月",
    year: "大约1年",
    years: "%d年",
    numbers: [],
    wordSeparator: ""
};
```

但是注意我们没必要在`application.js`中引入这个文件，因为它不是必须的，如果用户使用的是英文环境，那么是不需要引入`jquery.timeago.zh-CN.js`文件的。

所以我们需要新加一个编译的文件，在`app/assets/javascripts`目录下新建`locales/zh-CN.js`

###### zh-CN.js

```javascript
//= require locales/jquery.timeago.zh-CN
```

接着在`production.rb`中设置要增加的编译文件

```bash
config.assets.precompile += %w(locales/zh-CN.js)
```

最后我们在`application.html.erb`的`head`部分判断`I18n.locale`，如果不是`en`的话我们就引入相应的`locale`文件，比如

```html
<% if I18n.locale != 'en' %>
<%= javascript_include_tag "locales/#{I18n.locale}" %>
<% end %>
```

至此，我们关于`time_ago`显示的过程中出现的`缓存`和`I18n`问题都解决了。