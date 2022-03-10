## 29 Rails Heroku の PostgreSQL を初期化する 〜デプロイ編〜

- `root $ cd api`を実行<br>

* `api $ tree config -L 1`を実行して確認してみる<br>

```:terminal
config
├── application.rb
├── boot.rb
├── cable.yml
├── credentials.yml.enc
├── database.yml
├── environment.rb
├── environments
├── initializers
├── locales
├── master.key
├── puma.rb
├── routes.rb
├── spring.rb
└── storage.yml

3 directories, 11 files
```

- `api $ pbcopy < config/master.key`を実行する<br>

* `api $ heroku config:set RAILS_MASTER_KEY=83776cbdbec83d15dfe1acddc2d6f30e`を実行<br>

```:terminal
Setting RAILS_MASTER_KEY and restarting ⬢ udemy-railsnuxtv2-api... done, v7
RAILS_MASTER_KEY: 83776cbdbec83d15dfe1acddc2d6f30e
```

- `api $ heroku config`で確認ができる<br>

```:terminal
=== udemy-railsnuxtv2-api Config Vars
DATABASE_URL:             postgres://ojvtalfglzoyht:dc62033f03237c4e32f7eb647da68c74459425ab66da79dbf18c0ead742bceb6@ec2-52-44-209-165.compute-1.amazonaws.com:5432/d6jqvf0m4deanr
RACK_ENV:                 production
RAILS_ENV:                production
RAILS_LOG_TO_STDOUT:      enabled
RAILS_MASTER_KEY:         83776cbdbec83d15dfe1acddc2d6f30e
RAILS_SERVE_STATIC_FILES: enabled
```

- `api $ heroku run rails db:migrate`を実行<br>

```:terminal
Running rails db:migrate on ⬢ udemy-railsnuxtv2-api... up, run.5112 (Free)
D, [2022-03-10T17:46:40.771930 #3] DEBUG -- :    (93.9ms)  CREATE TABLE "schema_migrations" ("version" character varying NOT NULL PRIMARY KEY)
D, [2022-03-10T17:46:40.790423 #3] DEBUG -- :    (12.4ms)  CREATE TABLE "ar_internal_metadata" ("key" character varying NOT NULL PRIMARY KEY, "value" character varying, "created_at" timestamp(6) NOT NULL, "updated_at" timestamp(6) NOT NULL)
D, [2022-03-10T17:46:40.833281 #3] DEBUG -- :    (1.2ms)  SELECT pg_try_advisory_lock(2992471314263377065)
D, [2022-03-10T17:46:40.871271 #3] DEBUG -- :    (1.6ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
D, [2022-03-10T17:46:40.882746 #3] DEBUG -- :   ActiveRecord::InternalMetadata Load (1.2ms)  SELECT "ar_internal_metadata".* FROM "ar_internal_metadata" WHERE "ar_internal_metadata"."key" = $1 LIMIT $2  [["key", "environment"], ["LIMIT", 1]]
D, [2022-03-10T17:46:40.890165 #3] DEBUG -- :    (1.2ms)  BEGIN
D, [2022-03-10T17:46:40.894414 #3] DEBUG -- :   ActiveRecord::InternalMetadata Create (3.7ms)  INSERT INTO "ar_internal_metadata" ("key", "value", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "key"  [["key", "environment"], ["value", "production"], ["created_at", "2022-03-10 08:46:40.888212"], ["updated_at", "2022-03-10 08:46:40.888212"]]
D, [2022-03-10T17:46:40.897025 #3] DEBUG -- :    (2.0ms)  COMMIT
D, [2022-03-10T17:46:40.898423 #3] DEBUG -- :    (1.2ms)  SELECT pg_advisory_unlock(2992471314263377065)
```

- `api $ heroku open /api/v1/hello`を実行すると heroku の api が開ける<br>

* `api $ heroku ps`を実行(config/puma.rb が表示されていれば OK)<br>

```:terminal
Free dyno hours quota remaining this month: 999h 20m (99%)
Free dyno usage for this app: 0h 3m (0%)
For more information on dyno sleeping and how to upgrade, see:
https://devcenter.heroku.com/articles/dyno-sleeping

=== web (Free): /bin/sh -c bundle\ exec\ puma\ -C\ config/puma.rb (1)
web.1: up 2022/03/10 17:43:18 +0900 (~ 7m ago)
```

- `api $ heroku run sh`を実行して heroku のコンテナに入る<br>

`$ date`を実行する(JST になっていれば OK)<br>

```:terminal
Thu Mar 10 17:54:03 JST 2022
```

`$ exit`を実行してコンテナから抜ける<br>

`api $ heroku pg:info`を実行<br>

```:terminal
=== DATABASE_URL
Plan:                  Hobby-dev
Status:                Available
Connections:           0/20
PG Version:            13.5
Created:               2022-03-09 04:55 UTC
Data Size:             8.0 MB/1.00 GB (In compliance)
Tables:                0
Rows:                  0/10000 (In compliance) - refreshing
Fork/Follow:           Unsupported
Rollback:              Unsupported
Continuous Protection: Off
Add-on:                postgresql-encircled-05500
```

`api $ heroku pg:psql postgresql-encircled-05500`を実行(下記のようになれば DB の中に入っていることになる)<br>

```:terminal
--> Connecting to postgresql-encircled-05500
psql (13.0, server 13.5 (Ubuntu 13.5-2.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

udemy-railsnuxtv2-api::DATABASE=>
```

`api $ udemy-railsnuxtv2-api::DATABASE=> show timezone;`を実行(UTC になっていれば OK)<br>

```
 TimeZone
----------
 Etc/UTC
(1 row)
```

+`api $ udemy-railsnuxtv2-api::DATABASE=> exit`実行して抜ける<br>

## 31 Nuxt.js DockerFile の編集と heroku.yml の作成 〜デプロイ準備編〜<br>

- `front/Dockerfile`を編集<br>

```:Dockerfile
# FROM node:14.4.0-alpine 使用不可
FROM node:16.13.1-alpine

ARG WORKDIR
ARG API_URL

ENV HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo \
  # これを指定しないとブラウザからhttp://localhost へアクセスすることができない。
  # コンテナのNuxt.jsをブラウザから参照するためにip:0.0.0.0に紐付ける
  # https://ja.nuxtjs.org/faq/host-port/
  HOST=0.0.0.0 \
  API_URL=${API_URL}

WORKDIR ${HOME}

# 追加
# ローカル上のpackageをコンテナにコピーしてインストールする
COPY package*.json ./
RUN yarn install

# コンテナにNuxtプロジェクトをコピー
COPY . ./

# 本番環境にアプリを構築
RUN yarn run build
# ここまで

# 公開用ポート番号を指定
# 3000番が入ってくる(http://localhost(0.0.0.0):3000)
# EXPOSE ${CONTAINER_PORT}

# 2021.12.13追記
# FROM node:14.15.1-alpine
# node v14.15.1は、$ yarn create nuxt-app appコマンド時に下記エラーが発生するので使用不可
# eslint-plugin-vue@8.2.0: The engine "node" is incompatible with this module. Expected version "^12.22.0 || ^14.17.0 || >=16.0.0". Got "14.15.1"

# create nuxt-appコマンド成功確認済みのnode version
# FROM node:16.13.1-alpine
# or
# FROM node:16-alpine(node v16.13.1)

# 現在のnode推奨版はこちらから => https://nodejs.org/ja/download/
```

- `root $ cd front`を実行<br>

- `front $ touch heroku.yml`を実行<br>

```yml:heroku.yml
setup:
  config:
    NODE_ENV: production
build:
  docker:
    web: Dockerfile
  config:
    WORKDIR: app
    API_URL: 'https://udemy-railsnuxtv2-api.herokuapp.com'
run:
  web: yarn run start
```

- `front`ディレクリを git commit する<br>
