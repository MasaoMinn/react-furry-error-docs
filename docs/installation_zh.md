# react-furry-error 安装指南

## 安装

### 1. 安装包

使用 npm:

```bash
npm install react-furry-error
```

或使用 pnpm / yarn:

```bash
pnpm add react-furry-error
# 或
yarn add react-furry-error
```

### 2. 启用覆盖层

在您的应用程序入口文件中（例如 main.tsx 或 index.tsx）：

```tsx
import { initFurryDevOverlay} from "react-furry-error";

if (import.meta.env.MODE === "development") {
  initFurryDevOverlay();
}
```

就是这样。
启用后，运行时错误将自动触发毛茸茸的错误覆盖层。

### 3. 使用 ErrorTest 测试您的环境（可选）

```tsx

import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { initFurryDevOverlay,ErrorTest} from "react-furry-error";

if (import.meta.env.MODE === "development") {
  initFurryDevOverlay();
}

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <ErrorTest />
  </StrictMode>,
)

```

### 4. 它可以捕获哪些错误？

react-furry-error 可以检测并显示：

- 运行时 JavaScript 错误
- React 渲染错误
- Hook 误用错误
- Promise / fetch 失败
- 水合不匹配（运行时）
- 热模块替换（HMR）运行时崩溃

⚠️ 编译时语法错误（例如无效的 JSX）不能被任何运行时库拦截。当应用程序无法启动时，无法挂载覆盖层。
