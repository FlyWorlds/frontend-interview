# V8 引擎与内存管理 (进阶)

## 1. V8 执行原理

### V8 是什么?

V8 是 Google 开发的开源高性能 JavaScript 和 WebAssembly 引擎,用于 Chrome 浏览器和 Node.js。

### 编译流水线 (The Pipeline)

V8 不仅仅解释执行 JS,它采用了 **JIT (Just-In-Time)** 编译技术。

1.  **Parse (解析)**: 将 JS 源码解析为 AST (抽象语法树)。
2.  **Ignition (解释器)**: 将 AST 转换为 Bytecode (字节码) 并解释执行。
    - _为什么不直接转机器码?_ 字节码更节省内存,且启动速度快。
3.  **TurboFan (优化编译器)**:
    - 在代码运行过程中,收集**类型信息 (Type Feedback)**。
    - 将"热点代码" (Hot Function) 编译为高效的**机器码 (Machine Code)**。
    - **Deoptimization (去优化)**: 如果后续运行中类型假设失效 (比如原本认为是整数的变量突然变成了字符串),V8 会丢弃优化代码,回退到 Ignition 解释执行。

### 核心优化机制

#### 隐藏类 (Hidden Classes / Shapes)

JS 是动态语言,对象属性可随时增删。V8 为了加速属性访问,会在后台动态创建"隐藏类"。

- 拥有相同属性结构的对象共享同一个隐藏类。
- **优化建议**:
  - 在构造函数中初始化所有属性 (避免后续动态添加)。
  - 属性初始化顺序保持一致。

#### 内联缓存 (Inline Caching, IC)

V8 会缓存属性访问的查找结果 (偏移量)。下次访问同一对象的同名属性时,直接使用缓存,无需查表。

---

## 2. 垃圾回收 (Garbage Collection)

V8 的内存主要分为 **栈 (Stack)** 和 **堆 (Heap)**。GC 主要管理**堆内存**。

### 内存分代策略

V8 将堆内存分为两代,采用不同的 GC 算法:

1.  **新生代 (New Space)**: 存放生命周期短的对象 (如临时变量)。容量小 (1-8MB)。
2.  **老生代 (Old Space)**: 存放生命周期长或常驻的对象。容量大。

### 新生代 GC: Scavenge 算法

使用 **Cheney 算法**,将空间平分为 `From Space` 和 `To Space`。

1.  新对象分配在 From Space。
2.  GC 开始: 检查 From Space 中的存活对象。
3.  将存活对象**复制**到 To Space (有序排列,无碎片)。
4.  清空 From Space。
5.  交换角色: To 变 From, From 变 To。

- **晋升 (Promotion)**: 对象经过 2 次 Scavenge 还存活,或 To Space 占用超过 25%,则晋升到老生代。

### 老生代 GC: Mark-Sweep-Compact

1.  **Mark (标记)**: 从根对象 (Root) 开始遍历,标记所有可达对象 (三色标记法)。
2.  **Sweep (清除)**: 遍历堆内存,释放未标记对象的空间 (产生内存碎片)。
3.  **Compact (整理)**: 将存活对象向一端移动,合并碎片 (开销大,只在必要时执行)。

### 优化技术

- **Incremental Marking (增量标记)**: 将一口气完成的标记任务拆分成小步,穿插在 JS 执行之间,减少全停顿 (Stop-The-World) 时间。
- **Lazy Sweeping (惰性清除)**: 按需清除垃圾。
- **Concurrent (并发)**: 辅助线程辅助 GC,不阻塞主线程。

---

## 3. 内存泄漏 (Memory Leaks)

即使有 GC,不当的代码仍会导致内存泄漏 (对象不再需要但无法被回收)。

### 常见场景

#### 1. 意外的全局变量

```javascript
function foo() {
  bar = "potential leak"; // 相当于 window.bar
}
```

#### 2. 被遗忘的计时器或回调

```javascript
const bigData = getData();
setInterval(() => {
  // 引用了 bigData,导致其无法回收
  console.log(bigData);
}, 1000);
```

#### 3. 闭包 (Closures)

```javascript
function outer() {
  const heavyObj = new Array(10000).fill("*");
  return function inner() {
    // inner 引用了 heavyObj,只要 inner 存在,heavyObj 就不会回收
    console.log(heavyObj[0]);
  };
}
```

#### 4. 分离的 DOM 节点 (Detached DOM)

DOM 节点已从页面移除,但 JS 变量仍持有引用。

```javascript
let detachedNodes;
function create() {
  const ul = document.createElement("ul");
  for (let i = 0; i < 10; i++) {
    const li = document.createElement("li");
    ul.appendChild(li);
  }
  detachedNodes = ul; // 即使 ul 没添加到 body,也还在内存中
}
```

### 排查工具

- **Chrome DevTools > Memory Panel**:
  - **Heap Snapshot**: 拍摄堆快照,对比 GC 前后的差异。
  - **Allocation Instrumentation on timeline**: 实时查看内存分配。
- **Performance Monitor**: 实时监控 JS Heap 大小。

---

## 4. 高频面试题

### Q1: V8 执行 JS 代码是直接解释执行还是编译执行?

**答**: 混合模式。有些面试官可能只知道"JS 是解释型语言",但准确说是 **JIT (即时编译)**。V8 先用 Ignition 解释器转字节码执行 (启动快),运行中 TurboFan 编辑器将热点代码转可以在 CPU 直接运行的机器码 (运行快)。

### Q2: 为什么 `[] !== []`?

**答**: 数组是引用类型,存储在堆内存中。两个 `[]` 在堆中是两个不同的内存地址,比较的是引用地址。

### Q3: 什么是"全停顿" (Stop-The-World)? V8 怎么解决?

**答**: GC 执行时必须暂停 JS 应用逻辑 (否则对象引用关系在 GC 途中变了就乱套了)。
V8 通过 **增量标记 (Incremental Marking)** 和 **并发标记 (Concurrent Marking)** 解决,将大暂停拆成微小的暂停,让用户几乎感知不到卡顿。
