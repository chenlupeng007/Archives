## new 的模拟实现

```javascript
function New() {
  // 1. 创建一个空对象 obj
  let obj = {};

  // 2. 从 arguments 里取出作为第一个参数的构造函数 func
  let arg = Array.from(arguments);
  let func = arg.shift();

  // 3. 将 obj 的 [[Prototype]] 赋值成构造函数 func 的 prototype
  obj.__proto__ = func.prototype;

  // 4. 以 this = obj 调用 func
  let result = func.call(obj, ...arg);

  // 5. 根据构造函数的返回值是否是 object，决定返回 result 还是 obj
  return typeof result == 'object' ? result: obj;
}
```

## 对象属性的获取
- `Object.keys(obj)`: 返回一个数组, 包括对象自身的(不含继承的)所有可枚举属性(不含 Symbol 属性).
- `for...in`: 循环遍历对象自身的和继承的可枚举属性(不含 Symbol 属性).
- `Object.getOwnPropertyDescriptors()` 返回的属性包含 Symbol 属性和不可枚举属性(不含继承的).
