---
layout: post
title: "初识 Docker"
date: 2014-10-12 17:40:33
---
![](http://2.bp.blogspot.com/-6bjnf703MRY/U6mzyawC2KI/AAAAAAAABic/RdB3XkyLduY/s1600/docker_logo.png)

这家伙现在实在是太火了，以前对类似的工具没怎么使用过，这一次当然不能错过了。于是花了点时间，做了一个简单的了解和一些 `Hello World!` 的尝试。

下面是 Docker 的官方给出的定义:

> Docker is an open platform for developers and sysadmins to build, ship, an> d run distributed applications.

### 安装
Mac 用户使用 Boot2Docker 来安装

今天先来说说 Docker 的几大特性:

### 和虚拟机的区别

看看下面这张图就能明白个大致了:
![](http://blog.trifork.com/wp-content/uploads/2013/07/Screenshot_from_docker.io_about.png)

传统的虚拟机不仅包括了我们运行的应用程序，以及必要的二进制文件和库，还同时包含了整个用户操作系统，这可能就占据了 10 GB 的容量。

而 `Docker Engine` 的容器由应用和它的依赖组成。Docker 容器的运行不需要额外的 hypervisor 支持，它是内核级的虚拟化，可以实现更高的性能和效率。

Docker 包括三个基本概念

* 镜像 (Image)
* 容器 (Container)
* 仓库 (Repository)

这三个概念共同组成了 Docker 的整个生命周期

### 镜像
Docker 镜像就是一个只读的模版，例如一个镜像可以包含一个完整的 Ubuntu 系统，里面仅安装了 Nginx 或者你需要的其他的应用。`镜像可以用来创建 Docker 容器`。

### 容器
Docker 利用容器来运行应用

容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。可以把容器看作是一个应用程序。`镜像是只读的，容器在启动的时候创建一层可写层作为最上层`。

### 仓库
仓库是集中存放镜像文件的场所，分为`公有仓库`和`私有仓库`2种形式。

仓库和 Git 中的仓库很类似，目前公有的仓库有 `Docker Hub`，它提供了一个数量庞大的镜像库供用户下载。

用户创建了自己的镜像之后，就可以使用 `push` 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像的时候，只需要从仓库上 `pull` 下来就可以了。

这是这周利用闲暇时间了解了一下 Docker，还停留在一个比较初步的阶段。关于 Docker 在开发中更多的使用，下周再和大家分享。如果对 Docker 比较感兴趣，可以先浏览一些官网的 guides，能让我们对一些常用的基本命令有一个全面的认识。
