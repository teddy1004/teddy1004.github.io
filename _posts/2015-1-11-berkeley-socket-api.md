---
layout: post
title: "《TCP Sockets 编程》书摘"
date: 2015-1-11 15:40:33
---
这周在读一本关于关于 TCP Sockets 编程的书，叫做 `Working with TCP Sockets`。书中主要是以一个 Ruby 程序员的角度从较高层来讲 TCP Sockets 编程。对于我这个非计算机专业出身的程序员来说，读起来非常浅显易懂，学到了很多以前不知道的知识，这里主要纪录一下基本的知识。

### Berkeley 套接字

Berkeley 套接字 API 是一种`编程 API`，运作在实际的协议实现之上。它关注的是连接两个端点共享数据，而非处理分组和序列号。Berkeley 套接字 API 之所以能屹立不倒的一个关键原因就是: _你可以在无需了解底层协议的情况下使用套接字_。Ruby 提供了套接字 API 的包装类，让我们可以更轻易的使用这一套 API。

### 服务器生命周期

服务器套接字用于__侦听连接__而非__发起连接__，其典型的生命周期如下:

1. 创建
2. 绑定
3. 侦听
4. 接受
5. 关闭

下面是一段 Ruby 的示意代码:

```ruby
require 'socket'

# 创建一个新的套接字
socket = Socket.new :INET, :STREAM

# 创建一个 c 结构体用来保存用于侦听的地址
addr = Socket.pack_sockaddr_in 4481, '0.0.0.0'

# 执行绑定
socket.bind addr
```

### 客户端生命周期

客户端的生命周期要比服务器短一些:

1. 创建
2. 绑定
3. 连接
4. 关闭

```ruby
require 'socket'

socket = Socket.new :INET, :STREAM

# 发起到 google.com 端口 80 的连接 (梯子自备:))
remote_addr = Socket.pack_sockaddr_in 80, 'google.com'
socket.connect remote_addr
```

### 交换数据

可以将 TCP 连接想像成一串连接了本地套接字和远程套接字的`管子`，我们可以沿着这跟管子发送、接受数据。
前面的代码中，我们创建套接字的时候传入了一个 `:STREAM` 的选项，该选项表明我们希望使用一个 `流套接字`。TCP 是一个基于流的协议。如果我们在创建套接字的时候没有传入 :STREAM 选项，那就无法创建 TCP 套接字。

从应用程序的代码的角度上来说，TCP 连接提供了一个不间断的、有序的通信流。`只有流，别无其他`。

```ruby
data = %w(a b c)

data.each do |piece|
  write_to_connection(piece)
end

# 下面的代码在一次操作中读取全部数据
result = read_from_connection #=> ['a', 'b', 'c']
```

上面代码表明，`流并没有消息边界`的概念。即便是客户端分别发送了3份数据，服务器在读取时，也是将其作为一份数据来接受。它并不知道客户端时分批发送的数据。要注意的时，极端并不保留消息边界，但是流中内容的次序还是会保留的。
