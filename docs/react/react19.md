# React 19 新特性

React 19 带来了许多激动人心的新特性，重点在于简化开发体验和提升性能。

## 核心更新

### 1. React Compiler (自动记忆化)

React Compiler 是一个构建时工具，能自动优化组件，无需手动使用 `useMemo`、`useCallback` 和 `React.memo`。

**优势：**

- **自动缓存**：编译器自动识别需要缓存的计算和组件。
- **简化代码**：开发者可以专注于业务逻辑，而不必担心重新渲染的性能问题。
- **细粒度更新**：只更新真正变化的部分，而不仅是组件级别。

**以前的代码：**

```jsx
const expensiveValue = useMemo(() => compute(a, b), [a, b]);
const handleClick = useCallback(() => { ... }, []);
```

**React 19 (使用 Compiler)：**

```jsx
// 无需手动 memo
const expensiveValue = compute(a, b);
const handleClick = () => { ... };
```

### 2. Server Actions

允许在服务端直接运行函数，简化数据变异（Data Mutation）流程。

**特点：**

- **原生支持**：不仅限于 Next.js，React 核心库现在原生支持。
- **RPC 风格**：客户端可以直接调用服务端定义的函数。
- **表单处理**：特别适合处理 `<form>` 提交。

```jsx
// actions.js
"use server";

export async function updateUser(formData) {
  const name = formData.get("name");
  await db.users.update({ name });
}

// Component.jsx
import { updateUser } from "./actions";

function Profile() {
  return (
    <form action={updateUser}>
      <input name="name" />
      <button type="submit">Update</button>
    </form>
  );
}
```

## 新 Hooks

### 1. `use()`

用于在组件中读取 Promise 或 Context 的值。

**读取 Promise：**

```jsx
import { use } from "react";

function Comments({ commentsPromise }) {
  // 会挂起组件直到 Promise resolve
  const comments = use(commentsPromise);
  return comments.map((c) => <p key={c.id}>{c.text}</p>);
}
```

**读取 Context：**

```jsx
// 可以在条件语句和循环中使用！
if (show) {
  const theme = use(ThemeContext);
}
```

### 2. `useOptimistic`

用于处理乐观 UI 更新（Optimistic UI）。在请求完成前，立即显示预期结果。

```jsx
import { useOptimistic } from "react";

function Thread({ messages, sendMessage }) {
  // messages 是实际状态，optimisticMessages 是乐观状态
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [...state, newMessage]
  );

  async function formAction(formData) {
    const message = formData.get("message");
    // 立即更新 UI
    addOptimisticMessage(message);
    // 发送真实请求
    await sendMessage(message);
  }

  return (
    <div>
      {optimisticMessages.map((m, i) => (
        <div key={i}>{m}</div>
      ))}
      <form action={formAction}>
        <input name="message" />
      </form>
    </div>
  );
}
```

### 3. `useFormStatus`

获取父级 `<form>` 的状态（如 loading），无需传递 props。

```jsx
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending}>{pending ? "Submitting..." : "Submit"}</button>
  );
}

function Form() {
  return (
    <form action={action}>
      <SubmitButton /> {/* 自动感知 form 状态 */}
    </form>
  );
}
```

### 4. `useActionState`

替代 `useFormState`，用于管理 Action 的状态（如 error, success）。

```jsx
import { useActionState } from "react";

function Form() {
  const [state, formAction, isPending] = useActionState(updateUser, null);

  return (
    <form action={formAction}>
      <input name="name" />
      {state?.error && <p>{state.error}</p>}
      <button disabled={isPending}>Update</button>
    </form>
  );
}
```

## 其他改进

### 1. `ref` 作为 Prop

不再需要 `forwardRef`！现在 `ref` 可以像普通 prop 一样传递。

```jsx
// ✅ React 19
function MyInput({ placeholder, ref }) {
  return <input placeholder={placeholder} ref={ref} />;
}

// ❌ 以前
const MyInput = forwardRef(({ placeholder }, ref) => {
  return <input placeholder={placeholder} ref={ref} />;
});
```

### 2.文档元数据支持

原生支持 `<title>`, `<meta>` 等标签，会自动提升到 `<head>`，且支持服务端渲染。

```jsx
function BlogPost({ title }) {
  return (
    <article>
      <title>{title}</title>
      <meta name="description" content="Blog post description" />
      <h1>{title}</h1>
    </article>
  );
}
```

### 3. useDeferredValue 初始值

`useDeferredValue` 现在支持第二个参数作为初始值，避免首次渲染时的额外开销。

```jsx
const value = useDeferredValue(deferredValue, initialValue);
```

## 常见面试题

### Q1: React Compiler 解决了什么问题？

A: 解决了 React 中长期存在的"过度渲染"问题和手动优化（useMemo/useCallback）的心智负担。它通过编译器自动分析依赖，实现细粒度的更新和自动缓存。

### Q2: `use()` Hook 和 `useContext` 有什么区别？

A: `use()` 更加灵活，可以在条件语句、循环和嵌套函数中调用，而 `useContext` 必须在组件顶层调用。此外，`use()` 还可以用于解包 Promise，配合 Suspense 使用。

### Q3: 为什么 React 19 移除了 `forwardRef`？

A: 为了简化 API。在 React 19 中，`ref` 被视为普通 prop，会自动在组件间传递，不再需要高阶组件 `forwardRef` 来转发。

### Q4: Server Actions 和普通 API 请求有什么区别？

A: Server Actions 是一种 RPC（远程过程调用）机制，隐式地处理了客户端到服务端的通信。它与 React 的表单和 transition 深度集成，支持渐进增强（即使没有 JS 也能工作），并且可以由 React 管理 loading 和 error 状态。
