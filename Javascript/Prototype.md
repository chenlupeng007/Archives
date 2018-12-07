### descriptor

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

注意:
- `Object.keys(obj)`: 返回一个数组, 包括对象自身的(不含继承的)所有可枚举属性(不含 Symbol 属性).
- `for...in`: 循环遍历对象自身的和继承的可枚举属性(不含 Symbol 属性).
- `Object.getOwnPropertyDescriptors` 返回的属性包含 Symbol 属性和不可枚举属性(不含继承的).

### getter & setter
