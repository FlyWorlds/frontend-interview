# 面试准备笔记

## 高频面试题总结

### JavaScript 基础

#### 1. typeof 和 instanceof 的区别?

**typeof:**
- 返回字符串，表示类型
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

#### 2. == 和 === 的区别?

**===** 严格相等:
- 不进行类型转换
- 类型不同直接返回 false

**==** 宽松相等:
- 会进行类型转换
- 转换规则复杂

```javascript
1 === "1"  // false
1 == "1"   // true
null == undefined  // true
null === undefined // false
```

**建议: 始终使用 ===**

#### 3. var/let/const 的区别?

| 特性     | var        | let        | const      |
| -------- | ---------- | ---------- | ---------- |
| 作用域   | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 是         | TDZ        | TDZ        |
| 重复声明 | 允许       | 不允许     | 不允许     |
| 重新赋值 | 允许       | 允许       | 不允许     |

#### 4. 深拷贝和浅拷贝的区别?

**浅拷贝**: 只复制第一层
```javascript
Object.assign({}, obj)
{ ...obj }
[].concat(arr)
[...arr]
```

**深拷贝**: 递归复制所有层
```javascript
JSON.parse(JSON.stringify(obj))  // 有限制

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

---

## 框架相关

### Vue

#### Vue 2 vs Vue 3 的区别?

1. **响应式原理**
   - Vue 2: Object.defineProperty
   - Vue 3: Proxy

2. **Composition API**
   - Vue 3 新增，更好的逻辑复用

3. **性能优化**
   - Vue 3: 更小的包体积，更快的渲染

4. **TypeScript 支持**
   - Vue 3: 更好的 TS 支持

### React

#### Hooks 的使用规则?

1. 只能在函数组件或自定义 Hook 中使用
2. 只能在顶层调用，不能在循环、条件或嵌套函数中调用
3. 依赖数组要完整

#### useState 和 useEffect 的区别?

**useState:**
- 用于管理组件状态
- 返回 [state, setState]

**useEffect:**
- 用于处理副作用
- 在组件渲染后执行
- 可以返回清理函数

---

## 性能优化

### 如何优化首屏加载时间?

1. **代码分割和懒加载**
2. **资源压缩和 CDN**
3. **预加载关键资源**
4. **减少 HTTP 请求**
5. **使用 HTTP/2 或 HTTP/3**
6. **服务端渲染 (SSR)**

### 如何优化大列表渲染?

1. **虚拟滚动**: 只渲染可见区域
2. **分页加载**: 分批加载数据
3. **使用 DocumentFragment**: 批量 DOM 操作
4. **防抖/节流**: 优化滚动事件

---

## 手写代码

### 防抖 (Debounce)

```javascript
function debounce(fn, delay) {
  let timer = null;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

### 节流 (Throttle)

```javascript
function throttle(fn, delay) {
  let lastTime = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    }
  };
}
```

### 深拷贝

```javascript
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

---

## 面试技巧

### 回答问题的结构

1. **先说概念**: 解释是什么
2. **再说原理**: 解释为什么
3. **举例说明**: 给出实际例子
4. **总结应用**: 说明应用场景

### 常见问题准备

1. **自我介绍**: 突出技术栈和项目经验
2. **项目介绍**: 准备 STAR 法则 (Situation, Task, Action, Result)
3. **技术选型**: 说明为什么选择这个技术
4. **遇到问题**: 说明如何排查和解决

---

## 总结

### 准备清单

- [ ] JavaScript 核心概念
- [ ] 框架原理 (Vue/React)
- [ ] 性能优化
- [ ] 手写代码
- [ ] 项目经验
- [ ] 系统设计

### 面试建议

1. **充分准备**: 系统复习知识点
2. **多练习**: 多写代码，多思考
3. **保持自信**: 相信自己的能力
4. **诚实回答**: 不会的就说不会，但要说明学习能力

**加油！相信自己！** 💪
