---
layout: post
title: "Struct OpenStruct Hash in Ruby"
date: 2014-8-31 11:45:31
---
Ruby 中有三个看起来挺类似的数据结构，`Struct`, `OpenStruct`, `Hash`，一直不是很清楚他们具体使用的场景，周末花时间了解了一番，觉得大致可以总结如下：

### Struct 使用场景

* 需要一个容器，并且这个容器里面的字段在写代码的时候是已经定义好的
* 想要快速定义一个包含几个字段的 `Class like` 的东西
* 需要检测出由于误输入字段名称导致的错误

### OpenStruct 使用场景

* 字段数量在运行时候是确定的，但是可能在开发阶段中有频繁的变化（通常是由于传入的数据不定导致的）
* 其他情况来说，OpenStruct 和 Struct 的场景是差不多的，但是他们有一个重要的区别，`Struct` 是 Ruby 解释器内置，用 C 实现；而 `OpenStruct` 是 Ruby 标准库，用 Ruby 实现，所以 Struct 的性能要远远好于 Struct

### Hash 使用场景

* 字段的数量在编码的过程中是不确定的
* 通常从文件中读取 `key value` 的时候, Hash 也会经常使用到，因为这时字段的数量也是不固定的而且有可能会很大
