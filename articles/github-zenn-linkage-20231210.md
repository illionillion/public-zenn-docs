---
title: "WebNFCでNFC読み書きWebアプリを作った話"
emoji: "💳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "TypeScript", "NFC", "WebNFC"]
published: true
---

# はじめに

ブラウザのAPI **WebNFC** を使ってNFCを読み書きできるアプリを作った話の記事です。

# NFCとは？

NFC（Near Field Communication）とは「近距離無線通信」を意味する技術・規格で、非接触ICチップを使ってかざすだけで通信できる通信規格です。
おサイフ機能付きのスマートフォンや、Suica、PASMOなどの交通系IC、家電、交通機関で使用する IC カードなど、幅広い機器で活用されています。

それの読み書きをブラウザでする技術がWebNFCです。

https://developer.mozilla.org/ja/docs/Web/API/Web_NFC_API

# 開発したアプリについて

リンクは以下になります。

アプリ：https://illionillion.github.io/react-read-nfc-app/read
リポジトリ：https://github.com/illionillion/react-read-nfc-app

書き込み機能
![](https://storage.googleapis.com/zenn-user-upload/2a87ab36b486-20231215.jpg)

読み込み機能
![](https://storage.googleapis.com/zenn-user-upload/054b5eac187c-20231215.jpg)

- テキスト
- URL
- JSON

でのデータの読み書きが可能ですが、Android端末での一部のブラウザでしか動作しません。
https://developer.mozilla.org/ja/docs/Web/API/Web_NFC_API#%E3%83%96%E3%83%A9%E3%82%A6%E3%82%B6%E3%83%BC%E3%81%AE%E4%BA%92%E6%8F%9B%E6%80%A7

# 使用技術

- React
- TypeScript
- Chakra UI
- GitHub Actions
- PWA

# 実装のメイン部分

Reactで書いているので見た目は大分違いますが、読み書きの部分はここなどを参考にしました。

https://zenn.dev/ktpi2000/articles/webnfc-playground

# まとめ

- ブラウザでNFCの読み書きができる（テキスト・URL・JSONなど）
- 画像などのバイナリはbase64に変換してもNFCの容量的に書き込み不可
- 使える端末やブラウザに制限がある
- 交通系ICなどは対応しておらず、あくまで市販のNFCのみ対応
- 開発環境で `https://` が使える環境でないといけない（今回はViteでSSL化）
- 実際にNFCを操作するならネイティブアプリで実装した方が良い
