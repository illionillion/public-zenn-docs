---
title: "ツリーシェイキングと副作用 ツリーシェイキングが効く時と効かない時"
emoji: "🌳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "TypeScript", "React", "Vite", "Webpack"]
published: true
---

# 用語の整理：「ビルド」と「バンドル」

一般的に「ビルド」は、ソースコードを本番用に変換する一連の工程全体（トランスパイル・圧縮・バンドルなどを含む広義の用語）を指します。

一方「バンドル」は、その中の一工程で、複数のファイルやモジュールを1つまたは少数のファイルにまとめる処理です。ツリーシェイキングもこのバンドル工程の中で行われます。

> **補足：「ビルド」と「バンドル」の関係**
>
> 「ビルド」は、ソースコードを本番用に変換・最適化する全体の工程を指し、その中に「バンドル」（複数ファイルの結合）や「トランスパイル」「圧縮」などが含まれます。  
> 「バンドル」は「ビルド」の一工程（または一形態）であり、ツリーシェイキングもバンドル工程の中で実行されます。

本記事では“バンドルツール”という表現を用いていますが、これはビルド工程の中でも特にバンドル（ファイル結合・最適化）を担うツール（Webpack, Vite, Rollupなど）を指します。

---

# ツリーシェイキングとは

WebpackやViteなどの現代的なバンドルツールが持つプロセスの一つで、プログラム内の未使用の部分（使われていない関数や変数など）を自動で削除してくれる最適化処理です。

これにより、未使用の関数や変数、ライブラリの一部などはバンドルサイズに含まれなくなります。
たとえば、`node_modules`内のライブラリでも、使っていない部分はバンドルから除外されます（ただし形式や副作用によっては除外されない場合もあります）。

名前は、木を揺らして不要な葉を落とすことに例えています。

![ツリーシェイキングのイメージ](https://storage.googleapis.com/zenn-user-upload/6765e27768ab-20250614.gif)

# `rollup-plugin-visualizer`でバンドルファイルを可視化

以下のような構成の場合コンポーネントA・B・Cどれがバンドルに含まれるのでしょうか？

![サンプル](https://storage.googleapis.com/zenn-user-upload/db439813a96f-20250614.png)

:::details 詳しい実装

ディレクトリ構成

```
.
└── src/
    ├── App.tsx
    └── components/
        ├── index.ts
        ├── component-a.tsx
        ├── component-b.tsx
        └── component-c.tsx
```

`App.tsx`で`<ComponentA />`と`<ComponentB />`を読み込んでる

```tsx
import { ComponentA, ComponentB } from "./components"

function App() {

  return (
    <>
      <ComponentA />
      <ComponentB />
    </>
  )
}

export default App
```

`components/index.ts`でA・B・Cを再エクスポート

```ts
export { ComponentA } from "./component-a";
export { ComponentB } from "./component-b";
export { ComponentC } from "./component-c";
```

`/components/component-a.tsx`

```tsx
import { FC } from "react";

export const ComponentA:FC = () => {
    return <div>コンポーネントA</div>
}
```

`/components/component-b.tsx`

```tsx
import { FC } from "react";

export const ComponentB:FC = () => {
    return <div>コンポーネントB</div>
}
```

`/components/component-c.tsx`

```tsx
import { FC } from "react";

export const ComponentC:FC = () => {
    return <div>コンポーネントC</div>
}
```

:::

最初の説明の話を踏まえると、正解は`<ComponentA />`と`<ComponentB />`だけがバンドルに含まれ、`<ComponentC/>`はバンドルには含まれません。

ビルド後のバンドルファイルを[rollup-plugin-visualizer](https://www.npmjs.com/package/rollup-plugin-visualizer)で可視化すると以下のようになります。

https://www.npmjs.com/package/rollup-plugin-visualizer


![可視化されたバンドルファイル](https://storage.googleapis.com/zenn-user-upload/30a545653d9e-20250614.png)

これで実際にツリーシェイキングがどのように作用するかを理解できたと思います。

# ツリーシェイキングが効くための前提条件

そんなツリーシェイキングですが、これが効くのには条件があります。

## ESM形式である

コードを静的解析して未使用の部分を削除するので、`import / export`の形式で書かれている必要があります。
CJS時代の`require`やもっと昔の`<script src="./js/main.js" ></script>`を複数書いてグローバルに読み込ませる形式では静的解析できないのでツリーシェイキングが効きません。

## バンドルツールを使っている

Reactなどは基本WebpackやViteなどのバンドルツールが付いているので意識することはないですが、Vanillaで`<script type="module" src="./js/main.js"></script>`のようにESM形式で読み込ませる場合、バンドルツールを使っていないので未使用の部分はそのまま読み込まれます。

:::details Vanilla環境での例

`index.html`

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sample</title>
    <script type="module" src="./js/main.js"></script>
</head>
<body>
</body>
</html>
```

`main.js`

```js
import { componentA, componentB } from "./components/index.js";

function main() {
    componentA();
    componentB();
    // バンドルツールを使用していないので未使用のcomponentCも読み込まれる
}

document.addEventListener("DOMContentLoaded", main);
```

`components/index.js`

```js
export { componentA } from "./component-a.js";
export { componentB } from "./component-b.js";
export { componentC } from "./component-c.js";
```

`components/component-a.js`

```js
export function componentA() {
  console.log("Component A initialized");
  document.body.appendChild(
    document.createElement("div")
  ).textContent = "Component A is active";
}
```

`components/component-b.js`

```js
export function componentB() {
  console.log("Component B initialized");
  document.body.appendChild(
    document.createElement("div")
  ).textContent = "Component B is active";
}
```

`components/component-c.js`

```js
export function componentC() {
  console.log("Component C initialized");
  document.body.appendChild(
    document.createElement("div")
  ).textContent = "Component C is active";
}
```

このように、バンドルツールを使わずESM形式で読み込んだ場合、未使用の`componentC`もネットワーク経由で読み込まれ、開発者ツールの「ソース」タブなどで確認できます。

![Vanilla環境の例](https://storage.googleapis.com/zenn-user-upload/ecefba0bae68-20250614.png)

:::

# 前提条件を満たしていてもツリーシェイキングが効かない場合

これらの条件を満たしていてもツリーシェイキングが効かない場合があります。

![ツリーシェイキングが効かなかった場合](https://storage.googleapis.com/zenn-user-upload/1660197b1025-20250614.png)

## 効かない時1：全部同じファイル内にある時

1つのファイル内で複数の関数やコンポーネントを定義し、まとめてexportしている場合、バンドルツールによっては未使用のものもバンドルに含まれてしまうことがあります。特にデフォルトエクスポートや、export文がファイル末尾にまとめて書かれている場合は注意が必要です。

```tsx
// index.tsx にA/B/Cすべて定義されている例
// AとBのみをimportしていてもCは含まれてしまう

export const ComponentA = () => {
  return <div>コンポーネントA</div>;
};

export const ComponentB = () => {
  return <div>コンポーネントB</div>;
};

export const ComponentC = () => {
  return <div>コンポーネントC</div>;
};
```

このように1ファイルにまとめて定義・exportしている場合、たとえComponentCを使っていなくても、バンドルツールによってはCもバンドルに含まれてしまうことがあります。

## 効かない時2: `export default`でまとめてエクスポートしている時

`export default { ComponentA, ComponentB, ComponentC }` のようにオブジェクトでまとめてdefault exportし、`import Components from './components'` のようにdefault importで受け取る場合は、バンドルツールがどのプロパティが使われているか静的に判断できず、未使用のものもバンドルに含まれてしまうことがあります。

### 例：export defaultでまとめている場合

```ts
// components/index.ts
import { ComponentA } from './component-a';
import { ComponentB } from './component-b';
import { ComponentC } from './component-c';

export default { ComponentA, ComponentB, ComponentC };
```

```tsx
// App.tsx
import Components from './components';

function App() {
  return (
    <>
      <Components.ComponentA />
      <Components.ComponentB />
    </>
  );
}
```

このようにdefault exportでまとめている場合、未使用のComponentCもバンドルに含まれる可能性が高くなります。

## 効かない時3: 副作用がある場合

たとえ`<ComponentC/>`を使っていなくても、`/components/component-c.tsx`内でトップレベルに何らかの処理（例: `console.log()`やグローバル変数の書き換えなど）が書かれていると、それが「副作用」とみなされ、バンドルツールは安全のためそのモジュール全体をバンドルに含めてしまいます。

```tsx
import { FC } from "react";

export const ComponentC: FC = () => {
  return <div>コンポーネントC</div>;
};

console.log("componentC"); // このような副作用があると未使用でもバンドルに含まれる
```

このような副作用がある場合、たとえimportされていなくても、バンドルサイズが大きくなる原因となります。

# 補足：`sideEffects: false`で副作用を無効化

`package.json`で`sideEffects: false`を指定すると、「このプロジェクト（または特定ファイル）には副作用がない」とバンドルツールに伝えることができます。これにより、未使用のimportがあっても副作用を気にせず安全に除去でき、バンドルサイズをさらに小さくできます。

```json
{
  "sideEffects": false
}
```

ただし、単に副作用が「ない」かどうかではなく、たとえ副作用があっても`sideEffects: false`で無視してもバンドル後の挙動に影響が出ないかどうかを十分に確認した上で指定してください。副作用があるのに`false`にすると、バンドル後の挙動が変わる危険があります。OSSのライブラリでもバンドルサイズ削減のために指定されていることがありますが、利用時は注意が必要です。

https://github.com/yamada-ui/yamada-ui/blob/97e70ca854e85c3dea7a5ed1c584a00e79ea1992/packages/components/accordion/package.json#L25

![Yamada UI](https://storage.googleapis.com/zenn-user-upload/1d632da56645-20250615.png)

---

# バンドルサイズを小さくするのに気をつけること

- ファイルは論理的に分割し、不要な依存がバンドルに含まれないようにする（細かくしすぎると管理が煩雑になるためバランスも大事）
- `export default {}` でオブジェクト形式でまとめてexportしない（必要なものだけ個別export・importを使う）
- 副作用がないか、または副作用をsideEffects: falseで無視しても本当に影響がないかを必ず確認する
- `rollup-plugin-visualizer`などでバンドルサイズや内容を定期的に可視化・確認する
- 使用しているライブラリがESM形式で、ツリーシェイキングに対応しているか調べる
- CommonJS形式（CJS）のライブラリは極力避ける、またはESM版があればそちらを使う
- 名前空間import（import * as ...）やdefault exportのまとめは避け、必要なものだけ個別importする
- package.jsonの`sideEffects`フィールドの設定を適切に行う

※主要なバンドルツール（Webpack, Vite, Rollupなど）はここで紹介した挙動にほぼ準拠していますが、細かなバージョンや設定によって例外もあるため、公式ドキュメントや実際のバンドル結果も確認しましょう。

これらの点を意識することで、無駄なコードを含まない最適なバンドルを作ることができます。

このリポジトリで検証ができるので、気になる方はクローンして動かしてみてください
フォーク・プルリクエスト・スター大歓迎です！！

https://github.com/illionillion/tree-shaking-learning