# react-furry-error Implementation Mechanism

## This section briefly explains the internal implementation ideas of react-furry-error to help developers understand its core working mechanism

### 1.  Global Error Listening

The core entry point of the library is browser-level global error listening, which captures various runtime exceptions by binding the following native events:

- window.onerror: Captures synchronous JavaScript errors
- window.onunhandledrejection: Captures unhandled Promise rejections (such as failed fetch requests, uncaught async/await errors)

Through the above listening, key error information can be obtained:

- Error description message (error message)
- File name, line number, and column number where the error occurred
- Error stack trace (if supported by the browser)

Here is the implementation in index.ts:

```javascript
export function initFurryDevOverlay(): void {
  // Mark as initialized
  setInitialized();

  // Initialize WebSocket patch (retained regardless of enabled value)
  patchWebSocket();
  // When enabled: Use custom error overlay
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

### 2.  Error Classification Logic

I roughly determine the error type based on the error message and classify errors into 4 types, each corresponding to a unique furry character image. Here is the classification logic code in classify.ts:

```javascript
// classify.ts

import type { FurryImageType } from "./types";

export function classifyFurryType(message: string): FurryImageType {
  const text = message.toLowerCase();

  // Hook-related errors (highest priority)
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

  // DOM/hydration errors (second priority)
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

  // Searching category (missing resources, not found, execution failure)
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

  // Default fallback
  return "confused";
}

```

### 3. Overlay Rendering Method

After an error occurs, an error overlay is displayed through independent and isolated rendering logic to avoid affecting the application itself:
Dynamically create an independent DOM container (div#react-furry-error-overlay)
Use React 18's new createRoot API to mount the component (supports concurrent rendering features)
Render an isolated ErrorOverlay component (completely separate from the application's React tree)
Core display content of the overlay:
Cute illustrations corresponding to the error type
Concise error description (extracting key information)
Collapsible detailed stack trace (for debugging)
Quick refresh button (one-click restart of the application)

### 4. Security and Cleanup Mechanism

To ensure no pollution of the application environment and no memory leaks, strict security policies have been designed:
🛡️ Overlay uses singleton pattern: Only one root instance exists globally to avoid repeated rendering
🧹 Automatic cleanup: Unmounts the previous Overlay instance before each new error, releasing DOM and memory
🚧 Isolated design: Does not modify the application's React root node, does not inject global variables, and does not interfere with the application's own logic
Core Summary
The essence of react-furry-error is:
A lightweight tool that only focuses on runtime errors, through the design of "non-intrusive listening + classification visualization", it reduces the cost of error understanding and debugging during development without modifying React's internal mechanisms.
