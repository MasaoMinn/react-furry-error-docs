# 安装

## 1．furry-ts-errors（VSCode插件）安装指南

## 1.1 安装 VSIX 文件

[furry-ts-errors项目地址](https://github.com/MasaoMinn/furry-ts-errors/tree/feature/fursona-errors "furry-ts-errors项目地址")

点击连接后找到release，下载furry-ts-errors.vsix．（可以顺便给我个⭐，感谢）

## 1.2在 Visual Studio 和 Visual Studio Code 中安装 VSIX 文件的方法如下

## Visual Studio

1．打开 Visual Studio。
2．选择＂扩展＂菜单，然后选择＂管理扩展＂。
3．在＂管理扩展＂窗口中，选择＂安装来自 VSIX＂。
4．浏览并选择要安装的 VSIX 文件，然后点击＂安装＂。

## Visual Studio Code

1．打开 Visual Studio Code。
2．在扩展视图中，点击右上角的＂．．．＂按钮，选择＂从 VSIX 安装＂。
3．浏览并选择要安装的 VSIX 文件，然后点击＂安装＂。
4．安装完成后，重启 Visual Studio Code 即可。

## 使用命令行安装 VSIX 文件

你也可以使用命令行来安装 VSIX 文件。以下是具体步骤：

- 打开命令行窗口（如 Terminal 或 PowerShell）。
- 输入以下命令来安装 VSIX 文件：

```bash
code --install-extension path/to/your-extension.vsix
```

－执行命令后，重启 Visual Studio Code 即可看到安装的扩展。

## 2．react－furry－error（React 开发环境依赖）安装指南

## 0．项目地址

[react-furry-error](https://github.com/MasaoMinn/react-furry-error "react-furry-error")（可以顺便给我个，感谢）

## 1．安装

使用 npm：

```bash
npm install react-furry-error
```

或使用 pnpm／yarn：

```bash
1 pnpm add react-furry-error
2 # 或
3 yarn add react-furry-error
```

## 2．启用覆盖层

在您的应用程序入口文件中（例如 main．tsx 或 index．tsx）：

```JSX
    import { initFurryDevOverlay} from "react-furry-error";
    if (import.meta.env.MODE === "development") {
        initFurryDevOverlay();
    }
```

如果是Next.js等框架，以上代码不能使用，请使用

```JSX
    import { initFurryDevOverlay } from "react-furry-error";
    if (process.env.NODE_ENV === "development") {
        initFurryDevOverlay();
    }
```

启用后，运行时错误（runtime）将自动触发毛茸茸的错误覆盖层。

## 3．使用 ErrorTest 测试您的环境（可选）

你可以使用 **ErrorTest** 组件来测试 react-furry-error 是否运行正常

```JSX
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
