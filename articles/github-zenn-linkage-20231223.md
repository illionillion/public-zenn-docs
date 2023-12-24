---
title: "Tensorflow.jsで画像から人を検出するアプリのサンプルを作ってみた"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "Tensorflow", "ESM", "importmap", "CDN"]
published: true
---

# はじめに

Tensorflow.jsで画像から人検出するアプリを作ったので解説を書いていきます。

# Tensorflow.jsとは？

JavaScriptで機械学習を行うためのオープンソースのライブラリで、事前いさまざまなモデルがトレーニング済みなのでそれを使用してブラウザ上で様々な処理ができます。

# ディレクトリ構成

```sh
.
├── css
│   └── style.css
├── index.html
└──── js
      └── main.js
```

動作させるにはサーバー環境が必要なので、XAMPP・Live Serverなどサーバー環境を用意しましょう。

自分はNode.jsの`http-server`を使います。

```sh
npm init -y # package.json作成
npm i -D http-server # http-serverインストール
```

`package.json`を以下のように編集

```json
{
  "name": "tensorflow-javascript-sample",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "server": "http-server -o .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "http-server": "^14.1.1"
  }
}
```

サーバー起動

```sh
npm run server
```

# HTML

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Tensorflow.jsサンプル</title>
    <link rel="stylesheet" href="./css/style.css" />
    <script type="importmap">
      {
        "imports": {
          "tfjs": "https://cdn.jsdelivr.net/npm/@tensorflow/tfjs",
          "coco-ssd": "https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"
        }
      }
    </script>
    <script type="module" src="./js/main.js"></script>
  </head>
  <body>
    <h1>オブジェクト検出サンプル</h1>
    <div class="input-container">
      <label for="file-input">画像を選択</label>
      <input type="file" id="file-input" accept="image/*" multiple />
    </div>
    <div class="canvas-container">
      <canvas id="result-canvas"></canvas>
    </div>
    <div id="loading-container" class="none">
      <p>処理中・・・</p>
    </div>
    <div id="result-preview" class="none">
      <img id="result-preview-image" src="" alt="" />
      <div class="result-controller">
        <input type="button" value="&lt;" id="prev-button" />
        <a id="result-preview-download" href="" download="">ダウンロード</a>
        <input type="button" value="&gt;" id="next-button" />
      </div>
    </div>
  </body>
</html>
```

今回はESMで記述するので、`importmap`を使用します。

```html
<script type="importmap">
  {
    "imports": {
      "tfjs": "https://cdn.jsdelivr.net/npm/@tensorflow/tfjs",
      "coco-ssd": "https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"
    }
  }
</script>
<script type="module" src="./js/main.js"></script>
```

# CSS

```css
* {
  margin: auto;
  padding: auto;
}

h1 {
  text-align: center;
  padding-block: 1rem;
  text-shadow: 1px 2px 3px #808080;
}

.input-container {
  max-width: 300px;
  min-width: 300px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-content: center;
}

.input-container label[for="file-input"] {
  width: 100%;
  text-align: left;
}

.input-container #file-input {
  width: 100%;
  text-align: left;
}

.canvas-container {
  display: none;
}

.none {
  display: none;
}

#result-preview {
  width: 50%;
  padding-block: 1rem;
  text-align: center;
}

#result-preview #result-preview-image {
  width: 100%;
}

#loading-container {
  text-align: center;
  padding-block: 1rem;
}

.result-controller {
  display: flex;
}

#result-preview-download {
  flex: 5;
}

#prev-button {
  flex: 1;
}

#next-button {
  flex: 1;
}
```

# JavaScript

```js
"use strict";
import "tfjs";
import "coco-ssd";
(() => {
  let imageList = [];
  let currentIndex = 0;

  const handleFileChange = async (e) => {
    if (e.target.files.length === 0) return;
    const loading = document.getElementById("loading-container");
    const resultPreview = document.getElementById("result-preview");
    loading.classList.remove("none");
    resultPreview.classList.add("none");

    imageList = [];
    currentIndex = 0;

    for (let i = 0; i < e.target.files.length; i++) {
      const file = e.target.files[i];
      const imageObj = new Image();
      imageObj.src = URL.createObjectURL(file);

      // モデルの読み込み
      const model = await cocoSsd.load();
      // 画像からオブジェクト認識
      const predictions = await model.detect(imageObj);
      console.log(predictions);
      const canvas = document.getElementById("result-canvas");
      canvas.width = imageObj.width;
      canvas.height = imageObj.height;
      const context = canvas.getContext("2d");
      context.drawImage(imageObj, 0, 0, imageObj.width, imageObj.height);

      // 描画
      for (const p of predictions) {
        if (p.class === "person") {
          context.beginPath();
          context.rect(...p.bbox);
          context.lineWidth = 1;
          context.fillStyle = generateRandomColor();
          context.stroke();
          context.fill();
        }
      }

      const base64 = canvas.toDataURL("image/jpeg");
      imageList.push({
        name: file.name,
        url: base64,
      });
    }

    resultPreview.classList.remove("none");
    updatePreview();
    loading.classList.add("none");
  };

  const prev = () => {
    if (currentIndex === 0 || !imageList.length) return;
    currentIndex--;
    updatePreview();
  };

  const next = () => {
    if (currentIndex >= imageList.length - 1 || !imageList.length) return;
    currentIndex++;
    updatePreview();
  };

  const updatePreview = () => {
    document.getElementById("result-preview-image").src =
      imageList[currentIndex].url;
    document.getElementById("result-preview-download").href =
      imageList[currentIndex].url;
    document.getElementById("result-preview-download").download =
      imageList[currentIndex].name;
  };

  const generateRandomColor = () => {
    const rand = () => Math.floor(Math.random() * 256);
    const r = rand();
    const g = rand();
    const b = rand();
    console.log(`rgb(${r},${g},${b})`);
    return `rgb(${r},${g},${b})`;
  };

  const main = async () => {
    document
      .getElementById("file-input")
      .addEventListener("change", handleFileChange);
    document.getElementById("prev-button").addEventListener("click", prev);
    document.getElementById("next-button").addEventListener("click", next);
  };

  window.addEventListener("load", main);
})();
```

これで完成です。
実際に人が写っている写真を選択するとマスキングされた画像が生成されます。

Webアプリ：https://illionillion.github.io/tensorflow-javascript-sample/
Repo：https://github.com/illionillion/tensorflow-javascript-sample/