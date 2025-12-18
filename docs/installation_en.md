# react-furry-error Installation Guide

## Installation

### 1. Install the package

Using npm:

```bash
npm install react-furry-error --save-dev
```

Or with pnpm / yarn:

```bash
pnpm add react-furry-error --save-dev
# or
yarn add react-furry-error --dev
```

### 2. Enable the overlay

In your application entry file (for example main.tsx or index.tsx):

```tsx
import { initFurryDevOverlay} from "react-furry-error";

if (import.meta.env.MODE === "development") {
  initFurryDevOverlay();
}
```

If you are using **Next.js** or some other frameworks that triggers error on the code above, you may try this code:

```javascript
import { initFurryDevOverlay } from "react-furry-error";

if (process.env.NODE_ENV === "development") {
  initFurryDevOverlay();
}
```

That’s it.
Once enabled, runtime errors will automatically trigger the furry error overlay.

### 3. Test your environment using ErrorTest (Optional)

The component `<ErrorTest />` is writen to check if react-furry-error works well. Here is an example :

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

### 4. What errors can it catch?

react-furry-error can detect and display:

- Runtime JavaScript errors
- React render errors
- Hook misuse errors
- Promise / fetch failures
- Hydration mismatches (runtime)
- Hot Module Replacement (HMR) runtime crashes

⚠️ Compile-time syntax errors (e.g. invalid JSX) cannot be intercepted by any runtime library. When the app cannot start, no overlay can be mounted.
