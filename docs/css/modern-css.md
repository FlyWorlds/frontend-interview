# 现代 CSS 新特性 (2024/2025)

CSS 在过去几年发展迅速，许多以前需要 JS 或预处理器才能实现的功能现在已经有了原生支持。

## 1. 容器查询 (Container Queries)

媒体查询 (`@media`) 基于视口大小，而容器查询 (`@container`) 基于由于父容器大小的变化而改变样式的能力。这对于组件化开发至关重要。

### 使用方法

```css
/* 1. 定义容器 */
.card-container {
  container-type: inline-size; /* 监听宽度变化 */
  container-name: card; /* 可选命名 */
}

/* 2. 使用容器查询 */
@container card (max-width: 400px) {
  .card-content {
    flex-direction: column;
    font-size: 14px;
  }
}
```

### 为什么重要？

它实现了真正的"响应式组件"，组件不再依赖页面的整体布局，而是根据自身可用空间自适应。

## 2. 原生 CSS 嵌套 (Nesting)

不再需要 Sass/Less，浏览器原生支持嵌套语法。

```css
/* 以前 (Sass) */
.card {
  color: black;
  &:hover {
    color: blue;
  }
  .title {
    font-weight: bold;
  }
}

/* 现在 (原生 CSS) */
.card {
  color: black;

  /* & 符号代表父选择器 */
  &:hover {
    color: blue;
  }

  /* 也可以直接嵌套类名 */
  .title {
    font-weight: bold;
  }

  /* 媒体查询也可以嵌套 */
  @media (min-width: 500px) {
    padding: 20px;
  }
}
```

## 3. `:has()` 伪类 (父选择器)

CSS 终于有了"父选择器"！`:has()` 允许我们根据子元素的状态来选择父元素。

```css
/* 如果 card 包含 img，则 card 背景变黑 */
.card:has(img) {
  background: #000;
  color: #fff;
}

/* 表单验证：如果组内有无效输入，标记整个组 */
.form-group:has(input:invalid) {
  border-color: red;
}

/* 后面跟随 p 的 h1 */
h1:has(+ p) {
  margin-bottom: 0;
}
```

## 4. CSS Layers (`@layer`)

用于控制 CSS 的级联（Cascade）优先级，解决样式覆盖冲突问题。

```css
/* 定义层级 */
@layer reset, base, components, utilities;

/* 即使 utilities 在代码前面，它的优先级也更高 */
@layer utilities {
  .padding-lg {
    padding: 2rem;
  }
}

@layer base {
  body {
    margin: 0;
  }
}
```

**优先级顺序**：越靠后的 layer 优先级越高（不考虑特指度）。Utilities > Components > Base > Reset。

## 5. View Transitions API

无需复杂的 JS 动画库，提供原生且平滑的页面/状态切换动画。

```javascript
// JS 中触发
document.startViewTransition(() => {
  // 更新 DOM 的操作
  updateTheDOM();
});
```

```css
/* CSS 自定义过渡效果 */
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 0.5s;
}
```

## 6. 滚动驱动动画 (Scroll-driven Animations)

将动画进度绑定到滚动容器的滚动位置，无需 JS 监听 scroll 事件。

```css
/* 顶部进度条 */
.progress-bar {
  scale: 0 1;
  animation: grow auto linear;
  animation-timeline: scroll(); /* 绑定到滚动时间轴 */
}

@keyframes grow {
  to {
    scale: 1 1;
  }
}
```

## 常见面试题

### Q1: 容器查询 (`@container`) 和媒体查询 (`@media`) 的区别？

A: `@media` 查询的是浏览器的视口（Viewport）尺寸，适用于页面级布局。`@container` 查询的是最近的容器元素的尺寸，适用于组件级布局，让组件本身具备响应式能力，无论放在哪里都能自适应。

### Q2: `:has()` 选择器有什么实际应用场景？

A:

1. **父元素样式控制**：根据子元素是否存在改变父元素（如：卡片有图和无图的不同布局）。
2. **表单验证**：根据内部 input 的状态（valid/invalid/checked）改变表单组的样式。
3. **复杂兄弟选择**：虽然是父选择器，但结合 `+` 或 `~` 可以实现"前一个个兄弟选后一个兄弟"的效果。

### Q3: 为什么需要 CSS Layers？

A: 为了更好地管理 CSS 优先级（Specificity）。在大型项目中，经常遇到引入的第三方库样式覆盖困难，或者工具类被组件样式覆盖的问题。Layers 允许开发者显式定义层的优先级，而不必依赖 `!important` 或增加选择器权重。
