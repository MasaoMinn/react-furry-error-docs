#react-furry-error 実装メカニズム

このセクションでは、react-furry-error の内部実装の考え方を簡単に説明し、開発者がそのコア動作メカニズムを理解できるようにします。

### 1.  グローバルエラーリスニング

ライブラリのコアエントリーポイントはブラウザレベルのグローバルエラーリスニングであり、以下のネイティブイベントをバインドすることで、さまざまなランタイム例外をキャプチャします。

- window.onerror：同期的な JavaScript エラーをキャプチャ
- window.onunhandledrejection：未処理の Promise 拒否をキャプチャ（fetch の失敗、async/await の未キャッチエラーなど）

上記のリスニングを通じて、次のような重要なエラー情報を取得できます：
- エラーの説明メッセージ（error message）
- エラーが発生したファイル名、行番号、列番号
- エラースタックトレース（ブラウザがサポートしている場合）

以下は index.ts の実装です：

```javascript
export function initFurryDevOverlay(): void {
  // 初期化済みとしてマーク
  setInitialized();

  // WebSocket パッチの初期化（enabled の値に関係なく保持）
  patchWebSocket();
  // 有効な場合：カスタムエラーオーバーレイを使用
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

### 2.  エラー分類ロジック

エラーメッセージに基づいてエラータイプを大まかに判断し、エラーを 4 つのタイプに分類し、それぞれに固有のファーリーキャラクター画像を対応させています。以下は classify.ts の分類ロジックコードです：

```javascript
// classify.ts

import type { FurryImageType } from "./types";

export function classifyFurryType(message: string): FurryImageType {
  const text = message.toLowerCase();

  // Hook 関連のエラー（最優先）
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

  // DOM/hydration エラー（第二優先）
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

  // Searching カテゴリー（リソースが不足している、見つからない、実行に失敗した）
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

  // デフォルトのフォールバック
  return "confused";
}

```

### 3. オーバーレイのレンダリング方法

エラーが発生した後、アプリケーション自体に影響を与えないように、独立して分離されたレンダリングロジックによってエラーオーバーレイが表示されます：
独立した DOM コンテナ（div#react-furry-error-overlay）を動的に作成
React 18 の新しい createRoot API を使用してコンポーネントをマウント（コンカレントレンダリング機能をサポート）
分離された ErrorOverlay コンポーネントをレンダリング（アプリケーションの React ツリーと完全に分離）
オーバーレイのコア表示内容：
エラータイプに対応する可愛いイラスト
簡潔なエラー説明（重要な情報を抽出）
折りたたみ可能な詳細なスタックトレース（デバッグ用）
クイックリフレッシュボタン（アプリケーションのワンクリック再起動）



### 4. セキュリティとクリーンアップメカニズム
アプリケーション環境を汚染せず、メモリリークがないことを保証するために、厳格なセキュリティポリシーが設計されています：
🛡️ オーバーレイはシングルトンパターンを使用：グローバルには 1 つのルートインスタンスのみが存在し、繰り返しレンダリングを回避
🧹 自動クリーンアップ：新しいエラーが発生する前に前のオーバーレイインスタンスをアンマウントし、DOM とメモリを解放
🚧 分離された設計：アプリケーションの React ルートノードを変更せず、グローバル変数を注入せず、アプリケーション自身のロジックを妨げない
コアサマリー
react-furry-error の本質は：
ランタイムエラーのみに焦点を当てた軽量ツールで、「非侵入的なリスニング + 分類可視化」の設計により、React の内部メカニズムを変更することなく、開発中のエラー理解とデバッグのコストを削減します。