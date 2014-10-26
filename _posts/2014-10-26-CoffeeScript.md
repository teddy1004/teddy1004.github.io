---
layout: post
title: "Write better JavaScript with CoffeeScript"
date: 2014-10-26 17:40:33
---
对于一个 Rails 开发者来说，很多时候我们需要和 JavaScript 打交道。用习惯了优美的 Ruby 之后，有时实在是难以忍受丑陋的 JavaScript。语法丑陋不说，这个语言的坑还不少。写起来时常要小心翼翼，却还是难免出些难以发现的问题。`CoffeeScript` 的出现算是解放了很多 Rails 开发者。从此写 JS 也慢慢可以变成一种享受的事情。

简单的来说，CoffeeScript 其实就是原生的 JavaScript 加了许多的语法糖，在正式运行部署前，由 CoffeeScript compiler 将它编译为常规的 JavaScript 然后运行。所以首先它是不会影响效率的，因为最后运行的还是 JavaScript，只是需要我们在发布前多一个编译的步骤，但是开发死后效率却能大大的提升。

### Syntax

CoffeeScript 的语法很像是 `Python` 和 `Ruby` 的结合体，对于 `Ruby` 开发者来说，CoffeeScript 的学习成本还是很低的

#### 定义变量

```coffeescript
a = 1
```

这个就和 Ruby 中是一样的，这里它就帮我们跳过了一个小坑。在 JavaScript 中我们这样定义 `var a = 1;`，有时我们发现不加前面的 `var` 也是可以，但是如果真的不加就很危险。因为不加的话这个变量的作用域就是全局的，导致污染了全局的命名空间。而在 CoffeeScript 中这些我们不需要关心，compiler 在最后编译的时候会自行为我们处理。

#### 定义函数

先来看 JavaScript 中，我们通常是这样定义的

```javascript
function sayHello() {
  console.log("Hello, world");
}
```

在来看 CoffeeScript 中，我们定义的是这样的

```coffeescript
sayHello = ->
  console.log "Hello, world"
```

看起来不错吧，我是觉得那个 `->` 很性感。对了，CoffeeScript 中方法调用也是可以不加括号的。还要说说，我们在 CoffeeScript 中定义方法都强制要将它赋给一个变量。这样的好处是，这段 JS 在运行的时候，不会被预加载进来，只有调用的时候才会加载。即使在写原生的 JS 的时候，我们也应该使用这种定义方法的规范。

必包呢？也很容易

```javascript
(function() {
  console.log("Hello, world");
}())
```

```coffeescript
do ->
  console.log "Hello, world"
```

哪个优雅一目了然！

#### JavaScript 恶心的作用域

初学 JavaScript 的时候我们肯定被 `this` 这个东西恶心到过，它总是变来变去，有时候稍微不注意就记错了 `this` 到底指向的是哪里。下面是我以前写过的一个 Backbone 的代码

```javascript
remove: function () {
  this.model.destroy({
    success: function () {
      $('.loading').hide();
      this.$el.remove();
    }
  });
}
```
上面的代码的意思是当点击 view 层的 `remove` 之后去移除 `model` 层的数据，当 model 层的数据移除成功之后。将 view 层对应的 `element` 移除掉。这段代码看起来没问题吧？但是它是没法工作的。`this.$el.remove()` 这里，这里已经进入了 model 的作用域，this 指的是 model，它是没有 `element($el)` 的。我们得这样写：

```javascript
remove: function () {
  var _self = this;
  this.model.destroy({
    success: function () {
      $('.loading').hide();
      _self.$el.remove();
    }
  });
}
```

CoffeeScript 可以帮我们轻松解决这个问题，在定义方法时候，我们使用 `=>` 来定义，它会帮助我们锁定上下文

```coffeescript
remove: =>
  @model.destroy
    success: ->
      $('.loading').hide()
      @$el.remove()
```

对了，在 CoffeeScript 中 `@` 就代表的是 `this`。

#### 定义 class

这也是 JavaScript 奇葩的一点，它不是用累积成的，而是一种略奇怪的原型继承的方式。CoffeeScript 帮我们让 JS 也面向对象了。

```
class Dog
  constructor: (@name) ->

  bark: ->
    console.log "wang wang wang"

  introduce: ->
    console.log "I'm #{@name}"
```

要用的时候去 new 就好咯。对了 CoffeeScript 中和 Ruby 一样使用 `#{}` 的 `String interpolation`。

这是使用 CoffeeScript 中一些感觉比较爽的地方，关于它的一些更多的基本语法 http://coffeescript.org 中可以看到，它还有一个在线的交互台可以让你更快的了解。`Rails` 中现在已经默认对 `CoffeeScript` 支持了，我们在最后 `rake assets:precompile` 的时候它会帮我们来编译，不需要我们做额外的设置。其他的平台，也有诸如 `Grunt` 之类的工具来编译，很方便。总之，如果你经常和 JavaScript 打交道的话，CoffeeScript 是绝对值得去学习的。但是，在学习之前，也还是有必要对 JavaScript 有一定的了解，这样能让我们更好的使用 CoffeeScript。