# Symbol

* 所有symbol不相等
* 带一个描述参数，可用于调试
* 通过`Symbol.for`和`Symbol.keyFor`全局获取Symbol表（`Symbol('foo')`不会在全局symbol表中注册）
* `getOwnPropertySymbols`返回目标对象中所有Symbol属性的数组
* `Symbol.iterator` 创建自定义迭代器

## Symbol不相等
```js
const sb1 = Symbol();
const sb2 = Symbol();
sb1 === sb2 // false
```

## Symbol全局表
```js
Symbol.for("foo"); // 创建一个 symbol 并放入 symbol 注册表中，键为 "foo"
Symbol.for("foo"); // 从 symbol 注册表中读取键为"foo"的 symbol


Symbol.for("bar") === Symbol.for("bar"); // true，证明了上面说的
Symbol("bar") === Symbol("bar"); // false，Symbol() 函数每次都会返回新的一个 symbol


var sym = Symbol.for("mario");
sym.toString();
// "Symbol(mario)"，mario 既是该 symbol 在 symbol 注册表中的键名，又是该 symbol 自身的描述字符串
```

## 对象Symbol属性数组
```js
var obj = {};
var a = Symbol("a");
var b = Symbol.for("b");

obj[a] = "localSymbol";
obj[b] = "globalSymbol";

var objectSymbols = Object.getOwnPropertySymbols(obj);

console.log(objectSymbols.length); // 2
console.log(objectSymbols)         // [Symbol(a), Symbol(b)]
console.log(objectSymbols[0])      // Symbol(a)
```

## 迭代器属性
```js
var myIterable = {}
myIterable[Symbol.iterator] = function* () {
  console.log(1)
  yield 1;
  console.log(2)
  yield 2;
  console.log(3)
  yield 3;
};
[...myIterable] // [1, 2, 3]
```