## 84 認証モジュールのテストをする

- `root $ docker compose run --rm api rails g controller Api::V1::Projects`を実行<br>

* `api/app/controllers/api/v1/projects_controller.rb`を編集<br>

```rb:projects_controller.rb
class Api::V1::ProjectsController < ApplicationController
  before_action :authenticate_active_user

  def index
    projects = []
    date = Date.new(2021, 4, 1)
    10.times do |n|
      id = n + 1
      name = "#{current_user.name} project #{id.to_s.rjust(2, '0')}"
      updated_at = date + (id * 6).hours
      projects << { id: id, name: name, updatedAt: updated_at }
    end # 本来はcurrent_user.projects

    render json: projects
  end
end
```

- `api/config/routes.rb`を編集<br>

```rb:routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      ## aauth_token
      resources :auth_token, only: %i[create] do
        post :refresh, on: :collection
        delete :destroy, on: :collection
      end

      # 追加
      # projects
      resources :projects, only: %i[index]
    end
  end
end
```

- `root $ docker compose run --rm api rails g integration_test Authentication`を実行<br>

* `api/test/integration/authentication_test.rb`を編集<br>

```rb:authentication_test.rb
require 'test_helper'

class AuthenticationTest < ActionDispatch::IntegrationTest
  def setup
    @user = active_user
    @params = { auth: { email: @user.email, password: 'password' } }
    @access_lifetime = UserAuth.access_token_lifetime
    @session_key = UserAuth.session_key.to_s
    @access_token_key = 'token'
  end

  # プロジェクトapi
  def projects_api(token)
    get api('/projects'), xhr: true, headers: auth(token)
  end

  # 認証メソッドテスト
  test 'authenticate_user_method' do
    login(@params)
    access_token = res_body[@access_token_key]

    # 有効なtokenでアクセスできているか
    projects_api(access_token)
    assert_response 200
    assert response.body.present?

    # 有効期限切れは401が返っているか
    travel_to (@access_lifetime.from_now) do
      # アクセス前のcookieは存在するか
      assert_not cookies[@session_key].blank?

      # レスポンスは想定通りか
      projects_api(access_token)
      assert_response 401
      assert_not response.body.present?

      # cookieは削除されているか
      assert cookies[@session_key].blank?
    end

    # 不正なtokenが投げられた場合
    invalid_token = 'a.' + access_token
    projects_api(invalid_token)

    assert_response 401
    assert_not response.body.present?
  end
end
```

- `$ docker compose run --rm api rails t test/integration`を実行<br>

```
users...---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=] 0% Time: 00:00:00,  ETA: ??:??:??
users...
users = 10
users = 10
  9/9: [==============================================================================================================================================================================] 100% Time: 00:00:04, Time: 00:00:04

Finished in 4.10767s
9 tests, 57 assertions, 0 failures, 0 errors, 0 skips
```

## 85 本番環境にクレデンシャル設定をして Heroku にデプロイ

### CORS のクレデンシャル設定

認証情報とは = Cookie, 認証ヘッダー, クライアント証明書<br>

`クライアント` ----XMLHttpRequest.withCredentials: true-----> `サーバー`<br>

`クライアント` <----Access-Control-Allow-Credentials: true----- `サーバー`<br>

### ハンズオン

- `api/config/initializers/cors.rb`を編集<br>

```rb:cors.rb
# Be sure to restart your server when you modify this file.

# Avoid CORS issues when API is called from the frontend app.
# Handle Cross-Origin Resource Sharing (CORS) in order to accept cross-origin AJAX requests.

# Read more: https://github.com/cyu/rack-cors

Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins ENV['API_DOMAIN'] || ''

    resource '*',
             headers: :any,
             methods: %i[get post put patch delete options head],
             # 追加
             credentials: true
  end
end
```

### Cookie の SameSite 属性

SameSite 属性とは・・・Cookie の送信を制御する属性<br>

https://web.dev/same-site-same-origin/ <br>

| 指定できる値 |                                 Cookie の送信を許可するとき                                  | 開発環境<br>localhost | 本番環境 ①<br>herokuapp.com | 本番環境 ②<br>同じドメイン |
| :----------: | :------------------------------------------------------------------------------------------: | :-------------------: | :-------------------------: | -------------------------- |
|    Strict    |                                        同一サイトのみ                                        |           ○           |              x              | ○                          |
|     Lax      | 同一サイト or トップレベルナビゲーションの<br>Get リクエストの場合(URL バーの変更が伴う遷移) |           ○           |              x              | ○                          |
|     None     |           クロスサイトへの Cookie 送信を許可<br>(Secure 属性が true の場合にのみ)            |           x           |              ○              | ○                          |
|   指定なし   |                          Chrome v80〜 デフォルト値 Lax が付与される                          |           -           |              -              | -                          |

### ハンズオン

- `api/config/application.rb`を編集<br>

```rb:application.rb
require_relative 'boot'

require 'rails' # Pick the frameworks you want:
require 'active_model/railtie'
require 'active_job/railtie'
require 'active_record/railtie'
require 'active_storage/engine'
require 'action_controller/railtie'
require 'action_mailer/railtie'
require 'action_mailbox/engine'
require 'action_text/engine'
require 'action_view/railtie'
require 'action_cable/engine' # require "sprockets/railtie"
require 'rails/test_unit/railtie'

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)

module App
  class Application < Rails::Application # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.0

    # Railsアプリのタイムゾーン(default 'UTC')
    # TimeZoneList: http://api.rubyonrails.org/classes/ActiveSupport/TimeZone.html
    config.time_zone = ENV['TZ']

    # データベースの読み書きに使用するタイムゾーン(:local | :utc(default))
    config.active_record.default_timezone = :utc

    # i18nで使われるデフォルトのロケールファイルの指定(default :en)
    config.i18n.default_locale = :ja

    # $LOAD_PATHにautoload pathを追加しない(Zeitwerk有効時false推奨)
    config.add_autoload_paths_to_load_path = false

    # Cookieを処理するmeddleware
    config.middleware.use ActionDispatch::Cookies

    # 追加
    # Cookietのsamesite属性を変更する(Rails v6.1〜, :strict, :lax, :none)
    if Rails.env.production?
      config.action_dispatch.cookies_same_site_protection =
        ENV['COOKIES_SAME_SITE'].to_sym
    end

    config.api_only = true
  end
end
```

- `api $ heroku config:set COOKIES_SAME_SITE=none`を実行<br>
  `

* `api $ heroku config:get COOKIES_SAME_SITE`を実行して確認<br>

```
none が表示されればOK
```

- コミットして heroku にデプロイする<br>

* `api $ heroku run rails db:migrate`を実行<br>

* `api $ heroku run rails r 'puts User.column_names`'を実行<br>

```
id
name
email
password_digest
activated
admin
created_at
updated_at
```

- `api $ heroku config:get BASE_URL | pbcopy`を実行<br>

* `curl -X POST https://tk-railsnuxtv1-api.herokuapp.com/api/v1/auth_token \ -H "X-Requested-With: XMLHttpRequest" \ -H "Content-Type: application/json" \ -d '{"auth": {"email": "user0@example.com", "password": "password"}}'`を実行<br>

```
{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg5ODkyNjYsInN1YiI6Iit5NmZhOWFDRENDWitCY2UvdGNqeTMxY0tZelNKekZjcVltaVpCajdHTTRuZ1o2UkhUcnN6dGpDNVcrK2hMd3VQaDZZcE9NZmNQQjVTQkY1YVBqcWJ2TnBtYWdSMnQ3d013dz0tLUFacllzZi9zT1pIRHRhcXotLUxzUS9XRVA3cXVTdUhueVJhRGQvOVE9PSIsImlzcyI6Imh0dHBzOi8vdGstcmFpbHNudXh0djEtYXBpLmhlcm9rdWFwcC5jb20iLCJhdWQiOiJodHRwczovL3RrLXJhaWxzbnV4dHYxLWFwaS5oZXJva3VhcHAuY29tIn0.4PAg7PDAvOP9Lmc-4DQZIWTCb5jRygIHB3KSbzxxR-c","expires":1648989266,"user":{"id":1,"name":"user0","sub":"+y6fa9aCDCCZ+Bce/tcjy31cKYzSJzFcqYmiZBj7GM4ngZ6RHTrsztjC5W++hLwuPh6YpOMfcPB5SBF5aPjqbvNpmagR2t7wMww=--AZrYsf/sOZHDtaqz--LsQ/WEP7quSuHnyRaDd/9Q=="}}
```

- `api $ heroku logs --tail`を実行<br>

```
2022-04-03T11:43:21.518340+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/lib/puma/cli.rb:80:in `run'
2022-04-03T11:43:21.518340+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/bin/puma:10:in `<top (required)>'
2022-04-03T11:43:21.518341+00:00 app[web.1]: /usr/local/bundle/bin/puma:23:in `load'
2022-04-03T11:43:21.518341+00:00 app[web.1]: /usr/local/bundle/bin/puma:23:in `<top (required)>'
2022-04-03T11:43:21.749962+00:00 heroku[web.1]: Process exited with status 1
2022-04-03T11:43:21.834313+00:00 heroku[web.1]: State changed from starting to crashed
2022-04-03T11:43:21.852967+00:00 heroku[web.1]: State changed from crashed to starting
2022-04-03T11:43:30.009785+00:00 heroku[web.1]: Starting process with command `/bin/sh -c bundle\ exec\ puma\ -C\ config/puma.rb`
2022-04-03T11:43:32.050287+00:00 app[web.1]: [3] Puma starting in cluster mode...
2022-04-03T11:43:32.050311+00:00 app[web.1]: [3] * Version 4.3.11 (ruby 2.7.2-p137), codename: Mysterious Traveller
2022-04-03T11:43:32.050312+00:00 app[web.1]: [3] * Min threads: 5, max threads: 5
2022-04-03T11:43:32.050312+00:00 app[web.1]: [3] * Environment: production
2022-04-03T11:43:32.050313+00:00 app[web.1]: [3] * Process workers: 2
2022-04-03T11:43:32.050313+00:00 app[web.1]: [3] * Preloading application
2022-04-03T11:43:35.235011+00:00 app[web.1]: [3] ! Unable to load application: NoMethodError: undefined method `action_despatch' for #<Rails::Application::Configuration:0x00007f7dea184150>
2022-04-03T11:43:35.235299+00:00 app[web.1]: bundler: failed to load command: puma (/usr/local/bundle/bin/puma)
2022-04-03T11:43:35.235673+00:00 app[web.1]: NoMethodError: undefined method `action_despatch' for #<Rails::Application::Configuration:0x00007f7dea184150>
2022-04-03T11:43:35.235674+00:00 app[web.1]: /usr/local/bundle/gems/railties-6.0.4.6/lib/rails/railtie/configuration.rb:96:in `method_missing'
2022-04-03T11:43:35.235675+00:00 app[web.1]: /app/config/application.rb:44:in `<class:Application>'
2022-04-03T11:43:35.235676+00:00 app[web.1]: /app/config/application.rb:24:in `<module:App>'
2022-04-03T11:43:35.235676+00:00 app[web.1]: /app/config/application.rb:22:in `<top (required)>'
2022-04-03T11:43:35.235677+00:00 app[web.1]: /app/config/environment.rb:2:in `require_relative'
2022-04-03T11:43:35.235677+00:00 app[web.1]: /app/config/environment.rb:2:in `<top (required)>'
2022-04-03T11:43:35.235678+00:00 app[web.1]: config.ru:3:in `require_relative'
2022-04-03T11:43:35.235678+00:00 app[web.1]: config.ru:3:in `block in <main>'
2022-04-03T11:43:35.235679+00:00 app[web.1]: /usr/local/bundle/gems/rack-2.2.3/lib/rack/builder.rb:116:in `eval'
2022-04-03T11:43:35.235679+00:00 app[web.1]: /usr/local/bundle/gems/rack-2.2.3/lib/rack/builder.rb:116:in `new_from_string'
2022-04-03T11:43:35.235680+00:00 app[web.1]: /usr/local/bundle/gems/rack-2.2.3/lib/rack/builder.rb:105:in `load_file'
2022-04-03T11:43:35.235680+00:00 app[web.1]: /usr/local/bundle/gems/rack-2.2.3/lib/rack/builder.rb:66:in `parse_file'
2022-04-03T11:43:35.235681+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/lib/puma/configuration.rb:321:in `load_rackup'
2022-04-03T11:43:35.235681+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/lib/puma/configuration.rb:246:in `app'
2022-04-03T11:43:35.235682+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/lib/puma/runner.rb:155:in `load_and_bind'
2022-04-03T11:43:35.235682+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/lib/puma/cluster.rb:413:in `run'
2022-04-03T11:43:35.235682+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/lib/puma/launcher.rb:172:in `run'
2022-04-03T11:43:35.235683+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/lib/puma/cli.rb:80:in `run'
2022-04-03T11:43:35.235683+00:00 app[web.1]: /usr/local/bundle/gems/puma-4.3.11/bin/puma:10:in `<top (required)>'
2022-04-03T11:43:35.235683+00:00 app[web.1]: /usr/local/bundle/bin/puma:23:in `load'
2022-04-03T11:43:35.235683+00:00 app[web.1]: /usr/local/bundle/bin/puma:23:in `<top (required)>'
2022-04-03T11:43:35.392233+00:00 heroku[web.1]: Process exited with status 1
2022-04-03T11:43:35.453754+00:00 heroku[web.1]: State changed from starting to crashed
2022-04-03T11:44:39.477639+00:00 app[api]: Starting process with command `rails db:migrate` by user takaki55730317@gmail.com
2022-04-03T11:44:48.408812+00:00 heroku[run.3733]: State changed from starting to up
2022-04-03T11:44:48.637535+00:00 heroku[run.3733]: Awaiting client
2022-04-03T11:44:48.677783+00:00 heroku[run.3733]: Starting process with command `rails db:migrate`
2022-04-03T11:44:59.984537+00:00 heroku[run.3733]: Process exited with status 1
2022-04-03T11:45:00.232168+00:00 heroku[run.3733]: State changed from up to complete
2022-04-03T11:46:56.448324+00:00 app[api]: Starting process with command `rails db:migrate` by user takaki55730317@gmail.com
2022-04-03T11:47:04.512539+00:00 heroku[run.7430]: State changed from starting to up
2022-04-03T11:47:04.641982+00:00 heroku[run.7430]: Awaiting client
2022-04-03T11:47:04.663254+00:00 heroku[run.7430]: Starting process with command `rails db:migrate`
2022-04-03T11:47:12.197824+00:00 heroku[run.7430]: Process exited with status 1
2022-04-03T11:47:12.307617+00:00 heroku[run.7430]: State changed from up to complete
2022-04-03T11:53:08.000000+00:00 app[api]: Build started by user takaki55730317@gmail.com
2022-04-03T11:54:35.000000+00:00 app[api]: Build succeeded
2022-04-03T11:54:35.579189+00:00 app[api]: Deploy 5c327b28 by user takaki55730317@gmail.com
2022-04-03T11:54:35.579189+00:00 app[api]: Release v14 created by user takaki55730317@gmail.com
2022-04-03T11:54:35.819458+00:00 heroku[web.1]: State changed from crashed to starting
2022-04-03T11:54:43.771510+00:00 heroku[web.1]: Starting process with command `/bin/sh -c bundle\ exec\ puma\ -C\ config/puma.rb`
2022-04-03T11:54:45.853823+00:00 app[web.1]: [3] Puma starting in cluster mode...
2022-04-03T11:54:45.853887+00:00 app[web.1]: [3] * Version 4.3.11 (ruby 2.7.2-p137), codename: Mysterious Traveller
2022-04-03T11:54:45.853887+00:00 app[web.1]: [3] * Min threads: 5, max threads: 5
2022-04-03T11:54:45.853888+00:00 app[web.1]: [3] * Environment: production
2022-04-03T11:54:45.853888+00:00 app[web.1]: [3] * Process workers: 2
2022-04-03T11:54:45.853888+00:00 app[web.1]: [3] * Preloading application
2022-04-03T11:54:51.132045+00:00 app[web.1]: [3] * Listening on tcp://0.0.0.0:4902
2022-04-03T11:54:51.132221+00:00 app[web.1]: [3] ! WARNING: Detected 1 Thread(s) started in app boot:
2022-04-03T11:54:51.132372+00:00 app[web.1]: [3] ! #<Thread:0x00007faf0da8cf28 /usr/local/bundle/gems/activerecord-6.0.4.6/lib/active_record/connection_adapters/abstract/connection_pool.rb:334 sleep> - /usr/local/bundle/gems/activerecord-6.0.4.6/lib/active_record/connection_adapters/abstract/connection_pool.rb:337:in `sleep'
2022-04-03T11:54:51.132607+00:00 app[web.1]: [3] Use Ctrl-C to stop
2022-04-03T11:54:51.144374+00:00 app[web.1]: [3] - Worker 0 (pid: 5) booted, phase: 0
2022-04-03T11:54:51.154348+00:00 app[web.1]: [3] - Worker 1 (pid: 18) booted, phase: 0
2022-04-03T11:54:51.640090+00:00 heroku[web.1]: State changed from starting to up
2022-04-03T11:55:01.829357+00:00 app[api]: Starting process with command `rails db:migrate` by user takaki55730317@gmail.com
2022-04-03T11:55:07.984563+00:00 heroku[run.4268]: Awaiting client
2022-04-03T11:55:07.999365+00:00 heroku[run.4268]: Starting process with command `rails db:migrate`
2022-04-03T11:55:08.066642+00:00 heroku[run.4268]: State changed from starting to up
2022-04-03T11:55:14.527894+00:00 heroku[run.4268]: Process exited with status 0
2022-04-03T11:55:14.599417+00:00 heroku[run.4268]: State changed from up to complete
2022-04-03T11:57:30.725430+00:00 app[api]: Starting process with command `rails r "puts User.column_name"` by user takaki55730317@gmail.com
2022-04-03T11:57:38.235065+00:00 heroku[run.4323]: State changed from starting to up
2022-04-03T11:57:38.369453+00:00 heroku[run.4323]: Awaiting client
2022-04-03T11:57:38.386849+00:00 heroku[run.4323]: Starting process with command `rails r "puts User.column_name"`
2022-04-03T11:57:46.690066+00:00 heroku[run.4323]: Process exited with status 1
2022-04-03T11:57:46.808599+00:00 heroku[run.4323]: State changed from up to complete
2022-04-03T11:58:03.401028+00:00 app[api]: Starting process with command `rails r "puts User.column_names"` by user takaki55730317@gmail.com
2022-04-03T11:58:12.918779+00:00 heroku[run.1844]: State changed from starting to up
2022-04-03T11:58:13.214111+00:00 heroku[run.1844]: Awaiting client
2022-04-03T11:58:13.252112+00:00 heroku[run.1844]: Starting process with command `rails r "puts User.column_names"`
2022-04-03T11:58:25.413089+00:00 heroku[run.1844]: Process exited with status 0
2022-04-03T11:58:25.501873+00:00 heroku[run.1844]: State changed from up to complete
2022-04-03T12:04:26.356567+00:00 app[web.1]: I, [2022-04-03T21:04:26.356375 #18]  INFO -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c] Started POST "/api/v1/auth_token" for 113.156.7.172 at 2022-04-03 21:04:26 +0900
2022-04-03T12:04:26.361596+00:00 app[web.1]: I, [2022-04-03T21:04:26.361502 #18]  INFO -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c] Processing by Api::V1::AuthTokenController#create as */*
2022-04-03T12:04:26.361703+00:00 app[web.1]: I, [2022-04-03T21:04:26.361660 #18]  INFO -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c]   Parameters: {"auth"=>{"email"=>"user0@example.com", "password"=>"[FILTERED]"}, "auth_token"=>{"auth"=>{"email"=>"user0@example.com", "password"=>"[FILTERED]"}}}
2022-04-03T12:04:26.426430+00:00 app[web.1]: D, [2022-04-03T21:04:26.426319 #18] DEBUG -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c]   User Load (3.2ms)  SELECT "users".* FROM "users" WHERE "users"."email" = $1 AND "users"."activated" = $2 LIMIT $3  [["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
2022-04-03T12:04:26.716410+00:00 app[web.1]: D, [2022-04-03T21:04:26.716302 #18] DEBUG -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c]   User Load (1.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
2022-04-03T12:04:26.720986+00:00 app[web.1]: D, [2022-04-03T21:04:26.720897 #18] DEBUG -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c]    (1.3ms)  BEGIN
2022-04-03T12:04:26.723963+00:00 app[web.1]: D, [2022-04-03T21:04:26.723689 #18] DEBUG -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c]   User Load (1.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
2022-04-03T12:04:26.729150+00:00 app[web.1]: D, [2022-04-03T21:04:26.729048 #18] DEBUG -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c]   User Update (1.6ms)  UPDATE "users" SET "refresh_jti" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["refresh_jti", "b60f0628e83caf8bb281e871e686ccfe"], ["updated_at", "2022-04-03 12:04:26.725815"], ["id", 1]]
2022-04-03T12:04:26.731958+00:00 app[web.1]: D, [2022-04-03T21:04:26.731868 #18] DEBUG -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c]    (2.2ms)  COMMIT
2022-04-03T12:04:26.737807+00:00 app[web.1]: I, [2022-04-03T21:04:26.737721 #18]  INFO -- : [c28a2231-2c60-4f1a-a659-9c0a76607f4c] Completed 200 OK in 376ms (Views: 0.5ms | ActiveRecord: 45.0ms | Allocations: 11648)
2022-04-03T12:04:26.739200+00:00 heroku[router]: at=info method=POST path="/api/v1/auth_token" host=tk-railsnuxtv1-api.herokuapp.com request_id=c28a2231-2c60-4f1a-a659-9c0a76607f4c fwd="113.156.7.172" dyno=web.1 connect=0ms service=387ms status=200 bytes=1636 protocol=https
```

- `api $ curl -X POST https://tk-railsnuxtv1-api.herokuapp.com/api/v1/auth_token \ -H "X-Requested-With: XMLHttpRequest" \ -H "Content-Type: application/json" \ -d '{"auth": {"email": "user0@example.com", "password": "passwor"}}'`を実行(password を変えてみる)<br>

* heroku の log には`service=335ms status=404 bytes=429 protocol=https`が返ってきている<br>

### このチャプターのまとめ

リフレッシュとアクセスを使った Rails のログイン機能の構築<br>

|         種類         |                     役割                     |       保存先・取得先       | 有効期限 |       トークンの無効化        |
| :------------------: | :------------------------------------------: | :------------------------: | :------: | :---------------------------: |
| リフレッシュトークン |      セッション管理<br>(アクセスと発行)      |           Cookie           | 24 時間  | ユーザーテーブルの jti を削除 |
|   アクセストークン   | リソースの保護<br>(本人認証とコンテンツ保護) | メモリ・リクエストヘッダー |  30 分   |               -               |
