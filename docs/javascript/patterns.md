# 设计模式与函数式编程

## 1. 常见设计模式 (Design Patterns)

### 单例模式 (Singleton)

保证一个类只有一个实例,并提供一个访问它的全局访问点。

```javascript
class Singleton {
  static instance;

  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    Singleton.instance = this;
  }
}
```

- **应用**: Vuex/Redux Store, 全局 Config, 数据库连接池。

### 观察者模式 (Observer)

对象间的一对多依赖关系,当一个对象改变状态时,所有依赖于它的对象都会收到通知并自动更新。

```javascript
class Subject {
  constructor() {
    this.observers = [];
  }
  add(observer) {
    this.observers.push(observer);
  }
  notify(msg) {
    this.observers.forEach((obs) => obs.update(msg));
  }
}

class Observer {
  update(msg) {
    console.log("收到通知:", msg);
  }
}
```

- **应用**: DOM 事件监听, Vue 响应式系统 (`Dep` & `Watcher`).

### 发布订阅模式 (Pub-Sub)

与观察者模式类似,但多了一个**调度中心 (Event Bus)**。发布者和订阅者互不认识。

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  on(event, cb) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(cb);
  }
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach((cb) => cb(...args));
    }
  }
}
```

- **应用**: Node.js `EventEmitter`, Vue Event Bus.

### 代理模式 (Proxy)

为其他对象提供一种代理以控制对这个对象的访问。

- **应用**: ES6 `Proxy` 实现数据劫持, 解决跨域 (Nginx/Webpack Dev Server 代理), 懒加载图片。

### 工厂模式 (Factory)

定义一个创建对象的接口,但让子类决定实例化哪一个类。

- **应用**: React `createElement`, Vue `h` 函数。

---

## 2. 函数式编程 (Functional Programming)

### 核心概念

1.  **纯函数 (Pure Function)**:
    - 相同输入永远得到相同输出。
    - 无**副作用 (Side Effects)** (不修改外部状态,不打印 log,不发网络请求)。
2.  **不可变性 (Immutability)**: 数据一旦创建就不能被修改。修改数据时返回新的数据。

### 高阶函数 (HOF)

接收函数作为参数,或返回一个函数的函数。

- `map`, `filter`, `reduce`
- React HOC (Higher-Order Component)

### 函数柯里化 (Currying)

将接受多个参数的函数变换成接受一个单一参数的函数,并且返回接受余下参数的新函数。

```javascript
// 普通函数
const add = (a, b) => a + b;

// 柯里化
const curriedAdd = (a) => (b) => a + b;
const add10 = curriedAdd(10);
console.log(add10(5)); // 15
```

- **作用**: 参数复用, 延迟执行。

### 函数组合 (Composition)

将多个函数组合成一个函数,数据从右向左流动 (`f(g(x))`)。

```javascript
const compose =
  (...fns) =>
  (x) =>
    fns.reduceRight((y, f) => f(y), x);

const add1 = (x) => x + 1;
const double = (x) => x * 2;

const addThenDouble = compose(double, add1);
// double(add1(2)) -> double(3) -> 6
console.log(addThenDouble(2)); // 6
```

- **应用**: Redux `compose`, Lodash `flowRight`.
