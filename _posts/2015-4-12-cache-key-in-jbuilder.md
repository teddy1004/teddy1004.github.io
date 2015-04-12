---
layout: post
title: "Cache key in jbuilder"
date: 2015-4-12 22:00:33
---
本周工作中遇到一个 bug，功能大致是这样的: 在后台配置 app 首页的 sliders， 然后向客户端返回配置的 slider 的 api。由于考虑到这个 slider 变化不会太频繁，所以使用 `json.cache! 'a/b/c'` 来显示的命名 `cache_key` 为 `a/b/c`，缓存设为永不过期。在后台更新 sliders 数据后，去主动清楚这个 cache_key 对应的内容。

这种做法与在 model 中通常使用的方式一样，在 model 中有时会使用 `Rails.cache.write 'cache_key_name'`，然后用 `Rails.cache.delete 'cache_key_name'` 去主动使缓存过期。

但是在上线后，发现后台配置更改后缓存并未过期。思考觉得应该是 jbuilder 的 `cache!` 方法生成的 key 会在我的 key 上加上一些东西，请教了 `源码cai` 之后，他告诉去看`哪个文件夹的哪个文件的多少行`，答案就有了。看了之后豁然开朗，果然多读源码多多益善，好了，上代码。

```ruby
# File 'lib/jbuilder/jbuilder_template.rb', line 51

def cache!(key=nil, options={})
  if @context.controller.perform_caching
    value = ::Rails.cache.fetch(_cache_key(key, options), options) do
      _scope { yield self }
    end

    merge! value
  else
    yield
  end
end
```

好了，关键就是 `_cache_key(key, options)` 这里，继续看代码

```ruby
# File 'lib/jbuilder/jbuilder_template.rb', line 108

def _cache_key(key, options)
  key = _fragment_name_with_digest(key, options)
  key = url_for(key).split('://', 2).last if ::Hash === key
  ::ActiveSupport::Cache.expand_cache_key(key, :jbuilder)
end
```

关键就在 `::ActiveSupport::Cache.expand_cache_key(key, :jbuilder)` 这里，这里专门传入了 `:jbuilder` 这个 `namespace`，可见会有一些特殊的处理，继续 step in。

```ruby
# activesupport/lib/active_support/cache.rb

def expand_cache_key(key, namespace = nil)
  expanded_cache_key = namespace ? "#{namespace}/" : ""

  if prefix = ENV["RAILS_CACHE_ID"] || ENV["RAILS_APP_VERSION"]
    expanded_cache_key << "#{prefix}/"
  end

  expanded_cache_key << retrieve_cache_key(key)
  expanded_cache_key
end
```

由于传入了 `namesapce` 导致 Rails 会对 `cache_key` 进行进一步的包装，与我们自己主动声明的并不完全一样，所以在清理 key 的时候会出现问题，因为生成的 key 是 `expanded`。不过，Rails 为什么会对 jbuilder 的 cache_key 进行拓展，还不是很明白，有待进一步发掘。
