# 正则表达式完全指南

## 概述

正则表达式 (Regular Expression) 是用于匹配字符串中字符组合的模式。在 JavaScript 中，正则表达式也是对象，是前端面试的高频考点。

---

## 一、创建正则表达式

### 1. 两种创建方式

```javascript
// 1. 字面量方式 (推荐)
const regex1 = /pattern/flags

// 2. 构造函数方式
const regex2 = new RegExp('pattern', 'flags')

// 示例
const re1 = /hello/i           // 匹配 hello，忽略大小写
const re2 = new RegExp('hello', 'i')

// 动态创建正则时使用构造函数
const keyword = 'user'
const dynamicRe = new RegExp(keyword, 'g')
```

### 2. 修饰符 (Flags)

| 修饰符 | 说明 |
|--------|------|
| `g` | 全局匹配，查找所有匹配项 |
| `i` | 忽略大小写 |
| `m` | 多行模式，^ 和 $ 匹配每行 |
| `s` | dotAll 模式，. 匹配换行符 |
| `u` | Unicode 模式 |
| `y` | 粘性匹配，从 lastIndex 开始 |

```javascript
// g - 全局匹配
'hello hello'.match(/hello/)   // ['hello']
'hello hello'.match(/hello/g)  // ['hello', 'hello']

// i - 忽略大小写
/hello/i.test('HELLO')  // true

// m - 多行模式
const str = 'line1\nline2'
str.match(/^line/g)   // ['line']
str.match(/^line/gm)  // ['line', 'line']

// s - dotAll 模式
/a.b/.test('a\nb')   // false
/a.b/s.test('a\nb')  // true

// u - Unicode 模式
/\u{1F600}/u.test('😀')  // true
/^.$/u.test('😀')        // true (正确识别 emoji 为1个字符)

// y - 粘性匹配
const sticky = /foo/y
sticky.lastIndex = 3
'xxxfoo'.match(sticky)  // ['foo']
```

---

## 二、元字符

### 1. 基本元字符

| 元字符 | 说明 | 示例 |
|--------|------|------|
| `.` | 匹配任意单个字符（除换行符） | `/a.c/` 匹配 "abc" |
| `\d` | 匹配数字 [0-9] | `/\d+/` 匹配 "123" |
| `\D` | 匹配非数字 | `/\D+/` 匹配 "abc" |
| `\w` | 匹配字母数字下划线 [a-zA-Z0-9_] | `/\w+/` 匹配 "hello_123" |
| `\W` | 匹配非字母数字下划线 | `/\W/` 匹配 "@" |
| `\s` | 匹配空白字符（空格、制表符等） | `/\s/` 匹配 " " |
| `\S` | 匹配非空白字符 | `/\S+/` 匹配 "hello" |
| `\b` | 匹配单词边界 | `/\bword\b/` |
| `\B` | 匹配非单词边界 | `/\Bword/` |

```javascript
// \d 和 \D
'abc123'.match(/\d+/)   // ['123']
'abc123'.match(/\D+/)   // ['abc']

// \w 和 \W
'hello_world!'.match(/\w+/)  // ['hello_world']
'hello_world!'.match(/\W/)   // ['!']

// \s 和 \S
'hello world'.match(/\s/)    // [' ']
'hello world'.match(/\S+/g)  // ['hello', 'world']

// \b 单词边界
'hello world'.match(/\bworld\b/)  // ['world']
'helloworld'.match(/\bworld\b/)   // null

// 匹配整个单词
const text = 'cat category catalog'
text.match(/\bcat\b/g)  // ['cat']
```

### 2. 位置元字符

```javascript
// ^ - 匹配开头
/^hello/.test('hello world')  // true
/^hello/.test('say hello')    // false

// $ - 匹配结尾
/world$/.test('hello world')  // true
/world$/.test('world hello')  // false

// 同时使用 ^ 和 $
/^hello$/.test('hello')        // true (精确匹配)
/^hello$/.test('hello world')  // false

// 多行模式下
const multiline = 'line1\nline2\nline3'
multiline.match(/^line\d/gm)  // ['line1', 'line2', 'line3']
```

---

## 三、量词

### 1. 基本量词

| 量词 | 说明 | 示例 |
|------|------|------|
| `*` | 0 次或多次 | `/a*/` |
| `+` | 1 次或多次 | `/a+/` |
| `?` | 0 次或 1 次 | `/a?/` |
| `{n}` | 恰好 n 次 | `/a{3}/` |
| `{n,}` | 至少 n 次 | `/a{2,}/` |
| `{n,m}` | n 到 m 次 | `/a{2,4}/` |

```javascript
// * - 0次或多次
'color'.match(/colou*r/)   // ['color']
'colour'.match(/colou*r/)  // ['colour']
'colouur'.match(/colou*r/) // ['colouur']

// + - 1次或多次
'ac'.match(/ab+c/)    // null
'abc'.match(/ab+c/)   // ['abc']
'abbc'.match(/ab+c/)  // ['abbc']

// ? - 0次或1次
'color'.match(/colou?r/)   // ['color']
'colour'.match(/colou?r/)  // ['colour']

// {n} - 恰好n次
'aaa'.match(/a{3}/)   // ['aaa']
'aa'.match(/a{3}/)    // null

// {n,} - 至少n次
'aa'.match(/a{2,}/)   // ['aa']
'aaaa'.match(/a{2,}/) // ['aaaa']

// {n,m} - n到m次
'aaa'.match(/a{2,4}/) // ['aaa']
'aaaaa'.match(/a{2,4}/) // ['aaaa']
```

### 2. 贪婪与非贪婪

```javascript
// 贪婪模式 (默认) - 尽可能多地匹配
const html = '<div>content</div>'
html.match(/<.+>/)   // ['<div>content</div>']

// 非贪婪模式 (加?) - 尽可能少地匹配
html.match(/<.+?>/)  // ['<div>']

// 各量词的非贪婪形式
// *?  - 0次或多次，非贪婪
// +?  - 1次或多次，非贪婪
// ??  - 0次或1次，非贪婪
// {n,}? - 至少n次，非贪婪
// {n,m}? - n到m次，非贪婪

const str = 'aaaaaa'
str.match(/a+/)    // ['aaaaaa'] 贪婪
str.match(/a+?/)   // ['a'] 非贪婪

str.match(/a{2,4}/)   // ['aaaa'] 贪婪
str.match(/a{2,4}?/)  // ['aa'] 非贪婪
```

---

## 四、字符组

### 1. 字符集合

```javascript
// [...] - 匹配方括号中的任意一个字符
/[abc]/.test('a')   // true
/[abc]/.test('d')   // false

// 范围表示
/[a-z]/.test('m')   // true (小写字母)
/[A-Z]/.test('M')   // true (大写字母)
/[0-9]/.test('5')   // true (数字)

// 组合使用
/[a-zA-Z0-9]/.test('A')  // true
/[a-zA-Z0-9]/.test('5')  // true

// [^...] - 取反，匹配不在方括号中的字符
/[^abc]/.test('d')  // true
/[^abc]/.test('a')  // false

/[^0-9]/.test('a')  // true (非数字)
```

### 2. 特殊字符转义

```javascript
// 在字符集中，大多数特殊字符不需要转义
/[.?*+]/.test('?')   // true

// 但以下字符仍需转义
/[\]]/.test(']')     // true (右方括号)
/[\\]/.test('\\')    // true (反斜杠)
/[\^]/.test('^')     // true (在开头时)
/[\-]/.test('-')     // true (在中间时)

// 或者放在特殊位置
/[]a]/.test(']')     // true (]放开头)
/[a-]/.test('-')     // true (-放结尾)
/[^a]/.test('^')     // true (^不在开头)
```

---

## 五、分组与捕获

### 1. 捕获组

```javascript
// (...) - 捕获组
const match = 'hello world'.match(/(hello) (world)/)
// ['hello world', 'hello', 'world']
// match[0] - 完整匹配
// match[1] - 第一个捕获组
// match[2] - 第二个捕获组

// 命名捕获组 (?<name>...)
const dateMatch = '2024-01-15'.match(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/)
console.log(dateMatch.groups)
// { year: '2024', month: '01', day: '15' }

// 在替换中使用捕获组
'hello world'.replace(/(hello) (world)/, '$2 $1')  // 'world hello'

// 使用命名捕获组替换
'2024-01-15'.replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  '$<month>/$<day>/$<year>'
)  // '01/15/2024'
```

### 2. 非捕获组

```javascript
// (?:...) - 非捕获组，只分组不捕获
const match1 = 'hello world'.match(/(hello) (world)/)
// ['hello world', 'hello', 'world']

const match2 = 'hello world'.match(/(?:hello) (world)/)
// ['hello world', 'world']
// 只有一个捕获组

// 常用于需要分组但不需要捕获的场景
/(?:http|https):\/\//.test('https://example.com')  // true
```

### 3. 反向引用

```javascript
// \n - 引用第 n 个捕获组的内容
// 匹配重复的单词
const text = 'hello hello world'
text.match(/(\w+) \1/)  // ['hello hello', 'hello']

// 匹配 HTML 标签
const html = '<div>content</div>'
html.match(/<(\w+)>.*<\/\1>/)  // ['<div>content</div>', 'div']

// 命名反向引用 \k<name>
const repeatWord = 'hello hello'
repeatWord.match(/(?<word>\w+) \k<word>/)  // ['hello hello', 'hello']

// 查找重复字符
'aabbcc'.match(/(.)\1/g)  // ['aa', 'bb', 'cc']
```

---

## 六、断言

### 1. 先行断言

```javascript
// (?=...) - 正向先行断言 (后面是...)
// 匹配后面跟着 'world' 的 'hello'
'hello world'.match(/hello(?= world)/)  // ['hello']
'hello there'.match(/hello(?= world)/)  // null

// 密码验证：匹配至少包含一个数字的字符串
/^(?=.*\d).+$/.test('abc123')  // true
/^(?=.*\d).+$/.test('abcdef')  // false

// (?!...) - 负向先行断言 (后面不是...)
// 匹配后面不跟数字的位置
'test1 test'.match(/test(?!\d)/g)  // ['test'] (第二个)

// 匹配不是 .js 结尾的文件名
/^[\w-]+(?!\.js$)/.test('app.ts')   // true
/^[\w-]+(?!\.js$)/.test('app.js')   // false
```

### 2. 后行断言

```javascript
// (?<=...) - 正向后行断言 (前面是...)
// 匹配前面是 $ 的数字
'$100 and 200'.match(/(?<=\$)\d+/)  // ['100']

// (?<!...) - 负向后行断言 (前面不是...)
// 匹配前面不是 $ 的数字
'$100 and 200'.match(/(?<!\$)\d+/)  // ['00'] 或 ['200']

// 提取价格数字
const prices = '$10 €20 ¥30'
prices.match(/(?<=[$€¥])\d+/g)  // ['10', '20', '30']
```

### 3. 常见断言应用

```javascript
// 密码强度验证：至少8位，包含大写、小写、数字
const strongPassword = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/

strongPassword.test('Abc12345')   // true
strongPassword.test('abc12345')   // false (无大写)
strongPassword.test('ABCD1234')   // false (无小写)
strongPassword.test('Abcdefgh')   // false (无数字)

// 千分位格式化
function formatNumber(num) {
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',')
}
formatNumber(1234567)  // '1,234,567'

// 匹配不在引号内的逗号
const csv = 'a,"b,c",d'
// 复杂场景需要更复杂的正则或其他解析方式
```

---

## 七、常用方法

### 1. RegExp 方法

```javascript
// test() - 测试是否匹配
/hello/.test('hello world')  // true

// exec() - 执行匹配，返回详细信息
const regex = /(\w+) (\w+)/
const result = regex.exec('hello world')
// ['hello world', 'hello', 'world', index: 0, input: 'hello world', groups: undefined]

// 全局匹配时需要循环
const re = /\d+/g
const str = 'a1b2c3'
let match
while ((match = re.exec(str)) !== null) {
  console.log(match[0], match.index)
}
// '1' 1
// '2' 3
// '3' 5
```

### 2. String 方法

```javascript
// match() - 返回匹配结果
'hello world'.match(/\w+/)   // ['hello']
'hello world'.match(/\w+/g)  // ['hello', 'world']

// matchAll() - 返回迭代器 (ES2020)
const str = 'test1 test2'
const matches = [...str.matchAll(/test(\d)/g)]
// [['test1', '1'], ['test2', '2']]

// search() - 返回匹配位置
'hello world'.search(/world/)  // 6
'hello world'.search(/xyz/)    // -1

// replace() - 替换
'hello world'.replace(/world/, 'JavaScript')  // 'hello JavaScript'
'hello world'.replace(/o/g, '0')  // 'hell0 w0rld'

// 使用函数替换
'hello world'.replace(/\w+/g, (match) => match.toUpperCase())
// 'HELLO WORLD'

// 替换函数参数：match, p1, p2, ..., offset, string, groups
'2024-01-15'.replace(
  /(\d{4})-(\d{2})-(\d{2})/,
  (match, year, month, day) => `${month}/${day}/${year}`
)  // '01/15/2024'

// replaceAll() - 替换所有 (ES2021)
'hello hello'.replaceAll('hello', 'hi')  // 'hi hi'

// split() - 分割
'a,b;c|d'.split(/[,;|]/)  // ['a', 'b', 'c', 'd']

// 保留分隔符（使用捕获组）
'a1b2c3'.split(/(\d)/)  // ['a', '1', 'b', '2', 'c', '3', '']
```

---

## 八、常用正则表达式

### 1. 表单验证

```javascript
// 手机号（中国大陆）
const phoneReg = /^1[3-9]\d{9}$/
phoneReg.test('13812345678')  // true

// 邮箱
const emailReg = /^[\w.-]+@[\w-]+(\.\w+)+$/
emailReg.test('test@example.com')  // true

// 身份证号（18位）
const idCardReg = /^[1-9]\d{5}(19|20)\d{2}(0[1-9]|1[0-2])(0[1-9]|[12]\d|3[01])\d{3}[\dXx]$/
idCardReg.test('110101199001011234')  // true

// URL
const urlReg = /^https?:\/\/[\w-]+(\.[\w-]+)+([/?#].*)?$/
urlReg.test('https://www.example.com/path?query=1')  // true

// IPv4
const ipv4Reg = /^((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$/
ipv4Reg.test('192.168.1.1')  // true

// 用户名（字母开头，允许字母数字下划线，4-16位）
const usernameReg = /^[a-zA-Z]\w{3,15}$/
usernameReg.test('user_123')  // true

// 密码强度（8-20位，必须包含大小写字母和数字）
const passwordReg = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d]{8,20}$/
passwordReg.test('Abc12345')  // true

// 中文
const chineseReg = /^[\u4e00-\u9fa5]+$/
chineseReg.test('你好')  // true

// 邮政编码（中国）
const postalReg = /^[1-9]\d{5}$/
postalReg.test('100000')  // true

// 车牌号
const plateReg = /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领][A-Z][A-HJ-NP-Z0-9]{4,5}[A-HJ-NP-Z0-9挂学警港澳]$/
plateReg.test('京A12345')  // true
```

### 2. 字符串处理

```javascript
// 去除首尾空格
function trim(str) {
  return str.replace(/^\s+|\s+$/g, '')
}
trim('  hello  ')  // 'hello'

// 去除所有空格
function removeSpaces(str) {
  return str.replace(/\s+/g, '')
}

// 驼峰转连字符
function camelToKebab(str) {
  return str.replace(/([A-Z])/g, '-$1').toLowerCase()
}
camelToKebab('helloWorld')  // 'hello-world'

// 连字符转驼峰
function kebabToCamel(str) {
  return str.replace(/-([a-z])/g, (_, letter) => letter.toUpperCase())
}
kebabToCamel('hello-world')  // 'helloWorld'

// 首字母大写
function capitalize(str) {
  return str.replace(/^\w/, c => c.toUpperCase())
}
capitalize('hello')  // 'Hello'

// 每个单词首字母大写
function titleCase(str) {
  return str.replace(/\b\w/g, c => c.toUpperCase())
}
titleCase('hello world')  // 'Hello World'

// 压缩连续空格
function compressSpaces(str) {
  return str.replace(/\s+/g, ' ')
}
compressSpaces('hello    world')  // 'hello world'

// 转义 HTML 特殊字符
function escapeHtml(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;')
}

// 提取 URL 参数
function getUrlParams(url) {
  const params = {}
  url.replace(/[?&]([^=&#]+)=([^&#]*)/g, (_, key, value) => {
    params[key] = decodeURIComponent(value)
  })
  return params
}
getUrlParams('https://example.com?name=John&age=25')
// { name: 'John', age: '25' }

// 格式化金额
function formatMoney(num) {
  return num.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ',')
}
formatMoney(1234567.89)  // '1,234,567.89'

// 脱敏处理
function maskPhone(phone) {
  return phone.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2')
}
maskPhone('13812345678')  // '138****5678'

function maskIdCard(id) {
  return id.replace(/(\d{4})\d{10}(\d{4})/, '$1**********$2')
}

function maskEmail(email) {
  return email.replace(/(.{2}).+(.{2}@)/, '$1****$2')
}
```

### 3. HTML 处理

```javascript
// 提取 HTML 标签内容
function getTagContent(html, tag) {
  const regex = new RegExp(`<${tag}[^>]*>([\\s\\S]*?)<\\/${tag}>`, 'gi')
  const matches = []
  let match
  while ((match = regex.exec(html)) !== null) {
    matches.push(match[1])
  }
  return matches
}
getTagContent('<p>Hello</p><p>World</p>', 'p')  // ['Hello', 'World']

// 去除 HTML 标签
function stripHtml(html) {
  return html.replace(/<[^>]+>/g, '')
}
stripHtml('<p>Hello <b>World</b></p>')  // 'Hello World'

// 提取所有链接
function extractLinks(html) {
  const regex = /href=["']([^"']+)["']/g
  const links = []
  let match
  while ((match = regex.exec(html)) !== null) {
    links.push(match[1])
  }
  return links
}

// 提取所有图片
function extractImages(html) {
  const regex = /src=["']([^"']+)["']/g
  const images = []
  let match
  while ((match = regex.exec(html)) !== null) {
    images.push(match[1])
  }
  return images
}

// 高亮关键词
function highlightKeyword(text, keyword) {
  const regex = new RegExp(`(${keyword})`, 'gi')
  return text.replace(regex, '<mark>$1</mark>')
}
highlightKeyword('Hello World', 'world')  // 'Hello <mark>World</mark>'
```

---

## 九、性能优化

### 1. 避免回溯灾难

```javascript
// 危险的正则 - 可能导致 ReDoS 攻击
const badRegex = /^(a+)+$/

// 测试
console.time('bad')
badRegex.test('aaaaaaaaaaaaaaaaaaaaab')  // 非常慢
console.timeEnd('bad')

// 改进版本
const goodRegex = /^a+$/

// 避免嵌套量词
// Bad:  (a+)+
// Good: a+

// Bad:  (a|a)+
// Good: a+
```

### 2. 优化建议

```javascript
// 1. 预编译正则（避免循环中创建）
// Bad
for (let i = 0; i < 1000; i++) {
  /\d+/.test(str)  // 每次创建新正则
}

// Good
const numReg = /\d+/
for (let i = 0; i < 1000; i++) {
  numReg.test(str)  // 复用
}

// 2. 使用非捕获组
// Bad
/(foo|bar|baz)/

// Good (如果不需要捕获)
/(?:foo|bar|baz)/

// 3. 具体化字符类
// Bad
/.*foo/

// Good
/[^f]*foo/

// 4. 使用 test() 而非 match() 做验证
// Bad (创建数组)
if ('hello'.match(/hello/)) {}

// Good
if (/hello/.test('hello')) {}

// 5. 锚定正则
// Bad
/hello/

// Good (如果要匹配完整字符串)
/^hello$/
```

---

## 十、高频面试题

### 1. 实现模板字符串解析

```javascript
function render(template, data) {
  return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
    return data[key] !== undefined ? data[key] : match
  })
}

const template = 'Hello, {{name}}! You are {{age}} years old.'
const data = { name: 'John', age: 25 }
render(template, data)  // 'Hello, John! You are 25 years old.'
```

### 2. 版本号比较

```javascript
function compareVersion(v1, v2) {
  const arr1 = v1.split('.').map(Number)
  const arr2 = v2.split('.').map(Number)
  const len = Math.max(arr1.length, arr2.length)

  for (let i = 0; i < len; i++) {
    const n1 = arr1[i] || 0
    const n2 = arr2[i] || 0
    if (n1 > n2) return 1
    if (n1 < n2) return -1
  }
  return 0
}

compareVersion('1.2.3', '1.2.4')   // -1
compareVersion('1.10.0', '1.9.0') // 1
compareVersion('1.0', '1.0.0')    // 0
```

### 3. 检查括号是否匹配（简化版）

```javascript
function isBalanced(str) {
  // 移除非括号字符
  const brackets = str.replace(/[^()[\]{}]/g, '')

  let prev = ''
  while (brackets !== prev) {
    prev = brackets
    brackets = brackets
      .replace(/\(\)/g, '')
      .replace(/\[\]/g, '')
      .replace(/\{\}/g, '')
  }

  return brackets === ''
}

isBalanced('(a + b) * [c - d]')  // true
isBalanced('(()')                // false
```

### 4. 千分位格式化

```javascript
// 方法1：正则
function formatNumber(num) {
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',')
}

// 方法2：toLocaleString
function formatNumber2(num) {
  return num.toLocaleString()
}

formatNumber(1234567890)  // '1,234,567,890'
```

### 5. 驼峰命名转换

```javascript
// 下划线转驼峰
function toCamelCase(str) {
  return str.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase())
}
toCamelCase('hello_world_test')  // 'helloWorldTest'

// 驼峰转下划线
function toSnakeCase(str) {
  return str.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`)
}
toSnakeCase('helloWorldTest')  // 'hello_world_test'
```

### 6. 解析 URL

```javascript
function parseUrl(url) {
  const regex = /^(https?):\/\/([^/:]+)(?::(\d+))?(\/[^?#]*)?(?:\?([^#]*))?(?:#(.*))?$/
  const match = url.match(regex)

  if (!match) return null

  return {
    protocol: match[1],
    host: match[2],
    port: match[3] || (match[1] === 'https' ? '443' : '80'),
    path: match[4] || '/',
    query: match[5] || '',
    hash: match[6] || ''
  }
}

parseUrl('https://www.example.com:8080/path?name=test#section')
// {
//   protocol: 'https',
//   host: 'www.example.com',
//   port: '8080',
//   path: '/path',
//   query: 'name=test',
//   hash: 'section'
// }
```

### 7. 正则表达式和字符串方法的区别？

```
test() vs match():
- test() 返回布尔值，性能更好
- match() 返回匹配结果数组

exec() vs match():
- exec() 可以配合 lastIndex 循环匹配
- match() 全局模式返回所有匹配，但无捕获组

replace() vs replaceAll():
- replace() 非全局模式只替换第一个
- replaceAll() 替换所有（ES2021）

split() 可以使用正则作为分隔符
```

### 8. 贪婪匹配和非贪婪匹配的区别？

```javascript
const html = '<div>hello</div><div>world</div>'

// 贪婪匹配 - 尽可能多地匹配
html.match(/<div>.*<\/div>/)
// ['<div>hello</div><div>world</div>']

// 非贪婪匹配 - 尽可能少地匹配
html.match(/<div>.*?<\/div>/)
// ['<div>hello</div>']

// 非贪婪量词：*?, +?, ??, {n,}?, {n,m}?
```

### 9. 如何避免正则表达式的性能问题？

```
1. 避免嵌套量词：(a+)+ 改为 a+
2. 避免回溯：使用更具体的字符类
3. 预编译正则：避免循环中创建
4. 使用非捕获组：(?:...) 代替 (...)
5. 锚定正则：使用 ^ 和 $
6. 避免过度使用 .*
7. 测试边界情况和长字符串
```
