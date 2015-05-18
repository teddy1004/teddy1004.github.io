---
layout: post
title: "Render JSONP response in Rails"
date: 2015-5-18 18:30:33
---
今天在给前端提供一个接口的时候, 由于有跨域的问题, 加之都是 `get` 请求, 所以就通
过 [jsonp](http://en.wikipedia.org/wiki/JSONP) 来比较简单的实现。

Rails 中提供了很简便的接口, 通常我们这样返回 json 数据

```ruby
render json: { hello: "world" }
```

对于需要加上 json callback 的情况我们只需要多加一个 option 就可以了

```ruby
render json: { hello: "world" }, callback: params[:callback]
```

这样当我们通过 ajax 去请求别的域名的数据的时候, 加上一个 callback 参数, 比如
 `?callback=abc`。这样返回的数据就会被包裹在一个 JavaScript 的方法里, 像下面这样

```javascript
abc({
  hello: "world"
})
```

这时前端同学反应希望返回的数据是 `abc && abc()` 的形式, 这种形式对于当服务端返回一个客户端
不识别的方法有着更好的容错处理, 看起来觉得很简单, 把返回的数据包装一下就可以了呗。做起来发现
有问题了。

开始我是这样解决的

```ruby
render json: "#{callback} && #{callback}(#{json})"
```

问题一出现了, 里面的那些 JSON 数据格式不对, 返回的直接是 Ruby 的 Hash 的数据格式 (突然我
领悟了一些人说 Node.js 一大好处是统一前后端的语言, 当时还一直不明白这统一有毛用啊)。那就 encode
一下呗

```ruby
json = ActiveSupport::JSON.encode json
render json: "#{callback} && #{callback}(#{json})"
```

这下总可以了吧, 结果客户端又抱错了, 看了报错信息我一下就懵逼了, Mime Type 不对。不应该用
`application/json`。可是 Rails 那个方法不也是 `render json` 吗。OK, 菜菜不在, 只能自己查
源码了。

```ruby
# rails/actionpack/lib/action_controller/metal/renderers.rb line 115
add :json do |json, options|
  json = json.to_json(options) unless json.kind_of?(String)

  if options[:callback].present?
    if content_type.nil? || content_type == Mime::JSON
      self.content_type = Mime::JS
    end

    "/**/#{options[:callback]}(#{json})"
  else
    self.content_type ||= Mime::JSON
    json
  end
end
```

原来对于传入 `callback` 参数的这种, 虽然我们写的是 `render json`, 但是其实返回的 content_type
 是 `application/js`。jsonp 需要的数据就是这样的类型滴, 不然那个 JS 方法如何执行呢。
