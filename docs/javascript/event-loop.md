# 事件循环 (Event Loop) 深度解析

## 1. 原理概述

![JS Event Loop Mechanism](/images/event-loop.png)

JavaScript 是单线程语言,为了在处理高耗时操作(如网络请求、IO)时不阻塞主线程,引入了**事件循环机制**。

### 核心组件

1.  **Call Stack (调用栈)**: 执行同步代码。
2.  **Heap (堆)**: 内存分配。
3.  **Task Queue (宏任务队列)**: `setTimeout`, `setInterval`, I/O, UI Rendering.
4.  **Microtask Queue (微任务队列)**: `Promise`, `process.nextTick`, `MutationObserver`.

---

## 2. 浏览器与 Node.js 的区别

### 浏览器 Event Loop

执行顺序如下:

1.  执行完主线程中的同步代码 (Call Stack 清空)。
2.  **清空微任务队列**: 执行**所有**微任务。
3.  **UI 渲染**: 浏览器判断是否需要更新页面。
4.  **执行一个宏任务**: 从宏任务队列取出一个执行。
5.  回到步骤 2。

> **重点**: 微任务优先级极高,会在渲染前和下一个宏任务前执行完毕。

### Node.js Event Loop

Node.js (11.0+) 的行为已经与浏览器非常接近,但在底层阶段划分上更细致 (libuv)。

**6 个阶段 (Phases)**:

1.  **Timers**: 执行 `setTimeout` 和 `setInterval` 的回调。
2.  **Pending Callbacks**: 执行系统操作的回调 (如 TCP 错误)。
3.  **Idle, Prepare**: 内部使用。
4.  **Poll**: 获取新的 I/O 事件; 执行 I/O 回调; Node 会在这里阻塞等待。
5.  **Check**: 执行 `setImmediate` 回调。
6.  **Close Callbacks**: 执行关闭回调 (如 `socket.on('close')`)。

> **Node 11+ 更新**: 以前是"执行完一个阶段的所有宏任务"才清空微任务。现在改为"执行完**一个宏任务**"就清空微任务 (与浏览器一致)。

---

## 3. 宏任务 vs 微任务

| 类型                   | 代表 API                                                                                                 | 优先级                            |
| :--------------------- | :------------------------------------------------------------------------------------------------------- | :-------------------------------- |
| **Microtask (微任务)** | `Promise.then/catch/finally`<br>`process.nextTick` (Node)<br>`MutationObserver`<br>`queueMicrotask`      | **高** (在当前任务结束后立即执行) |
| **Macrotask (宏任务)** | `setTimeout`<br>`setInterval`<br>`setImmediate` (Node)<br>`requestAnimationFrame`<br>I/O 操作<br>UI 渲染 | **低** (在微任务清空后执行)       |

**注意 `process.nextTick`**: 在 Node.js 中,它的优先级高于 `Promise`。它属于"微任务",但往往在微任务队列的最前面执行。

---

## 4. 实战题目解析

### 题目 1: 基础顺序

```javascript
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

console.log("4");
```

**解析**:

1.  同步: `1`, `4`
2.  微任务: `3` (Promise)
3.  宏任务: `2` (timer)
    **输出**: `1 -> 4 -> 3 -> 2`

### 题目 2: 宏微嵌套

```javascript
setTimeout(() => {
  console.log("timer1");
  Promise.resolve().then(() => {
    console.log("promise1");
  });
}, 0);

Promise.resolve().then(() => {
  console.log("promise2");
  setTimeout(() => {
    console.log("timer2");
  }, 0);
});
```

**解析**:

1.  第一轮:
    - 宏任务队列: [timer1]
    - 微任务队列: [promise2]
    - 执行微任务 `promise2` -> 输出 'promise2' -> 产生宏任务 `timer2`
    - 此时宏任务: [timer1, timer2]
2.  第二轮 (宏任务 timer1):
    - 输出 'timer1'
    - 产生微任务 `promise1`
    - **立即清空微任务**: 输出 'promise1'
3.  第三轮 (宏任务 timer2): \* 输出 'timer2'
    **输出**: `promise2 -> timer1 -> promise1 -> timer2`

### 题目 3: async/await

```javascript
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async function async2() {
  console.log("async2");
}

console.log("script start");

setTimeout(function () {
  console.log("setTimeout");
}, 0);

async1();

new Promise(function (resolve) {
  console.log("promise1");
  resolve();
}).then(function () {
  console.log("promise2");
});

console.log("script end");
```

**解析**:

- `await` 后面的代码相当于 `Promise.then` (微任务)。

1.  `script start`
2.  `async1 start`
3.  `async2` (同步执行)
4.  `async1 end` 放入微任务 (await 挂起)
5.  `promise1`
6.  `promise2` 放入微任务
7.  `script end`
8.  清空微任务: `async1 end`, `promise2`
9.  宏任务: `setTimeout`

**输出**:
`script start` -> `async1 start` -> `async2` -> `promise1` -> `script end` -> `async1 end` -> `promise2` -> `setTimeout`

> _注意_: 旧版浏览器/Node 可能 `async1 end` 比 `promise2` 慢,但现代环境通常如上所述。

---

## 5. 常见面试题

### Q1: `setTimeout` 准时吗?

**不准时**。
它只是承诺在指定时间后将回调**放入宏任务队列**。如果主线程有大量同步计算,或微任务队列一直不清空,`setTimeout` 的执行会被推迟。

### Q2: 什么是 "UI Render" 时机?

浏览器通常尝试每 16.6ms (60fps) 渲染一次。

- 如果执行栈空了,且微任务空了,浏览器会判断是否需要渲染。
- `requestAnimationFrame` 会在**渲染前**执行。
- `setTimeout(0)` 不一定在渲染前或后,取决于宏任务执行的恰当时机。

### Q3: Node.js 的 `active handles` 是什么?

Node.js 会持续运行直到没有 active handles (如打开的 socket, 运行的 timer)。如果代码执行完但进程没退出,通常是因为有遗留的 timer 或连接没关闭。
