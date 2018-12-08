## class 语法糖
`class` 语法糖提供了一个看起来更优雅的语法来实现原型继承. 实际上, `class` 的继承在技术上它是基于原型继承实现的.

基本语法:
```javscript
class MyClass {
  constructor(...) {
    // ...
  }
  method1(...) {}
  method2(...) {}
  get something(...) {}
  set something(...) {}
  static staticMethod(..) {}
  // ...
}
```

我们将之前的 `Rabbit` 类用 `class` 的语法改写:
```javscript
class Rabbit {
  constructor(name) {
    this.name = name;
  }

  jump() {
    console.log(`${this.name} jumps!`);
  }
}
```

虽然 `Rabbit` 前有 `class` 关键字, 但是实际上它是一个函数: `typeof Rabbit == 'function'`, 毕竟 JavaScript 里并没有 `class` 这种数据类型. `class Rabbit() {...}` 实际上完成了两件事:

1. 声明了一个名为 `Rabbit` 的变量, 并将它的值指向了 "constructor" 函数.
2. 把所有类中定义的方法“挂”到 `Rabbit.prototype` 上. 如示例中的 `jump` 和 `constructor` 两个方法.

综上可以发现 `class` 是一个特殊的语法, 它可以同时定义类的构造函数和它原型链上的方法. 不过它与传统定义方法间还是存在一些细微差别:

1. class 必须与 `new` 关键字一起使用.
2. class 中的方法是 non-enumerable(不可枚举)的. 在 class 中, 所有 "prototype" 上的方法, 其 `enumerable` 标志会被自动设置为 `false`. 这很棒, 因为当我们使用 `for..in` 遍历 object 的属性时, 我们通常不希望结果中包含有类上的方法.
3. class 拥有一个默认的 `constructor() {}`. 如果在 class 结构中没有定义 `constructor`, 那么 JavaScript 会生成一个默认的空构造函数.
4. class 永远是 `use strict` 的.

Getter/Setter:

```javascript
class User {
  get name() {
    return this._name
  }

  set name(name) {
    this._name = name
  }
}
```
在 JavaScript 内部, getter 和 setter 的实现都是在 `User.prototype` 上创建相应的方法:
```javascript
Object.defineProperties(User.prototype, {
  name: {
    get() {
      return this._name
    },
    set(name) {
      this._name = name
    },
    enumerable: false,
    configurable: true
  }
});
```

静态方法:

```javascript
class User {
  static staticMethod() {}
}
```

在 ES5 中相当于
```
function User() { }

User.staticMethod = function() {};
```

静态方法并不位于 `constructor.prototype` 里, 是 class 独有的方法, 由于 class 本质上也是一个函数, 所以也是一个 object, 当然可以拥有自己的属性. 当使用 `new` 构造出的实例并不会拥有这个方法.

### class 继承
```javascript
class Animal {
  stop() {}
}

class Rabbit extends Animal{}

let rabbit = new Rabbit();
// rabbit.stop()
```

`extends` 关键字实际上是让 `Rabbit.prototype` 的属性 `[[Prototype]]` 指向 `Animal.prototype`.

如果我们在 `Rabbit` 中定义了自己的 `stop` 方法, 那么它将被用来代替 `Animal` 的 `stop` 方法.

```javascript
class Rabbit extends Animal {
  stop() {
    // ...这将用于 rabbit.stop()
  }
}
```
但是通常来说, 我们不希望完全替换父类的方法, 而是希望基于它做一些调整或者扩展. 我们需要在 `stop` 的执行过程中调用父类的方法. 为此, 类提供了 `super` 关键字:
- `super.method(...)` 调用一个父类的方法.
- `super(...)` 调用父类的构造函数

方法的 overriding:

```javascript
class Animal {
  stop() {
    console.log('stop');
  }
}

class Rabbit extends Animal{
  stop() {
    setTimeout(()=> super.stop(), 2000)
  }
}
```
`setTimeout(()=> super.stop(), 2000)` 不能写成 `setTimeout(function() {super.stop()}, 2000)`. 普通函数是有 `super` 属性的, `setTimeout(function() {super.stop()}, 2000)` 里函数的 `super` 此时是 `undefined`. 但是箭头函数是没有 `super` 的, 它的查找与普通变量搜索完全相同, 是基于词法上下文的.

构造函数 overriding:

继承类的构造函数必须调用 `super(...)`, 并且一定要在使用 `this` 之前调用, 如果一个类继承了另一个类并且没有构造函数, 那么将自动生成以下构造函数:
```javascript
class Rabbit extends Animal {
  // 为没有构造函数的继承类生成以下的构造函数
  constructor(...args) {
    super(...args);
  }
}
```
当一个普通构造函数执行时, 它会创建一个空对象作为 `this` 并继续执行, 但是当派生的构造函数执行时, 它并不会做这件事, 它期望父类的构造函数来完成这项工作. 因此, 如果我们构建了我们自己的构造函数, 我们必须调用 `super`, 因为否则的话 `this` 指向的对象不会被创建, 并且我们会收到一个报错.

```javascript
class Animal {
  constructor(speed) {
    this.speed = speed;
  }
  stop() {
    console.log('stop');
  }
}

class Rabbit extends Animal{
  constructor(name, speed) {
    super(speed);
    this.name = name;
  }
  stop() {
    setTimeout(()=> super.stop(), 2000)
  }
}
```
[super 的内部实现](http://zh.javascript.info/class-inheritance#super-nei-bu-ji-yu-homeobject-shi-xian)

静态方法的继承:
```javascript
class Animal {
  static compare() {}
}

class Rabbit extends Animal{}
// Rabbit 会继承 compare 作为自己的静态函数
```
它是如何工作的? 正如你已经猜测的那样, 继承也给 `Rabbit` 的 `[[Prototype]]` 添加一个引用指向 `Animal`.

<img src='http://zh.javascript.info/article/class-inheritance/animal-rabbit-static@2x.png' width='50%' />

但是内置类却没有这种静态 `[[Prototype]]` 的引用. 例如, `Object` 有 `Object.defineProperty`, `Object.keys` 等静态方法, 但是 `Array, Date` 等等并不会继承他们. 这里有一张图来描述 Date 和 Object 的结构:

<img src='http://zh.javascript.info/article/class-inheritance/object-date-inheritance@2x.png' width='50%'/>

进一步阅读:
- [Babel 是如何编译 Class 的(上)](https://github.com/mqyqingfeng/Blog/issues/105)
- [Babel 是如何编译 Class 的(下)](https://github.com/mqyqingfeng/Blog/issues/106)
