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
      config.action_despatch.cookies_same_site_protection =
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
