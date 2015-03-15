---
layout: post
title: "Customize ember-data serializer"
date: 2015-3-15 21:30:42
---
Ember.js 有一个官方配套的用于管理数据模型的 framework 叫做 `ember-data`，它在很多方面和 Rails 的 ActiveRecord 很像，初次使用起来会觉得很方便。这也是它的设计哲学之一 `convention over configuration` 带来的好处。

但是，往往这种 `opinioned` 的 framework 都是你按它的意思来会觉得很爽，反之你会抱怨这玩意儿怎么这么坑。可是这里的问题是，`ember-data` 是直接和后端 API 打交道的，前端对于 API 怎么定实在是无能无力哎，而且我至今对它的那套 `sideload` 方式心存疑问。先来看看具体的示例吧。

假设我们有两个 model `user` 和 `post`，它们之间是 `1-N` 的关系。

我们在 ember-data 中这样定义它:

```javascript
App.User = DS.Model.extend({
      name: DS.attr('string'),
      posts: hasMany('posts')
    });

App.Post = DS.Model.extend({
      body: DS.attr('string')
    });
```

ember-data 期待后端返回的数据是这样的:

```json
{
  "user": {
    "id": 1,
    "name": "Teddy",
    "posts": [1, 2, 3]
  },

  "posts": [
    {"id": 1, "body": "Hello, world"},
    {"id": 2, "body": "Hello, world"},
    {"id": 3, "body": "Hello, world"},
  ]
}
```

这就是 ember-data 推行的 `sideload` 的形式，它列举了一堆这样写的好处是。可是后端 API 不是这样啊，而且 Rails 通常的 API 返回的习惯也不是这样。所以每次接到后端的数据，就会报错。

由于一开始入了 ember-data 的坑，发现这个问题的时候已经写了一大半了总不能推倒重来，而且 ember-data 确实提供了其他很多的便利。于是开始摸索它们的代码，发现在 `DS.ActiveModelSerializer.extend` 有这样一个方法 `normalizeRelationships`，我们可以 hack 它来实现我们对自己的数据的序列化。

大致代码是这样的

```javascript
normalizeRelationships: function (type, payload) {
  var _self = this;

  this._super(type, payload);

  type.eachRelationship(function (key, relationship) {
    var relatedTypeKey = relationship.type.typeKey;

    if (relationship.options.embedded) {
      if (relationship.kind === 'hasMany') {
        payload[key] = payload[key].map(function(embeddedHash) {
          return _self._serializeRelationships(relationship, relatedTypeKey, embeddedHash, key);
        });
      } else if (relationship.kind === 'belongsTo') {
        payload[key] = _self._serializeRelationships(relationship,
          relatedTypeKey, payload[key], key);
      }
    }
  });
}
```

大概就是根据 type 是否是 `hasMany` 或者 `belongsTo` 然后去进行我们自己的序列化。

ember-data 总的来说还是为开发提供了不少便利，但是对于后端 API 不够 restful 或者不符合他们期望的结构的时候，用起来会很痛苦。最近在看 `Discourse` 的源代码，发现作者并没有用到 ember-data 而是直接用 ember 自己的 `Ember.object` 来做模型对象，受到了很多启发，在下来空闲的时间我也会尝试以这种形式来做。
