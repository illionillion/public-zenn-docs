---
title: "ツリーシェイキングと副作用 ツリーシェイキングが効く時と効かない時"
emoji: "🌳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "TypeScript", "React"]
published: false
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
    // ビルドツールを使用していないので未使用のcomponentCも読み込まれる
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

## 効かない時3: 副作用が効いている時

モジュール内でグローバル変数の書き換えや、import時に何らかの処理（副作用）が発生する場合、バンドルツールは安全のためそのモジュール全体をバンドルに含めてしまいます。

# 補足：`sideEffect: false`で副作用を無効化

---

# バンドルサイズを小さくするのに気をつけること

<!-- ## その他ツリーシェイキングが効かない場合

上記以外にも、以下のようなケースではツリーシェイキングが効きません：

- **CommonJS形式（CJS）で書かれている場合**
  - `require`や`module.exports`を使ったCJS形式は静的解析が難しく、未使用部分が除去されません。
- **動的importや動的参照**
  - `import('./foo.js')`や`obj[funcName]()`のような動的な参照は静的解析できず、すべて含まれることがあります。
- **バンドルツールや設定の問題**
  - バンドルツールの設定（`sideEffects: true`や`minify: false`など）によってはツリーシェイキングが効かない場合があります。
- **Babelなどのトランスパイル後に副作用が入る場合**
  - Babelの設定や一部プラグインによっては、tree-shakingしにくいコードに変換されてしまうことがあります。

これらの点にも注意しながら、バンドルサイズの最適化を意識しましょう。 -->