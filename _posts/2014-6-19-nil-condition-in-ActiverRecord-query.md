---
layout: post
title: "ActiveRecord 中的一个小陷阱"
date: 2014-6-3 14:56:30
---
我们在 ActiveRecord 中使用`where`查询时，通常是下面这样写

```ruby
User.where("name = ?", name)
```

在查询中我们使用`?`来为我们转义，这样有效的避免了`SQL injection`。今天发现在有有一种情况下需要我们特别的注意，那就是当查询条件为`nil`的时候。根据条件我们希望生成的`SQL`语句是这样`SELECT * FROM users WHERE users.name IS NULL`。

按照之前的查询，我们通常的查询是

```ruby
User.where("name = ?", nil)
# => User Load (1.8ms)  SELECT "users".* FROM "users" WHERE (email = NULL)
```

可以看到生成的查询语句并不是我们所要的，原因就是`Rails 没有那么聪明`，这里`?`并不知道我们需要把`email = NULL`转换为`email IS NULL`。

所以在`nil`的情况下我们改用这种方式查询

```ruby
User.where(name: nil)
# => User Load (0.3ms) SELECT "users".* FROM "users" WHERE "users"."name" IS NULL
```

可以看到，后者的速度相比前者有很大的提升。
