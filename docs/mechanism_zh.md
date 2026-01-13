# 本节将简要说明项目的内部实现思路，帮助开发者理解它的核心工作机制

## 1. furry-ts-errors 实现原理

我Fork了 [yoavbls/pretty-ts-errors](https://github.com/yoavbls/pretty-ts-errors "yoavbls/pretty-ts-errors") ，在这个项目的基础上在**webview**中添加了图片逻辑：

```javascript
/* eslint-disable @typescript-eslint/ban-ts-comment */
// @ts-nocheck

/**
 * Extension for the message listener
 * This allows extending the message handling functionality without modifying index.js
 */
(function() {
  if (typeof window === 'undefined') {
    return;
  }

  const originalAddEventListener = window.addEventListener;
  const messageListeners = [];
  window.addEventListener = function(type, listener, options) {
    if (type === 'message') {
      messageListeners.push(listener);
    }
    return originalAddEventListener.call(this, type, listener, options);
  };
  const originalRemoveEventListener = window.removeEventListener;
  window.removeEventListener = function(type, listener, options) {
    if (type === 'message') {
      const index = messageListeners.indexOf(listener);
      if (index > -1) {
        messageListeners.splice(index, 1);
      }
    }
    return originalRemoveEventListener.call(this, type, listener, options);
  };
  if (window.document.readyState === 'loading') {
    window.document.addEventListener('DOMContentLoaded', addCustomMessageListener);
  } else {
    addCustomMessageListener();
  }
  function addCustomMessageListener() {
    // Add our own message listener that will be called after the original one
    originalAddEventListener.call(window, 'message', function(event) {
      const message = event.data;
      switch (message.command) {
        case 'update-content': {
          console.log('Custom update-content handler called');
          const $furryError = window.document.querySelector('#furry-error');
          if ($furryError) {
            // Get the image path based on message classification
            const imagePath = window.classifyMessage(message.content);
            let imageUri;
            
            // Select the appropriate image URI based on the classified path
            if (imagePath === '/images/hook.png') {
              imageUri = message.hookImageUri || './images/hook.png';
            } else if (imagePath === '/images/dom.png') {
              imageUri = message.domImageUri || './images/dom.png';
            } else if (imagePath === '/images/not_found_wink.png') {
              imageUri = message.notFoundWinkImageUri || './images/not_found_wink.png';
            } else {
              imageUri = message.confusedImageUri || './images/confused.png';
            }
            
            // Set the innerHTML with the appropriate image
            $furryError.innerHTML = `<img src="${imageUri}" alt="Furry error" />`;
          } else {
            console.error('Could not find #furry-error element');
          }
          break;
        }
      }
    });
  }

  // Export for potential use in other modules
  window.furryErrorExtension = {
    getMessageListeners: () => messageListeners,
    addCustomMessageListener
  };
})();
```

分类逻辑，根据报错信息进行分类，显示对应图片

```javascript
// Convert to a global function for use in webview
function classifyMessage(message) {
  const text = message.toLowerCase();

  if (
    text.includes('hook') ||
    text.includes('hooks') ||
    text.includes('useeffect') ||
    text.includes('usestate') ||
    text.includes('usememo') ||
    text.includes('usecallback') ||
    text.includes('usereducer')
  ) {
    return '/images/hook.png';
  }

  if (
    text.includes('react child') ||
    text.includes('cannot read properties of undefined') ||
    text.includes('cannot read property') ||
    text.includes('each child in a list should have a unique "key"') ||
    text.includes('hydrate') ||
    text.includes('hydration') ||
    text.includes('did not match') ||
    text.includes('expected server html') ||
    text.includes('adjacent jsx elements') ||
    text.includes('failed to execute') ||
    text.includes('appendchild') ||
    text.includes('removechild') ||
    text.includes('dom') ||
    text.includes('doms')
  ) {
    return '/images/dom.png';
  }

  if (
    text.includes('found') ||
    text.includes('find') ||
    text.includes('varient') ||
    text.includes('import') || 
    text.includes('module') ||
    text.includes('export')
  ) {
    return '/images/not_found_wink.png';
  }

  return '/images/confused.png';
}

// Add to global window object for use in furryError.js
window.classifyMessage = classifyMessage;

```

# 2. react-furry-error 实现原理

## 2.1 全局错误监听

库的核心入口是浏览器层级的全局错误监听，通过绑定以下原生事件捕获各类运行时异常：

- window.onerror：捕获同步 JavaScript 错误
- window.onunhandledrejection：捕获未处理的 Promise 拒绝（如 fetch 失败、async/await 未捕获错误）
通过上述监听，可获取关键错误信息：
- 错误描述信息（error.message）
- 错误发生的文件名、行号与列号
- 错误堆栈跟踪（stack trace，若浏览器支持）
以下是index.ts代码实现：

```javascript
export function initFurryDevOverlay(): void {
  // 标记已经初始化
  setInitialized();

  // 初始化WebSocket补丁（无论enabled为何值都保留）
  patchWebSocket();
  // 启用时：使用自定义错误覆盖层
  window.addEventListener("error", (event) => {
    const err = event.error;
    if (!err) return;

    const type = classifyFurryType(err.message);
    const msg: DevOverlayMessage = {
      type,
      message: err.message,
      stack: err.stack,
    };

    showOverlay(msg);
  });

  window.addEventListener("unhandledrejection", (event) => {
    const reason = event.reason;
    if (!reason) return;

    const errorMessage = reason.message ?? String(reason);
    const type = classifyFurryType(errorMessage);
    const msg: DevOverlayMessage = {
      type,
      message: errorMessage,
      stack: reason.stack,
    };

    showOverlay(msg);
  });

}
```

2.2 错误分类逻辑
我根据错误信息来大致判断错误类型，将错误分类为4种，每种对应专属的兽设图像。以下是分类逻辑代码classify.ts ：

```javascript
// classify.ts

import type { FurryImageType } from "./types";

export function classifyFurryType(message: string): FurryImageType {
  const text = message.toLowerCase();

  // hook 类错误（优先级最高）
  if (
    text.includes("invalid hook call") ||
    text.includes("too many re-renders") ||
    text.includes("maximum update depth") ||
    text.includes("hooks can only be used") ||
    text.includes("dependency cycle") ||
    text.includes("cannot update a component while rendering")
  ) {
    return "hook-error";
  }

  // dom/hydration 错误（第二）
  if (
    text.includes("hydration") ||
    text.includes("markup") ||
    text.includes("unmounted") ||
    text.includes("container is null") ||
    text.includes("invalid react element") ||
    text.includes("properties")
  ) {
    return "dom-broken";
  }

  // searching 类（资源缺失、找不到、执行失败）
  if (
    text.includes("not found") ||
    text.includes("cannot resolve") ||
    text.includes("failed to load") ||
    text.includes("failed to fetch") ||
    text.includes("undefined is not a function") ||
    text.includes("is not defined")
  ) {
    return "searching";
  }

  // 默认兜底
  return "confused";
}
```

## 2.3 Overlay 渲染方式

错误发生后，通过独立隔离的渲染逻辑展示错误覆盖层，避免影响应用本身：

- 动态创建独立的 DOM 容器（div#react-furry-error-overlay）
- 使用 React 18 新增的 createRoot API 挂载组件（支持并发渲染特性）
- 渲染隔离的 ErrorOverlay 组件（与应用 React 树完全分离）
- 覆盖层核心展示内容：
- 对应错误类型的萌系插画
- 简洁的错误描述（提炼关键信息）
- 可折叠的详细堆栈跟踪（便于调试）
- 快速刷新按钮（一键重启应用）

## 2.4 安全性与清理机制

为确保不污染应用环境、无内存泄漏，设计了严格的安全策略：

- 🛡️ Overlay 采用单例模式：全局仅存在一个 root 实例，避免重复渲染
- 🧹 自动清理：每次新错误触发前，卸载上一个 Overlay 实例，释放 DOM 与内存
- 🚧 隔离设计：不修改应用的 React 根节点、不注入全局变量、不干扰应用自身逻辑

## react-furry-error 的本质是

一个只聚焦运行时错误的轻量工具，通过「非侵入式监听 + 分类可视化」的设计，在不修改 React 内部机制的前提下，降低开发过程中的错误理解与调试成本
