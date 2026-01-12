# 性能优化

## 概述

性能优化是前端工程师必须掌握的核心能力之一。面试中会重点考察你对**性能指标**、**优化手段**、**监控方案**的理解和实践经验。

## 性能指标

### Core Web Vitals (核心 Web 指标)

Google 提出的三个核心指标:

| 指标 | 全称 | 含义 | 目标值 |
|------|------|------|--------|
| **LCP** | Largest Contentful Paint | 最大内容绘制时间 | < 2.5s |
| **FID** | First Input Delay | 首次输入延迟 | < 100ms |
| **CLS** | Cumulative Layout Shift | 累积布局偏移 | < 0.1 |

### 其他重要指标

- **FCP** (First Contentful Paint): 首次内容绘制
- **TTI** (Time to Interactive): 可交互时间
- **TBT** (Total Blocking Time): 总阻塞时间
- **FPS** (Frames Per Second): 帧率

## 优化策略分类

### 1. 加载性能优化
- 资源优化(压缩、合并、CDN)
- 懒加载和预加载
- HTTP 缓存策略
- 代码分割
- 服务端渲染(SSR)

### 2. 运行时性能优化
- 虚拟列表
- 防抖和节流
- 减少重排重绘
- Web Worker
- requestAnimationFrame

### 3. 构建优化
- Tree Shaking
- 压缩和混淆
- 图片优化
- 依赖分析
- 打包体积分析

---

## 加载性能优化

### 资源压缩

```javascript
// Webpack 配置
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      // JS 压缩
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,  // 移除 console
            pure_funcs: ['console.log']
          }
        }
      }),

      // CSS 压缩
      new CssMinimizerPlugin()
    ]
  },

  // Gzip 压缩
  plugins: [
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240,  // 10KB 以上才压缩
      minRatio: 0.8
    })
  ]
};
```

### 代码分割

```javascript
// 1. 路由懒加载
const routes = [
  {
    path: '/home',
    component: () => import('./views/Home.vue')  // 动态导入
  },
  {
    path: '/about',
    component: () => import('./views/About.vue')
  }
];

// 2. 组件懒加载
const HeavyComponent = defineAsyncComponent(() =>
  import('./components/HeavyComponent.vue')
);

// 3. 动态导入
button.addEventListener('click', async () => {
  const module = await import('./heavy-module.js');
  module.doSomething();
});

// 4. Webpack 魔法注释
import(
  /* webpackChunkName: "my-chunk" */
  /* webpackPrefetch: true */
  './module.js'
);
```

### 图片优化

```javascript
// 1. 图片懒加载
<img
  src="placeholder.jpg"
  data-src="real-image.jpg"
  class="lazy"
/>

<script>
const images = document.querySelectorAll('.lazy');

const imageObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.classList.remove('lazy');
      imageObserver.unobserve(img);
    }
  });
});

images.forEach(img => imageObserver.observe(img));
</script>

// 2. 响应式图片
<picture>
  <source media="(min-width: 1200px)" srcset="large.jpg">
  <source media="(min-width: 768px)" srcset="medium.jpg">
  <img src="small.jpg" alt="responsive image">
</picture>

// 3. WebP 格式
<picture>
  <source type="image/webp" srcset="image.webp">
  <source type="image/jpeg" srcset="image.jpg">
  <img src="image.jpg" alt="image">
</picture>

// 4. 图片压缩配置
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|jpeg|gif)$/,
        use: [
          {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: { quality: 65 },
              pngquant: { quality: [0.65, 0.90] }
            }
          }
        ]
      }
    ]
  }
};
```

### CDN 加速

```javascript
// 1. 静态资源 CDN
module.exports = {
  output: {
    publicPath: 'https://cdn.example.com/'
  }
};

// 2. 第三方库使用 CDN
<script src="https://cdn.jsdelivr.net/npm/vue@3.3.4/dist/vue.global.js"></script>

// Webpack 配置
module.exports = {
  externals: {
    vue: 'Vue',
    'vue-router': 'VueRouter'
  }
};
```

---

## 运行时性能优化

> 💡 **详细内容请参考**: [运行时性能优化详解](./runtime.md) - 包含 JavaScript 代码优化、内存优化、DOM 操作优化、性能监控等完整内容

### 虚拟列表

```vue
<template>
  <div class="virtual-list" @scroll="handleScroll">
    <div class="list-phantom" :style="{ height: totalHeight + 'px' }"></div>
    <div class="list-content" :style="{ transform: `translateY(${offset}px)` }">
      <div
        v-for="item in visibleData"
        :key="item.id"
        class="list-item"
        :style="{ height: itemHeight + 'px' }"
      >
        {{ item.text }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  data: Array,
  itemHeight: { type: Number, default: 50 },
  visibleCount: { type: Number, default: 20 }
});

const scrollTop = ref(0);

// 总高度
const totalHeight = computed(() => props.data.length * props.itemHeight);

// 起始索引
const startIndex = computed(() => Math.floor(scrollTop.value / props.itemHeight));

// 结束索引
const endIndex = computed(() => startIndex.value + props.visibleCount);

// 可见数据
const visibleData = computed(() =>
  props.data.slice(startIndex.value, endIndex.value)
);

// 偏移量
const offset = computed(() => startIndex.value * props.itemHeight);

function handleScroll(e) {
  scrollTop.value = e.target.scrollTop;
}
</script>
```

### 防抖和节流

```javascript
// 防抖: 延迟执行,多次触发只执行最后一次
function debounce(fn, delay = 300) {
  let timer = null;

  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// 使用场景: 搜索输入
const search = debounce((keyword) => {
  console.log('搜索:', keyword);
}, 500);

input.addEventListener('input', (e) => search(e.target.value));

// 节流: 固定时间内只执行一次
function throttle(fn, delay = 300) {
  let lastTime = 0;

  return function(...args) {
    const now = Date.now();
    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    }
  };
}

// 使用场景: 滚动事件
const handleScroll = throttle(() => {
  console.log('滚动位置:', window.scrollY);
}, 200);

window.addEventListener('scroll', handleScroll);
```

### 减少重排重绘

```javascript
// ❌ 多次重排重绘
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.style.width = '100px';
  div.style.height = '100px';
  document.body.appendChild(div);
}

// ✅ 批量操作,减少重排
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.style.width = '100px';
  div.style.height = '100px';
  fragment.appendChild(div);
}
document.body.appendChild(fragment);

// ✅ 使用 transform 代替 top/left
// ❌ 触发重排
element.style.left = '100px';

// ✅ 只触发重绘
element.style.transform = 'translateX(100px)';

// ✅ 读写分离
// ❌ 读写交替,触发强制重排
div.style.width = div.offsetWidth + 10 + 'px';
div.style.height = div.offsetHeight + 10 + 'px';

// ✅ 先读后写
const width = div.offsetWidth;
const height = div.offsetHeight;
div.style.width = width + 10 + 'px';
div.style.height = height + 10 + 'px';
```

### requestAnimationFrame

```javascript
// ❌ 使用 setInterval
let left = 0;
setInterval(() => {
  left += 1;
  element.style.left = left + 'px';
}, 16);

// ✅ 使用 requestAnimationFrame
let left = 0;
function animate() {
  left += 1;
  element.style.left = left + 'px';

  if (left < 100) {
    requestAnimationFrame(animate);
  }
}
requestAnimationFrame(animate);
```

---

## 性能监控

### Performance API

```javascript
// 1. 获取性能指标
const perfData = window.performance.timing;

const pageLoadTime = perfData.loadEventEnd - perfData.navigationStart;
const domReadyTime = perfData.domContentLoadedEventEnd - perfData.navigationStart;
const firstPaintTime = perfData.responseEnd - perfData.fetchStart;

console.log('页面加载时间:', pageLoadTime);
console.log('DOM 解析时间:', domReadyTime);

// 2. 监控资源加载
const resources = window.performance.getEntriesByType('resource');
resources.forEach(resource => {
  console.log(`${resource.name}: ${resource.duration}ms`);
});

// 3. 监控 FCP、LCP
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('LCP:', entry.renderTime || entry.loadTime);
  }
}).observe({ entryTypes: ['largest-contentful-paint'] });

// 4. 自定义性能标记
performance.mark('start');
// ... 执行代码
performance.mark('end');
performance.measure('操作耗时', 'start', 'end');

const measures = performance.getEntriesByType('measure');
console.log(measures[0].duration);
```

### 错误监控

```javascript
// 全局错误捕获
window.addEventListener('error', (event) => {
  console.error('错误:', event.error);

  // 上报错误
  fetch('/api/log', {
    method: 'POST',
    body: JSON.stringify({
      message: event.error.message,
      stack: event.error.stack,
      url: window.location.href,
      userAgent: navigator.userAgent
    })
  });
});

// Promise 错误捕获
window.addEventListener('unhandledrejection', (event) => {
  console.error('未处理的 Promise 错误:', event.reason);
});

// Vue 错误处理
app.config.errorHandler = (err, instance, info) => {
  console.error('Vue 错误:', err, info);
};
```

---

## 实战案例

### 首屏优化

```javascript
// 1. 关键 CSS 内联
<style>
  /* 首屏关键样式内联到 HTML */
  .header { ... }
  .hero { ... }
</style>

// 2. 预加载关键资源
<link rel="preload" href="critical.js" as="script">
<link rel="preload" href="hero-image.jpg" as="image">

// 3. DNS 预解析
<link rel="dns-prefetch" href="https://api.example.com">

// 4. 骨架屏
<div class="skeleton">
  <div class="skeleton-header"></div>
  <div class="skeleton-content"></div>
</div>

// 5. SSR 服务端渲染
// server.js
import { createSSRApp } from 'vue';
import { renderToString } from 'vue/server-renderer';

app.get('*', async (req, res) => {
  const app = createSSRApp({...});
  const html = await renderToString(app);
  res.send(`
    <!DOCTYPE html>
    <html>
      <body>
        <div id="app">${html}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `);
});
```

### 长列表优化

```javascript
// 1. 分页加载
const pageSize = 20;
let currentPage = 1;

async function loadMore() {
  const data = await fetchData(currentPage, pageSize);
  list.value.push(...data);
  currentPage++;
}

// 2. 无限滚动
const observer = new IntersectionObserver((entries) => {
  if (entries[0].isIntersecting) {
    loadMore();
  }
});

observer.observe(loadMoreTrigger);

// 3. 虚拟滚动(见上文虚拟列表)
```

---

## 总结

### 优化清单

**加载优化**:
- ✅ 资源压缩(Gzip、Brotli)
- ✅ 代码分割和懒加载
- ✅ 图片优化(WebP、懒加载、CDN)
- ✅ HTTP 缓存(强缓存、协商缓存)
- ✅ 预加载、预连接

**运行时优化**:
- ✅ 虚拟列表
- ✅ 防抖节流
- ✅ 减少重排重绘
- ✅ 使用 Web Worker
- ✅ requestAnimationFrame

**构建优化**:
- ✅ Tree Shaking
- ✅ 压缩和混淆
- ✅ 打包分析
- ✅ 按需引入

**监控**:
- ✅ Performance API
- ✅ 错误监控
- ✅ 用户行为追踪

### 面试加分项
- 有实际的性能优化案例和数据对比
- 了解性能指标的具体含义
- 能手写虚拟列表、防抖节流等工具
- 熟悉性能监控工具(Lighthouse、WebPageTest)

---

## 高频面试题

### 1. 前端性能优化有哪些手段？

**一句话答案**: 从加载、渲染、运行三个阶段优化，包括资源压缩、代码分割、懒加载、缓存策略、减少重排重绘等。

#### 详细解答

性能优化可以分为三个维度：

**1. 加载性能优化**

```javascript
// (1) 资源压缩 - Webpack 配置
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,  // 生产环境移除 console
            pure_funcs: ['console.log']
          }
        }
      })
    ],
    // 代码分割
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        },
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    }
  }
};

// (2) 路由懒加载
const routes = [
  {
    path: '/home',
    component: () => import(/* webpackChunkName: "home" */ './views/Home.vue')
  },
  {
    path: '/dashboard',
    component: () => import(/* webpackChunkName: "dashboard" */ './views/Dashboard.vue')
  }
];

// (3) 图片懒加载
const lazyLoadImages = () => {
  const images = document.querySelectorAll('img[data-src]');
  const imageObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
        imageObserver.unobserve(img);
      }
    });
  });

  images.forEach(img => imageObserver.observe(img));
};

// (4) 预加载关键资源
// HTML 中添加
<link rel="preload" href="critical.js" as="script">
<link rel="preconnect" href="https://cdn.example.com">
<link rel="dns-prefetch" href="https://api.example.com">
```

**2. 渲染性能优化**

```javascript
// (1) 减少重排重绘
// 错误做法 - 频繁操作 DOM
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  document.body.appendChild(div);  // 触发 1000 次重排
}

// 正确做法 - 使用 DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  fragment.appendChild(div);
}
document.body.appendChild(fragment);  // 只触发 1 次重排

// (2) 使用 transform 代替 top/left
// 错误 - 触发重排
element.style.top = '100px';

// 正确 - 只触发合成
element.style.transform = 'translateY(100px)';

// (3) 批量读写 DOM
// 错误 - 读写交替
const width1 = div1.offsetWidth;
div1.style.width = width1 + 10 + 'px';
const width2 = div2.offsetWidth;
div2.style.width = width2 + 10 + 'px';

// 正确 - 先读后写
const width1 = div1.offsetWidth;
const width2 = div2.offsetWidth;
div1.style.width = width1 + 10 + 'px';
div2.style.width = width2 + 10 + 'px';
```

**3. 运行时性能优化**

```javascript
// (1) 防抖 - 搜索输入
function debounce(fn, delay = 300) {
  let timer = null;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const handleSearch = debounce((keyword) => {
  fetch(`/api/search?q=${keyword}`);
}, 500);

// (2) 节流 - 滚动事件
function throttle(fn, delay = 300) {
  let lastTime = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    }
  };
}

const handleScroll = throttle(() => {
  console.log('滚动位置:', window.scrollY);
}, 200);

// (3) 虚拟列表 - 长列表渲染
class VirtualList {
  constructor(options) {
    this.data = options.data;
    this.itemHeight = options.itemHeight;
    this.visibleCount = options.visibleCount;
    this.container = options.container;
    this.scrollTop = 0;

    this.init();
  }

  init() {
    this.container.style.height = this.itemHeight * this.visibleCount + 'px';
    this.container.style.overflow = 'auto';

    this.phantom = document.createElement('div');
    this.phantom.style.height = this.data.length * this.itemHeight + 'px';

    this.content = document.createElement('div');
    this.content.style.transform = 'translateY(0)';

    this.container.appendChild(this.phantom);
    this.container.appendChild(this.content);

    this.container.addEventListener('scroll', () => this.handleScroll());
    this.render();
  }

  handleScroll() {
    this.scrollTop = this.container.scrollTop;
    this.render();
  }

  render() {
    const startIndex = Math.floor(this.scrollTop / this.itemHeight);
    const endIndex = startIndex + this.visibleCount;
    const visibleData = this.data.slice(startIndex, endIndex);

    this.content.innerHTML = visibleData.map((item, index) => `
      <div style="height: ${this.itemHeight}px">
        ${item.text}
      </div>
    `).join('');

    this.content.style.transform = `translateY(${startIndex * this.itemHeight}px)`;
  }
}

// (4) Web Worker - 复杂计算
// main.js
const worker = new Worker('worker.js');
worker.postMessage({ data: largeData });
worker.onmessage = (e) => {
  console.log('计算结果:', e.data);
};

// worker.js
self.onmessage = (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};
```

**4. 缓存策略**

```javascript
// Service Worker 缓存
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) => {
      return cache.addAll([
        '/',
        '/style.css',
        '/script.js',
        '/image.jpg'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});

// HTTP 缓存设置
// 强缓存
Cache-Control: max-age=31536000  // 1 年

// 协商缓存
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Wed, 21 Oct 2023 07:28:00 GMT
```

#### 面试口语化回答模板

"前端性能优化我一般从三个方面来做：

第一是**加载性能**。我会做资源压缩，比如使用 Gzip 或 Brotli 压缩文件；做代码分割，通过路由懒加载和动态 import 按需加载模块；图片方面会使用 WebP 格式、懒加载和 CDN 加速；还会配置 HTTP 缓存策略，利用强缓存和协商缓存减少请求。

第二是**渲染性能**。我会减少重排重绘，比如使用 DocumentFragment 批量操作 DOM，用 transform 代替 top/left 定位，避免读写 DOM 交替操作。对于长列表，我会使用虚拟列表只渲染可视区域。

第三是**运行时性能**。对于频繁触发的事件，我会使用防抖和节流来优化，比如搜索输入用防抖，滚动事件用节流。对于复杂计算，我会考虑用 Web Worker 放到后台线程执行。

在我之前的项目中，通过这些优化手段，首屏加载时间从 5 秒降到了 2 秒以内，LCP 指标也达到了 2.5 秒的标准。"

---

### 2. 首屏加载优化怎么做？

**一句话答案**: 减小首屏资源体积、加快资源加载速度、提前请求关键资源、使用 SSR/SSG，配合骨架屏提升体验。

#### 详细解答

**1. 减小资源体积**

```javascript
// (1) 路由懒加载
const router = createRouter({
  routes: [
    {
      path: '/',
      name: 'Home',
      component: () => import('./views/Home.vue')  // 首屏不加载
    },
    {
      path: '/about',
      component: () => import('./views/About.vue')
    }
  ]
});

// (2) 组件按需引入
// 错误 - 全量引入
import ElementPlus from 'element-plus';
app.use(ElementPlus);

// 正确 - 按需引入
import { ElButton, ElInput } from 'element-plus';
app.component('ElButton', ElButton);
app.component('ElInput', ElInput);

// 配合 unplugin-auto-import 自动按需引入
// vite.config.js
import AutoImport from 'unplugin-auto-import/vite';
import Components from 'unplugin-vue-components/vite';
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers';

export default {
  plugins: [
    AutoImport({
      resolvers: [ElementPlusResolver()]
    }),
    Components({
      resolvers: [ElementPlusResolver()]
    })
  ]
};

// (3) Tree Shaking
// package.json
{
  "sideEffects": false  // 开启 Tree Shaking
}

// 只导入需要的方法
import { debounce, throttle } from 'lodash-es';  // 正确
import _ from 'lodash';  // 错误 - 会打包整个库
```

**2. 资源加载优化**

```html
<!DOCTYPE html>
<html>
<head>
  <!-- (1) DNS 预解析 -->
  <link rel="dns-prefetch" href="https://api.example.com">
  <link rel="dns-prefetch" href="https://cdn.example.com">

  <!-- (2) 预连接 -->
  <link rel="preconnect" href="https://fonts.googleapis.com">

  <!-- (3) 预加载关键资源 -->
  <link rel="preload" href="critical.js" as="script">
  <link rel="preload" href="hero-image.jpg" as="image">
  <link rel="preload" href="fonts/main.woff2" as="font" type="font/woff2" crossorigin>

  <!-- (4) 内联关键 CSS -->
  <style>
    /* 首屏关键样式直接内联 */
    .header { height: 60px; background: #fff; }
    .hero { min-height: 400px; }
  </style>

  <!-- (5) 异步加载非关键 CSS -->
  <link rel="preload" href="non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="non-critical.css"></noscript>
</head>
<body>
  <!-- (6) 骨架屏 -->
  <div id="app">
    <div class="skeleton">
      <div class="skeleton-header"></div>
      <div class="skeleton-content">
        <div class="skeleton-line"></div>
        <div class="skeleton-line"></div>
        <div class="skeleton-line"></div>
      </div>
    </div>
  </div>

  <!-- (7) defer 加载脚本 -->
  <script defer src="main.js"></script>
</body>
</html>
```

```css
/* 骨架屏样式 */
.skeleton {
  animation: pulse 1.5s infinite;
}

.skeleton-header {
  height: 60px;
  background: #f0f0f0;
  border-radius: 4px;
  margin-bottom: 20px;
}

.skeleton-line {
  height: 20px;
  background: #f0f0f0;
  border-radius: 4px;
  margin-bottom: 10px;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
```

**3. SSR/SSG 服务端渲染**

```javascript
// Nuxt.js SSR 示例
// nuxt.config.js
export default {
  ssr: true,  // 开启 SSR

  // 性能优化配置
  render: {
    bundleRenderer: {
      shouldPreload: (file, type) => {
        // 预加载关键资源
        return ['script', 'style', 'font'].includes(type);
      }
    },
    http2: {
      push: true  // HTTP/2 服务器推送
    }
  },

  // 静态站点生成
  target: 'static',
  generate: {
    routes: ['/about', '/contact']
  }
};

// pages/index.vue
export default {
  // 服务端预取数据
  async asyncData({ $axios }) {
    const data = await $axios.$get('/api/data');
    return { data };
  },

  // 页面元信息
  head() {
    return {
      title: '首页',
      meta: [
        { hid: 'description', name: 'description', content: '页面描述' }
      ]
    };
  }
};
```

**4. 图片优化**

```javascript
// (1) 图片懒加载
const lazyLoadObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.classList.add('loaded');
      lazyLoadObserver.unobserve(img);
    }
  });
}, {
  rootMargin: '50px'  // 提前 50px 开始加载
});

document.querySelectorAll('img[data-src]').forEach(img => {
  lazyLoadObserver.observe(img);
});

// (2) 响应式图片
<picture>
  <source
    media="(min-width: 1200px)"
    srcset="large.webp"
    type="image/webp">
  <source
    media="(min-width: 768px)"
    srcset="medium.webp"
    type="image/webp">
  <source
    media="(min-width: 1200px)"
    srcset="large.jpg">
  <source
    media="(min-width: 768px)"
    srcset="medium.jpg">
  <img src="small.jpg" alt="responsive image" loading="lazy">
</picture>

// (3) 渐进式图片加载
<img
  src="placeholder-blur.jpg"  // 10KB 模糊占位图
  data-src="full-image.jpg"   // 完整图片
  class="progressive-image"
  alt="progressive">

<style>
.progressive-image {
  filter: blur(10px);
  transition: filter 0.3s;
}

.progressive-image.loaded {
  filter: blur(0);
}
</style>
```

**5. CDN 和缓存**

```javascript
// Webpack 配置 CDN
module.exports = {
  output: {
    publicPath: process.env.NODE_ENV === 'production'
      ? 'https://cdn.example.com/'
      : '/'
  },

  // 外部依赖使用 CDN
  externals: {
    vue: 'Vue',
    'vue-router': 'VueRouter',
    axios: 'axios'
  }
};

// HTML 引入 CDN
<script src="https://cdn.jsdelivr.net/npm/vue@3.3.4/dist/vue.global.prod.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue-router@4.2.4/dist/vue-router.global.prod.js"></script>

// Service Worker 缓存策略
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open('v1').then((cache) => {
      return cache.match(event.request).then((response) => {
        // 缓存优先策略
        if (response) {
          return response;
        }

        // 网络请求
        return fetch(event.request).then((networkResponse) => {
          // 缓存新请求
          cache.put(event.request, networkResponse.clone());
          return networkResponse;
        });
      });
    })
  );
});
```

**6. 性能监控**

```javascript
// 监控首屏加载时间
window.addEventListener('load', () => {
  const perfData = performance.timing;
  const pageLoadTime = perfData.loadEventEnd - perfData.navigationStart;
  const domReadyTime = perfData.domContentLoadedEventEnd - perfData.navigationStart;

  // 上报性能数据
  fetch('/api/performance', {
    method: 'POST',
    body: JSON.stringify({
      pageLoadTime,
      domReadyTime,
      url: window.location.href,
      userAgent: navigator.userAgent
    })
  });
});

// 监控 LCP
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];

  console.log('LCP:', lastEntry.renderTime || lastEntry.loadTime);

  // 上报 LCP
  fetch('/api/lcp', {
    method: 'POST',
    body: JSON.stringify({
      lcp: lastEntry.renderTime || lastEntry.loadTime,
      element: lastEntry.element?.tagName
    })
  });
}).observe({ entryTypes: ['largest-contentful-paint'] });
```

#### 面试口语化回答模板

"首屏加载优化我主要从四个方面来做：

第一是**减小资源体积**。我会做路由懒加载，让非首屏的页面不在首屏加载；使用按需引入，比如 Element Plus 只引入需要的组件；开启 Tree Shaking 去除未使用的代码。

第二是**加快资源加载**。我会使用 DNS 预解析和预连接，提前建立连接；对关键资源使用 preload 预加载；把首屏关键 CSS 内联到 HTML 中；非关键 CSS 异步加载。我还会配置 CDN 加速静态资源访问，使用强缓存减少重复请求。

第三是**SSR 服务端渲染**。对于需要 SEO 的页面，我会使用 Nuxt.js 做服务端渲染，让用户直接看到渲染好的 HTML，不用等 JS 执行完才能看到内容。如果页面是静态的，还可以用静态站点生成(SSG)，性能会更好。

第四是**提升用户体验**。在页面加载时展示骨架屏，让用户感觉页面在快速响应；图片使用懒加载和渐进式加载，优先加载可视区域的图片。

在我之前的项目中，通过这些优化，首屏 LCP 从 4 秒降到了 1.8 秒，用户体验有明显提升。"

---

### 3. 如何监控前端性能？

**一句话答案**: 使用 Performance API 收集性能指标，配合 PerformanceObserver 监控 Core Web Vitals，结合错误监控和用户行为追踪，最后上报到监控平台分析。

#### 详细解答

**1. Performance API 基础监控**

```javascript
// (1) 页面加载性能
class PerformanceMonitor {
  constructor() {
    this.metrics = {};
  }

  // 收集基础性能指标
  collectPageMetrics() {
    if (!window.performance || !window.performance.timing) {
      return;
    }

    const timing = performance.timing;

    // 各阶段耗时
    this.metrics = {
      // DNS 查询耗时
      dnsTime: timing.domainLookupEnd - timing.domainLookupStart,

      // TCP 连接耗时
      tcpTime: timing.connectEnd - timing.connectStart,

      // SSL 握手耗时
      sslTime: timing.secureConnectionStart
        ? timing.connectEnd - timing.secureConnectionStart
        : 0,

      // 请求耗时
      requestTime: timing.responseEnd - timing.requestStart,

      // 响应耗时
      responseTime: timing.responseEnd - timing.responseStart,

      // DOM 解析耗时
      domParseTime: timing.domInteractive - timing.domLoading,

      // 资源加载耗时
      resourceLoadTime: timing.loadEventStart - timing.domContentLoadedEventEnd,

      // 首屏时间
      firstScreenTime: timing.domContentLoadedEventEnd - timing.navigationStart,

      // 页面完全加载时间
      pageLoadTime: timing.loadEventEnd - timing.navigationStart,

      // 白屏时间
      whiteScreenTime: timing.responseStart - timing.navigationStart
    };

    return this.metrics;
  }

  // 收集资源加载性能
  collectResourceMetrics() {
    const resources = performance.getEntriesByType('resource');

    const resourceMetrics = resources.map(resource => ({
      name: resource.name,
      type: resource.initiatorType,
      duration: resource.duration,
      size: resource.transferSize,
      protocol: resource.nextHopProtocol,
      startTime: resource.startTime
    }));

    // 统计各类型资源
    const summary = {
      js: [],
      css: [],
      img: [],
      xhr: [],
      other: []
    };

    resourceMetrics.forEach(resource => {
      const type = resource.type;
      if (type === 'script') {
        summary.js.push(resource);
      } else if (type === 'link' || type === 'css') {
        summary.css.push(resource);
      } else if (type === 'img') {
        summary.img.push(resource);
      } else if (type === 'xmlhttprequest' || type === 'fetch') {
        summary.xhr.push(resource);
      } else {
        summary.other.push(resource);
      }
    });

    return {
      resources: resourceMetrics,
      summary,
      totalCount: resources.length,
      totalSize: resourceMetrics.reduce((sum, r) => sum + r.size, 0),
      totalDuration: Math.max(...resourceMetrics.map(r => r.startTime + r.duration))
    };
  }
}

// 使用
window.addEventListener('load', () => {
  const monitor = new PerformanceMonitor();
  const pageMetrics = monitor.collectPageMetrics();
  const resourceMetrics = monitor.collectResourceMetrics();

  console.log('页面性能:', pageMetrics);
  console.log('资源性能:', resourceMetrics);

  // 上报数据
  sendToAnalytics({ pageMetrics, resourceMetrics });
});
```

**2. Core Web Vitals 监控**

```javascript
// (1) LCP - 最大内容绘制
class WebVitalsMonitor {
  constructor() {
    this.metrics = {};
    this.init();
  }

  init() {
    this.observeLCP();
    this.observeFID();
    this.observeCLS();
    this.observeFCP();
    this.observeTTFB();
  }

  // 监控 LCP
  observeLCP() {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1];

      this.metrics.lcp = {
        value: lastEntry.renderTime || lastEntry.loadTime,
        element: lastEntry.element?.tagName,
        url: lastEntry.url,
        rating: this.getRating('lcp', lastEntry.renderTime || lastEntry.loadTime)
      };

      console.log('LCP:', this.metrics.lcp);
    });

    observer.observe({ entryTypes: ['largest-contentful-paint'] });
  }

  // 监控 FID - 首次输入延迟
  observeFID() {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();

      entries.forEach(entry => {
        this.metrics.fid = {
          value: entry.processingStart - entry.startTime,
          eventType: entry.name,
          rating: this.getRating('fid', entry.processingStart - entry.startTime)
        };

        console.log('FID:', this.metrics.fid);
      });
    });

    observer.observe({ entryTypes: ['first-input'] });
  }

  // 监控 CLS - 累积布局偏移
  observeCLS() {
    let clsValue = 0;

    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      }

      this.metrics.cls = {
        value: clsValue,
        rating: this.getRating('cls', clsValue)
      };

      console.log('CLS:', this.metrics.cls);
    });

    observer.observe({ entryTypes: ['layout-shift'] });
  }

  // 监控 FCP - 首次内容绘制
  observeFCP() {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();

      entries.forEach(entry => {
        if (entry.name === 'first-contentful-paint') {
          this.metrics.fcp = {
            value: entry.startTime,
            rating: this.getRating('fcp', entry.startTime)
          };

          console.log('FCP:', this.metrics.fcp);
        }
      });
    });

    observer.observe({ entryTypes: ['paint'] });
  }

  // 监控 TTFB - 首字节时间
  observeTTFB() {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();

      entries.forEach(entry => {
        this.metrics.ttfb = {
          value: entry.responseStart - entry.requestStart,
          rating: this.getRating('ttfb', entry.responseStart - entry.requestStart)
        };

        console.log('TTFB:', this.metrics.ttfb);
      });
    });

    observer.observe({ entryTypes: ['navigation'] });
  }

  // 评级
  getRating(metric, value) {
    const thresholds = {
      lcp: { good: 2500, poor: 4000 },
      fid: { good: 100, poor: 300 },
      cls: { good: 0.1, poor: 0.25 },
      fcp: { good: 1800, poor: 3000 },
      ttfb: { good: 800, poor: 1800 }
    };

    const threshold = thresholds[metric];
    if (value <= threshold.good) return 'good';
    if (value <= threshold.poor) return 'needs-improvement';
    return 'poor';
  }

  // 获取所有指标
  getMetrics() {
    return this.metrics;
  }

  // 上报数据
  report() {
    // 页面卸载时上报
    window.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.sendBeacon();
      }
    });

    window.addEventListener('pagehide', () => {
      this.sendBeacon();
    });
  }

  sendBeacon() {
    const data = JSON.stringify({
      metrics: this.metrics,
      url: window.location.href,
      userAgent: navigator.userAgent,
      timestamp: Date.now()
    });

    navigator.sendBeacon('/api/performance', data);
  }
}

// 使用
const vitalsMonitor = new WebVitalsMonitor();
vitalsMonitor.report();
```

**3. 错误监控**

```javascript
class ErrorMonitor {
  constructor() {
    this.errors = [];
    this.init();
  }

  init() {
    this.captureJsError();
    this.capturePromiseError();
    this.captureResourceError();
    this.captureVueError();
  }

  // JavaScript 错误
  captureJsError() {
    window.addEventListener('error', (event) => {
      const error = {
        type: 'javascript',
        message: event.error?.message || event.message,
        stack: event.error?.stack,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        timestamp: Date.now()
      };

      this.errors.push(error);
      this.reportError(error);
    }, true);
  }

  // Promise 错误
  capturePromiseError() {
    window.addEventListener('unhandledrejection', (event) => {
      const error = {
        type: 'promise',
        message: event.reason?.message || event.reason,
        stack: event.reason?.stack,
        timestamp: Date.now()
      };

      this.errors.push(error);
      this.reportError(error);
    });
  }

  // 资源加载错误
  captureResourceError() {
    window.addEventListener('error', (event) => {
      const target = event.target || event.srcElement;

      if (target !== window && target.tagName) {
        const error = {
          type: 'resource',
          tagName: target.tagName,
          url: target.src || target.href,
          timestamp: Date.now()
        };

        this.errors.push(error);
        this.reportError(error);
      }
    }, true);
  }

  // Vue 错误
  captureVueError() {
    // 在 Vue 应用中配置
    // app.config.errorHandler = (err, instance, info) => {
    //   const error = {
    //     type: 'vue',
    //     message: err.message,
    //     stack: err.stack,
    //     info: info,
    //     componentName: instance?.$options?.name,
    //     timestamp: Date.now()
    //   };
    //
    //   this.errors.push(error);
    //   this.reportError(error);
    // };
  }

  // 上报错误
  reportError(error) {
    const data = {
      ...error,
      url: window.location.href,
      userAgent: navigator.userAgent,
      viewport: {
        width: window.innerWidth,
        height: window.innerHeight
      }
    };

    // 使用 sendBeacon 保证数据发送
    navigator.sendBeacon('/api/error', JSON.stringify(data));

    // 或使用 fetch
    fetch('/api/error', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
      keepalive: true  // 页面卸载后仍然发送
    });
  }
}

// 使用
const errorMonitor = new ErrorMonitor();
```

**4. 用户行为监控**

```javascript
class UserBehaviorMonitor {
  constructor() {
    this.events = [];
    this.init();
  }

  init() {
    this.trackPageView();
    this.trackClick();
    this.trackScroll();
    this.trackStayTime();
  }

  // 页面访问
  trackPageView() {
    const pageView = {
      type: 'pageview',
      url: window.location.href,
      referrer: document.referrer,
      timestamp: Date.now()
    };

    this.events.push(pageView);
    this.report(pageView);
  }

  // 点击事件
  trackClick() {
    document.addEventListener('click', (e) => {
      const target = e.target;
      const event = {
        type: 'click',
        tagName: target.tagName,
        className: target.className,
        id: target.id,
        text: target.innerText?.slice(0, 50),
        x: e.clientX,
        y: e.clientY,
        timestamp: Date.now()
      };

      this.events.push(event);
    });
  }

  // 滚动深度
  trackScroll() {
    let maxScrollDepth = 0;

    const throttledScroll = this.throttle(() => {
      const scrollDepth = Math.round(
        (window.scrollY / (document.documentElement.scrollHeight - window.innerHeight)) * 100
      );

      if (scrollDepth > maxScrollDepth) {
        maxScrollDepth = scrollDepth;

        // 达到特定深度时上报
        if ([25, 50, 75, 100].includes(scrollDepth)) {
          const event = {
            type: 'scroll',
            depth: scrollDepth,
            timestamp: Date.now()
          };

          this.events.push(event);
          this.report(event);
        }
      }
    }, 1000);

    window.addEventListener('scroll', throttledScroll);
  }

  // 页面停留时间
  trackStayTime() {
    const startTime = Date.now();

    const reportStayTime = () => {
      const stayTime = Date.now() - startTime;
      const event = {
        type: 'stay-time',
        duration: stayTime,
        url: window.location.href,
        timestamp: Date.now()
      };

      this.report(event);
    };

    window.addEventListener('beforeunload', reportStayTime);
    window.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        reportStayTime();
      }
    });
  }

  // 工具函数
  throttle(fn, delay) {
    let lastTime = 0;
    return function(...args) {
      const now = Date.now();
      if (now - lastTime >= delay) {
        fn.apply(this, args);
        lastTime = now;
      }
    };
  }

  // 上报数据
  report(event) {
    const data = {
      ...event,
      sessionId: this.getSessionId(),
      userId: this.getUserId()
    };

    navigator.sendBeacon('/api/behavior', JSON.stringify(data));
  }

  getSessionId() {
    // 从 sessionStorage 获取或生成
    let sessionId = sessionStorage.getItem('sessionId');
    if (!sessionId) {
      sessionId = `${Date.now()}_${Math.random()}`;
      sessionStorage.setItem('sessionId', sessionId);
    }
    return sessionId;
  }

  getUserId() {
    // 从 localStorage 或 cookie 获取
    return localStorage.getItem('userId') || 'anonymous';
  }
}

// 使用
const behaviorMonitor = new UserBehaviorMonitor();
```

**5. 完整监控方案**

```javascript
// 综合监控类
class PerformanceTracker {
  constructor(options = {}) {
    this.appId = options.appId;
    this.reportUrl = options.reportUrl || '/api/monitor';
    this.batchSize = options.batchSize || 10;
    this.reportInterval = options.reportInterval || 5000;

    this.dataQueue = [];
    this.timer = null;

    this.init();
  }

  init() {
    // 初始化各种监控
    this.performanceMonitor = new PerformanceMonitor();
    this.webVitalsMonitor = new WebVitalsMonitor();
    this.errorMonitor = new ErrorMonitor();
    this.behaviorMonitor = new UserBehaviorMonitor();

    // 启动定时上报
    this.startAutoReport();

    // 页面卸载时上报
    this.reportOnUnload();
  }

  // 添加数据到队列
  addData(data) {
    this.dataQueue.push({
      ...data,
      appId: this.appId,
      url: window.location.href,
      timestamp: Date.now()
    });

    // 达到批量大小立即上报
    if (this.dataQueue.length >= this.batchSize) {
      this.report();
    }
  }

  // 定时上报
  startAutoReport() {
    this.timer = setInterval(() => {
      if (this.dataQueue.length > 0) {
        this.report();
      }
    }, this.reportInterval);
  }

  // 上报数据
  report() {
    if (this.dataQueue.length === 0) return;

    const data = [...this.dataQueue];
    this.dataQueue = [];

    // 使用 sendBeacon 保证可靠性
    const blob = new Blob([JSON.stringify(data)], {
      type: 'application/json'
    });
    navigator.sendBeacon(this.reportUrl, blob);
  }

  // 页面卸载时上报
  reportOnUnload() {
    window.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.report();
      }
    });

    window.addEventListener('pagehide', () => {
      this.report();
    });
  }

  // 销毁
  destroy() {
    clearInterval(this.timer);
    this.report();
  }
}

// 使用
const tracker = new PerformanceTracker({
  appId: 'my-app',
  reportUrl: 'https://monitor.example.com/api/report',
  batchSize: 20,
  reportInterval: 10000
});
```

#### 面试口语化回答模板

"前端性能监控我主要从四个维度来做：

第一是**性能指标监控**。我会使用 Performance API 收集页面加载的各个阶段的耗时，比如 DNS 查询、TCP 连接、资源加载等。还会使用 PerformanceObserver 监控 Core Web Vitals，也就是 Google 提出的三个核心指标：LCP(最大内容绘制)、FID(首次输入延迟)、CLS(累积布局偏移)。这些指标能很好地反映用户体验。

第二是**资源监控**。通过 Performance API 的 getEntriesByType 方法，可以获取所有资源的加载时间、大小、协议等信息，统计 JS、CSS、图片等各类资源的加载情况，找出加载慢的资源进行优化。

第三是**错误监控**。我会监听 window 的 error 事件捕获 JavaScript 错误，监听 unhandledrejection 事件捕获 Promise 错误，还会捕获资源加载失败的错误。对于 Vue 项目，还会配置全局的 errorHandler 捕获组件错误。捕获到错误后，会记录错误信息、堆栈、发生时间等，上报到服务端。

第四是**用户行为监控**。我会追踪用户的页面访问、点击、滚动等行为，记录页面停留时间、滚动深度等数据，这些数据可以帮助我们了解用户如何使用产品，哪些功能使用频率高。

在数据上报方面，我会使用 sendBeacon API，它在页面卸载时也能保证数据发送成功。还会做批量上报和定时上报，避免频繁请求影响性能。

在我之前的项目中，通过这套监控体系，我们能及时发现性能问题和错误，LCP 超过 3 秒的用户占比从 30% 降到了 5%，错误率也下降了 50%。"
