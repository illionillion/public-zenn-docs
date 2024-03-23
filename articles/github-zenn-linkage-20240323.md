---
title: "Next14のAppRouterでPageRouterのAPIを作る方法"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nextjs", "AppRouter", "PageRouter", "TypeScript"]
published: true
---

`next.config.js`内に`bodyParser: false`を記述

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // 以下を指定
  api: {
    bodyParser: false,
  },
};

module.exports = nextConfig;
```

これでAppRouterのプロジェクト内でもPageRouterのAPIを作ることができる