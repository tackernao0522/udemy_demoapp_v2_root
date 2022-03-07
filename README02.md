## 15 Nuxt.js アプリを立ち上げる

- `$ docker-compose run --rm front yarn create nuxt-app`を実行(このままだとエラーが出て create できない)<br>

* `$ docker-compose run --rm front yarn create nuxt-app app`を実行<br>

? Project name: (app) そのまま`Enter`で進む<br>
? Programming language: (Use arrow keys)は`JavaScript`を選択して`Enter`<br>
? Package manager: (Use arrow keys)は`Yarn`を選択して`Enter`<br>
? UI framework: (Use arrow keys) は `None`を選択して`Enter`(後ほどモジュールで vuetify を導入することとする)<br>
? Nuxt.js modules: (Press <space> to select, <a> to toggle all, <i> to invert selection)は`Axios`を選択して`Enter`<br>
? Linting tools: (Press <space> to select, <a> to toggle all, <i> to invert selection)は`ESlint`を選択して`Enter`<br>
? Testing framework: (Use arrow keys)は`None`を選択して`Enter`<br>
? Rendering mode: (Use arrow keys)は`Single Page App`を選択して`Enter`<br>
? Deployment target: (Use arrow keys)は`Server`を選択して`Enter`<br>
? Development tools: (Press <space> to select, <a> to toggle all, <i> to invert selection)は何も選択せずに`Enter`<br>
? Continuous integration: (Use arrow keys)は`None`を選択して`Enter`<br>
? Version control system: (Use arrow keys)は`Git`を選択して`Enter`<br>

- `$ mv front/app/{*,.*} front`を実行<br>

- `$ rmdir front/app/`を実行<br>

* `$ docker compose up front`を実行<br>

- `localhost:8080`にアクセスしてみる<br>

- `Control + C`でサーバー停止<br>

* `$ docker compose down`でコンテナ削除<br>

# セクション 4: 複数プロジェクトの Git 管理

## 16 Git 管理の全体像を理解する

submodule ... 別リポジトリの編集履歴を現在のリポジトリで管理する<br>

## 17 api と front リポジトリのコミット

- `$ cd api`を実行<br>

* `$ vi .gitignore`を実行<br>

```vi:.gitignore
# Ignore all logfiles and tempfiles.
/log/*
/tmp/*
!/log/.keep
!/tmp/.keep

# Ignore pidfiles, but keep the directory.
/tmp/pids/*
!/tmp/pids/
!/tmp/pids/.keep

# Ignore uploaded files in development.
/storage/*
!/storage/.keep
.byebug_history

# Ignore master key for decrypting credentials and more.
/config/master.key

// 追記
# original
.DS_Store
```

- `$ git add -A`を実行<br>

- `$ git commit -m "Create Rails App"`を実行<br>

* `$ cd ../front`を実行<br>

- `$ git init`を実行<br>

* `$ git add -A`を実行<br>

- `$ git commit -m "Create Nuxt.js App"`を実行<br>

## 18 root リポジトリのコミット

- `root ディレクトに移動`<br>

- 今回は一旦`$ rm -rf .git`を実行してやり直した。<br>

* `$ git init`を実行<br>

* `$ git add -A`を実行<br>

- `$ git submodule add ./api api`を実行すると下記のようになる<br>

```:terminal
groovy@groovy-no-MacBook-Pro udemy-rails-nuxt % git submodule add ./api api
fatal: 'api' already exists in the index
groovy@groovy-no-MacBook-Pro udemy-rails-nuxt %
```

- `$ git commit -m "Create App"`を実行<br>

* `$ git ls-files`を実行して確認してみる<br>

- `$ vi .gitmodules`を実行<br>

```vi:.bitmodules
[submodule "api"]
  path = api
  url = ./api
[submodule "front"]
  patch = front
  url = ./front
```

- `$ git add -A`を実行<br>

* `$ git commit -m "Add .gitmodules"`を実行<br>

- GitHub に`udemy_demoapp_v2_root`リポジトリを作成<br>

* GitHub に`udemy_demoapp_v2_api`リポジトリを作成<br>

* GitHub に`udemy_demoapp_v2_front`リポジトリを作成<br>

- GitHub のユーザーアイコンのプルダウンにある`your projects`をクリック<br>

- `New Project`をクリック<br>

* Project board name に `udemy_demoapp_v2`を入力<br>

- Linked repositories に `udemy_demoapp_v2_root`, `udemy_demoapp_v2_api`, `udemy_demoapp_v2_front`を選択して入れる<br>

* `Create project`をクリック<br>

- GitHub の`your projects`を確認してみる<br>

* `cd api`を実行<br>

* `$ git remote add origin git@github.com:tackernao0522/udemy_demoapp_v2_api.git`を実行<br>

- `$ git push`を実行<br>

下記のようになる<br>

```:terminal
groovy@groovy-no-MacBook-Pro api % git push
fatal: The current branch main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main

groovy@groovy-no-MacBook-Pro api %
```

- 最初の一回だけ`$ git push --set-upstream origin main`を実行<br>

* `$ cd front`を実行<br>

- `$ git remote add origin git@github.com:tackernao0522/udemy_demoapp_v2_front.git`を実行<br>

- 最初の一回だけ`$ git push --set-upstream origin main`を実行<br>

- `.gitmodules`を編集<br>

```:.gitmodules
[submodule "api"]
  path = api
  url = https://github.com/tackernao0522/udemy_demoapp_v2_api
[submodule "front"]
  path = front
  url = https://github.com/tackernao0522/udemy_demoapp_v2_front
```

- `rootディレクトリ`に移動<br>

- `$ git commit -am "Edit .gitmodules"`を実行<br>

* `$ git remote add origin git@github.com:tackernao0522/udemy_demoapp_v2_root.git`を実行<br>

- `$ git push --set-upstream origin main`を実行<br>

# セクション 5: RailsApi x Nuxt.js 初めての API 通信

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
      API_DOMAIN: 'localhost:$FRONT_PORT' # 追加
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
        API_URL: 'http://localhost:$API_PORT' # 追加
    # コンテナで実行したいコマンド(CMD)
    command: yarn run dev
    volumes:
      - './front:/$WORKDIR'
    ports:
      - '$FRONT_PORT:3000'
    depends_on:
      - api
```

- `front/Dockerfile`を編集<br>

```:Dockerfile
# FROM node:14.4.0-alpine 使用不可
FROM node:16.13.1-alpine

ARG WORKDIR
ARG API_URL # 追記

ENV HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo \
  # これを指定しないとブラウザからhttp://localhost へアクセスすることができない。
  # コンテナのNuxt.jsをブラウザから参照するためにip:0.0.0.0に紐付ける
  # https://ja.nuxtjs.org/faq/host-port/
  HOST=0.0.0.0 \
  API_URL=${API_URL} # 追記

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

- `root $ docker compose build front`を実行<br>

* `root $ docker-compose run --rm api rails g controller api::v1::hello`を実行<br>

- `api/app/controllers/api/v1/hello_controller.rb`を編集<br>

```rb:hello_controller.rb
class Api::V1::HelloController < ApplicationController
  def index
    render json: 'Hello'
  end
end
```

- `api/config/routes.rb`を編集<br>

```rb:routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      # api test action
      resources :hello, only: %i[index]
    end
  end
end
```

- `root $ docker compose up api`を実行<br>

* http://localhost:3000/api/v1/hello にアクセスしてみる<br>
