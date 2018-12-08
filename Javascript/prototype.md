## 函数的 prototype
每个函数都有叫作 "prototype" 的常规属性(相对于访问器属性). 它的值必须是一个对象或 `null`, 其他值将不起作用. 默认的 "prototype" 是一个只有属性 `constructor` 的对象, `constructor` 又指向这个函数本身.

<img src='http://zh.javascript.info/article/function-prototype/function-prototype-constructor@2x.png' width='50%'/>

```javascript
function Rabbit() {}

console.log(Object.getOwnPropertyDescriptor(Rabbit, 'prototype'));
/*
{ value: Rabbit {}, // constructor 没有被列出来
  writable: true,
  enumerable: false,
  configurable: false }
*/

console.log(Object.getOwnPropertyDescriptor(Rabbit.prototype, 'constructor'));
/*
{ value: [Function: Rabbit],
  writable: true,
  enumerable: false,
  configurable: true }
*/
```
由于可以人为地修改 "prototype" 对象, 我们不能确保 "constructor" 函数值指向正确的构造函数. 特别是, 如果我们将整个默认 "prototype" 替换掉, 那么其中就不会有构造函数. 因此, 为了确保正确的 "constructor", 我们可以选择添加/删除属性到默认 "prototype" 而不是将其整个覆盖, 也可以手动重新创建正确的 `constructor` 属性.

## 对象的 [[Prototype]]
每个对象有一个特殊的隐藏属性 `[[Prototype]]`, 它的值必须是 `null` 或者是另一个引用对象. 属性 `[[Prototype]]` 是内部的而且隐藏的, 你无法直接用 `.` 运算符对它进行读写操作. 后面会看到如何设置对象的 `[[Prototype]]`.

假设我们有一个构造函数 `F`, 用 `new F()` 创建一个新的对象时, 如果 `F` 有一个 `prototype` 属性, 该属性具有一个对象类型的值, 那么 `new` 运算符就会使用它为新对象设置 `[[Prototype]]`. 具体的过程请看文章 [模拟 new 的实现](./Topics.md). 以 `obj = new Object()` 为例, 当 `new Object()` 被用来创建一个对象时, 这个对象的 `[[Prototype]]` 属性被设置为 `Object.prototype`:

<img src='http://zh.javascript.info/article/native-prototypes/object-prototype-1@2x.png' width='50%' />

我们看到 `Object.prototype` 所引用的对象里有很多函数, 其中还包括了我们将用来设置 `[[Prototype]]` 的访问属性 `__proto__`:

<img src='http://zh.javascript.info/article/prototype-methods/object-prototype-2@2x.png' width='50%' />

## 原型链
假设对象 `obj1` 的 `[[Prototype]]` 引用 `obj2`, `obj2` 也是一个对象, 它也有自己的 `[[Prototype]]` 属性, `obj2` 的 `[[Prototype]]` 也可以引用另一个对象 `obj3`, 可以有限地进行下去最后终止于 `null`, 最后形成了原型链.

如果我们想要读取 `obj` 的属性或者调用一个方法, 而且它不存在该对象中, 那么 JavaScript 就会尝试在原型中查找它. 写/删除直接在对象上进行操作, 它们涉及它的原型对象(除非属性实际上是一个 setter).

设置 `obj` 的 `[[Prototype]]` 的方式是通过设置 `obj.__proto__` 来实现的. `__proto__` 属性并不存在于 `obj` 中, 而是存在于原型链上的 `Object.prototype` 对象中, 它是 `Object.prototype` 的一个访问属性, 如果 `obj.__proto__` 被读取或者赋值, 那么实际上是 `Object.prototype` 上的 `__proto__` 的 getter/setter 被调用, 它会获取/设置 `[[Prototype]]`.

```javascript
let animal = {
  eats: true,
  walk() {
    console.log("Animal walk");
  }
};

let rabbit = {
  jumps: true,
  __proto__: animal
};

let longEar = {
  earLength: 10,
  __proto__: rabbit
}

// walk is taken from the prototype chain
longEar.walk(); // Animal walk
console.log(longEar.jumps); // true (from rabbit)

```

<img src='http://zh.javascript.info/article/prototype-inheritance/proto-animal-rabbit-chain@2x.png' height='350px'/>

注意:
1. 对象属性的读写删操作: 如果该属性直接存在于该对象中, 那么操作的是该对象的属性. 如果该属性不直接存在于该对象中, 而是在原型上, 那么读取的是原型上的属性, 写则直接写入该对象而不是原型对象(除非属性是原型对象上的一个 setter), 删除属性直接在对象上进行操作, 不涉及它的原型对象.
2. `this`: 如果我们调用 `obj.method()`, 而且 `method` 是从原型中获取的, `method` 中的 `this` 仍然会引用 `obj`. 这是因为 `this` 是函数被调用执行的时候才会被确定, `this` 指向的永远是调用者, 而不是这个函数存在于哪个对象中.

## 原型的类和继承
我们可以用构造函数实现类, 例如
```JavaScript
function User(arg1,arg2) {
  function fun1(...) {}
  this.method1 = function(...) {};
  this.method2 = function(...) {};
}

// arg1, arg2, fun1 是内部的, 对对象来说是私有的. 他们只对它的内部可见.
// method1, method2 是外部的, 公有的方法.
```
它把属性和方法写在一起, 当我们用 `new` 构造出实例的时候, 每个实例对每个属性和方法都各自保存着一份副本, 这造成了冗余.

推荐采用如下的基于函数原型的写法
```javascript
function User(name) {
  this.name = name;
}

User.prototype.method1 = function(...) {}
```
在原型模式中, 构造器 `User` 只是初始化当前对象的状态, 方法被添加到 `User.prototype` 中. 所有的方法都在 `User.prototype` 上, 并对所有的实例共享, 对象本身只存储数据, 所以原型模式存储效率更高.

类基于原型的继承: 假设我们有两个基于原型的类 `Rabbit`,
```javaScript
function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype.jump = function() {
  console.log(`${this.name} jumps!`);
};

let rabbit = new Rabbit("My rabbit");
```
<img src='http://zh.javascript.info/article/class-patterns/rabbit-animal-independent-1@2x.png' width='50%'/>

`Animal`:
```javaScript
function Animal(name) {
  this.name = name;
}

Animal.prototype.eat = function() {
  console.log(`${this.name} eats.`);
};

let animal = new Animal("My animal");
```

<img src='http://zh.javascript.info/article/class-patterns/rabbit-animal-independent-2@2x.png' width='50%'/>

我们希望 `Rabbit` 来继承 `Animal`. 换句话说, Rabbit 应该是基于 Animal 的, 可以访问 Animal 的所有方法并且用它自己的方法扩展它们. 我们让 `Rabbit.prototype.__proto__ = Animal.prototype;`, 关系如下:

<img src='http://zh.javascript.info/article/class-patterns/class-inheritance-rabbit-animal@2x.png' height='350px'/>

这样就实现了继承.
