---
title: "AppRouterのAPIでテストコードを書く方法"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["NextJS", "Jest", "TypeScript", "API", "Test"]
published: true
---

以下のようなAPIがあるとする

```ts:/src/app/api/hello/route.ts
import { NextResponse } from "next/server";

export async function GET() {
    return NextResponse.json({
        message: "Hello!!"
    })
}
```

[next-test-api-route-handler](https://www.npmjs.com/package/next-test-api-route-handler)をインストールしてテストコードを書く

```ts:/src/test/helle.test.ts
import * as appHandler from "@/app/api/hello/route";
import { testApiHandler } from "next-test-api-route-handler";
describe("HelloAPIのテスト", () => {
  test("Helloが返ってくるか", async () => {
    await testApiHandler({
      appHandler,
      async test({ fetch }) {
        const res = await fetch({
          method: "GET",
        });
        expect(await res.json()).toStrictEqual({
          message: "Hello!!"
        });
      },
    });
  });
});
```

`import * as appHandler from "@/app/api/hello/route";`の書き方で、`GET`関数を`appHandler`の中に入れる感じでインポートして、`testApiHandler`につっこんでいる。

`POST`やパラメータを渡して送信する際はこう

```ts:/src/app/api/send/route.ts
import * as appHandler from "@/app/api/send/route";
import { testApiHandler } from 'next-test-api-route-handler'; 

describe("Send APIのテスト", () => {
  test("sendで送信されるか", async () => {
    await testApiHandler({
      appHandler,
      async test({ fetch }) {
        const res = await fetch({ method: 'POST', body: JSON.stringify({
          name: "John Doe", email: "sample@email.com", content: "Hello World."
        }) });
        expect(await res.json()).toStrictEqual({ message: "送信に成功しました。" });
      }
    });
  });
});
```

`test`関数の引数のオブジェクト内にある`fetch`関数がブラウザなどで使う`fetch`関数と似たような実装になっている。

あとはファイルのインポートのパスが`@/*`から始まっているので、`jest.config.ts`でその設定などを忘れずにしたら動く。