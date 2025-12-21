# 前端安全 (Web Security)

## 1. XSS (Cross-Site Scripting) 跨站脚本攻击

### 原理

攻击者在目标网站上注入恶意脚本 (通常是 JS),当用户访问时,脚本在用户浏览器中运行,窃取 Cookie、Token 或伪造用户操作。

### 分类

1.  **存储型 (Stored)**: 恶意脚本存储在数据库中 (如评论区)。危害最大,所有访问者都会中招。
2.  **反射型 (Reflected)**: 恶意脚本存在于 URL 参数中。服务器接收参数并反射回页面。需要诱导用户点击链接。
3.  **DOM 型**: 纯前端漏洞。JS 取出 URL 参数或输入值,直接操作 DOM (如 `innerHTML`, `document.write`) 导致执行。

### 防御

- **输入过滤**: 过滤特殊字符 (`<`, `>`, `&`, `"`, `'`)。
- **输出转义 (Encoding)**: 在输出到 HTML 时进行转义 (如将 `<` 转为 `&lt;`)。现代框架 (React/Vue) 默认已做相关处理。
  - _注意_: 慎用 `dangerouslySetInnerHTML` 或 `v-html`。
- **HttpOnly Cookie**: 禁止 JS 读取敏感 Cookie (防止 Session ID 被窃取)。
- **CSP (Content Security Policy)**: 浏览器白名单机制,限制加载外部资源和行内脚本执行。

---

## 2. CSRF (Cross-Site Request Forgery) 跨站请求伪造

### 原理

攻击者诱导用户访问恶意网站 B,而用户在网站 A 已登录。恶意网站 B 向网站 A 发送请求 (如转账),利用用户在 A 的登录凭证 (Cookie) 自动带上的特性,冒充用户操作。

### 防御

- **SameSite Cookie 属性**:
  - `Strict`: 完全禁止第三方 Cookie。
  - `Lax` (默认): 仅允许导航到目标网址的 Get 请求携带 Cookie。
- **CSRF Token**:
  - 服务器生成一个随机 Token,放在页面隐藏域或 Meta 标签中。
  - 提交请求时必须带上这个 Token。
  - 攻击者无法获取第三方页面的 DOM,因此拿不到 Token。
- **检测 Referer / Origin 头部**: 拒绝来自非信任域名的请求。

---

## 3. 原型污染 (Prototype Pollution)

### 原理

JS 对象继承自原型链。如果攻击者能控制对象的键名 (如 `__proto__`),就能修改 `Object.prototype`,影响所有对象。

```javascript
// 攻击示例
const payload = JSON.parse('{"__proto__": {"admin": true}}');
const obj = {};
Object.assign(obj, payload); // 或 deepMerge

console.log({}.admin); // true -> 所有对象都被污染了!
```

### 场景

- 解析查询字符串 (Query String)。
- 深度合并对象 (Deep Merge)。

### 防御

- 使用 `Object.create(null)` 创建无原型对象。
- `Object.freeze(Object.prototype)`。
- 在合并库中过滤 `__proto__`, `constructor`, `prototype` 等键。
- 使用 `Map` 代替普通对象存储键值对。

---

## 4. CSP (Content Security Policy)

### 什么是 CSP?

CSP 是一种 HTTP 头部 (`Content-Security-Policy`),允许站点管理员声明允许加载哪些资源 (JS, CSS, 图片等),防止 XSS 和数据注入。

### 使用示例

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com;
```

- `default-src 'self'`: 默认只允许加载同源资源。
- `script-src ...`: JS 只能从同源或指定 CDN 加载,禁止内联脚本 (`<script>...`)。

---

## 5. 高频面试题

### Q1: XSS 和 CSRF 的区别?

**答**:

- **XSS (跨站脚本)**: 攻击者在你的网页里**注入代码**,脚本在用户浏览器运行。核心是"窃取数据"或"控制页面"。
  - _防_: 转义, CSP, HttpOnly。
- **CSRF (跨站请求伪造)**: 攻击者利用用户**已登录的身份**,在第三方网站发请求。核心是"借用身份"执行操作。
  - _防_: SameSite, CSRF Token, Origin 校验。

### Q2: 为什么 React/Vue 很少有 XSS?

**答**: 它们的数据绑定默认会自动对内容进行转义 (Escaping)。除非你显式使用 `dangerouslySetInnerHTML` 或 `v-html`,或者在 `href` 属性中使用了 `javascript:` 伪协议。

### Q3: 点击劫持 (Clickjacking) 是什么? 怎么防?

**答**: 攻击者用 iframe 嵌套你的网页,并将其设为透明,覆盖在诱导按钮上。用户以为在点按钮,实际点的是你网页上的操作 (如"关注", "付款")。

- **防**: `X-Frame-Options: DENY` 或 `SAMEORIGIN` (禁止被 iframe 嵌套)。
