---
title: 变量的解构赋值
---

## 1. 数组的解构赋值

### 基本用法

ES6 允许按照一定模式, 从数组和对象中取值, 对变量进行赋值, 这被称为解构(Destructuring)

```javascript
let [a, b, c] = [1, 2, 3];
let [head, ...tail] = [1, 2, 3, 4];
```

如果解构不成功, 变量的值就等于 `undefined`

事实上, 只要某种数据结构具有 `Iterator` 接口, 都可以采用数组的形式解构赋值

### 默认值

解构允许你指定默认值

ES6 内部使用严格相等运算符( `===` ), 判断一个位置是否有效, 所以, 只有当一个数组成员严格等于 `undefined`, 默认值才会生效

```javascript
let [foo = true] = [];
foo // true

// 默认值生效
let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
// null 不严格等于 undefined 
let [x = 1] = [null];
x // null
```

## 2. 对象的解构赋值

解构不仅可以用于数组, 还可以用于对象, 比如 `import`

对象的解构与数组有一个重要的不同

数组的元素是按次序排列的, 变量的取值由它的位置决定, 而对象的属性没有次序, 变量必须与属性同名, 才可以取到正确的值

与数组一样, 解构也可以用于嵌套的数据解构

```javascript
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
start // Object {line: 1, column: 5}
loc  // Object {start: Object}
// 最后一次对line属性的解构赋值之中，只有line是变量，loc和start都是模式，不是变量
```

**对象的解构也可以指定默认值**

与数组解构一样, 默认值的生效条件是, 对象的属性严格等于 `undefined`

对象解构时要避免已经声明的变量

```javascript
let x;
{x} = {x: 1};
// SyntaxError: syntax error
// JavaScript 引擎会将 {x} 理解为一个代码块从而发生语法错误, 只有不将大括号写在首行, 避免 JavaScript 引擎将其解释为代码块
```

由于数组的本质是对象, 因此可以对数组进行对象属性的解构

```javascript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```

## 3. 字符串的解构赋值

字符串也可以结构赋值

这是因为此时, 字符串被转换为了一个类似数组的对象

```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

类似数组的对象都有一个 `length` 属性

```javascript
let {length : len} = 'hello';
len // 5
```

## 4. 函数和布尔值的解构赋值

解构赋值时, 如果等号右边是数值和布尔值, 则会先转为对象

```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

解构赋值的规则是, 只要右边的值不是对象或数组, 就先将其转为对象

由于 `undefined` 和 `null` 无法转为对象, 所以对它们进行解构赋值都会报错

## 5. 函数参数的解构赋值

函数的参数也可以使用解构赋值

```javascript
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
// 函数 add 的参数表面上是一个数组, 然而在传入参数的那一刻, 数组参数就被解构成为变量 x, y
```

`undefined` 会触发函数参数的默认值

```javascript
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```

## 6. 圆括号问题

解构赋值虽然很方便, 但是解析起来并不容易

对于编译器来说, 一个式子到底是模式, 还是表达式, 没有办法从一开始就知道, 必须解析到(或解析不到) 等号才知道

ES6 的负责是, 只要有可能导致解构的歧义, 就不得不使用圆括号

但是这条规则不是那么容易辨别, 处理起来相当麻烦, 因此建议只要有可能, 就不要在模式中放置圆括号

### 不能使用圆括号的情况

1. 变量声明语句

   ```javascript
   // 全部报错
   let [(a)] = [1];
   
   let {x: (c)} = {};
   let ({x: c}) = {};
   let {(x: c)} = {};
   let {(x): c} = {};
   
   let { o: ({ p: p }) } = { o: { p: 2 } };
   ```

2. 函数参数

   ```javascript
   // 报错
   function f([(z)]) { return z; }
   // 报错
   function f([z,(x)]) { return x; }
   ```

3. 赋值语句的模式

   ```javascript
   // 全部报错
   ({ p: a }) = { p: 42 };
   ([a]) = [5];
   ```

### 可以使用圆括号的情况

**情况只有一种, 赋值语句的非模式部分**

```javascript
[(b)] = [3]; // 正确
({ p: (d) } = {}); // 正确
[(parseInt.prop)] = [3]; // 正确
```

## 7. 用途

解构赋值有很多用途

### 交换变量的值

```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```

### 从函数返回多个值

函数只能返回一个值, 如果要返回多个值, 只能将他们放在数组或对象里返回

突然感觉以前的很多代码有了优化的空间

```javascript
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

### 函数参数的定义

```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

### 提取 JSON 部分

解构赋值对于提取 JSON 对象中的数据, 尤其有用

```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

### 函数参数的默认值

设置默认值后避免了在内部中书写 `var foo = config.foo || 'default foo';` 的语句

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

### 遍历 Map 结构

任何部署了 `Iterator` 接口的对象, 都可以使用 `for...of` 循环遍历

配合变量的解构赋值, 获取键名和键值就非常方便

```javascript
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world


// 获取键名
for (let [key] of map) {
  // ...
}
// 获取键值
for (let [,value] of map) {
  // ...
}
```

### 输入模块的指定方法

加载模块时, 往往需要指定哪些方法, 解构赋值使得语句非常清晰

```javascript
// 该使用方式与从函数返回多个值一致, 不过这个仅支持对象(毕竟是方法)
const { SourceMapConsumer, SourceNode } = require("source-map");
```

