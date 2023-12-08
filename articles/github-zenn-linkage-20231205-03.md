---
title: "【Next.js】AppRouterでお問い合わせフォームを作ってみた！！"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nextjs", "AppRouter", "TypeScript", "Docker", "React"]
published: false
---

# はじめに

Next.js 13でリリースされたAppRouterを活用してお問い合わせフォオームをインプット・アウトプットとして作ってみます。

# 使用技術

以下の物を使用

- Next.js（AppRouter）
- TypeScript
- Chakra UI
- MySQL
- Docker

# 環境構築

空のフォルダを作成し以下のようなディレクトリ構成にする

```
.
├── .env
├── .gitignore
├── Dockerfile
├── app
├── docker-compose.yml
├── initdb.d
│   └── create-table.sql
└── mysql
    └── my.cnf
```

`Dockerfile`には以下を記述

```Dockerfile
FROM node:lts-alpine  
WORKDIR /app
```

`docker-compose.yml`には以下を記述

```yml
version: '3'
services:
  app:
    container_name: app
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./app:/app
    command: sh -c "npm run dev"
    tty: true
    ports:
      - 3000:3000
    environment:
      - DB_HOST=${MYSQL_HOST}
      - DB_PORT=${MYSQL_PORT}
      - DB_USER=${MYSQL_USER}
      - DB_PASSWORD=${MYSQL_PASSWORD}
      - DB_DATABASE=${MYSQL_DATABASE}
    depends_on:
      - mysql
  mysql:
    container_name: db
    image: mysql:latest
    restart: always
    ports:
      - ${MYSQL_PORT}:${MYSQL_PORT}
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - TZ=${TZ} # タイムゾーン
    volumes:
      - ./initdb.d:/docker-entrypoint-initdb.d # DBの初期データ
      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf # bashで日本語文字化けする問題の解決
```

`/initdb.d/create-table.sql`は以下のように記述する

```sql
CREATE TABLE contact_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    content_question TEXT NOT NULL,
    postdate DATETIME(6) DEFAULT CURRENT_TIMESTAMP(6)
);
```

`/mysql/my.cnf`は以下

```conf
[mysqld]
character_set_server = utf8mb4
collation-server=utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4

[client]
default-character-set = utf8mb4
```

`.gitignore`

```gitignore
.DS_Store

.env.local
.env
```

`.env`

```
MYSQL_HOST=mysql
MYSQL_USER=root
MYSQL_PORT=3306
MYSQL_PASSWORD=password
MYSQL_DATABASE=contactform
TZ=Asia/Tokyo
```

必要なファイルの作成ができたのでこれからNext.jsの環境を作成します。
以下を実行

```sh
docker compose run --rm app npx create-next-app
```

- プロジェクト名は「.」を入力
- TypeScriptを選択
- ESLintを選択
- Tailwind CSSを選択
- `src/`ディレクトリを使うを選択
- App Routerを選択

![](https://storage.googleapis.com/zenn-user-upload/8a097e079b6d-20231205.png)

これで`/app`フォルダ内にNext.jsの環境ができました。

試しに起動するか確認しましょう

```sh
docker compose down # 念の為、コンテナを修了
docker compose up -d
```

`localhost:3000`にアクセスするとNext.jsのページが表示されると思います。

データベースの確認は以下のコマンドでできます。

```sh
docker exec -it db sh # コンテナの中に入る
mysql -u root -p # DBにログイン
use contactform; # データベースを選択
show tables; # テーブル一覧を表示

exit # 表示できたらSQLを終了
exit # コンテナからも出る
```

これで`contact_table`のテーブルが表示されればうまくデータベースは動いています。

![](https://storage.googleapis.com/zenn-user-upload/f954915461ce-20231208.png)

次に使用するライブラリのインストールをします。

```sh
docker exec -it ap sh # コンテナの中に入る
npm i framer-motion mysql2
npm i -D @chakra-ui/react @emotion/react @emotion/styled @types/mysql
exit # インストールできたらコンテナから出る
```