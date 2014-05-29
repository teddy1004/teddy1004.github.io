---
layout: post
title: "不输入密码连接远程服务器"
date: 2014-5-29 14:13:25
---
在部署应用的时候，我们通常要连接远程服务器，如果不使用一些专门的 ssh 软件而是使用终端的话。首先要在 shell 输入

```bash
ssh teddy@192.168.1.1
```

然后再输入密码，这一长串难记的 IP 加上密码让每次登录远程服务器都是一个挺不爽的过程。

直到发现 `ssh-copy-id` 这个很方便的工具，它的功能其实就是将本机的一个 RSA 秘钥上传到远程服务器的 `authorized_keys`,
以后我们连接远程服务器的时候就会自动拿这个秘钥来验证，省去了我们自己输入密码的过程。

首先来在本机生成一个秘钥

```bash
ssh-keygen -t rsa
```
这里我给自己的秘钥的名称是 `teddy_rsa`，会生成两个文件 `teddy_rsa` 以及 `teddy_rsa.pub`

接下来就是使用 `ssh-copy-id` 了，Mac 下面这个东西是要自己安装的

```bash
brew install ssh-copy-id
```

安装完成之后就是可以使用 ssh-copy-id 上传我们自己的秘钥了

```bash
ssh-copy-id -i teddy_rsa.pub teddy@192.168.1.1
```

### OK

接着我们可以输入 `ssh teddy@192.168.1.1` 连接远程服务器，发现真的不用输入密码了。可是还是不够方便，
我们已然得每次输入那长长的 IP，其实是有办法解决的，我们在 `~/.ssh` 目录下建立一个 `config` 文件，
这个文件专门用来配置我们的服务器的信息的，我们现在的这个服务器可以这样配置

**config**

```bash
Host teddy
HostName 192.168.1.1
User teddy
```

这样之后，我们只需要在终端输入 `ssh teddy` 就可以直接连接远程服务器了！
