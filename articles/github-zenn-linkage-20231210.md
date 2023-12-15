---
title: "WebNFCでNFC読み書きWebアプリを作った話"
emoji: "💳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "TypeScript", "NFC", "WebNFC"]
published: false
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

でのデータの読み書きが可能です。

# 使用技術

- React
- TypeScript
- Chakra UI
- GitHub Actions
- PWA

# 実装のメイン部分

