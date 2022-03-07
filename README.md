セクション 3: Docker を使った Rails+Nuxt.js 環境構築

※ `$ mkdir udemy_demoapp_v2 && cd $_`<br>

## 07 Rails を動かす Dockerfile を作成する 1/2 〜作成編〜

### ハンズオン

- `$ mkdir {api,front}`(スペースは空けない)を実行<br>

* `api`ディレクトリに移動する<br>

* `$ touch {Dockerfile,Gemfile,Gemfile.lock}`を実行<br>

- `api/Gemfile`ファイルを編集<br>

```
source 'https://rubygems.org'
gem 'rails', '~> 6.0.0' # 6.0.0以上、6.1.0未満で最新 = 6.0.x
```

- `api/Dockerfile`ファイルを編集<br>

```
FROM ruby:2.7.1-alpine

ARG WORKDIR

ENV RUNTIME_PACKAGES="linux-headers libxml2-dev make gcc libc-dev nodejs tzdata postgresql-dev postgresql git" \
  DEV_PACKAGES="build-base curl-dev" \
  HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo

# ENV test（このRUN命令は確認のためなので無くても良い）
RUN echo ${HOME}

WORKDIR ${HOME}

COPY Gemfile* ./

RUN apk update && \
  apk upgrade && \
  apk add --no-cache ${RUNTIME_PACKAGES} && \
  apk add --virtual build-dependencies --no-cache ${DEV_PACKAGES} && \
  bundle install -j4 && \
  apk del build-dependencies

COPY . .

CMD ["rails", "server", "-b", "0.0.0.0"]
```

## 08 Rails を動かす Dockerfile を作成する 2/2 〜解説編〜

- ベースイメージの Ruby バージョンを調べる手順(下記 URL を参照)<br>

* 参考: https://blog.cloud-acct.com/posts/u-rails-dockerfile/#%E3%83%99%E3%83%BC%E3%82%B9%E3%82%A4%E3%83%A1%E3%83%BC%E3%82%B8%E3%81%AEruby%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3%E3%82%92%E8%AA%BF%E3%81%B9%E3%82%8B%E6%89%8B%E9%A0%86 <br>

- `api/Dockerfile`を編集<br>

```
# ベースイメージを指定する
# FROM ベースイメージ:タグ(タグはなくてもよいが最新のものが指定されることになる)
FROM ruby:2.7.2-alpine

# Dockerfile内で使用する変数を定義
# appという値が入る
ARG WORKDIR

# 環境変数を定義(Dockerfile, コンテナ参照可能)
# Rails ENV["TZ"] => Asia/Tokyoが出力される
ENV RUNTIME_PACKAGES="linux-headers libxml2-dev make gcc libc-dev nodejs tzdata postgresql-dev postgresql git" \
  DEV_PACKAGES="build-base curl-dev" \
  # /app
  HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo

# ベースイメージに対してコマンドを実行する
# ${HOME}, $HOME => /app
RUN echo ${HOME}

# Dockerfime内で指定した命令を実行する・・・RUN, COPY, ADD, ENTORYPOINT, CMD
# 作業ディレクトリを定義
# コンテナ/app/Railsアプリ
WORKDIR ${HOME}

# ホスト側の(PC)のファイルをコンテナにコピー
# COPY コピー元(ホスト) コピー先(コンテナ)
# Gemfile* ... Gemfileから始まるファイルを全指定(Gemfile, Gemfile, Gemfile.lock)
# コピー元(ホスト) ... Dockerfileがあるディレクトリ以下を指定(api) ../ NG
# コピー先(コンテナ) ... 絶対パス or 相対パス(./ ... 今いる(カレント)ディレクトリ)
COPY Gemfile* ./

    # apk ... Alpine Linuxのコマンド
    # apk update = パッケージの最新リストを取得
RUN apk update && \
  # apk upgrade = インストールパッケージを最新のものに
  apk upgrade && \
  # apk add = パッケージのインストールを実行
  # --no-cache = パッケージをキャッシュしない(Dokcerイメージを軽量化)
  apk add --no-cache ${RUNTIME_PACKAGES} && \
  # --virtual 名前(任意) = 仮想パッケージ
  apk add --virtual build-dependencies --no-cache ${DEV_PACKAGES} && \
  # Gemのインストールコマンド
  # -j4(jobs=4) = Gemインストールの高速化
  bundle install -j4 && \
  # パーケージを削除(Dokcerイメージを軽量化)
  apk del build-dependencies

# . ... Dockerfileがあるディレクトリ全てのファイル(サブディレクトリも含む)
COPY . ./

# コンテナ内で実行したいコマンドを定義
# -b ... バインド、プロセスを指定してip(0.0.0.0)アドレスに紐付け(バインド)する
CMD ["rails", "server", "-b", "0.0.0.0"]

# ホスト(PC)       | コンテナ
# ブラウザ(外部)    | Rails
```

## 09 Nuxt.js を動かす Dockerfile を作成する

- `$ cd front`を実行<br>

* `front/Dockerfile`ファイルを作成<br>

```:Dockerfile
# FROM node:14.4.0-alpine 使用不可
FROM node:16.13.1-alpine

ARG WORKDIR
ARG CONTAINER_PORT

ENV HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo \
  HOST=0.0.0.0

# ENV check（このRUN命令は確認のためなので無くても良い）
RUN echo ${HOME}
RUN echo ${CONTAINER_PORT}

WORKDIR ${HOME}

EXPOSE ${CONTAINER_PORT}

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

### 環境変数とは

環境変数を一言で言うと OS が保有している変数<br>

今回指定した環境変数の意味<br>

1. HOME ・・・ Home ディレクトリを指定。cd コマンドの戻り先<br>
2. LANG ・・・ ロケール => 使用する言語や単位（通貨や日付・その並び順）を定義したもの<br>
3. TZ ・・・ タイムゾーンの指定<br>

LANG=C.UTF-8 とは？<br>

1. LANG ・・・ コンテナ内で使用する言語や単位<br>

2. C ・・・ 言語（en_US, ja_JP)<br>

3. UTF-8 ・・・ 文字コード（SHIFT_JIS）<br>

C とは？<br>

コンピューターのルールに沿った英語<br>

C ≠ en_US<br>

まとめると・・・<br>

LANG = C.UTF-8<br>

コンピューター用の英語を文字コード UTF-8 で利用する<br>

## 10 Dockerfile のベストプラクティス

- `api/Dockerfile`を編集（改善)<br>

```Dockerfile:Dockerfile
# ベースイメージを指定する
# FROM ベースイメージ:タグ(タグはなくてもよいが最新のものが指定されることになる)
FROM ruby:2.7.2-alpine

# Dockerfile内で使用する変数を定義
# appという値が入る
ARG WORKDIR

ARG RUNTIME_PACKAGES="nodejs tzdata postgresql-dev postgresql git"

ARG DEV_PACKAGES="build-base curl-dev"

# 環境変数を定義(Dockerfile, コンテナ参照可能)
# Rails ENV["TZ"] => Asia/Tokyoが出力される
ENV HOME=/${WORKDIR} \
    LANG=C.UTF-8 \
    TZ=Asia/Tokyo

# Dockerfime内で指定した命令を実行する・・・RUN, COPY, ADD, ENTORYPOINT, CMD
# 作業ディレクトリを定義
# コンテナ/app/Railsアプリ
WORKDIR ${HOME}

# ホスト側の(PC)のファイルをコンテナにコピー
# COPY コピー元(ホスト) コピー先(コンテナ)
# Gemfile* ... Gemfileから始まるファイルを全指定(Gemfile, Gemfile, Gemfile.lock)
# コピー元(ホスト) ... Dockerfileがあるディレクトリ以下を指定(api) ../ NG
# コピー先(コンテナ) ... 絶対パス or 相対パス(./ ... 今いる(カレント)ディレクトリ)
COPY Gemfile* ./

# apk ... Alpine Linuxのコマンド
# apk update = パッケージの最新リストを取得
RUN apk update && \
  # apk upgrade = インストールパッケージを最新のものに
  apk upgrade && \
  # apk add = パッケージのインストールを実行
  # --no-cache = パッケージをキャッシュしない(Dokcerイメージを軽量化)
  apk add --no-cache ${RUNTIME_PACKAGES} && \
  # --virtual 名前(任意) = 仮想パッケージ
  apk add --virtual build-dependencies --no-cache ${DEV_PACKAGES} && \
  # Gemのインストールコマンド
  # -j4(jobs=4) = Gemインストールの高速化
  bundle install -j4 && \
  # パーケージを削除(Dokcerイメージを軽量化)
  apk del build-dependencies

# . ... Dockerfileがあるディレクトリ全てのファイル(サブディレクトリも含む)
COPY . ./

# コンテナ内で実行したいコマンドを定義
# -b ... バインド、プロセスを指定してip(0.0.0.0)アドレスに紐付け(バインド)する
CMD ["rails", "server", "-b", "0.0.0.0"]

# ホスト(PC)       | コンテナ
# ブラウザ(外部)    | Rails
```

- `front/Dockerfile`を編集（改善）<br>

```Dockerfile:Dockerfile
# FROM node:14.4.0-alpine 使用不可
FROM node:16.13.1-alpine

ARG WORKDIR

ENV HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo \
  # これを指定しないとブラウザからhttp://localhost へアクセスすることができない。
  # コンテナのNuxt.jsをブラウザから参照するためにip:0.0.0.0に紐付ける
  # https://ja.nuxtjs.org/faq/host-port/
  HOST=0.0.0.0

WORKDIR ${HOME}

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

## 11 docker-compose.yml の環境変数設計

- docker-compose.yml -> コンテナの一元管理<br>

* Dockerfile -> コンテナの設計書<br>

- コンテナ -> アプリ稼働<br>

### DokcerComponse で環境変数を定義する 4 つの方法

① docker-compose.yml のファイル内で宣言<br>

② コンテナ実行時にコマンドで渡す (`$ docker-compose run -e WORKDIR=app`)<br>

③ .env ファイル内で宣言<br>

④ 独自のファイルを用意して宣言<br>

### 環境変数の流れ

.env => docer-compose.yml => Dockerfile => 各コンテナに渡る(Rails, Nuxt.js)<br>

docker-compose.yml と同じディレクトリにある `.env`ファイルが自動で読まれる<br>

## 12 .env, .gitignore, docker-compose.yml を作成

- `$ touch {.gitignore,.env,docker-compose.yml}`を実行<br>

* `.gitigonore`ファイルを編集<br>

```
.env
/.DS_store
```

- `.env`ファイルを編集<br>

```
# commons
WORKDIR=app
API_PORT=3000
FRONT_PORT=8080

# db
POSTGRES_PASSWORD=password
```

- `docker-compose.yml`を編集<br>

```
version: '3.8'

services:
  db:
    image: postgres:12.3-alpine
    environment:
      TZ: UTC
      PGTZ: UTC
      POSTGRES_PASSWORD: $POSTGRES_PASSWORD
    volumes:
      - ./api/tmp/db:/var/lib/postgresql/data

  api:
    build:
      context: ./api
      args:
        WORKDIR: $WORKDIR
    environment:
      POSTGRES_PASSWORD: $POSTGRES_PASSWORD
    volumes:
      - ./api:/$WORKDIR
    depends_on:
      - db
    ports:
      - "$API_PORT:$CONTAINER_PORT"

  front:
    build:
      context: ./front
      args:
        WORKDIR: $WORKDIR
        CONTAINER_PORT: $CONTAINER_PORT
    command: yarn run dev
    volumes:
      - ./front:/$WORKDIR
    ports:
      - "$FRONT_PORT:$CONTAINER_PORT"
    depends_on:
      - api
```

## 13 docker-compose.yml を理解する

volumes とは？ ・・・ ホスト側のディレクトリをコンテナで使用する<br>

ホスト(./api/tem/db) -> コンテナ(/var/lib/postgresql/data)<br>

ホスト => \$ build => Docker イメージ => コンテナ<br>

- `docker-compose.yml`を編集<br>

```yml:docker-compose.yml
# composeファイルのバージョン指定
# Doc: https://docs.docker.com/compose/compose-file/compose-versioning/
version: '3.8'

services:
  # サービス(= コンテナ)
  db:
    # ベースイメージを定義
    image: postgres:13.1-alpine
    # 環境変数を定義
    environment:
      # OSのタイムゾーン
      TZ: UTC
      # postgresのタイムゾーン
      PGTZ: UTC
      # データベースのパスワード
      POSTGRES_PASSWORD: $POSTGRES_PASSWORD
    # ホスト側のディレクトリをコンテナで使用する
    # volumes: ホストパス(絶対 or 相対) : コンテナパス(絶対)
    volumes:
      - './api/tmp/db:/var/lib/postgresql/data'

  api:
    # ベースイメージとなるDockerfileを指定
    build:
      context: ./api
      # Dockerfileに変数を渡す
      args:
        WORKDIR: $WORKDIR
    environment:
      POSTGRES_PASSWORD: $POSTGRES_PASSWORD
    volumes:
      - './api:/$WORKDIR'
    # サービスの依存関係を定義(起動の順番)
    # 公開したいポート番号:コンテナポート
    depends_on:
      - db
    # 公開用ポートを指定
    ports:
      - '$API_PORT:3000'

  front:
    build:
      context: ./front
      args:
        WORKDIR: $WORKDIR
    # コンテナで実行したいコマンド(CMD)
    command: yarn run dev
    volumes:
      - './front:/$WORKDIR'
    ports:
      - '$FRONT_PORT:3000'
    depends_on:
      - api
```

## 14 Rails アプリを立ち上げる

- `$ docker compose build`を実行<br>

- `$ docker images`を実行してイメージが出来ているか確認`<br>

```:console
groovy@groovy-no-MacBook-Pro udemy-rails-nuxt % docker images
REPOSITORY               TAG       IMAGE ID       CREATED              SIZE
udemy-rails-nuxt_api     latest    1caed9e24b48   38 seconds ago       413MB
udemy-rails-nuxt_front   latest    701437d939af   About a minute ago   110MB
```

- `$ docker compose run --rm api rails new . -f -B -d postgresql --api`を実行<br>

* `$ docker compose build`を実行<br>

- `$ docker compose up api`を実行<br>

* localhost:3000 にアクセスしてみる(エラーが出る)<br>

- `api/config/database.yml`を編集<br>

```yml:database.yml
# PostgreSQL. Versions 9.3 and up are supported.
#
# Install the pg driver:
#   gem install pg
# On macOS with Homebrew:
#   gem install pg -- --with-pg-config=/usr/local/bin/pg_config
# On macOS with MacPorts:
#   gem install pg -- --with-pg-config=/opt/local/lib/postgresql84/bin/pg_config
# On Windows:
#   gem install pg
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
#
# Configure Using Gemfile
# gem 'pg'
#
default: &default
  adapter: postgresql
  encoding: unicode
  host: db # 追記
  username: postgres # 追記
  password: <%= ENV["POSTGRES_PASSWORD"] %> # 追記
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: app_development

  # The specified database role being used to connect to postgres.
  # To create additional roles in postgres see `$ createuser --help`.
  # When left blank, postgres will use the default role. This is
  # the same name as the operating system user that initialized the database.
  #username: app
  # The password associated with the postgres role (username).
  #password:
  # Connect on a TCP socket. Omitted by default since the client uses a
  # domain socket that doesn't need configuration. Windows does not have
  # domain sockets, so uncomment these lines.
  #host: localhost
  # The TCP port the server listens on. Defaults to 5432.
  # If your server runs on a different port number, change accordingly.
  #port: 5432
  # Schema search path. The server defaults to $user,public
  #schema_search_path: myapp,sharedapp,public
  # Minimum log levels, in increasing order:
  #   debug5, debug4, debug3, debug2, debug1,
  #   log, notice, warning, error, fatal, and panic
  # Defaults to warning.
  #min_messages: notice

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: app_test

# As with config/credentials.yml, you never want to store sensitive information,
# like your database password, in your source code. If your source code is
# ever seen by anyone, they now have access to your database.
#
# Instead, provide the password as a unix environment variable when you boot
# the app. Read https://guides.rubyonrails.org/configuring.html#configuring-a-database
# for a full rundown on how to provide these environment variables in a
# production deployment.
#
# On Heroku and other platform providers, you may have a full connection URL
# available as an environment variable. For example:
#
#   DATABASE_URL="postgres://myuser:mypass@localhost/somedatabase"
#
# You can use this database configuration with:
#
#   production:
#     url: <%= ENV['DATABASE_URL'] %>
#
production:
  <<: *default
  database: app_production
  username: app
  password: <%= ENV['APP_DATABASE_PASSWORD'] %>
```

- ターミナルで control + c を押してコンテナの停止をする<br>

* `$ docker compose down`を実行してコンテナを削除<br>

- `$ docker compose ps -a`を実行してコンテナが残っていないか確認<br>

* `$ docker compose run --rm api rails db:create`を実行して DB を作成<br>

- `$docker compose up api`を実行して api サービスを起動<br>

* localhost:3000 にアクセス(Railsの初期画面が表示されればOK)<br>

### Docker コマンド

```terminal
# コンテナ内で任意のコマンドを実行する
$ docker-compose run <サービス名> <任意のコマンド>

# Dockerイメージを作成する
$ docker-compose build

# 複数のコンテナの生成し起動する
$ docker-compose up

# コンテナを停止する
$ docker-compose stop

# 停止中のコンテナを削除する
$ docker-compose rm -f

# コンテナの停止・削除を一度に行う
$ docker-compose down

# コンテナの一覧を表示する
$ docker-compose ps -a
```