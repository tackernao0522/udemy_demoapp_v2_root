# セクション 6: heroku.yml を使った Docker デプロイ

## 25 Rails Puma の設定と heroku.yml の作成 〜 デプロイ準備編〜

- `api/config/puma.rb`を編集<br>

```rb:puma.rb
# Doc: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#config

# Specifies the number of `workers` to boot in clustered mode.
# Workers are forked web server processes. If using threads and workers together
# the concurrency of the application would be max `threads` * `workers`.
# Workers do not work on JRuby or Windows (both of which do not support
# processes).
#
workers Integer(ENV.fetch('WEB_CONCURRENCY') { 2 })

# Puma can serve each request in a thread from an internal thread pool.
# The `threads` method setting takes two numbers: a minimum and maximum.
# Any libraries that use thread pools should be configured to match
# the maximum value specified for Puma. Default is set to 5 threads for minimum
# and maximum; this matches the default thread size of Active Record.
#
max_threads_count = Integer(ENV.fetch('RAILS_MAX_THREADS') { 5 })
min_threads_count =
  Integer(ENV.fetch('RAILS_MIN_THREADS') { max_threads_count })
threads min_threads_count, max_threads_count

# Use the `preload_app!` method when specifying a `workers` number.
# This directive tells Puma to first boot the application and load code
# before forking the application. This takes advantage of Copy On Write
# process behavior so workers use less memory.
#
preload_app!

rackup DefaultRackup

# Specifies the `port` that Puma will listen on to receive requests; default is 3000.
#
port ENV.fetch('PORT') { 3000 }

# Specifies the `environment` that Puma will run in.
#
environment ENV.fetch('RACK_ENV') { 'development' }

on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
```

- `root $ touch api/heroku.yml`を実行<br>

```yml:heroku.yml
# アプリ環境を定義する場所
setup:
  # アプリ作成時にアドオンを自動で追加する
  addons:
    - plan: heroku-postgresql
  # 環境変数を指定する
  config:
    # Rackへ現在の環境を示す
    RACK_ENV: production
    # Railsへ現在の環境を示す
    RAILS_ENV: production
    # log出力のフラグ(enabled => 出力する)
    RAILS_LOG_TO_STDOUT: enabled
    # publicディレクトリからの静的ファイルを提供してもらうかのフラグ(enabled => 提供してもらう)
    RAILS_SERVE_STATIC_FILES: enabled
# ビルドを定義する場所
build:
  # 参照するDockerfileの場所を定義(相対パス)
  docker:
    web: Dockerfile
  # Dockerfileに渡す環境変数を指定
  config:
    WORKDIR: app
# プロセスを定義
run:
  # Bundlerでインストールされたgemを使用してコマンドを実行
  web: bundle exec puma -C config/puma.rb
```

## 26 Rails 開発環境の起動コマンドを変更する 〜デプロイ準備編〜

- `root $ docker images`を実行<br>

```:terminal
REPOSITORY               TAG           IMAGE ID       CREATED             SIZE
udemy-rails-nuxt_api     latest        f76000c6a21c   About an hour ago   529MB
udemy-rails-nuxt_front   latest        85fbf273eb47   47 hours ago        110MB
postgres                 13.1-alpine   8c6053d81a45   13 months ago       159MB
```

- `root $ docker inspect -f="{{ .Config.Cmd }}" f76000c6a21c`を実行<br>

```:terminal
[rails server -b 0.0.0.0]
```

- https://docs.docker.com/compose/rails/ へアクセス<br>

* `docker-compose.yml`を編集<br>

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
    command: /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'" # 追加
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

- `root $ docker compose run --rm api sh`を実行(api コンテナに入ることができる)<br>

* `/app # /bin/sh -c "echo TEST"`を実行<br>

```terminal:
TEST
```

- `/app # /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"<br>

```terminal:
=> Booting Puma
=> Rails 6.0.4.6 application starting in development
=> Run `rails server --help` for more startup options
[9] Puma starting in cluster mode...
[9] * Version 4.3.11 (ruby 2.7.2-p137), codename: Mysterious Traveller
[9] * Min threads: 5, max threads: 5
[9] * Environment: development
[9] * Process workers: 2
[9] * Preloading application
[9] * Listening on tcp://0.0.0.0:3000
[9] Use Ctrl-C to stop
[9] - Worker 0 (pid: 15) booted, phase: 0
[9] - Worker 1 (pid: 17) booted, phase: 0
```

- `/app # exit`を実行してコンテナから抜ける<br>

* `api/.gitignore`を編集<br>

```:.gitignore
# See https://help.github.com/articles/ignoring-files for more about ignoring files.
#
# If you find yourself ignoring temporary files generated by your text editor
# or operating system, you probably want to add a global ignore instead:
#   git config --global core.excludesfile '~/.gitignore_global'

# Ignore bundler config.
/.bundle

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

# original
.DS_Store
# 追加
/.ash_history
/.irb_history
```

- `api $ git rm --cached .ash_history`を実行<br>

* `api/Dockerfile`を編集<br>

```:Dockerfile
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

# 削除

# ホスト(PC)       | コンテナ
# ブラウザ(外部)    | Rails
```

- `root $ docker compose build api`を実行<br>

* `root $ docker images`を実行<br>

```:terminal
REPOSITORY               TAG           IMAGE ID       CREATED              SIZE
udemy-rails-nuxt_api     latest        7f8676ba32ab   About a minute ago   529MB
udemy-rails-nuxt_front   latest        85fbf273eb47   2 days ago           110MB
postgres                 13.1-alpine   8c6053d81a45   13 months ago        159MB
```

- `root $ docker inspect -f="{{ .Config.Cmd }}" 7f8676ba32ab`を実行<br>

```:terminal
[irb]
```

- `root $ docker image prune -f`を実行(古い imgae を削除する)<br>

## 27 Rails HerokuCLI-manifest を使ってアプリを構築する 〜デプロイ準備編〜

- `root $ heroku login`を実行<br>

* heroku: Press any key to open up the browser to login or q to exit: `Enter`を実行<br>

- ブラウザが開きログインをする<br>

* `root $ heroku update beta`を実行<br>

- `root $ heroku plugins:install @heroku-cli/plugin-manifest`を実行<br>

* `root $ heroku plugins`を実行して確認(下記のようになれば OK)<br>

```:terminal
heroku-repo 1.0.14
manifest 0.0.5
```

ちなみに plugin を削除するコマンドは `$ heroku plugins:remove manifest`<br>

beta 版から安定版に戻す場合は `$ hroku update stable`<br>

- `root $ cd api`を実行<br>

* `api $ heroku create udemy-railsnuxtv2-api --manifest`(アンダーバーは不可)を実行<br>

- `api $ heroku open`を実行<br>

* Heroku のダッシュボードを開く<br>

- `udemy-railsnuxtv2-api`を開く(add-ons が設定されているか確認する)<br>

* `Settings`の`Config Vars`の`Reveal Config Vars`をクリックする(5 個の値が設定されていれば OK)<br>

## 28 Heroku へ SSH 接続を行い Rails をデプロイする 〜デプロイ編〜

- 参考: https://devcenter.heroku.com/ja/articles/keys <br>

* `$ cd api`を実行<br>

- `api $ ssh-keygen -t rsa -f ~/.ssh/id_rsa_heroku -C groovy@macbookpro.local`を実行<br>

- そのまま`Enter`で進む<br>

* そのまま`Enter`で進む<br>

```:terminal
groovy@groovy-no-MacBook-Pro api % ssh-keygen -t rsa -f ~/.ssh/id_rsa_heroku -C groovy@macbookpro.local
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/groovy/.ssh/id_rsa_heroku.
Your public key has been saved in /Users/groovy/.ssh/id_rsa_heroku.pub.
The key fingerprint is:
SHA256:PbyH/SmxUNflQPHcNDXHhk+nwVVvnOA0Q6hqv7dqk2s groovy@macbookpro.local
The key's randomart image is:
+---[RSA 3072]----+
|            oOoBO|
|           .o O=&|
|          .  . @X|
|         +  . o.+|
|        S +. .   |
|       o  .=.    |
|      . . +.oo   |
|         E oo. . |
|        o+*...o  |
+----[SHA256]-----+
```

- `api $ ls ~/.ssh`を実行して確認する<br>

- `api $ heroku keys:add ~/.ssh/id_rsa_heroku.pub`を実行<br>

```:terminal
Uploading /Users/groovy/.ssh/id_rsa_heroku.pub SSH key... done
```

- Heroku ブラウザの Account Settings の SSH Keys に反映されていれば OK<br>

* `api $ heroku keys`を実行しても確認できる<br>

- `api $ vi ~/.ssh/config`を実行<br>

- https://devcenter.heroku.com/ja/articles/keys にアクセスして下記をコピーする<br>

```
Host heroku.com
  HostName heroku.com
  IdentityFile /path/to/key_file
  IdentitiesOnly yes
```

- `.ssh/config`を編集する<br>

```:.ssh/config
Host heroku.com
  HostName heroku.com
  IdentityFile ~/.ssh/id_rsa_heroku
  IdentitiesOnly yes
```

- `api $ ssh -v git@heroku.com`を実行して確認(下記の 2 文が記されていれば OK)<br>

```:terminal
debug1: Offering public key: /Users/groovy/.ssh/id_rsa_heroku RSA SHA256:PbyH/SmxUNflQPHcNDXHhk+nwVVvnOA0Q6hqv7dqk2s explicit
Authenticated to heroku.com ([50.19.85.154]:22).
```

- `api $ git remote -v`を実行<br>

```:terminal
heroku  https://git.heroku.com/udemy-railsnuxtv2-api.git (fetch)
heroku  https://git.heroku.com/udemy-railsnuxtv2-api.git (push)
origin  git@github.com:tackernao0522/udemy_demoapp_v2_api.git (fetch)
origin  git@github.com:tackernao0522/udemy_demoapp_v2_api.git (push)
```

- `api $ git remote remove heroku`を実行<br>

* `api $ git remote -v`を実行<br>

```:terminal
origin  git@github.com:tackernao0522/udemy_demoapp_v2_api.git (fetch)
origin  git@github.com:tackernao0522/udemy_demoapp_v2_api.git (push)
```

- `api $ git remote add heroku git@heroku.com:udemy-railsnuxtv2-api.git`を実行<br>

- `api $ git remote -v`を実行<br>

```:terminal
heroku  git@heroku.com:udemy-railsnuxtv2-api.git (fetch)
heroku  git@heroku.com:udemy-railsnuxtv2-api.git (push)
origin  git@github.com:tackernao0522/udemy_demoapp_v2_api.git (fetch)
origin  git@github.com:tackernao0522/udemy_demoapp_v2_api.git (push)
```

- `api $ heroku stack`を実行(container に'\*'が付いていれば OK)<br>

```:terminal
=== ⬢ udemy-railsnuxtv2-api Available Stacks
* container
  heroku-18
  heroku-20
```

- もし container に設定されていない場合は `api $ heroku stack:set container`を実行すると変更できる<br>

* `api $ git push heroku main`を実行(最後に下記の文が記されていれば OK)<br>

```:terminal
remote: Verifying deploy... done.
```
