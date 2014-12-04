---
layout: post
title: "Class Inheritance in JavaScript"
date: 2014-12-4 19:40:30
---
JavaScript 本身是一门 OO 语言，但是由于 JavaScript 中没有类的概念 (好像 ECMA6 中会加入)。在 JavaScript 中使用原型继承，这不同于一般的 OO 语言，简单的说，JavaScript 中的原型继承的工作方式大概如下:

* 一个对象有数个属性，这个属性既可以是 `attributes` 也可以是 `functions`
* 一个对象有一个 `parent property`，它就是这个对象的原型 (可以通过 `__proto__` 属性去访问)。
* 一个对象可以覆写父对象的属性
* `constructor` 用于创建对象
* 当一个对象被创建的时候，它的 `parent` 被设为这个对象的 `constructor` 的原型

用代码来解释一下上面的几点大概就是这样的:

```javascript
function Mammal() {
}

Mammal.prototype;
// {}

var mammal = new Mammal();

mammal.__proto__ === Mammal.prototype
// true
```

当我们需要实现继承的时候，写起来更是绕。CoffeeScript 解决了我们的这个痛点，它帮我们实现了类还有继承的功能，像下面这段代码：

```coffeescript
class Animal
  constructor: (@name) ->

  move: (meters) ->
    console.log "#{@name} moved #{meters}m."

class Snake extends Animal
  move: ->
    alert "Slithering..."
    super 5

class Horse extends Animal
  move: ->
    alert "Galloping..."
    super 45

sam = new Snake "Sammy the Python"
tom = new Horse "Tommy the Palomino"
```

看起来和 Ruby 中的继承的写法没什么区别，很长一段事件我就这样用着 CoffeeScript 写的很开心。突然有一天我想到，`CoffeeScript 不是最后还要编译成 JavaScript 的吗!` 也就是说通过 JavaScript 我们也可以自己去实现这样的一个需求。我们来一步一步的实现吧。


#### VERSION 1
```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.move = function(meters) {
  console.log(this.name + " moved " + meters + "m.");
};

function Snake() {
  return Animal.prototype.constructor.apply(this, arguments);
}

Snake.prototype = new Animal(); // 继承开始
Snake.prototype.constructor = Snake; // 不然 Snake 的 constructor 就会成为 Animal

Snake.prototype.move = function() {
  return Animal.prototype.move.call(this, 5);
};

var snake = new Snake('Teddy');
snake.move();
// Teddy moved 5m. 
```

上面的代码基本上实现了一个继承的感觉，但是代码的可拓展性很差，在 `Snake` 类中，我们是直接将 `Animal` 写死的。因此，我们有必要实现一个自己的继承的方法，经过一番改造时候，代码成了下面这样:

#### FINAL VERSION
```javascript
var __hasProp = {}.hasOwnProperty,
    __extends = function(child, parent) {
      // 将 `parent constructor` 中的属性复制到 `child constructor` 中
      for (var key in parent) {
        // 使用 __hasProp.call(parent, key) 而不是 parent.hasOwnProperty(key)
        // 主要是一方面据说效率有提升，还有就是程序的健壮性
        if (__hasProp.call(parent, key)) child[key] = parent[key] 
      }
      // 这是一个代理方法，在我们之前的实现中，我们使用
      // `Snake.prototype = new Animal()`的方式，我们这
      // 里需要的只是设置继承链，并没有并有去实例化一个父类
      function ctor() {
        this.constructor = child;
      }
      // 将夫类的原型链加入到代理方法的原型中
      ctor.prototype = parent.prototype;
      child.prototype = new ctor(); 
      // 这样就可以在子类中访问父类了
      child.__super__ = parent.prototype;
      return child
    };

var Animal = (function() {
    function Animal(name) {
      this.name = name;
    }

    Animal.prototype.move = function(meters) {
      return console.log(this.name + " moves " + meters + "m.");
    };

    return Animal;
})();

var Snake = (function(_super) {
    __extends(Snake, _super);

    function Snake() {
      return Snake.__super__.constructor.apply(this, arguments);
    }

    Snake.prototype.move = function() {
      return Snake.__super__.call(this, 5);
    };

    return Snake;
})(Animal);

var snake = new Snake('Teddy');
snake.move();
// Teddy moves 5m.
```

通过将这个继承的方法抽象出来，代码就清晰了很多，复用性也得到了增强。虽说 `Coffee 大法好`，但是了解一下 CoffeeScript 是如何帮我们绕开 JavaScript 中的这些坑还是很有必要的！
