# react-furry-error インストールガイド

## インストール

### 1. パッケージをインストール

npm を使用:

```bash
npm install react-furry-error
```

または pnpm / yarn を使用:

```bash
pnpm add react-furry-error
# または
yarn add react-furry-error
```

### 2. オーバーレイを有効にする

アプリケーションのエントリーファイル（例: main.tsx または index.tsx）で：

```tsx
import { initFurryDevOverlay} from "react-furry-error";

if (import.meta.env.MODE === "development") {
  initFurryDevOverlay();
}
```

以上です。
有効にすると、ランタイムエラーが自動的にファーリーエラーオーバーレイをトリガーします。

### 3. ErrorTest を使用して環境をテストする（オプション）

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

### 4. どのようなエラーをキャッチできますか？

react-furry-error は以下のエラーを検出して表示できます：

- ランタイム JavaScript エラー
- React レンダリングエラー
- Hook の誤用エラー
- Promise / fetch の失敗
- ハイドレーションの不整合（ランタイム）
- ホットモジュールリプレースメント（HMR）ランタイムクラッシュ

⚠️ コンパイル時の構文エラー（例：無効な JSX）は、どのランタイムライブラリでもインターセプトできません。アプリケーションが起動できない場合、オーバーレイをマウントできません.
