## 35 モデル開発に必要な Gem をインストール

- 参考: https://rubygems.org/ <br>

* `hirb`を検索する<br>

- https://rubygems.org/gems/hirb <br>

* `api/Gemfile`を編集<br>

```:Gemfile
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails', branch: 'main'
gem 'rails', '~> 6.0.4', '>= 6.0.4.6'
# Use postgresql as the database for Active Record
gem 'pg', '>= 0.18', '< 2.0'
# Use Puma as the app server
gem 'puma', '~> 4.1'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
# gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'

# 追加
# コンソールの出力結果を見やすく表示する
gem 'hirb', '~> 0.7.3'

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
end

group :development do
  gem 'listen', '~> 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- https://rubygems.org/ で`hirb-unicode`を検索する<br>

* https://rubygems.org/gems/hirb-unicode <br>

- `hirb-unicode-steakknife`をクリックする<br>

* `api/Gemfile`に追加<br>

```:Gemfile
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails', branch: 'main'
gem 'rails', '~> 6.0.4', '>= 6.0.4.6'
# Use postgresql as the database for Active Record
gem 'pg', '>= 0.18', '< 2.0'
# Use Puma as the app server
gem 'puma', '~> 4.1'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
# gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'

# コンソールの出力結果を見やすく表示する
gem 'hirb', '~> 0.7.3'

# 追加
# Hirbの文字列補正を行う
gem 'hirb-unicode-steakknife', '~> 0.0.9'

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
end

group :development do
  gem 'listen', '~> 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- https://rubygems.org/ で`bcrypt`を検索する<br>

* `bcrypt`をクリック<br>

`api/Gemfile`に追加<br>

```:Gemfile
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails', branch: 'main'
gem 'rails', '~> 6.0.4', '>= 6.0.4.6'
# Use postgresql as the database for Active Record
gem 'pg', '>= 0.18', '< 2.0'
# Use Puma as the app server
gem 'puma', '~> 4.1'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
# gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'

# コンソールの出力結果を見やすく表示する
gem 'hirb', '~> 0.7.3'

# Hirbの文字列補正を行う
gem 'hirb-unicode-steakknife', '~> 0.0.9'

# 追加
# パスワードを暗号化する
gem 'bcrypt', '~> 3.1', '>= 3.1.16'

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
end

group :development do
  gem 'listen', '~> 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- `root $ docker-compose build api`を実行<br>

* `root $ docker image prune -f`を実行<br>

* `root $ docker compose run --rm api bundle info hirb`を実行<br>

```:terminal
  * hirb (0.7.3)
	Summary: A mini view framework for console/irb that's easy to use, even while under its influence.
	Homepage: http://tagaholic.me/hirb/
	Path: /usr/local/bundle/gems/hirb-0.7.3
```

- `root $ docker compose run --rm api bundle info hirb-unicode-steakknife`を実行<br>

```:terminal
  * hirb-unicode-steakknife (0.0.9)
	Summary: Unicode support for hirb
	Homepage:
	Path: /usr/local/bundle/gems/hirb-unicode-steakknife-0.0.9
```

- `root $ docker compose run --rm api bundle info bcrypt`を実行<br>

```:terminal
  * bcrypt (3.1.16)
	Summary: OpenBSD's bcrypt() password hashing algorithm.
	Homepage: https://github.com/codahale/bcrypt-ruby
	Path: /usr/local/bundle/gems/bcrypt-3.1.16
```

- `root $ vi api/.irbrc`を実行<br>

```vi:.irbrc
# コンソール起動時
if defined? Rails::Console
  # Hirb.enableの有効化
  Hirb.enable if defined? Hirb
end
```

保存する<br>

- `root $ docker compose run --rm api rails c`を実行<br>

`irb(main):001:0> Hirb::View.enabled?`を実行<br>

```:terminal
irb(main):002:0> Hirb::View.enabled?
=> true
irb(main):003:0>
```

- `irb(main):003:0> puts Hirb::Helpers::Table.render [[1,2], [2,3]]`を実行<br>

```:terminal
irb(main):003:0> puts Hirb::Helpers::Table.render [[1,2], [2,3]]
+---+---+
| 0 | 1 |
+---+---+
| 1 | 2 |
| 2 | 3 |
+---+---+
2 rows in set
=> nil
irb(main):004:0>
```

- `irb(main):004:0> puts Hirb::Helpers::Table.render [{:age=>10, :weight=>100}, {:age=>80, :weight=>500}]`を実行<br>

```:terminal
irb(main):004:0> puts Hirb::Helpers::Table.render [{:age=>10, :weight=>100}, {:age=>80, :weight=>500}]
+-----+--------+
| age | weight |
+-----+--------+
| 10  | 100    |
| 80  | 500    |
+-----+--------+
2 rows in set
=> nil
irb(main):005:0>
```

- `api $ git push heroku`を実行<br>

- `api $ heroku run rails c`を実行<br>

- `irb(main):001:0> puts Hirb::Helpers::Table.render [[1,2], [2,3]]`を実行<br>

```:terminal
irb(main):001:0> puts Hirb::Helpers::Table.render [[1,2], [2,3]]
+---+---+
| 0 | 1 |
+---+---+
| 1 | 2 |
| 2 | 3 |
+---+---+
2 rows in set
=> nil
irb(main):002:0>
```

# セクション 8: ユーザーモデル開発

## 36 ユーザーテーブル設計と認証設計を理解する

### ユーザーテーブル設計

|     カラム      |   型    | デフォルト | NULL  | 長さ |       内容       |
| :-------------: | :-----: | :--------: | :---: | :--: | :--------------: |
|      name       | String  |            | false |  30  |    ユーザー名    |
|      email      | String  |            | false | 255  |  メールアドレス  |
| password_digest | String  |            | false |  72  |    パスワード    |
|    activated    | Boolean |   false    | false |      | メール認証フラグ |
|      admin      | Boolean |   false    | false |      |   管理者フラグ   |

### Email のバリデーション

`OK => activated = false && user.email == email`<br>

`NG => activated = true && user.email == email`<br>

認証済みメールアドレスの一意性を保つ<br>

## 37. User テーブルを作成する

- `root $ docker-compose run --rm api rails g model User --no-fixture`を実行<br>

* `api/db/migrate/xxx_create_users.rb`を編集<br>

```rb:xxx_create_users.rb
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :name, null: false
      t.string :email, null: false
      t.string :password_digest, null: false
      t.boolean :activated, null: false, default: false
      t.boolean :admin, null: false, default: false
      t.timestamps
    end
  end
end
```

- `root $ docker-compose run --rm api rails db:migrate`を実行<br>

* `root $ docker compose run --rm api rails dbconsole`を実行<br>

```:terminal
/app # rails dbconsole
Password for user postgres: password
```

- `password`を入力して`Enter`<br>

```:terminal
/app # rails dbconsole
Password for user postgres:
psql (13.6, server 13.1)
Type "help" for help.

app_development=#
```

- `app_development=# \d`を実行<br>

```:terminal
                  List of relations
 Schema |         Name         |   Type   |  Owner
--------+----------------------+----------+----------
 public | ar_internal_metadata | table    | postgres
 public | schema_migrations    | table    | postgres
 public | users                | table    | postgres
 public | users_id_seq         | sequence | postgres
(4 rows)
```

- `app_development=# \d users`を実行<br>

```:terminal
                                            Table "public.users"
     Column      |              Type              | Collation | Nullable |              Default
-----------------+--------------------------------+-----------+----------+-----------------------------------
 id              | bigint                         |           | not null | nextval('users_id_seq'::regclass)
 name            | character varying              |           | not null |
 email           | character varying              |           | not null |
 password_digest | character varying              |           | not null |
 activated       | boolean                        |           | not null | false
 admin           | boolean                        |           | not null | false
 created_at      | timestamp(6) without time zone |           | not null |
 updated_at      | timestamp(6) without time zone |           | not null |
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
```

## 38 User モデルのバリデーション設定

### User モデルのバリデーション設計

|     カラム      |       内容       | 入力 | 最小文字数 |     最大文字数      | 書式チェック | 一意性 |
| :-------------: | :--------------: | :--: | :--------: | :-----------------: | :----------: | :----: |
|      name       |    ユーザー名    | 必須 |            |                     |      30      | しない | なし |
|      email      |  メールアドレス  | 必須 |            |         255         |     する     |  なし  |
| password_digest |    パスワード    | 必須 |     8      | 72（bcrypt の仕様） |     する     |  なし  |
|    activated    | メール認証フラグ |      |     -      |          -          |      -       |   -    |
|      admin      |   管理者フラグ   |      |     -      |          -          |      -       |   -    |

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> User.all`を実行するとユーザー登録されていれば閲覧される<br>

* `irb(main):002:0> User.first`を実行すると ID: 01 のユーザーが出力される<br>

- `irb(main):003:0> User.first.password_digest`暗号化されたパスワードの出力<br>

* `irb(main):005:0> User.first.authenticate("password")`パスワードが一致しているかどうか確認出来る<br>

- `api/app/models/user.rb`を編集<br>

```rb:user.rb
class User < ApplicationRecord
  # gem bcrypt
  # 1. passwordを暗号化する
  # 2. password_digest => password
  # 3. password_confirmation => パスワードの一致確認
  # 4. 一致のバリデーション追加
  # 5. authenticate()
  # 6. 最大文字数 72文字まで
  # 7. User.create() => 入力必須バリデーション、User.update() => x
  has_secure_password # 新規登録時はpassoword入力必須になる

  # validates
  # User.create(name: "                                        ")
  # 名前を入力してください。文字数は30文字まで(空白の場合には出ないようにする allow_blank: true)
  validates :name, presence: true,      # 入力必須
                    length: {
                      maximum: 30,      # 最大文字数
                      allow_blank: true # Null(nil), 空白文字の場合スキップ(空白文字の場合には無駄な検証を行わない)
                    }

  VALID_PASSWORD_REGEX = /\A[\w\-]+\z/ # 先頭から末尾まで、全て「a-zA-Z0-9_」と「-」にマッチする文字列を許容する。
  # \A     => 文字列の先頭にマッチ
  # [\w\-] => a-zA-X0-9_-
  # +      => 1文字以上繰り返す
  # \z     => 文字列の末尾にマッチ
  validates :password, presence: true, # nameのみのupdate時にpasswordがnilになっていてもpassword必須とはならない(allow_nilが利く)
                      length: {               # 最小文字数
                        minimum: 8,
                        allow_blank: true
                      },
                      format: {               # 書式チェック
                        with: VALID_PASSWORD_REGEX,
                        allow_blank: true
                      },
                      allow_nil true # 空パスワードのアップデートを許容する。(Null(nil)の場合スキップ)
end
```
