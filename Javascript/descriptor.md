## descriptor

对象的属性除了属性值以外还有三个特殊属性(所谓的“标志” descriptor):
- `writable`
- `enumerable`
- `configurable`

它们默认都设置成 `true`. 如何获取这些标志?

```javascript
let user = {
  name: 'John',
};

let descriptor = Object.getOwnPropertyDescriptor(user, 'name'); // propertyName 不包括原型链继承下来的属性

console.log(JSON.stringify(descriptor, null, 2)); // 2 表示缩进两个空格

/* property descriptor:
{
"value": "John",
"writable": true,
"enumerable": true,
"configurable": true
}
*/
```

`Object.defineProperty(obj, propertyName, descriptor)` 可以用来修改标志和定义属性.

```javascript
let id = Symbol('id');

let user = {
  name: 'John',
  [id]: '492472'
};

Object.defineProperty(user, 'name', {
  writable: false
});

Object.defineProperty(user, 'age', {
  value: 28,
  writable: true,
  enumerable: false,
  configurable: true
});

Object.defineProperty(user, 'gender', {
  value: 'woman',
  writable: true,
  enumerable: true,
  configurable: false
});

user.name = 'Tom'
console.log(user.name); // John

for(let key in user) console.log(key);
/*
name
gender
*/

delete user.gender
for(let key in user) console.log(key);
/*
name
gender
*/
```

- `writable` 设置为 `false` 表示只读, 不能修改.
- `enumerable` 设置为 `false` 时, 在 `for...in` 和 `Object.keys(obj)` 时该属性不会被列出.
- `configurable` 设置为 `false` 时, 无法改变属性值和删除该属性.

## getter & setter
对象有两种类型的属性: 第一种是数据属性, 之前我们谈论的都是数据属性; 第二种是访问器属性(accessor properties), 它们本质上是获取和设置值的函数, 但是使用起来却像常规的数据属性.

```javascript
let user = {
  firstName: 'John',
  lastName: 'Smith',
  get fullName() {
    return `${this.firstName} ${this.lastName}`
  },
  set fullName(name) {
    [this.name, this.lastName] = name.split('');
  }
}
```

你可以直接访问它的值 `user.fullName`, 也可以使用 `user.fullName = ...`, 就好像真的有一个数据属性 `fullName` 一样, 而本质上它是调用了获取和设置值的两个函数. 访问器属性的描述符与数据属性相比是不同的. 对于访问器属性, 没有 `value` 和 `writable`, 但是有 `get` 和 `set` 函数. 所以访问器描述符包括:
- `get` --- 一个没有参数的函数, 在读取属性时工作,
- `set` --- 带有一个参数的函数, 当属性被设置时调用,
- `enumerable` --- 与数据属性相同, 例如 `for...in` 遍历时会包含 `fullName`,
- `configurable` --- 与数据属性相同.

总之, 表面上这种属性与数据属性没什么区别, 不过我们却可以在 getter/setter 函数里对数据进行进一步的加工处理.
