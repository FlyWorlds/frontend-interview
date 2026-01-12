# JavaScript 核心笔记

## 事件循环 (Event Loop)

### ❌ 常见错误理解

**错误说法**: "所有的宏任务执行完，进入下一次事件循环"

这种理解是错误的！很多人认为事件循环是：
1. 执行完所有宏任务
2. 然后清空微任务
3. 然后进入下一次循环

### ✅ 正确的事件循环流程

**正确流程**: 
```
执行一个宏任务 → 清空微任务 → 执行下一个宏任务 → 清空微任务...
```

### 详细执行顺序

```javascript
console.log('1'); // 同步代码

setTimeout(() => {
  console.log('2'); // 宏任务
  Promise.resolve().then(() => {
    console.log('3'); // 微任务
  });
}, 0);

Promise.resolve().then(() => {
  console.log('4'); // 微任务
  setTimeout(() => {
    console.log('5'); // 宏任务
  }, 0);
});

console.log('6'); // 同步代码

// 输出顺序: 1, 6, 4, 2, 3, 5
```

**执行过程解析:**

1. **同步代码执行**
   - 输出: `1`
   - 输出: `6`

2. **清空微任务队列**
   - 执行 `Promise.resolve().then()` → 输出: `4`
   - 在微任务中又注册了新的宏任务 `setTimeout`

3. **执行一个宏任务**
   - 执行第一个 `setTimeout` → 输出: `2`
   - 在宏任务中注册了新的微任务 `Promise.resolve().then()`

4. **清空微任务队列**
   - 执行刚才注册的微任务 → 输出: `3`

5. **执行下一个宏任务**
   - 执行第二个 `setTimeout` → 输出: `5`

### 核心要点

1. **每次只执行一个宏任务**
   - 不是执行完所有宏任务，而是执行完**一个**宏任务后，立即清空微任务队列

2. **微任务优先级极高**
   - 每次宏任务执行完后，会**立即清空所有微任务**
   - 微任务中注册的微任务也会在当前轮次执行完

3. **执行顺序**
   ```
   同步代码 → 微任务 → 宏任务 → 微任务 → 宏任务 → ...
   ```

### 宏任务和微任务分类

**宏任务 (MacroTask):**
- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- I/O 操作
- UI 渲染
- `script` (整体代码)

**微任务 (MicroTask):**
- `Promise.then/catch/finally`
- `queueMicrotask`
- `MutationObserver`
- `process.nextTick` (Node.js, 优先级最高)

### 经典面试题

```javascript
console.log('start');

setTimeout(() => {
  console.log('timeout1');
  Promise.resolve().then(() => {
    console.log('promise1');
  });
}, 0);

Promise.resolve().then(() => {
  console.log('promise2');
  setTimeout(() => {
    console.log('timeout2');
  }, 0);
});

console.log('end');

// 输出: start, end, promise2, timeout1, promise1, timeout2
```

**解析:**
1. 同步代码: `start`, `end`
2. 清空微任务: `promise2` (同时注册了 `timeout2`)
3. 执行宏任务: `timeout1` (同时注册了 `promise1`)
4. 清空微任务: `promise1`
5. 执行宏任务: `timeout2`

### 记忆口诀

> **"一个宏，清微任，再宏，再微任，循环往复"**

或者：

> **"宏任务一个一个来，微任务一清到底"**

---

## Vue 的 $nextTick

### ❌ 常见错误理解

**错误说法**: "渲染的微任务执行完，再执行其他微任务"

很多人认为 `$nextTick` 的执行顺序是：
1. Vue 的 DOM 更新微任务执行完
2. 然后执行用户注册的 `$nextTick` 回调
3. 然后执行其他微任务

### ✅ 正确的理解

**正确流程**: 
```
在同一个微任务中，先执行 DOM 更新，再执行用户注册的 $nextTick 回调
```

### 详细执行机制

```javascript
// Vue 源码简化版
let callbacks = []  // 存储用户注册的 nextTick 回调
let pending = false

function nextTick(callback) {
  callbacks.push(callback)
  
  if (!pending) {
    pending = true
    // 使用微任务
    Promise.resolve().then(flushCallbacks)
  }
}

function flushCallbacks() {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  
  // 执行所有回调
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

// Vue 的更新队列
function queueWatcher(watcher) {
  // 将 watcher 加入队列
  // ...
  
  // 在同一个微任务中执行更新
  nextTick(flushSchedulerQueue)
}

function flushSchedulerQueue() {
  // 1. 先执行 DOM 更新
  watchers.forEach(watcher => {
    watcher.run()  // 更新 DOM
  })
  
  // 2. 然后执行用户注册的 nextTick 回调
  // (这些回调已经在 callbacks 队列中)
}
```

### 实际执行顺序

```javascript
// 示例代码
this.message = 'Hello'
this.$nextTick(() => {
  console.log('nextTick callback')
  console.log(this.$el.textContent)  // 'Hello'
})

Promise.resolve().then(() => {
  console.log('Promise callback')
})
```

**执行过程:**

1. **同步代码执行**
   - `this.message = 'Hello'` → 触发响应式更新
   - Vue 将更新任务加入队列，调用 `nextTick(flushSchedulerQueue)`
   - 用户调用 `this.$nextTick()` → 回调加入 `callbacks` 队列
   - `Promise.resolve().then()` → 回调加入微任务队列

2. **微任务执行 (同一个微任务中)**
   - 执行 `flushSchedulerQueue()` → **DOM 更新完成**
   - 执行 `flushCallbacks()` → **执行用户注册的 nextTick 回调**
   - 执行 `Promise.resolve().then()` → **执行 Promise 回调**

**输出顺序:**
```
nextTick callback
Hello
Promise callback
```

### 核心要点

1. **DOM 更新和 nextTick 回调在同一个微任务中**
   - 不是"渲染微任务执行完，再执行其他微任务"
   - 而是"在同一个微任务中，先更新 DOM，再执行 nextTick 回调"

2. **执行顺序**
   ```
   同步代码 → 微任务开始
              ├─ Vue DOM 更新 (flushSchedulerQueue)
              ├─ 用户 nextTick 回调 (flushCallbacks)
              └─ 其他微任务 (Promise.then 等)
   ```

3. **为什么能保证 DOM 已更新？**
   - 因为 DOM 更新和 nextTick 回调在**同一个微任务**中顺序执行
   - DOM 更新在前，nextTick 回调在后

### 代码示例

```javascript
// Vue 组件
export default {
  data() {
    return {
      message: '初始值'
    }
  },
  
  methods: {
    updateMessage() {
      this.message = '新值'
      
      // 此时 DOM 还未更新
      console.log(this.$el.textContent)  // '初始值'
      
      this.$nextTick(() => {
        // DOM 已更新
        console.log(this.$el.textContent)  // '新值'
      })
      
      Promise.resolve().then(() => {
        // 也在同一个微任务中，但执行顺序在 nextTick 之后
        console.log('Promise', this.$el.textContent)  // '新值'
      })
    }
  }
}
```

### 记忆要点

> **"DOM 更新和 nextTick 回调在同一个微任务中，DOM 更新在前，nextTick 回调在后"**

或者：

> **"不是渲染微任务执行完再执行其他微任务，而是在同一个微任务中先更新 DOM，再执行 nextTick 回调"**

---

## 总结

### 关键理解

- ❌ **错误**: 所有宏任务执行完 → 清空微任务 → 下一次循环
- ✅ **正确**: 一个宏任务 → 清空微任务 → 下一个宏任务 → 清空微任务 → ...

### 实际应用

理解事件循环对于：
- 调试异步代码
- 优化性能
- 理解框架原理 (如 Vue 的 nextTick)
- 面试答题

都非常重要！
