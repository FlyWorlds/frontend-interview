# JavaScript 核心知识

## 概述

JavaScript 是前端开发的核心语言，不仅要会用，更要深入理解其运行机制和底层原理。本章节涵盖所有**高频面试八股文**。

---

---

## 进阶专题 (New)

- [**V8 引擎与内存管理**](./v8-engine.md) (垃圾回收, 执行管线, 内存泄漏)
- [**事件循环 (Event Loop)**](./event-loop.md) (宏任务/微任务, Node.js 区别)
- [**前端安全**](./security.md) (XSS, CSRF, CSP, 原型污染)
- [**设计模式与函数式编程**](./patterns.md) (单例/观察者/发布订阅, 纯函数/柯里化)
- [**性能优化专题**](../performance/runtime.md) (代码优化, 内存优化, 渲染优化, 2025最佳实践) ⭐ 2025新增
- [**ES2024/ES2025 新特性**](./es2024-2025.md) (最新特性详解, 应用场景, 面试重点) ⭐ 2025新增

---

## 一、数据类型

### 1. 基本类型与引用类型

```javascript
/**
 * 基本类型 (7种):
 * - number
 * - string
 * - boolean
 * - null
 * - undefined
 * - symbol (ES6)
 * - bigint (ES2020)
 *
 * 引用类型:
 * - Object (包括 Array, Function, Date, RegExp, Error 等)
 */

// 存储区别
// 基本类型: 存储在栈内存,按值访问
// 引用类型: 存储在堆内存,按引用访问

let a = 1;
let b = a; // 复制值
b = 2;
console.log(a); // 1 (不受影响)

let obj1 = { name: "Alice" };
let obj2 = obj1; // 复制引用
obj2.name = "Bob";
console.log(obj1.name); // 'Bob' (被修改)
```

### 2. 类型判断

```javascript
// 1. typeof
typeof 123        // 'number'
typeof 'abc'      // 'string'
typeof true       // 'boolean'
typeof undefined  // 'undefined'
typeof Symbol()   // 'symbol'
typeof 123n       // 'bigint'

// 特殊情况
typeof null       // 'object' (历史遗留 bug)
typeof []         // 'object'
typeof {}         // 'object'
typeof function() {} // 'function'

// 2. instanceof (检查原型链)
[] instanceof Array   // true
[] instanceof Object  // true
{} instanceof Object  // true

// 手写 instanceof
function myInstanceof(obj, constructor) {
  if (typeof obj !== 'object' || obj === null) return false

  let proto = Object.getPrototypeOf(obj)
  while (proto !== null) {
    if (proto === constructor.prototype) return true
    proto = Object.getPrototypeOf(proto)
  }
  return false
}

// 3. Object.prototype.toString.call() (最准确)
Object.prototype.toString.call(123)        // '[object Number]'
Object.prototype.toString.call('abc')      // '[object String]'
Object.prototype.toString.call(true)       // '[object Boolean]'
Object.prototype.toString.call(null)       // '[object Null]'
Object.prototype.toString.call(undefined)  // '[object Undefined]'
Object.prototype.toString.call([])         // '[object Array]'
Object.prototype.toString.call({})         // '[object Object]'
Object.prototype.toString.call(function(){}) // '[object Function]'
Object.prototype.toString.call(new Date()) // '[object Date]'
Object.prototype.toString.call(/abc/)      // '[object RegExp]'

// 封装通用类型判断函数
function getType(value) {
  const type = Object.prototype.toString.call(value)
  return type.slice(8, -1).toLowerCase()
}

getType(123)    // 'number'
getType([])     // 'array'
getType(null)   // 'null'

// 4. Array.isArray()
Array.isArray([])   // true
Array.isArray({})   // false

// 5. 判断 NaN
Number.isNaN(NaN)   // true (推荐)
isNaN('abc')        // true (不推荐,会先转换类型)
NaN === NaN         // false (唯一一个不等于自身的值)
Object.is(NaN, NaN) // true
```

### 3. 类型转换

```javascript
/**
 * 隐式转换规则:
 *
 * 1. 转 Boolean: 使用 Boolean() 或 !!
 *    假值: false, 0, -0, '', null, undefined, NaN
 *    其他都是真值 (包括 [], {}, '0')
 *
 * 2. 转 Number: 使用 Number() 或 +
 *    true → 1
 *    false → 0
 *    null → 0
 *    undefined → NaN
 *    '' → 0
 *    '123' → 123
 *    '12a' → NaN
 *    [] → 0
 *    [1] → 1
 *    [1,2] → NaN
 *
 * 3. 转 String: 使用 String() 或 + ''
 *    null → 'null'
 *    undefined → 'undefined'
 *    [1,2,3] → '1,2,3'
 */

// == 比较规则 (优先转 Number)
1 == '1'        // true ('1' → 1)
1 == true       // true (true → 1)
'1' == true     // true ('1' → 1, true → 1)
null == undefined // true (特殊规则)
null == 0       // false (null 只等于 undefined)
NaN == NaN      // false

// [] 和 {} 的转换
[] == ![]       // true
// ![] → false
// [] → '' → 0
// false → 0
// 0 == 0 → true

[] + []         // '' (两个空数组拼接)
[] + {}         // '[object Object]'
{} + []         // 0 ({}被解析为代码块)
({}) + []       // '[object Object]'

// 对象转原始值 (ToPrimitive)
// 优先调用 [Symbol.toPrimitive]
// 否则: 转 Number 先 valueOf 再 toString
//       转 String 先 toString 再 valueOf

const obj = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return 42
    if (hint === 'string') return 'hello'
    return true  // default
  }
}

+obj        // 42 (hint: 'number')
`${obj}`    // 'hello' (hint: 'string')
obj + ''    // 'true' (hint: 'default')

// 经典面试题: 让 a == 1 && a == 2 && a == 3
const a = {
  value: 1,
  valueOf() {
    return this.value++
  }
}

console.log(a == 1 && a == 2 && a == 3)  // true
```

---

## 二、执行上下文与作用域

### 1. 执行上下文

```javascript
/**
 * 执行上下文类型:
 * 1. 全局执行上下文
 * 2. 函数执行上下文
 * 3. eval 执行上下文
 *
 * 执行上下文包含:
 * - 变量环境 (var 声明)
 * - 词法环境 (let/const 声明)
 * - this 绑定
 * - 外部环境引用 (作用域链)
 */

// 执行上下文栈 (调用栈)
function foo() {
  console.log("foo");
  bar();
  console.log("foo end");
}

function bar() {
  console.log("bar");
}

foo();
// 调用栈变化:
// 1. [全局]
// 2. [全局, foo]
// 3. [全局, foo, bar]
// 4. [全局, foo] (bar 出栈)
// 5. [全局] (foo 出栈)
```

### 2. 变量提升

```javascript
/**
 * var: 存在变量提升,初始化为 undefined
 * let/const: 存在"提升"但有 TDZ (暂时性死区)
 * function: 整体提升,包括函数体
 */

console.log(a); // undefined
var a = 1;

console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 2;

// 函数提升优先于变量提升
console.log(foo); // [Function: foo]
var foo = 1;
function foo() {}

// 等价于:
function foo() {}
var foo;
console.log(foo); // [Function: foo]
foo = 1;

// TDZ (暂时性死区)
var x = 1;
{
  console.log(x); // ReferenceError
  let x = 2;
}
// let x 的声明会被提升到块作用域顶部
// 但在声明前的区域就是 TDZ,访问会报错
```

### 3. 作用域与作用域链

```javascript
/**
 * 作用域类型:
 * 1. 全局作用域
 * 2. 函数作用域
 * 3. 块级作用域 (let/const)
 *
 * 作用域链: 由当前作用域和所有父级作用域组成
 * 查找变量时沿作用域链向上查找
 */

var globalVar = "global";

function outer() {
  var outerVar = "outer";

  function inner() {
    var innerVar = "inner";

    console.log(innerVar); // 'inner' (当前作用域)
    console.log(outerVar); // 'outer' (父作用域)
    console.log(globalVar); // 'global' (全局作用域)
  }

  inner();
}

outer();

// 作用域链在函数定义时确定,不是调用时
var x = 10;

function foo() {
  console.log(x);
}

function bar() {
  var x = 20;
  foo();
}

bar(); // 10 (foo 的作用域链在定义时确定)

// 块级作用域
{
  let a = 1;
  const b = 2;
  var c = 3;
}
// console.log(a)  // ReferenceError
// console.log(b)  // ReferenceError
console.log(c); // 3 (var 不受块级作用域限制)
```

---

## 三、闭包

### 1. 闭包定义

```javascript
/**
 * 闭包 = 函数 + 函数能够访问的自由变量
 *
 * 特点:
 * 1. 函数嵌套函数
 * 2. 内部函数引用外部函数的变量
 * 3. 外部函数执行完后,变量不会被回收
 */

function outer() {
  let count = 0;

  return function inner() {
    count++;
    console.log(count);
  };
}

const counter = outer();
counter(); // 1
counter(); // 2
counter(); // 3
// count 变量被闭包保持,不会被回收
```

### 2. 闭包应用场景

```javascript
// 1. 模块化 (IIFE)
const module = (function () {
  let privateVar = 0;

  function privateMethod() {
    return privateVar++;
  }

  return {
    publicMethod: function () {
      return privateMethod();
    },
    getPrivateVar: function () {
      return privateVar;
    },
  };
})();

module.publicMethod(); // 0
module.publicMethod(); // 1
// privateVar 无法直接访问

// 2. 柯里化
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function (...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

curriedAdd(1)(2)(3); // 6
curriedAdd(1, 2)(3); // 6
curriedAdd(1)(2, 3); // 6

// 3. 防抖节流
function debounce(fn, delay) {
  let timer = null; // 闭包保存 timer

  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// 4. 缓存 (Memoization)
function memoize(fn) {
  const cache = {}; // 闭包保存缓存

  return function (...args) {
    const key = JSON.stringify(args);

    if (key in cache) {
      return cache[key];
    }

    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

const expensiveFn = memoize((n) => {
  console.log("Computing...");
  return n * 2;
});

expensiveFn(5); // Computing... 10
expensiveFn(5); // 10 (from cache)

// 5. 私有变量
function Person(name) {
  let _name = name; // 私有变量

  this.getName = function () {
    return _name;
  };

  this.setName = function (newName) {
    _name = newName;
  };
}

const person = new Person("Alice");
console.log(person._name); // undefined
console.log(person.getName()); // 'Alice'
```

### 3. 闭包经典问题

```javascript
// 问题: 输出什么?
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, 1000);
}
// 输出: 5 5 5 5 5
// 原因: var 是函数作用域,循环结束后 i = 5

// 解决方案 1: 使用 let
for (let i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, 1000);
}
// 输出: 0 1 2 3 4

// 解决方案 2: 闭包
for (var i = 0; i < 5; i++) {
  (function (j) {
    setTimeout(function () {
      console.log(j);
    }, 1000);
  })(i);
}
// 输出: 0 1 2 3 4

// 解决方案 3: setTimeout 第三个参数
for (var i = 0; i < 5; i++) {
  setTimeout(
    function (j) {
      console.log(j);
    },
    1000,
    i
  );
}
// 输出: 0 1 2 3 4
```

---

## 四、this 指向

### 1. this 绑定规则

```javascript
/**
 * this 绑定优先级 (从高到低):
 * 1. new 绑定
 * 2. 显式绑定 (call/apply/bind)
 * 3. 隐式绑定 (对象方法调用)
 * 4. 默认绑定 (独立函数调用)
 */

// 1. 默认绑定
function foo() {
  console.log(this);
}
foo(); // window (严格模式下是 undefined)

// 2. 隐式绑定
const obj = {
  name: "Alice",
  sayName() {
    console.log(this.name);
  },
};
obj.sayName(); // 'Alice'

// 隐式绑定丢失
const fn = obj.sayName;
fn(); // undefined (默认绑定)

// 3. 显式绑定
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const user = { name: "Bob" };

greet.call(user, "Hello"); // 'Hello, Bob'
greet.apply(user, ["Hi"]); // 'Hi, Bob'

const boundGreet = greet.bind(user);
boundGreet("Hey"); // 'Hey, Bob'

// 4. new 绑定
function Person(name) {
  this.name = name;
}

const person = new Person("Charlie");
console.log(person.name); // 'Charlie'

// 箭头函数没有自己的 this
const arrow = {
  name: "Dave",
  sayName: () => {
    console.log(this.name); // undefined (继承外层 this)
  },
  sayNameRegular() {
    const inner = () => {
      console.log(this.name); // 'Dave' (继承 sayNameRegular 的 this)
    };
    inner();
  },
};

arrow.sayName(); // undefined
arrow.sayNameRegular(); // 'Dave'
```

### 2. 手写 call/apply/bind

```javascript
// 手写 call
Function.prototype.myCall = function (context, ...args) {
  // 处理 null/undefined
  context = context ?? window;

  // 将函数设为对象的方法
  const key = Symbol("fn");
  context[key] = this;

  // 调用方法
  const result = context[key](...args);

  // 删除临时方法
  delete context[key];

  return result;
};

// 手写 apply
Function.prototype.myApply = function (context, args = []) {
  context = context ?? window;

  const key = Symbol("fn");
  context[key] = this;

  const result = context[key](...args);

  delete context[key];

  return result;
};

// 手写 bind
Function.prototype.myBind = function (context, ...args) {
  const fn = this;

  const bound = function (...moreArgs) {
    // 如果是 new 调用,this 指向实例
    return fn.apply(
      this instanceof bound ? this : context,
      args.concat(moreArgs)
    );
  };

  // 继承原型
  if (fn.prototype) {
    bound.prototype = Object.create(fn.prototype);
  }

  return bound;
};

// 测试
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const user = { name: "Alice" };

console.log(greet.myCall(user, "Hello", "!")); // 'Hello, Alice!'
console.log(greet.myApply(user, ["Hi", "?"])); // 'Hi, Alice?'
console.log(greet.myBind(user, "Hey")("...")); // 'Hey, Alice...'
```

---

## 五、原型与继承

### 1. 原型链

```javascript
/**
 * 每个对象都有 __proto__ 属性,指向其构造函数的 prototype
 * 原型链: 对象 → 原型 → 原型的原型 → ... → null
 */

function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log(`Hello, I'm ${this.name}`);
};

const person = new Person("Alice");

// 原型链关系
console.log(person.__proto__ === Person.prototype); // true
console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null); // true

// 属性查找
person.sayHello(); // 'Hello, I'm Alice' (原型上)
console.log(person.toString()); // '[object Object]' (Object.prototype)

// 判断属性来源
console.log(person.hasOwnProperty("name")); // true (自身属性)
console.log(person.hasOwnProperty("sayHello")); // false (原型属性)
console.log("sayHello" in person); // true (包括原型)
```

### 2. 继承方式

```javascript
// 1. 原型链继承
function Parent() {
  this.colors = ["red", "blue"];
}

function Child() {}
Child.prototype = new Parent();

// 问题: 引用类型共享
const c1 = new Child();
const c2 = new Child();
c1.colors.push("green");
console.log(c2.colors); // ['red', 'blue', 'green']

// 2. 构造函数继承
function Parent(name) {
  this.name = name;
  this.colors = ["red", "blue"];
}

function Child(name) {
  Parent.call(this, name);
}

// 问题: 无法继承原型方法

// 3. 组合继承
function Parent(name) {
  this.name = name;
  this.colors = ["red", "blue"];
}

Parent.prototype.sayName = function () {
  console.log(this.name);
};

function Child(name, age) {
  Parent.call(this, name); // 第一次调用 Parent
  this.age = age;
}

Child.prototype = new Parent(); // 第二次调用 Parent
Child.prototype.constructor = Child;

// 问题: Parent 构造函数被调用两次

// 4. 寄生组合继承 (最优)
function Parent(name) {
  this.name = name;
  this.colors = ["red", "blue"];
}

Parent.prototype.sayName = function () {
  console.log(this.name);
};

function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}

// 关键: 使用 Object.create
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

const child = new Child("Alice", 18);
child.sayName(); // 'Alice'
console.log(child.colors); // ['red', 'blue']

// 5. ES6 Class 继承
class Parent {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name); // 必须先调用 super
    this.age = age;
  }
}

const c = new Child("Bob", 20);
c.sayName(); // 'Bob'
```

### 3. 手写 new 和 Object.create

```javascript
// 手写 new
function myNew(constructor, ...args) {
  // 1. 创建新对象,原型指向构造函数的 prototype
  const obj = Object.create(constructor.prototype);

  // 2. 执行构造函数,绑定 this
  const result = constructor.apply(obj, args);

  // 3. 如果构造函数返回对象,则返回该对象
  return result instanceof Object ? result : obj;
}

// 手写 Object.create
function myCreate(proto, propertiesObject) {
  if (typeof proto !== "object" && typeof proto !== "function") {
    throw new TypeError("Object prototype may only be an Object or null");
  }

  function F() {}
  F.prototype = proto;

  const obj = new F();

  if (propertiesObject !== undefined) {
    Object.defineProperties(obj, propertiesObject);
  }

  return obj;
}

// 测试
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log("Hello");
};

const p = myNew(Person, "Alice");
p.sayHello(); // 'Hello'
```

---

## 六、异步编程

### 1. Promise

```javascript
/**
 * Promise 三种状态:
 * - pending: 进行中
 * - fulfilled: 已成功
 * - rejected: 已失败
 *
 * 状态一旦改变就不可逆
 */

// 基本使用
const promise = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    if (Math.random() > 0.5) {
      resolve("success");
    } else {
      reject(new Error("failed"));
    }
  }, 1000);
});

promise
  .then((result) => console.log(result))
  .catch((error) => console.error(error))
  .finally(() => console.log("done"));

// Promise 链式调用
Promise.resolve(1)
  .then((x) => x + 1)
  .then((x) => x * 2)
  .then((x) => console.log(x)); // 4

// Promise.all (全部成功才成功)
Promise.all([Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)]).then(
  (results) => console.log(results)
); // [1, 2, 3]

// Promise.allSettled (全部完成)
Promise.allSettled([
  Promise.resolve(1),
  Promise.reject(new Error("failed")),
  Promise.resolve(3),
]).then((results) => console.log(results));
// [
//   { status: 'fulfilled', value: 1 },
//   { status: 'rejected', reason: Error: failed },
//   { status: 'fulfilled', value: 3 }
// ]

// Promise.race (最快的)
Promise.race([
  new Promise((resolve) => setTimeout(() => resolve(1), 100)),
  new Promise((resolve) => setTimeout(() => resolve(2), 50)),
]).then((result) => console.log(result)); // 2

// Promise.any (第一个成功的)
Promise.any([
  Promise.reject(new Error("1")),
  Promise.resolve(2),
  Promise.resolve(3),
]).then((result) => console.log(result)); // 2
```

### 2. 手写 Promise

```javascript
class MyPromise {
  constructor(executor) {
    this.state = "pending";
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];

    const resolve = (value) => {
      if (this.state === "pending") {
        this.state = "fulfilled";
        this.value = value;
        this.onFulfilledCallbacks.forEach((fn) => fn());
      }
    };

    const reject = (reason) => {
      if (this.state === "pending") {
        this.state = "rejected";
        this.reason = reason;
        this.onRejectedCallbacks.forEach((fn) => fn());
      }
    };

    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (v) => v;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (e) => {
            throw e;
          };

    const promise2 = new MyPromise((resolve, reject) => {
      if (this.state === "fulfilled") {
        queueMicrotask(() => {
          try {
            const x = onFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      }

      if (this.state === "rejected") {
        queueMicrotask(() => {
          try {
            const x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      }

      if (this.state === "pending") {
        this.onFulfilledCallbacks.push(() => {
          queueMicrotask(() => {
            try {
              const x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch (error) {
              reject(error);
            }
          });
        });

        this.onRejectedCallbacks.push(() => {
          queueMicrotask(() => {
            try {
              const x = onRejected(this.reason);
              resolvePromise(promise2, x, resolve, reject);
            } catch (error) {
              reject(error);
            }
          });
        });
      }
    });

    return promise2;
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }

  finally(callback) {
    return this.then(
      (value) => MyPromise.resolve(callback()).then(() => value),
      (reason) =>
        MyPromise.resolve(callback()).then(() => {
          throw reason;
        })
    );
  }

  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise((resolve) => resolve(value));
  }

  static reject(reason) {
    return new MyPromise((_, reject) => reject(reason));
  }

  static all(promises) {
    return new MyPromise((resolve, reject) => {
      const results = [];
      let count = 0;

      promises.forEach((promise, index) => {
        MyPromise.resolve(promise).then((value) => {
          results[index] = value;
          if (++count === promises.length) {
            resolve(results);
          }
        }, reject);
      });
    });
  }

  static race(promises) {
    return new MyPromise((resolve, reject) => {
      promises.forEach((promise) => {
        MyPromise.resolve(promise).then(resolve, reject);
      });
    });
  }
}

function resolvePromise(promise2, x, resolve, reject) {
  if (promise2 === x) {
    return reject(new TypeError("Chaining cycle detected"));
  }

  if (x instanceof MyPromise) {
    x.then(resolve, reject);
  } else if (x !== null && (typeof x === "object" || typeof x === "function")) {
    let called = false;
    try {
      const then = x.then;

      if (typeof then === "function") {
        then.call(
          x,
          (y) => {
            if (called) return;
            called = true;
            resolvePromise(promise2, y, resolve, reject);
          },
          (r) => {
            if (called) return;
            called = true;
            reject(r);
          }
        );
      } else {
        resolve(x);
      }
    } catch (error) {
      if (called) return;
      called = true;
      reject(error);
    }
  } else {
    resolve(x);
  }
}
```

### 3. async/await

```javascript
// async/await 是 Generator + Promise 的语法糖
async function fetchData() {
  try {
    const response = await fetch("/api/data");
    const data = await response.json();
    return data;
  } catch (error) {
    console.error(error);
    throw error;
  }
}

// 并行执行
async function fetchAll() {
  const [user, posts] = await Promise.all([
    fetch("/api/user"),
    fetch("/api/posts"),
  ]);
  return { user, posts };
}

// 串行执行
async function fetchSequentially(urls) {
  const results = [];
  for (const url of urls) {
    const response = await fetch(url);
    results.push(await response.json());
  }
  return results;
}

// 错误处理
async function withErrorHandling() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    // 处理错误
    return null;
  }
}

// async 函数返回 Promise
async function foo() {
  return "hello";
}
foo().then(console.log); // 'hello'

async function bar() {
  throw new Error("failed");
}
bar().catch(console.error); // Error: failed
```

---

## 七、事件循环

### 1. 浏览器事件循环

```javascript
/**
 * 宏任务 (MacroTask):
 * - script (整体代码)
 * - setTimeout/setInterval
 * - setImmediate (Node.js)
 * - I/O
 * - UI rendering
 * - requestAnimationFrame
 *
 * 微任务 (MicroTask):
 * - Promise.then/catch/finally
 * - MutationObserver
 * - queueMicrotask
 * - process.nextTick (Node.js)
 *
 * 执行顺序:
 * 1. 执行一个宏任务
 * 2. 执行所有微任务
 * 3. 渲染 (如果需要)
 * 4. 回到步骤 1
 */

console.log("1");

setTimeout(() => {
  console.log("2");
  Promise.resolve().then(() => {
    console.log("3");
  });
}, 0);

Promise.resolve().then(() => {
  console.log("4");
  setTimeout(() => {
    console.log("5");
  }, 0);
});

console.log("6");

// 输出: 1, 6, 4, 2, 3, 5

// 复杂示例
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async function async2() {
  console.log("async2");
}

console.log("script start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

async1();

new Promise((resolve) => {
  console.log("promise1");
  resolve();
}).then(() => {
  console.log("promise2");
});

console.log("script end");

// 输出:
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```

### 2. Node.js 事件循环

```javascript
/**
 * Node.js 事件循环阶段:
 * 1. timers: setTimeout/setInterval 回调
 * 2. pending callbacks: I/O 回调
 * 3. idle, prepare: 内部使用
 * 4. poll: 检索新的 I/O 事件
 * 5. check: setImmediate 回调
 * 6. close callbacks: close 事件回调
 *
 * process.nextTick 优先级最高,在每个阶段之间执行
 */

console.log("start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

setImmediate(() => {
  console.log("setImmediate");
});

process.nextTick(() => {
  console.log("nextTick");
});

Promise.resolve().then(() => {
  console.log("Promise");
});

console.log("end");

// 输出:
// start
// end
// nextTick
// Promise
// setTimeout (或 setImmediate,顺序不确定)
// setImmediate (或 setTimeout)
```

---

## 八、垃圾回收

### 1. 垃圾回收机制

```javascript
/**
 * 引用计数:
 * - 优点: 实时回收,无停顿
 * - 缺点: 无法处理循环引用
 *
 * 标记清除:
 * - 从根对象开始标记所有可达对象
 * - 清除未标记的对象
 * - 现代浏览器主要使用
 *
 * V8 分代回收:
 * - 新生代: 存活时间短的对象,使用 Scavenge 算法
 * - 老生代: 存活时间长的对象,使用标记清除 + 标记整理
 */

// 循环引用示例
function circularReference() {
  const obj1 = {};
  const obj2 = {};
  obj1.ref = obj2;
  obj2.ref = obj1;
  // 引用计数无法回收
  // 标记清除可以回收 (函数执行完后,从根无法到达)
}

// 手动解除引用
let bigData = new Array(1000000).fill("x");
// 使用完后
bigData = null; // 帮助 GC
```

### 2. 内存泄漏场景

```javascript
// 1. 全局变量
function leak() {
  leakedVar = "global"; // 忘记 var/let/const
}

// 2. 闭包
function createLeak() {
  const hugeData = new Array(1000000);
  return function () {
    // hugeData 被保留
    console.log(hugeData.length);
  };
}

// 3. 定时器
const timer = setInterval(() => {
  const dom = document.getElementById("xxx");
  if (dom) {
    dom.innerHTML = Date.now();
  }
}, 1000);
// 忘记 clearInterval

// 4. DOM 引用
const elements = {
  button: document.getElementById("button"),
};

document.body.removeChild(elements.button);
// elements.button 仍然引用着 DOM

// 5. 事件监听
element.addEventListener("click", handler);
// 忘记 removeEventListener

// 6. Map/Set
const cache = new Map();
cache.set(key, value); // key 一直被引用

// 使用 WeakMap 解决
const weakCache = new WeakMap();
weakCache.set(obj, value); // obj 可被回收
```

---

## 高频面试题汇总

### 1. typeof 和 instanceof 的区别?

<details>
<summary>点击查看答案</summary>

**typeof:**

- 返回字符串,表示类型
- 可以判断基本类型 (除 null)
- 无法区分引用类型 (除 function)

**instanceof:**

- 返回布尔值
- 检查原型链
- 只能判断引用类型

```javascript
typeof null           // 'object' (bug)
typeof []             // 'object'
[] instanceof Array   // true
[] instanceof Object  // true
```

</details>

### 2. == 和 === 的区别? ⭐⭐⭐⭐⭐

<details>
<summary>点击查看答案</summary>

**===** 严格相等:

- 不进行类型转换
- 类型不同直接返回 false

**==** 宽松相等:

- 会进行类型转换
- 转换规则复杂

```javascript
1 === "1"; // false
1 == "1"; // true
null == undefined; // true
null === undefined; // false
NaN == NaN; // false
```

**建议: 始终使用 ===**

**详细内容**: 请参考 [数据类型与类型转换 - 每日一道面试八股文 Day 3](./data-types.md#day-3--和--的区别与隐式类型转换)

</details>

### 3. var/let/const 的区别?

<details>
<summary>点击查看答案</summary>

| 特性     | var        | let        | const      |
| -------- | ---------- | ---------- | ---------- |
| 作用域   | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 是         | TDZ        | TDZ        |
| 重复声明 | 允许       | 不允许     | 不允许     |
| 重新赋值 | 允许       | 允许       | 不允许     |
| 全局属性 | 是         | 否         | 否         |

```javascript
// var 挂载到 window
var a = 1;
window.a; // 1

// let/const 不挂载
let b = 2;
window.b; // undefined
```

</details>

### 4. 箭头函数和普通函数的区别?

<details>
<summary>点击查看答案</summary>

1. **this**: 箭头函数没有自己的 this,继承外层
2. **arguments**: 箭头函数没有 arguments 对象
3. **new**: 箭头函数不能作为构造函数
4. **prototype**: 箭头函数没有 prototype 属性
5. **yield**: 箭头函数不能用作 Generator

```javascript
const obj = {
  name: "Alice",
  regular() {
    console.log(this.name); // 'Alice'
  },
  arrow: () => {
    console.log(this.name); // undefined
  },
};
```

</details>

### 5. 深拷贝和浅拷贝的区别?

<details>
<summary>点击查看答案</summary>

**浅拷贝**: 只复制第一层

```javascript
Object.assign({}, obj)
{ ...obj }
[].concat(arr)
[...arr]
```

**深拷贝**: 递归复制所有层

```javascript
JSON.parse(JSON.stringify(obj)); // 有限制
// 无法处理: undefined, function, Symbol, 循环引用

// 完整深拷贝
function deepClone(obj, map = new WeakMap()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (map.has(obj)) return map.get(obj);

  const clone = Array.isArray(obj) ? [] : {};
  map.set(obj, clone);

  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key], map);
    }
  }

  return clone;
}
```

</details>

### 6. 如何实现一个支持取消的 Promise? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

```javascript
function cancellablePromise(executor) {
  let cancel;
  const promise = new Promise((resolve, reject) => {
    cancel = (reason) => {
      reject(new Error(reason || 'Cancelled'));
    };
    executor(resolve, reject, cancel);
  });
  promise.cancel = cancel;
  return promise;
}

// 使用 ES2024 的 Promise.withResolvers
function cancellablePromise(executor) {
  const { promise, resolve, reject } = Promise.withResolvers();
  const cancel = (reason) => reject(new Error(reason || 'Cancelled'));
  executor(resolve, reject, cancel);
  promise.cancel = cancel;
  return promise;
}

// 使用示例
const p = cancellablePromise((resolve, reject, cancel) => {
  const timer = setTimeout(() => resolve('done'), 5000);
  cancel = () => {
    clearTimeout(timer);
    reject(new Error('Cancelled'));
  };
});

p.then(console.log).catch(console.error);
p.cancel('User cancelled');
```

**应用场景**: 用户取消请求、组件卸载时取消异步操作

</details>

### 7. 如何实现并发控制的异步队列? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

```javascript
async function concurrentLimit(tasks, limit) {
  const results = [];
  const executing = [];
  
  for (const task of tasks) {
    const promise = Promise.resolve(task()).then(result => {
      executing.splice(executing.indexOf(promise), 1);
      return result;
    });
    
    results.push(promise);
    executing.push(promise);
    
    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }
  
  return Promise.all(results);
}

// 使用示例
const urls = Array.from({ length: 100 }, (_, i) => `/api/data/${i}`);
const tasks = urls.map(url => () => fetch(url).then(r => r.json()));
const results = await concurrentLimit(tasks, 5); // 最多5个并发
```

**应用场景**: 批量请求控制、文件上传控制、爬虫并发控制

</details>

### 8. 如何检测和解决内存泄漏? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

**检测方法:**

```javascript
// 1. 使用 Performance API
function monitorMemory() {
  if (performance.memory) {
    const { usedJSHeapSize, totalJSHeapSize, jsHeapSizeLimit } = performance.memory;
    console.log({
      used: `${(usedJSHeapSize / 1024 / 1024).toFixed(2)} MB`,
      total: `${(totalJSHeapSize / 1024 / 1024).toFixed(2)} MB`,
      limit: `${(jsHeapSizeLimit / 1024 / 1024).toFixed(2)} MB`
    });
  }
}

// 2. 使用 Chrome DevTools Memory Profiler
// 3. 使用 heap snapshot 对比

// 常见内存泄漏场景
// 1. 未清理的事件监听器
element.addEventListener('click', handler);
// 忘记: element.removeEventListener('click', handler);

// 2. 未清理的定时器
const timer = setInterval(() => {}, 1000);
// 忘记: clearInterval(timer);

// 3. 闭包引用大对象
function createLeak() {
  const hugeData = new Array(1000000);
  return function() {
    console.log(hugeData.length); // hugeData 无法回收
  };
}

// 4. DOM 引用未清理
const elements = {
  button: document.getElementById('button')
};
document.body.removeChild(elements.button);
// elements.button 仍然引用 DOM

// 解决方案
// 1. 使用 WeakMap/WeakSet
const cache = new WeakMap(); // 键可被垃圾回收

// 2. 及时清理引用
element = null;
timer = null;
```

</details>

### 9. ES2024 新特性有哪些? 如何应用? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

**主要新特性:**

1. **Array.findLast() / findLastIndex()** - 从后往前查找
2. **Array.toSorted() / toReversed() / toSpliced() / with()** - 非破坏性数组方法
3. **Object.groupBy() / Map.groupBy()** - 原生分组功能
4. **Promise.withResolvers()** - 简化 Promise 创建
5. **String.isWellFormed() / toWellFormed()** - Unicode 字符串检测和修复
6. **ArrayBuffer.transfer()** - 高效转移 ArrayBuffer 所有权

**应用示例:**

```javascript
// 1. 查找最后一个满足条件的元素
const lastError = logs.findLast(log => log.level === 'error');

// 2. 不可变数组操作 (React/Vue 友好)
const sorted = items.toSorted((a, b) => a.price - b.price);

// 3. 原生分组
const grouped = Object.groupBy(users, user => user.role);

// 4. 可取消的 Promise
const { promise, resolve, reject } = Promise.withResolvers();
```

详细内容请参考: [ES2024/ES2025 新特性详解](./es2024-2025.md)

</details>

### 10. 如何优化大列表渲染性能? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

**方案 1: 虚拟滚动**

```javascript
class VirtualList {
  constructor(container, items, itemHeight) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight);
    this.startIndex = 0;
    this.render();
    container.addEventListener('scroll', this.handleScroll.bind(this));
  }
  
  handleScroll() {
    const scrollTop = this.container.scrollTop;
    const newStartIndex = Math.floor(scrollTop / this.itemHeight);
    if (newStartIndex !== this.startIndex) {
      this.startIndex = newStartIndex;
      this.render();
    }
  }
  
  render() {
    const endIndex = Math.min(
      this.startIndex + this.visibleCount + 1,
      this.items.length
    );
    const visibleItems = this.items.slice(this.startIndex, endIndex);
    // 只渲染可见项...
  }
}
```

**方案 2: 使用 Intersection Observer**

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadItem(entry.target);
    }
  });
});

items.forEach(item => observer.observe(item));
```

**方案 3: 分页加载**

```javascript
async function loadPage(page) {
  const data = await fetch(`/api/items?page=${page}`).then(r => r.json());
  appendItems(data.items);
}
```

详细内容请参考: [性能优化专题](../performance/runtime.md)

</details>

### 11. 如何实现一个 LRU 缓存? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1;
    
    // 更新顺序: 删除后重新添加
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // 删除最久未使用的 (Map 的第一个元素)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }
}

// 使用示例
const cache = new LRUCache(2);
cache.put(1, 1);
cache.put(2, 2);
cache.get(1);    // 返回 1
cache.put(3, 3); // 该操作会使得密钥 2 作废
cache.get(2);    // 返回 -1 (未找到)
```

**应用场景**: HTTP 缓存、React Query 缓存、Vue 组件缓存

</details>

### 12. 如何实现请求重试机制? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

```javascript
async function retry(fn, options = {}) {
  const {
    retries = 3,
    delay = 1000,
    backoff = 2,
    onRetry = () => {}
  } = options;
  
  let lastError;
  
  for (let i = 0; i <= retries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (i < retries) {
        const waitTime = delay * Math.pow(backoff, i);
        onRetry(error, i + 1, waitTime);
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
  }
  
  throw lastError;
}

// 使用示例
const result = await retry(
  () => fetch('/api/data').then(r => r.json()),
  {
    retries: 3,
    delay: 1000,
    backoff: 2, // 指数退避: 1s, 2s, 4s
    onRetry: (error, attempt, waitTime) => {
      console.log(`重试第 ${attempt} 次，等待 ${waitTime}ms`);
    }
  }
);
```

**应用场景**: 网络请求重试、文件上传重试、API 调用重试

</details>

### 13. 如何实现发布订阅模式? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  off(event, callback) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }
  
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(...args));
    }
  }
  
  once(event, callback) {
    const wrapper = (...args) => {
      callback(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
}

// 使用示例
const emitter = new EventEmitter();
emitter.on('click', (data) => console.log('clicked', data));
emitter.emit('click', { x: 1, y: 2 });
```

**应用场景**: 组件通信、事件总线、状态管理

</details>

### 14. 如何优化首屏加载时间? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

**优化策略:**

1. **代码分割和懒加载**
```javascript
// 路由懒加载
const Home = lazy(() => import('./Home'));
const About = lazy(() => import('./About'));
```

2. **资源压缩和 CDN**
```javascript
// 使用 CDN 加载第三方库
<script src="https://cdn.example.com/react.min.js"></script>
```

3. **预加载关键资源**
```html
<link rel="preload" href="/critical.css" as="style">
<link rel="prefetch" href="/next-page.js" as="script">
```

4. **减少 HTTP 请求**
```javascript
// 合并请求
const [user, posts] = await Promise.all([
  fetch('/api/user'),
  fetch('/api/posts')
]);
```

5. **使用 HTTP/2 或 HTTP/3**
6. **服务端渲染 (SSR)**
7. **图片优化**: WebP、懒加载、响应式图片

详细内容请参考: [性能优化专题](../performance/runtime.md)

</details>

### 15. Object.groupBy 和传统 reduce 的区别? ⭐ 2025高频

<details>
<summary>点击查看答案</summary>

**传统方式:**

```javascript
const grouped = inventory.reduce((acc, item) => {
  const key = item.type;
  if (!acc[key]) {
    acc[key] = [];
  }
  acc[key].push(item);
  return acc;
}, {});
```

**ES2024 方式:**

```javascript
const grouped = Object.groupBy(inventory, ({ type }) => type);
```

**区别:**

1. **代码简洁性**: `Object.groupBy` 更简洁
2. **性能**: 原生实现，性能更好
3. **可读性**: 语义更清晰
4. **类型安全**: TypeScript 支持更好

**Map.groupBy 的优势:**

```javascript
// 键可以是任意类型
const grouped = Map.groupBy(items, item => item.id); // 数字键
const grouped2 = Map.groupBy(items, item => item.date); // 日期键
```

</details>

---

## 总结

### 核心考点

1. **数据类型**: 类型判断、类型转换
2. **作用域**: 作用域链、闭包、变量提升
3. **this**: 绑定规则、call/apply/bind
4. **原型**: 原型链、继承方式
5. **异步**: Promise、async/await、事件循环
6. **内存**: 垃圾回收、内存泄漏
7. **ES2024/ES2025**: 新特性、应用场景 ⭐ 2025新增
8. **性能优化**: 代码优化、内存优化、渲染优化 ⭐ 2025新增

### 面试技巧

1. **先说概念,再举例子**: 先解释原理,再给出实际应用场景
2. **结合实际项目经验**: 分享在项目中如何应用这些知识点
3. **展示深入理解**: 不仅要知道是什么,更要知道为什么
4. **主动提及相关知识点**: 展示知识体系的完整性
5. **关注最新特性**: 了解 ES2024/ES2025 新特性,展示学习能力 ⭐ 2025新增
6. **性能优化意识**: 在回答中体现性能优化的思考 ⭐ 2025新增

### 2025年面试趋势

1. **新特性应用**: ES2024/ES2025 新特性的实际应用场景
2. **性能优化**: 代码性能、内存优化、渲染优化
3. **工程化实践**: 模块化、打包优化、代码分割
4. **异步编程**: Promise 高级用法、并发控制、错误处理
5. **实际项目经验**: 解决实际问题的能力,而非纯理论

---

## 十三、ECMAScript 新特性 (ES2023/2024)

### 1. 数组非破坏性方法 (ES2023)

以前的 `sort`, `reverse`, `splice` 都会修改原数组，现在有了对应的不可变版本。

```javascript
/* 1. toSorted() */
const arr = [3, 1, 2];
const sorted = arr.toSorted();
console.log(arr); // [3, 1, 2] (原数组不变)
console.log(sorted); // [1, 2, 3]

/* 2. toReversed() */
const reversed = arr.toReversed();

/* 3. toSpliced() */
// 索引 1 开始，删除 1 个，插入 'a'
const spliced = arr.toSpliced(1, 1, "a");

/* 4. with() */
// 替换指定索引的值
const newArr = arr.with(1, "b"); // [3, 'b', 2]
```

### 2. Object.groupBy (ES2024)

原生分组功能，终于不用手写 `reduce` 或引入 lodash 了。

```javascript
const inventory = [
  { name: "asparagus", type: "vegetables", quantity: 5 },
  { name: "bananas", type: "fruit", quantity: 0 },
  { name: "goat", type: "meat", quantity: 23 },
  { name: "cherries", type: "fruit", quantity: 5 },
  { name: "fish", type: "meat", quantity: 22 },
];

const result = Object.groupBy(inventory, ({ type }) => type);

/* 结果:
{
  vegetables: [{ name: "asparagus", type: "vegetables", quantity: 5 }],
  fruit: [
    { name: "bananas", type: "fruit", quantity: 0 },
    { name: "cherries", type: "fruit", quantity: 5 }
  ],
  meat: [
    { name: "goat", type: "meat", quantity: 23 },
    { name: "fish", type: "meat", quantity: 22 }
  ]
}
*/
```

### 3. Promise.withResolvers (ES2024)

简化 Promise 的创建，直接获取 resolve 和 reject 函数。

```javascript
// 以前
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});

// 现在
const { promise, resolve, reject } = Promise.withResolvers();

// 应用场景: 在类中管理异步状态
class AsyncQueue {
  constructor() {
    const { promise, resolve, reject } = Promise.withResolvers();
    this.promise = promise;
    this.resolve = resolve;
    this.reject = reject;
  }
  
  async process() {
    try {
      const result = await this.promise;
      return result;
    } catch (error) {
      throw error;
    }
  }
}
```

### 4. Array findLast / findLastIndex (ES2023)

从数组末尾开始查找元素。

```javascript
const arr = [1, 2, 3, 4, 5, 4, 3, 2, 1];

// 查找最后一个满足条件的元素
const lastEven = arr.findLast(x => x % 2 === 0); // 2
const lastIndex = arr.findLastIndex(x => x % 2 === 0); // 7

// 应用场景: 查找最近的日志、最后一条记录等
const logs = [
  { level: 'info', message: 'start' },
  { level: 'error', message: 'failed' },
  { level: 'info', message: 'end' }
];
const lastError = logs.findLast(log => log.level === 'error');
```

### 5. Array toSorted / toReversed / toSpliced / with (ES2023)

数组的非破坏性方法，返回新数组而不修改原数组。

```javascript
const arr = [3, 1, 2];

// toSorted: 非破坏性排序
const sorted = arr.toSorted(); // [1, 2, 3]
console.log(arr); // [3, 1, 2] (原数组不变)

// toReversed: 非破坏性反转
const reversed = arr.toReversed(); // [2, 1, 3]

// toSpliced: 非破坏性 splice
const spliced = arr.toSpliced(1, 1, 'a', 'b'); // [3, 'a', 'b', 2]

// with: 替换指定索引的值
const newArr = arr.with(1, 'x'); // [3, 'x', 2]

// 应用场景: React/Vue 等框架中的不可变数据更新
const updatedItems = items.toSorted((a, b) => a.price - b.price);
```

### 6. String.prototype.isWellFormed / toWellFormed (ES2024)

检测和修复字符串中的代理对问题。

```javascript
// 检测字符串是否格式正确
const str1 = "Hello";
str1.isWellFormed(); // true

const str2 = "\uD800"; // 孤立的代理对
str2.isWellFormed(); // false

// 修复格式不正确的字符串
const fixed = str2.toWellFormed(); // "\uFFFD" (替换为替换字符)
```

### 7. ArrayBuffer.prototype.transfer / transferToFixedLength (ES2024)

高效地转移 ArrayBuffer 的所有权。

```javascript
const buffer1 = new ArrayBuffer(1024);
const buffer2 = buffer1.transfer(512); // 转移并调整大小

// buffer1 现在 detached (无法使用)
// buffer2 是新的 ArrayBuffer，大小为 512 字节

// 应用场景: Web Workers 间高效传输数据
const worker = new Worker('worker.js');
const data = new ArrayBuffer(1024);
// 转移所有权，避免复制
worker.postMessage(data.transfer(), [data]);
```

### 8. 正则表达式 v 标志 (ES2024)

增强的正则表达式功能，支持更强大的 Unicode 匹配。

```javascript
// 使用 v 标志启用新的 Unicode 特性
const regex = /[\p{Emoji}\p{Letter}]/v;

// 支持集合运算
const regex2 = /[\p{Letter}--\p{ASCII}]/v; // 非 ASCII 字母
```

### 9. Temporal API (提案阶段, 2025年关注)

更强大的日期时间处理 API。

```javascript
// 当前 Date API 的问题
const date1 = new Date('2024-01-01');
const date2 = new Date('2024-01-01T00:00:00Z');
date1.getTime() !== date2.getTime(); // 时区问题

// Temporal API (提案中)
// const instant = Temporal.Now.instant();
// const zoned = instant.toZonedDateTimeISO('Asia/Shanghai');
```

---

## 十四、2025年高频面试题补充

### 1. 如何实现一个支持取消的 Promise?

```javascript
function cancellablePromise(executor) {
  let cancel;
  const promise = new Promise((resolve, reject) => {
    cancel = (reason) => {
      reject(new Error(reason || 'Cancelled'));
    };
    executor(resolve, reject, cancel);
  });
  promise.cancel = cancel;
  return promise;
}

// 使用
const p = cancellablePromise((resolve, reject, cancel) => {
  setTimeout(() => resolve('done'), 5000);
});

p.then(console.log).catch(console.error);
p.cancel('User cancelled'); // 取消 Promise
```

### 2. 如何实现一个带超时的 Promise?

```javascript
function timeoutPromise(promise, timeout) {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout')), timeout)
    )
  ]);
}

// 使用
const fetchWithTimeout = timeoutPromise(
  fetch('/api/data'),
  5000
);
```

### 3. 如何实现一个并发控制的异步队列?

```javascript
async function concurrentLimit(tasks, limit) {
  const results = [];
  const executing = [];
  
  for (const task of tasks) {
    const promise = Promise.resolve(task()).then(result => {
      executing.splice(executing.indexOf(promise), 1);
      return result;
    });
    
    results.push(promise);
    executing.push(promise);
    
    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }
  
  return Promise.all(results);
}

// 使用
const tasks = Array.from({ length: 10 }, (_, i) => 
  () => fetch(`/api/data/${i}`)
);
const results = await concurrentLimit(tasks, 3); // 最多3个并发
```

### 4. 如何检测内存泄漏?

```javascript
// 使用 Performance API 监控内存
function monitorMemory() {
  if (performance.memory) {
    const used = performance.memory.usedJSHeapSize;
    const total = performance.memory.totalJSHeapSize;
    const limit = performance.memory.jsHeapSizeLimit;
    
    console.log({
      used: `${(used / 1024 / 1024).toFixed(2)} MB`,
      total: `${(total / 1024 / 1024).toFixed(2)} MB`,
      limit: `${(limit / 1024 / 1024).toFixed(2)} MB`,
      usage: `${((used / limit) * 100).toFixed(2)}%`
    });
  }
}

// 定期检查
setInterval(monitorMemory, 5000);
```

### 5. 如何优化大列表渲染性能?

```javascript
// 虚拟滚动实现
class VirtualList {
  constructor(container, items, itemHeight) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight);
    this.scrollTop = 0;
    
    this.render();
    container.addEventListener('scroll', () => this.handleScroll());
  }
  
  handleScroll() {
    this.scrollTop = this.container.scrollTop;
    this.render();
  }
  
  render() {
    const start = Math.floor(this.scrollTop / this.itemHeight);
    const end = Math.min(start + this.visibleCount + 1, this.items.length);
    
    const visibleItems = this.items.slice(start, end);
    const offsetY = start * this.itemHeight;
    
    // 更新 DOM
    this.container.innerHTML = '';
    visibleItems.forEach((item, index) => {
      const element = document.createElement('div');
      element.textContent = item;
      element.style.position = 'absolute';
      element.style.top = `${offsetY + index * this.itemHeight}px`;
      this.container.appendChild(element);
    });
    
    // 设置总高度
    this.container.style.height = `${this.items.length * this.itemHeight}px`;
  }
}
```

### 6. 如何实现一个 LRU 缓存?

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1;
    
    // 更新顺序: 删除后重新添加
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // 删除最久未使用的 (Map 的第一个元素)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }
}
```

### 7. 如何实现一个发布订阅模式?

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  off(event, callback) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }
  
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(...args));
    }
  }
  
  once(event, callback) {
    const wrapper = (...args) => {
      callback(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
}
```

### 8. 如何实现一个请求重试机制?

```javascript
async function retry(fn, options = {}) {
  const {
    retries = 3,
    delay = 1000,
    backoff = 2,
    onRetry = () => {}
  } = options;
  
  let lastError;
  
  for (let i = 0; i <= retries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (i < retries) {
        const waitTime = delay * Math.pow(backoff, i);
        onRetry(error, i + 1, waitTime);
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
  }
  
  throw lastError;
}

// 使用
const result = await retry(
  () => fetch('/api/data').then(r => r.json()),
  {
    retries: 3,
    delay: 1000,
    backoff: 2,
    onRetry: (error, attempt, waitTime) => {
      console.log(`重试第 ${attempt} 次，等待 ${waitTime}ms`);
    }
  }
);
```
