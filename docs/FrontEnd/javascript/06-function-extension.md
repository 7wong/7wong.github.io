---
title: 函数的扩展
---

## 1. 函数参数的默认值

### 基本用法

ES6 之前, 不能为函数的参数指定默认值, 只能采用变通的方法

```javascript
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World

// 这里存在一个问题, 第二个参数被赋予空字符, 结果被改为默认值, 需要添加判断来解决问题
if (typeof y === 'undefined') {
  y = 'World';
}
```

ES6 可以对参数赋予默认值

```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```

好处:

1. 阅读代码时, 可以立刻意识到哪些参数是可以省略的, 不用查看函数体或文档
2. 有利于将来的代码优化, 即使未来的版本在对外接口中, 彻底拿掉这个参数也不会导致不能运行

参数变量是默认声明的, 所以不能用 `let` 或 `const` 再次声明

### 与解构赋值默认值结合使用

参数默认值可以与解构赋值的默认值, 结合起来使用

```javascript
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // TypeError: Cannot read property 'x' of undefined
```

上面代码只使用了对象的解构赋值的默认值, 没有使用函数参数的默认值

只有当函数 `foo` 的参数是一个对象时, 变量 `x` 和 `y` 才会通过解构赋值生成. 如果函数 `foo` 调用时没有提供参数, 变量 `x` 和 `y` 就不会生成, 从而报错

通过提供函数参数的默认值, 就可以避免这种情况

```javascript
function foo({x, y = 5} = {}) {
  console.log(x, y);
}

foo() // undefined 5
```

再来一个符合现在代码的示例, 看到这种写法后瞬间懂了之前的一些代码是怎么回事

```javascript
function fetch(url, { body = '', method = 'GET', headers = {} } = {}) {
  // 函数的参数是空对象, 但是设置了对象解构赋值的默认值
  console.log(method);
}

fetch('http://example.com')
// "GET"
```

### 参数默认值的位置

通常情况下, 定义了默认值的参数, 应该是函数的尾参数

如果是非尾部的参数设置默认值, 实际上这个参数是没法省略的

```javascript
// 例一
function f(x = 1, y) {
  return [x, y];
}

f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 报错
f(undefined, 1) // [1, 1]

// 例二
function f(x, y = 5, z) {
  return [x, y, z];
}

f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 报错
f(1, undefined, 2) // [1, 5, 2]
```

### 函数的 length 属性

制定了默认值以后, 函数的 length 属性, 将返回没有指定默认值的参数个数

也就是说, 指定了默认值以后, length 属性将失真

```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```

这是因为 length 属性的含义是, 该函数预期传入的参数个数

某个参数指定默认值以后, 预期传入的参数个数就不包括这个参数了, 同理, 后文的 rest 参数也不会计入 length 属性

```javascript
(function(...args) {}).length // 0
```

如果设置了默认值的参数不是尾参数, 那么 length 属性也不再计入后面的参数了

```javascript
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

### 作用域

一旦设置了参数的默认值, 函数进行声明初始化时, 参数会形成一个单独的作用域 (context)

等到初始化结束, 这个作用域就会消失, 这种语法行为, 在不设置参数默认值时, 是不会出现的

```javascript
var x = 1;

function foo(x = x) {
  // ...
}

foo() // ReferenceError: x is not defined
```

上面代码中, 参数 `x = x` 形成一个作用域, 由于暂时性锁区导致顶部 `x` 无法引用外部的 `x` , 导致报错

**如果参数的默认值是一个函数, 也同理**

### 应用

可以利用参数默认值, 指定某一个参数不得省略, 如果省略就抛出一个错误

```javascript
function throwIfMissing() {
  throw new Error('Missing parameter');
}

function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}

foo()
// Error: Missing parameter
```

也可以将参数设置为 `undefined` , 表明这个参数是可以省略的

```javascript
function foo(optional = undefined) { ··· }
```

## 2. rest 参数

ES6 引入 rest 参数(形式为 `...变量名`) , 用于获取函数的多余参数, 这样就不需要使用 `arguments` 对象了

rest 参数搭配的变量是一个数组, 该变量将多余的参数放入数组中

```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
```

`arguments` 与 `rest` 进行对比

```javascript
// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();
```

> rest 参数之后不能再有其他参数

## 3. 严格模式

从 ES5 开始, 函数内部可以设定为严格模式

而 `ES2016` 做了一点修改, 规定只要函数参数使用了默认值, 解构赋值, 或者扩展运算符, 那么函数内部就不能显式设定为严格模式, 否则会报错

```javascript
// 报错
function doSomething(a, b = a) {
  'use strict';
  // code
}

// 报错
const doSomething = function ({a, b}) {
  'use strict';
  // code
};

// 报错
const doSomething = (...a) => {
  'use strict';
  // code
};

const obj = {
  // 报错
  doSomething({a, b}) {
    'use strict';
    // code
  }
};
```

这样规定的原因是, 函数内部的严格模式, 同时适用于函数体和函数参数

但是, 函数执行时, 先执行函数参数, 再执行函数体, 这样就存在不合理的地方, 只有从函数体之中才能获知参数是否应该以严格模式执行, 但是参数却应该先于函数体执行

举个例子

```javascript
// 报错
function doSomething(value = 070) {
  'use strict';
  return value;
}

// 参数 value 的默认值是八进制数 070, 但是严格模式下不能使用前缀 0 表示八进制, 所以应该报错
```

两种解决方法

+ 设定全局性的严格模式

  ```javascript
  'use strict';
  
  function doSomething(a, b = a) {
    // code
  }
  ```

+ 把函数包裹在一个无参数的立即执行函数中

  ```javascript
  const doSomething = (function () {
    'use strict';
    return function(value = 42) {
      return value;
    };
  }());
  ```

## 4. name 属性

函数的 `name` 属性, 返回该函数的函数名

```javascript
function foo() {}
foo.name // "foo"
```

该属性早被各种浏览器广泛支持, 但是直到 ES6, 才将其写入标准

ES5 与 ES6 环境还是存在区别的

1. 匿名函数

   ```javascript
   var f = function () {};
   
   // ES5
   f.name // ""
   
   // ES6
   f.name // "f"
   ```

2. 具名函数

   ```javascript
   const bar = function baz() {};
   
   // ES5
   bar.name // "baz"
   
   // ES6
   bar.name // "baz"
   ```

`Function` 构造函数返回的函数实例, `name` 属性的值为 `anonymous`

```javascript
(new Function).name // "anonymous"
```

`bind` 返回的函数, `name` 属性值会加上 `bound` 前缀

```javascript
function foo() {};
foo.bind({}).name // "bound foo"

(function(){}).bind({}).name // "bound "
```

## 5. 箭头函数

### 基本用法

ES6 允许使用箭头 (`=>`) 定义函数

```javascript
var f = v => v;

// 等同于
var f = function (v) {
  return v;
};
```

如果箭头函数不需要参数或需要多个参数

```javascript
var f = () => 5;
// 等同于
var f = function () { return 5 };

// 单语句可省略部分
var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};
```

由于大括号被解释为代码块, 所以如果在箭头函数直接返回一个对象, 必须在对象外加上括号

```javascript
// 报错
let getTempItem = id => { id: id, name: "Temp" };

// 不报错
let getTempItem = id => ({ id: id, name: "Temp" });
getTempItem(); //{id: undefined, name: "Temp"}
```

箭头函数可以有效的简化回调函数

```javascript
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);
```

箭头函数和 rest 参数结合

```javascript
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
```

### 使用注意事项

1. 函数体内的 `this` 对象, 就是定义时所在的对象, 而不是使用时所在的对象
2. 不可以当做构造函数, 也就是说, 不可以使用 `new` 命令, 否则会抛出一个错误
3. 不可以使用 `arguments` 对象, 该对象在函数体内不存在, 如果要使用, 可以使用 rest 参数代替
4. 不可以使用 `yield` 命令, 因此箭头函数不能用作 Generator 函数

`this` 指向的固定化, 并不是因为箭头函数内部有绑定 `this` 的机制, 实际原因是箭头函数根本没有自己的 `this`, 导致内部的 `this` 就是外层代码块的 `this`

正是因为它没有 `this` , 所以也就不能用作构造函数

```javascript
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this;

  setTimeout(function () {
    // 可以看到使用的是外层的 this
    console.log('id:', _this.id);
  }, 100);
}
```

### 不适用场合

由于箭头函数使得 `this` 从 "动态" 变成 "静态", 下面两个场合不应该使用箭头函数

1. 定义函数的方法, 且该方法内部包括 `this`

   ```javascript
   const cat = {
     lives: 9,
     jumps: () => {
       this.lives--;
     }
   }
   ```

   此时 `this` 会指向全局对象

2. 需要动态 `this` 的时候

   ```javascript
   var button = document.getElementById('press');
   button.addEventListener('click', () => {
     this.classList.toggle('on');
   });
   ```

   此时 `this` 会指向全局对象

3. 函数体复杂时

   当函数体很复杂, 有很多行, 或者函数内部有大量读写操作, 不单纯是为了计算值

   此时也不应当使用箭头函数

### 嵌套的箭头函数

箭头函数内部还可以再使用箭头函数

```javascript
function insert(value) {
  return {into: function (array) {
    return {after: function (afterValue) {
      array.splice(array.indexOf(afterValue) + 1, 0, value);
      return array;
    }};
  }};
}

insert(2).into([1, 3]).after(1); //[1, 2, 3]

// 修改为箭头函数
let insert = value => ({
  into: array => ({
    after: afterValue => {
      array.splice(array.indexOf(afterValue) + 1, 0, value);
      return array;
    }
  })
});

insert(2).into([1, 3]).after(1); //[1, 2, 3]
```

## 6. 双冒号运算符

函数箭头可以绑定 `this` 对象, 大大减少了显示绑定 `this` 对象的写法(`call`, `apply`, `bind`)

但是箭头函数并不适用于所有场合, 所以现在有一个提案, 提出了 "函数绑定" 运算符, 用来取代 `call`, `apply`, `bind` 调用

函数绑定运算符是并排的两个冒号(`::`), 双冒号左边是一个对象, 右边是一个函数

该运算符自动将左边的对象, 作为上下文环境(即 `this` 对象), 绑定到右边的函数上面

```javascript
foo::bar;
// 等同于
bar.bind(foo);

foo::bar(...arguments);
// 等同于
bar.apply(foo, arguments);

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return obj::hasOwnProperty(key);
}
```

如果双冒号左边为空, 右边是一个对象的方法, 则等于将该方法绑定在该对象上面

```javascript
var method = obj::obj.foo;
// 等同于
var method = ::obj.foo;

let log = ::console.log;
// 等同于
var log = console.log.bind(console);
```

如果双冒号运算符的运算结果还是一个对象, 就可以采用链式写法

```javascript
import { map, takeWhile, forEach } from "iterlib";

getPlayers()
::map(x => x.character())
::takeWhile(x => x.strength > 100)
::forEach(x => console.log(x));
```

## 7. 尾调用优化

尾调用 (Tail Call) 是函数式编程的一个重要概念, 本身非常简单, 就是指某个函数的最后一步是调用另一个函数(调用后就没有别的操作了才行)

```javascript
function f(x){
  // code...
  return g(x);
}
```

以下三种情况都不是尾调用

```javascript
// 情况一
function f(x){
  let y = g(x);
  return y;
}

// 情况二
function f(x){
  return g(x) + 1;
}

// 情况三
function f(x){
  g(x);
}
```

尾调用不一定出现在函数尾部, 只要是最后一步操作即可

```javascript
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
```

### 尾调用优化

调用函数会在内存形成一个 "调用记录", 又称 "调用帧", 保存调用位置和内部变量等信息

所有的调用帧, 就会形成一个 "调用栈"

尾调用由于是函数的最后一步操作, 所以不需要保留外层函数的调用帧, 因为调用位置, 内部变量等信息都不会再用到了, 只要直接用内层函数的调用帧即可

```javascript
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同于
function f() {
  return g(3);
}
f();

// 等同于
g(3); // 优化结果
```

这就叫做 "尾调用优化", 即只保留内层函数的调用帧

如果所有函数都是尾调用, 那么完全可以做到每次执行时, 调用帧只有一项, 这将大大节省内存

> 只有不再用到外层函数的内部变量, 内层函数的调用帧才会取代外层函数的调用帧
>
> 下面是个错误示例
>
> ```javascript
> function addOne(a){
>   var one = 1;
>   function inner(b){
>     return b + one;
>   }
>   return inner(a);
> }
> ```

### 尾递归

函数调用自身, 称为递归, 如果尾调用自身, 就称为尾递归

递归非常消耗内存, 因为需要同时保存成百上千个调用帧, 很容易发生 "栈溢出" 错误 (stack overflow)

但对于尾递归来说, 永远不会发生 "栈溢出" 错误, 因为只存在一个调用帧

```javascript
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5) // 120
```

上面代码是一个阶乘函数, 计算 `n` 的阶乘, 最多需要保存 `n` 个调用记录, 复杂度 `O(n)`, 修改成尾递归的形式, 只保留一个调用记录, 复杂度 `O(1)`

```javascript
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120
```

还有一个著名的例子, 计算 Fibonacci 数列

+ 非尾递归实现

  ```javascript
  function Fibonacci (n) {
    if ( n <= 1 ) {return 1};
  
    return Fibonacci(n - 1) + Fibonacci(n - 2);
  }
  
  Fibonacci(10) // 89
  Fibonacci(100) // 堆栈溢出 // 浏览器都直接无响应了
  Fibonacci(500) // 堆栈溢出
  ```

+ 尾递归实现

  ```javascript
  function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
    if( n <= 1 ) {return ac2};
  
    return Fibonacci2 (n - 1, ac2, ac1 + ac2);
  }
  
  Fibonacci2(100) // 573147844013817200000
  Fibonacci2(1000) // 7.0330367711422765e+208
  Fibonacci2(10000) // Infinity
  ```

### 递归函数的改写

尾递归的实现, 往往需要改写递归函数, 确保最后一步只调用自身, 做到这一点的方法就是把所有用到的内部变量改写成函数的参数

实现这个问题有两种解决方法

1. 尾递归函数之外, 再提供一个正常形式的函数

   ```javascript
   function tailFactorial(n, total) {
     if (n === 1) return total;
     return tailFactorial(n - 1, n * total);
   }
   
   function factorial(n) {
     return tailFactorial(n, 1);
   }
   
   factorial(5) // 120
   
   // 函数式编程有一个概念叫做 "柯里化", 意思是将多参数的函数转为单参数的形式
   function currying(fn, n) {
     return function(m) {
       return fn.call(this, m, n);
     };
   }
   
   function tailFactorial(n, total) {
     if (n === 1) return total;
     return tailFactorial(n - 1, n * total);
   }
   
   const factorial = currying(tailFactorial, 1);
   factorial(5) // 120
   
   ```

2. 采用 ES6 的函数默认值

   ```javascript
   function factorial(n, total = 1) {
     if (n === 1) return total;
     return factorial(n - 1, n * total);
   }
   
   factorial(5) // 120
   ```

> 递归本质是一种循环操作
>
> 纯粹的函数式编程没有循环操作命令, 所有的循环都用递归实现, 这就是为什么尾递归对这些语言极其重要

### 严格模式

ES6 的尾调用优化只在严格模式下开启, 正常模式是无效的

> 看了这么多突然感觉白了...

因为在正常模式下, 函数内部有两个变量, 可以跟踪函数的调用栈

+ `func.arguments` : 返回调用时函数的参数
+ `func.caller` : 返回调用当前函数的那个参数

尾调用优化时, 函数的调用栈会改写, 因此上面两个变量就会失真, 严格模式禁用这两个变量, 所以, 尾调用模式仅在严格模式下生效

```javascript
function restricted() {
  'use strict';
  restricted.caller;    // 报错
  restricted.arguments; // 报错
}
restricted();
```

### 尾递归优化的实现

在正常模式下, 及不支持该功能的环境中, 实现尾递归优化

原理很简单, 尾递归之所以需要优化, 原因是调用栈太多, 造成溢出, 只要减少调用栈, 就不会溢出

**采用 "循环" 替代 "递归"**

```javascript
function sum(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1);
  } else {
    return x;
  }
}

sum(1, 100000)
// Uncaught RangeError: Maximum call stack size exceeded(…)
```

蹦床函数(trampoline) 可以将递归执行转为循环执行

```javascript
function trampoline(f) {
  while (f && f instanceof Function) {
    f = f();
  }
  return f;
}

trampoline(sum(1, 100000))
// 100001
```

然而蹦床函数并不是真正的实现尾递归优化

```javascript
function tco(f) {
  var value;
  var active = false;
  var accumulated = [];

  return function accumulator() {
    accumulated.push(arguments);
    if (!active) {
      active = true;
      while (accumulated.length) {
        value = f.apply(this, accumulated.shift());
      }
      active = false;
      return value;
    }
  };
}

var sum = tco(function(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1)
  }
  else {
    return x
  }
});

sum(1, 100000)
// 100001
```

## 8. 函数参数的尾逗号

`ES2017` 允许函数的最后一个参数有尾逗号

```javascript
function clownsEverywhere(
  param1,
  param2,
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar',
);
```