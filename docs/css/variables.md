# CSS 变量与主题切换

## 概述

CSS 自定义属性（CSS Variables）是现代 CSS 的核心特性，为动态主题切换、组件样式定制提供了原生解决方案。相比 Sass/Less 变量，CSS 变量可以在运行时动态修改。

## CSS 变量基础

### 定义与使用

```css
/* 定义变量（通常在 :root 中定义全局变量） */
:root {
  --primary-color: #1890ff;
  --secondary-color: #52c41a;
  --font-size-base: 14px;
  --spacing-unit: 8px;
  --border-radius: 4px;
}

/* 使用变量 */
.button {
  background-color: var(--primary-color);
  font-size: var(--font-size-base);
  padding: calc(var(--spacing-unit) * 2);
  border-radius: var(--border-radius);
}
```

### 变量命名规范

```css
:root {
  /* 颜色系统 */
  --color-primary: #1890ff;
  --color-primary-light: #40a9ff;
  --color-primary-dark: #096dd9;
  --color-success: #52c41a;
  --color-warning: #faad14;
  --color-error: #f5222d;

  /* 文字颜色 */
  --text-color-primary: rgba(0, 0, 0, 0.85);
  --text-color-secondary: rgba(0, 0, 0, 0.65);
  --text-color-disabled: rgba(0, 0, 0, 0.25);

  /* 背景颜色 */
  --bg-color-base: #f5f5f5;
  --bg-color-light: #fafafa;
  --bg-color-component: #ffffff;

  /* 边框 */
  --border-color-base: #d9d9d9;
  --border-color-split: #f0f0f0;

  /* 字体 */
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto;
  --font-size-sm: 12px;
  --font-size-base: 14px;
  --font-size-lg: 16px;
  --font-size-xl: 20px;

  /* 间距 */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;

  /* 圆角 */
  --radius-sm: 2px;
  --radius-base: 4px;
  --radius-lg: 8px;

  /* 阴影 */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-base: 0 2px 8px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 4px 16px rgba(0, 0, 0, 0.15);

  /* 动画 */
  --transition-fast: 0.1s;
  --transition-base: 0.2s;
  --transition-slow: 0.3s;

  /* 层级 */
  --z-index-dropdown: 1000;
  --z-index-modal: 1050;
  --z-index-tooltip: 1100;
}
```

### 默认值与回退

```css
.element {
  /* 使用默认值 */
  color: var(--undefined-color, #333);

  /* 嵌套默认值 */
  background: var(--bg-custom, var(--bg-default, white));

  /* 多层回退 */
  font-size: var(--font-size-custom, var(--font-size-base, 14px));
}
```

### 作用域与继承

```css
/* 全局作用域 */
:root {
  --color: blue;
}

/* 组件作用域 */
.card {
  --color: red;  /* 覆盖全局变量 */
  --card-padding: 16px;  /* 局部变量 */
}

.card .title {
  color: var(--color);  /* 继承 .card 的 --color */
}

/* 元素级别覆盖 */
.card.primary {
  --color: green;
}
```

## 主题切换实现

### 方案一：CSS 类切换

```css
/* 默认主题（浅色） */
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
  --primary-color: #1890ff;
  --border-color: #e8e8e8;
}

/* 暗色主题 */
:root.dark {
  --bg-color: #1f1f1f;
  --text-color: #ffffff;
  --primary-color: #177ddc;
  --border-color: #434343;
}

/* 或使用 data 属性 */
:root[data-theme="dark"] {
  --bg-color: #1f1f1f;
  --text-color: #ffffff;
}

/* 使用变量 */
body {
  background-color: var(--bg-color);
  color: var(--text-color);
}
```

```javascript
// 切换主题
function toggleTheme() {
  document.documentElement.classList.toggle('dark');
}

// 或使用 data 属性
function setTheme(theme) {
  document.documentElement.dataset.theme = theme;
}

// 保存用户偏好
function saveTheme(theme) {
  localStorage.setItem('theme', theme);
}

// 初始化主题
function initTheme() {
  const savedTheme = localStorage.getItem('theme');
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

  if (savedTheme) {
    setTheme(savedTheme);
  } else if (prefersDark) {
    setTheme('dark');
  }
}
```

### 方案二：媒体查询自适应

```css
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
}

/* 跟随系统主题 */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-color: #1f1f1f;
    --text-color: #ffffff;
  }
}
```

```javascript
// 监听系统主题变化
const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');

mediaQuery.addEventListener('change', (e) => {
  if (e.matches) {
    console.log('系统切换到暗色模式');
  } else {
    console.log('系统切换到浅色模式');
  }
});
```

### 方案三：JavaScript 动态修改

```javascript
// 直接修改 CSS 变量
function setThemeColor(color) {
  document.documentElement.style.setProperty('--primary-color', color);
}

// 批量设置变量
function setTheme(theme) {
  const root = document.documentElement;
  Object.entries(theme).forEach(([key, value]) => {
    root.style.setProperty(`--${key}`, value);
  });
}

// 使用
setTheme({
  'bg-color': '#1f1f1f',
  'text-color': '#ffffff',
  'primary-color': '#177ddc'
});

// 获取变量值
function getThemeColor() {
  return getComputedStyle(document.documentElement)
    .getPropertyValue('--primary-color')
    .trim();
}
```

### 完整主题切换系统

```javascript
class ThemeManager {
  constructor() {
    this.themes = {
      light: {
        '--bg-color': '#ffffff',
        '--bg-secondary': '#f5f5f5',
        '--text-color': '#333333',
        '--text-secondary': '#666666',
        '--primary-color': '#1890ff',
        '--border-color': '#e8e8e8',
        '--shadow-color': 'rgba(0, 0, 0, 0.1)'
      },
      dark: {
        '--bg-color': '#1f1f1f',
        '--bg-secondary': '#2d2d2d',
        '--text-color': '#ffffff',
        '--text-secondary': '#a0a0a0',
        '--primary-color': '#177ddc',
        '--border-color': '#434343',
        '--shadow-color': 'rgba(0, 0, 0, 0.3)'
      }
    };

    this.currentTheme = 'light';
    this.init();
  }

  init() {
    // 优先读取本地存储
    const saved = localStorage.getItem('theme');
    if (saved && this.themes[saved]) {
      this.setTheme(saved);
      return;
    }

    // 其次跟随系统
    if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
      this.setTheme('dark');
    }

    // 监听系统变化
    window.matchMedia('(prefers-color-scheme: dark)')
      .addEventListener('change', (e) => {
        if (!localStorage.getItem('theme')) {
          this.setTheme(e.matches ? 'dark' : 'light');
        }
      });
  }

  setTheme(themeName) {
    const theme = this.themes[themeName];
    if (!theme) return;

    const root = document.documentElement;
    Object.entries(theme).forEach(([key, value]) => {
      root.style.setProperty(key, value);
    });

    root.dataset.theme = themeName;
    this.currentTheme = themeName;
    localStorage.setItem('theme', themeName);

    // 触发自定义事件
    window.dispatchEvent(new CustomEvent('themechange', {
      detail: { theme: themeName }
    }));
  }

  toggle() {
    this.setTheme(this.currentTheme === 'light' ? 'dark' : 'light');
  }

  // 支持自定义主题
  registerTheme(name, variables) {
    this.themes[name] = variables;
  }
}

// 使用
const themeManager = new ThemeManager();

// 切换主题
document.getElementById('theme-toggle').addEventListener('click', () => {
  themeManager.toggle();
});

// 监听主题变化
window.addEventListener('themechange', (e) => {
  console.log('主题已切换:', e.detail.theme);
});
```

## 框架集成

### Vue 3 主题系统

```vue
<!-- ThemeProvider.vue -->
<script setup>
import { ref, provide, onMounted, watch } from 'vue';

const themes = {
  light: { /* ... */ },
  dark: { /* ... */ }
};

const currentTheme = ref('light');

function setTheme(theme) {
  const root = document.documentElement;
  const vars = themes[theme];

  Object.entries(vars).forEach(([key, value]) => {
    root.style.setProperty(key, value);
  });

  currentTheme.value = theme;
  localStorage.setItem('theme', theme);
}

function toggleTheme() {
  setTheme(currentTheme.value === 'light' ? 'dark' : 'light');
}

onMounted(() => {
  const saved = localStorage.getItem('theme');
  if (saved) setTheme(saved);
});

provide('theme', {
  current: currentTheme,
  setTheme,
  toggleTheme
});
</script>

<template>
  <slot />
</template>
```

```vue
<!-- 使用主题 -->
<script setup>
import { inject } from 'vue';

const { current, toggleTheme } = inject('theme');
</script>

<template>
  <button @click="toggleTheme">
    当前: {{ current }}
  </button>
</template>
```

### React 主题系统

```jsx
// ThemeContext.jsx
import { createContext, useContext, useState, useEffect } from 'react';

const themes = {
  light: {
    '--bg-color': '#ffffff',
    '--text-color': '#333333',
  },
  dark: {
    '--bg-color': '#1f1f1f',
    '--text-color': '#ffffff',
  }
};

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  useEffect(() => {
    const saved = localStorage.getItem('theme');
    if (saved) setTheme(saved);
  }, []);

  useEffect(() => {
    const root = document.documentElement;
    Object.entries(themes[theme]).forEach(([key, value]) => {
      root.style.setProperty(key, value);
    });
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, setTheme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

```jsx
// 使用
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button onClick={toggleTheme}>
      {theme === 'light' ? '🌙' : '☀️'}
    </button>
  );
}
```

## 高级技巧

### 颜色计算

```css
:root {
  --primary-h: 210;
  --primary-s: 100%;
  --primary-l: 50%;
  --primary-color: hsl(var(--primary-h), var(--primary-s), var(--primary-l));

  /* 自动生成色阶 */
  --primary-light: hsl(var(--primary-h), var(--primary-s), 70%);
  --primary-dark: hsl(var(--primary-h), var(--primary-s), 30%);
}

/* 动态修改色调 */
.warning {
  --primary-h: 45;  /* 只改变色调 */
}
```

### 响应式变量

```css
:root {
  --container-width: 1200px;
  --font-size-base: 16px;
  --spacing-unit: 16px;
}

@media (max-width: 768px) {
  :root {
    --container-width: 100%;
    --font-size-base: 14px;
    --spacing-unit: 12px;
  }
}

@media (max-width: 480px) {
  :root {
    --font-size-base: 12px;
    --spacing-unit: 8px;
  }
}
```

### 组件级变量

```css
/* 按钮组件变量 */
.btn {
  --btn-height: 32px;
  --btn-padding: 0 16px;
  --btn-font-size: 14px;
  --btn-border-radius: var(--radius-base);
  --btn-bg: var(--color-primary);
  --btn-color: white;

  height: var(--btn-height);
  padding: var(--btn-padding);
  font-size: var(--btn-font-size);
  border-radius: var(--btn-border-radius);
  background: var(--btn-bg);
  color: var(--btn-color);
}

/* 尺寸变体 */
.btn-small {
  --btn-height: 24px;
  --btn-padding: 0 8px;
  --btn-font-size: 12px;
}

.btn-large {
  --btn-height: 40px;
  --btn-padding: 0 24px;
  --btn-font-size: 16px;
}

/* 样式变体 */
.btn-secondary {
  --btn-bg: transparent;
  --btn-color: var(--color-primary);
}
```

### 动画与过渡

```css
:root {
  --theme-transition: background-color 0.3s, color 0.3s, border-color 0.3s;
}

/* 主题切换时平滑过渡 */
body,
.card,
.button {
  transition: var(--theme-transition);
}

/* 禁用过渡（首次加载） */
.no-transition * {
  transition: none !important;
}
```

```javascript
// 首次加载时禁用过渡
document.documentElement.classList.add('no-transition');
initTheme();
// 强制重绘后移除
requestAnimationFrame(() => {
  requestAnimationFrame(() => {
    document.documentElement.classList.remove('no-transition');
  });
});
```

## CSS 变量 vs 预处理器变量

| 特性 | CSS 变量 | Sass/Less 变量 |
|------|----------|---------------|
| 运行时修改 | ✅ | ❌ |
| 作用域继承 | ✅ | ❌ |
| 媒体查询中使用 | ✅ | 需编译 |
| JavaScript 访问 | ✅ | ❌ |
| 浏览器支持 | IE 不支持 | 编译后通用 |
| 调试体验 | 开发者工具可见 | 编译后不可见 |

### 混合使用

```scss
// Sass 变量（编译时计算）
$breakpoints: (
  sm: 576px,
  md: 768px,
  lg: 992px
);

// CSS 变量（运行时动态）
:root {
  --color-primary: #1890ff;
}

// 结合使用
.element {
  color: var(--color-primary);

  @media (min-width: map-get($breakpoints, md)) {
    font-size: 16px;
  }
}
```

## 性能优化

### 减少变量查找

```css
/* 避免 */
.element {
  margin: var(--spacing-md) var(--spacing-md) var(--spacing-md) var(--spacing-md);
}

/* 推荐 */
.element {
  --_spacing: var(--spacing-md);
  margin: var(--_spacing);
}
```

### 合理使用作用域

```css
/* 组件内部变量使用下划线前缀 */
.card {
  --_padding: var(--spacing-md);
  --_radius: var(--radius-base);

  padding: var(--_padding);
  border-radius: var(--_radius);
}
```

## 面试常见问题

### 1. CSS 变量和 Sass 变量的区别？

- CSS 变量可在运行时修改，Sass 变量在编译时确定
- CSS 变量有作用域和继承，Sass 变量是全局的
- CSS 变量可通过 JavaScript 访问和修改
- CSS 变量在开发者工具中可见

### 2. 如何实现主题切换？

三种主要方案：
1. CSS 类切换（:root.dark）
2. data 属性切换（:root[data-theme="dark"]）
3. JavaScript 直接修改 CSS 变量

推荐方案 1 或 2，便于 CSS 管理。

### 3. CSS 变量的兼容性如何处理？

```css
.element {
  /* 回退值 */
  color: #333;
  color: var(--text-color, #333);
}

/* 或使用 @supports */
@supports (--css: variables) {
  .element {
    color: var(--text-color);
  }
}
```

### 4. 如何避免主题切换时的闪烁？

```javascript
// 在 HTML 头部立即执行
<script>
  const theme = localStorage.getItem('theme') ||
    (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
  document.documentElement.dataset.theme = theme;
</script>
```

## 总结

CSS 变量是现代前端主题系统的核心：

1. **原生支持**：无需编译，运行时动态修改
2. **作用域继承**：支持组件级和全局级变量
3. **主题切换**：配合类切换或 JavaScript 实现
4. **框架集成**：Vue/React 中配合状态管理
5. **性能优化**：合理使用作用域和命名

掌握 CSS 变量是实现现代化主题系统的必备技能。
