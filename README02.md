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
