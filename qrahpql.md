---
theme: "white"
---

<style type="text/css"> p,h1,li { text-align: left; }
</style>

## レンダリング最適化方法

---

### React Developer Tools

Facebook から提供されている公式の React 用のデバッグツール\
以下が可能になり、ページの状態を確認しながら開発ができる

- コンポーネントの構造を視覚的に把握できる
- コンポーネント間で渡されている Props の値が確認できる
- コンポーネント内の State の値が確認できる
- 実際に値を変えて動作の確認ができる

---

＜参考＞

- [Google Developer Tools のインストール](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)

---

### why did you render

- 不要な再レンダリングをコンソールで警告してくれるライブラリ
- オブジェクトが再生成された場合でも値が同じであれば警告を出す
- React Native、typescript 環境、でも使用可能
- React@18, React@17 , React@16 で動作保障

---

### 導入方法(typescript)

- インストール

```
npm install @welldone-software/why-did-you-render --save-dev
```

- wdyr.ts 作成

```typescript
/// <reference types="@welldone-software/why-did-you-render" />

import React from "react";

if (process.env.NODE_ENV === "development") {
  const whyDidYouRender = require("@welldone-software/why-did-you-render");
  whyDidYouRender(React);
}

---

- wdyr.tsをインポート
```

// index.tsx
import './wdyr'; // <--- first import
import React from "react";
import { createRoot } from "react-dom/client";
import App from "./components/App";

const container = document.getElementById("root");
const root = createRoot(container);

root.render(<App />);

```
---

### 動作確認

```

// App.tsx
import React, { useState } from "react";

const Child = () => {
return <p>子要素です</p>;
};

const App = () => {
const [count, setCount] = useState(0);
const addcount = () => {
setCount(count + 1);
};
return (
<>

<p>カウント：{count}</p>
<button onClick={addcount}>カウントアップ</button>
<Child />
</>
);
};
Child.whyDidYouRender = true; // <---監視対象とする

export default App;

```

---

### まとめ
- create react appで作成したプロジェクトには使えない
- ひとつずつ、whyDidYouRender = trueにしていく必要あり

```
