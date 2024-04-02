---
title: "Chakra UI使うならYamada UIを使おう"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["YamadaUI", "ChakraUI", "React", "TypeScript", "UI"]
published: true
---

# なぜYamada UIを勧めるのか

自分は以前まではChakra UIをよく使っていました。

理由としては、Material UIなど他のライブラリとは違ってCSSの値をそのままコンポーネントに指定できて、開発がしやすいからです。

```tsx
<Box
    color="red"
    mx="auto"
>
    Box
</Box>
```

ですが、この半年間Yamada UIの開発に携わって、Chakra UIを使うよりもYamada UIを使った方が良いと思うようになりました。

理由としては

- Yamada UIはChakra UIより便利なコンポーネントが多い
- また、CSSのプロパティも多いので使いやすい
- Yamada UIはChakra UIを参考に作られていてプロパティ名がほぼ同じなので移行しやすい
- 日本製のOSSなので公式ドキュメントが日本語対応している

からです。

`npm i @yamada-ui/react`を実行するだけインストールできて始めやすいので是非キャッチアップしてみてください！！
（Chakra UIだとインストールにいくつかライブラリ入れなきゃいけない）

[Yamada UIの解説](https://zenn.dev/hirotomoyamada/articles/15b6f46d12841b)
[Yamada UIドキュメント](https://yamada-ui.com/)
