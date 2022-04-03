## 82 認証を行うコントローラを作成する

- `root $ docker compose run --rm api rails g controller Api::v1::AuthToken`を実行<br>

* `api/app/controllers/application_controller.rb`を編集<br>

```rb:application_controller.rb
class ApplicationController < ActionController::API # Cookieを扱う
  include ActionController::Cookies # 認可を行う
  include UserAuthenticateService

  # CSRF対策
  before_action :xhr_request?

  private

  # XMLHttpRequestでない場合は403エラーを返す
  def xhr_request?
    return if request.xhr?
    render status: :forbidden, json: { status: 403, error: 'Forbidden' }
  end

  # Internal Server Error
  def response_500(msg = 'Internal Server Error')
    render status: 500, json: { status: 500, error: msg }
  end # リクエストヘッダ X-Requested-With: 'XMLHttpRequest' の存在を判定
end
```

- `api/app/controllers/api/v1/auth_token_controller.rb`を編集<br>

```rb:auth_token_controller.rb
class Api::V1::AuthTokenController < ApplicationController
  include UserSessionizeService

  # 404エラーが発生した場合にヘッダーのみを返す
  rescue_from UserAuth.not_found_exception_class, with: :not_found # refresh_tokenのInvalidJitErrorが発生した場合はカスタムエラーを返す
  rescue_from JWT::InvalidJtiError, with: :invalid_jti

  # userのログイン情報を確認する
  before_action :authenticate, only: %i[create] # 処理前にsessionを削除する
  before_action :delete_session, only: %i[create] # session_userを取得、存在しない場合は401を返す
  before_action :sessionize_user, only: %i[refresh destroy]

  # ログイン
  def create
    @user = login_user
    set_refresh_token_to_cookie
    render json: login_response
  end

  # リフレッシュ
  def refresh
    @user = session_user
    set_refresh_token_to_cookie
    render json: login_response
  end

  # ログアウト
  def destroy
    delete_session if session_user.forget
    if cookies[session_key].nil?
      head(:ok)
    else
      response_500('Could not delete session')
    end
  end

  private

  # params[:email]からアクティブなユーザーを返す
  def login_user
    @_login_user ||= User.find_by_activated(auth_params[:email])
  end

  # ログインユーザーが居ない、もしくはpasswordが一致しない場合404を返す
  def authenticate
    unless login_user.present? &&
             login_user.authenticate(auth_params[:password])
      raise UserAuth.not_found_exception_class
    end
  end

  # refresh_tokenをcookieにセットする
  def set_refresh_token_to_cookie
    cookies[session_key] = {
      value: refresh_token,
      expires: refresh_token_expiration,
      secure: Rails.env.production?,
      http_only: true
    }
  end

  # ログイン時のデフォルトレスポンス
  def login_response
    {
      token: access_token,
      expires: access_token_expiration,
      user: @user.response_json(sub: access_token_subject)
    }
  end

  # リフレッシュトークンのインスタンス生成
  def encode_refresh_token
    @_encode_refresh_token ||= @user.encode_refresh_token
  end

  # リフレッシュトークン
  def refresh_token
    encode_refresh_token.token
  end

  # リフレッシュトークンの有効期限
  def refresh_token_expiration
    Time.at(encode_refresh_token.payload[:exp])
  end

  # アクセストークンのインスタンス生成
  def encode_access_token
    @_encode_access_token ||= @user.encode_access_token
  end

  # アクセストークン
  def access_token
    encode_access_token.token
  end

  # アクセストークンの有効期限
  def access_token_expiration
    encode_access_token.payload[:exp]
  end

  # アクセストークンのsubjectクレーム
  def access_token_subject
    encode_access_token.payload[:sub]
  end

  # 404ヘッダーのみの返却を行う
  # Doc: https://gist.github.com/mlanett/a31c340b132ddefa9cca
  def not_found
    head(:not_found)
  end

  # decode jti != user.refresh_jti のエラー処理
  def invalid_jti
    msg = 'Invalid jti for refresh token'
    render status: 401, json: { status: 401, error: msg }
  end

  def auth_params
    params.require(:auth).permit(:email, :password)
  end
end
```

- `api/app/models/user.rb`を編集<br>

```rb:user.rb
require 'validator/email_validator'

class User < ApplicationRecord # Token生成モジュール
  include TokenGenerateService

  # Userクラスの一番上に追加
  # バリデーション直前
  before_validation :downcase_email

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
  validates :name,
            presence: true,
            length: {
              maximum: 30,
              # Null(nil), 空白文字の場合スキップ(空白文字の場合には無駄な検証を行わない)
              # 入力必須
              # 最大文字数
              allow_blank: true
            }
  validates :email, presence: true, email: { allow_blank: true }

  VALID_PASSWORD_REGEX = /\A[\w\-]+\z/ # \z     => 文字列の末尾にマッチ
  validates :password,
            presence: true,
            length: {
              minimum: 8,
              # nameのみのupdate時にpasswordがnilになっていてもpassword必須とはならない(allow_nilが利く)
              # 最小文字数
              allow_blank: true
            },
            format: {
              # 書式チェック
              with: VALID_PASSWORD_REGEX,
              message: :invalid_password,
              allow_blank: true
            },
            # 空パスワードのアップデートを許容する。(Null(nil)の場合スキップ)
            allow_nil: true

  ## methods
  # class method  ###########################
  class << self
    def find_by_activated(email)
      find_by(email: email, activated: true)
    end
  end # class method end #########################

  # 自分以外の同じemailのアクティブなユーザーがいる場合にtrueを返す
  def email_activated?
    users = User.where.not(id: id)
    users.find_by_activated(email).present?
  end

  # リフレッシュトークンのJWT IDを記憶する
  def remember(jti)
    update!(refresh_jti: jti)
  end

  # リフレッシュトークンのJWT IDを削除する
  def forget
    update!(refresh_jti: nil)
  end

  # 追加
  # 共通のJSONレスポンス
  def response_json(payload = {})
    as_json(only: %i[id name]).merge(payload).with_indifferent_access
  end

  private

  # email小文字化
  def downcase_email
    self.email.downcase! if email
  end # emailからアクティブなユーザーを返す
end
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> User.first.as_json`を実行<br>

```
  User Load (38.3ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> {"id"=>1, "name"=>"user0", "email"=>"user0@example.com", "password_digest"=>"$2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q", "activated"=>true, "admin"=>false, "created_at"=>"2022-03-16T11:15:58.112+09:00", "updated_at"=>"2022-04-02T17:48:02.158+09:00", "refresh_jti"=>"2b1fa6be5cf69bd80a67577d2e61e8c9"}
```

- `irb(main):002:0> User.first.as_json(sub: "sub")`を実行<br>

```
  User Load (11.4ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> {"id"=>1, "name"=>"user0", "email"=>"user0@example.com", "password_digest"=>"$2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q", "activated"=>true, "admin"=>false, "created_at"=>"2022-03-16T11:15:58.112+09:00", "updated_at"=>"2022-04-02T17:48:02.158+09:00", "refresh_jti"=>"2b1fa6be5cf69bd80a67577d2e61e8c9"}
```

- `irb(main):004:0> User.first.as_json(sub: "sub").with_indifferent_access`を実行<br>

```
  User Load (1.0ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> {"id"=>1, "name"=>"user0", "email"=>"user0@example.com", "password_digest"=>"$2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q", "activated"=>true, "admin"=>false, "created_at"=>"2022-03-16T11:15:58.112+09:00", "updated_at"=>"2022-04-02T17:48:02.158+09:00", "refresh_jti"=>"2b1fa6be5cf69bd80a67577d2e61e8c9"}
```

- `irb(main):005:0> User.first.response_json(sub: "aaa")`を実行<br>

```
  User Load (0.9ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> {"id"=>1, "name"=>"user0", "sub"=>"aaa"}
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
    end
  end
end
```

- `api/app/controllers/api/v1/users_controller.rb`を編集<br>

```rb:users_controller.rb
class Api::V1::UsersController < ApplicationController; end # 中身を削除
```

- `root $ docker compose up api`を実行<br>

- `root $ curl -X POST http://localhost:3000/api/v1/auth_token \<br> -H "X-Requested-With: XMLHttpRequest" \<br> -H "Content-Type: application/json" \<br> -d '{"auth": {"email": "user0@example.com", "password": "password"}}'`を実行<br>

```
{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg5Nzg0OTcsInN1YiI6IlVZWXhDa2ZwVmhJQmttVHpCTkQ1cnZqc1dOUDdGbjZCZnIzN2tWbzRFd2RkbHFiTExWNDViZGhYUGRaK3l4bkZFcXJYR08vaFFnYWxZajJwbHpkOCszTFExTVdmRkhFdGJTcz0tLWZ0WXJZYmFjdHFaMytvRzQtLUducnRqUW5zcWJhckdEVnNXMVB5UXc9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.61FtpWElOINnwO_RLnsFEMcw54l2cFoufriHIum2K4Y","expires":1648978497,"user":{"id":1,"name":"user0","sub":"UYYxCkfpVhIBkmTzBND5rvjsWNP7Fn6Bfr37kVo4EwddlqbLLV45bdhXPdZ+yxnFEqrXGO/hQgalYj2plzd8+3LQ1MWfFHEtbSs=--ftYrYbactqZ3+oG4--GnrtjQnsqbarGDVsW1PyQw=="}}%
```

- `root $ curl -X POST http://localhost:3000/api/v1/auth_token \ -H "X-Requested-With: XMLHttpRequest" \ -H "Content-Type: application/json" \ -d '{"auth": {"email": "user0@example.com", "password": "pass"}}'`を実行(password を間違えてみる)<br>

```
 Completed 404 Not Found in 548ms (ActiveRecord: 32.0ms | Allocations: 7460)
```

- `root $ curl -X POST http://localhost:3000/api/v1/auth_token \ -H "Content-Type: application/json" \ -d '{"auth": {"email": "user0@example.com", "password": "pass"}}'`を実行<br>

```
{"status":403,"error":"Forbidden"}%
```

## 83 認証コントローラのテスト

- `api/test/test_helper.rb`を編集<br>

```rb:test_helper.rb
ENV['RAILS_ENV'] ||= 'test'
require_relative '../config/environment'
require 'rails/test_help'

# gem minitest-reporters setup
require 'minitest/reporters'
Minitest::Reporters.use!

class ActiveSupport::TestCase # プロセスが分岐した直後に呼び出される # # 並列テスト ... 複数のプロセスを分岐させテスト時間の短縮を行う機能
  parallelize_setup do |worker|
    load # seedデータの読み込み
    "#{Rails.root}/db/seeds.rb"
  end

  # Run tests in parallel with specified workers
  # 並列テストの有効化・無効化
  # workers: プロセス数を渡す(2以上 => 有効、2未満 => 無効)
  # number_of_processors => 使用しているマシンのコア数(2)
  parallelize(workers: :number_of_processors) # parallelize(workers: 1) # 並列テストを無効にしたい場合

  # アクティブなユーザーを返す
  def active_user
    User.find_by(activated: true)
  end

  # 追加
  # api path
  def api(path = '/')
    "/api/v1#{path}"
  end

  # 認可ヘッダ
  def auth(token)
    { Authorization: "Bearer #{token}" }
  end

  # 引数のparamsでログインを行う
  def login(params)
    post api('/auth_token'), xhr: true, params: params
  end

  # ログアウトapi
  def logout
    delete api('/auth_token'), xhr: true
  end

  # レスポンスJSONをハッシュで返す
  def res_body
    JSON.parse(@response.body)
  end # ここまで
end
```

- `api/test/controllers/api/v1/auth_token_controller_test.rb`を編集<br>

```rb:auth_token_controller_test.rb
# テスト成功要件
# cookies[]の操作にはapplication.rbにCookieを処理するmeddlewareを追加
# config.middleware.use ActionDispatch::Cookies
require 'test_helper'

class Api::V1::AuthTokenControllerTest < ActionDispatch::IntegrationTest
  def setup
    @user = active_user
    @params = { auth: { email: @user.email, password: 'password' } }
    @access_lifetime = UserAuth.access_token_lifetime
    @refresh_lifetime = UserAuth.refresh_token_lifetime
    @session_key = UserAuth.session_key.to_s
    @access_token_key = 'token'
  end

  # tokenのリフレッシュを行うapi
  def refresh_api
    post api('/auth_token/refresh'), xhr: true
  end

  # 無効なリクエストで返ってくるレスポンスチェック
  def response_check_of_invalid_request(status, error_msg = nil)
    assert_response status
    @user.reload
    assert_nil @user.refresh_jti
    assert_not response.body.present? if error_msg.nil?
    assert_equal error_msg, res_body['error'] if !error_msg.nil?
  end

  # 有効なログイン
  test 'valid_login_from_create_action' do
    login(@params)
    assert_response 200
    access_lifetime_to_i = @access_lifetime.from_now.to_i
    refresh_lifetime_to_i = @refresh_lifetime.from_now.to_i

    # jtiは保存されているか
    @user.reload
    assert_not_nil @user.refresh_jti

    # レスポンスユーザーは正しいか
    assert_equal @user.id, res_body['user']['id']

    # レスポンス有効期限は想定通りか(1誤差許容)
    assert_in_delta access_lifetime_to_i, res_body['expires'], 1

    ## access_token
    access_token = User.decode_access_token(res_body[@access_token_key])

    # ユーザーはログイン本人と一致しているか
    assert_equal @user, access_token.entity_for_user

    # 有効期限はレスポンスと一致しているか
    token_exp = access_token.payload['exp']
    assert_equal res_body['expires'], token_exp

    ## cookie
    # cookieのオプションを取得する場合は下記を使用
    # @request.cookie_jar.instance_variable_get(:@set_cookies)[<key>]
    cookie =
      @request.cookie_jar.instance_variable_get(:@set_cookies)[@session_key]

    # expiresは想定通りか(1秒許容)
    assert_in_delta Time.at(refresh_lifetime_to_i), cookie[:expires], 1.seconds

    # secureは一致しているか
    assert_equal Rails.env.production?, cookie[:secure]

    # http_onlyはtrueか
    assert cookie[:http_only]

    ## refresh_token
    token_from_cookies = cookies[@session_key]
    refresh_token = User.decode_refresh_token(token_from_cookies)
    @user.reload

    # ログイン本人と一致しているか
    assert_equal @user, refresh_token.entity_for_user

    # jtiは一致しているか
    assert_equal @user.refresh_jti, refresh_token.payload['jti']

    # token有効期限とcookie有効期限は一致しているか
    assert_equal refresh_lifetime_to_i, refresh_token.payload['exp']
  end

  # 無効なログイン
  test 'invalid_login_from_create_action' do
    # 不正なユーザーの場合
    pass = 'password'
    invalid_params = { auth: { email: @user.email, password: pass + 'a' } }
    login(invalid_params)
    response_check_of_invalid_request 404

    # アクティブユーザーでない場合
    inactive_user = User.create(name: 'a', email: 'a@a.a', password: pass)
    invalid_params = { auth: { email: inactive_user.email, password: pass } }
    assert_not inactive_user.activated
    login(invalid_params)
    response_check_of_invalid_request 404

    # Ajax通信ではない場合
    post api('/auth_token'), xhr: false, params: @params
    response_check_of_invalid_request 403, 'Forbidden'
  end

  # 有効なリフレッシュ
  test 'valid_refresh_from_refresh_action' do
    # 有効なログイン
    login(@params)
    assert_response 200
    @user.reload
    old_access_token = res_body[@access_token_key]
    old_refresh_token = cookies[@session_key]
    old_user_jti = @user.refresh_jti

    # nilでないか
    assert_not_nil old_access_token
    assert_not_nil old_refresh_token
    assert_not_nil old_user_jti

    # refreshアクションにアクセス
    refresh_api
    assert_response 200
    @user.reload
    new_access_token = res_body[@access_token_key]
    new_refresh_token = cookies[@session_key]
    new_user_jti = @user.refresh_jti

    # nilでないか
    assert_not_nil new_access_token
    assert_not_nil new_refresh_token
    assert_not_nil new_user_jti

    # tokenとjtiが新しく発行されているか
    assert_not_equal old_access_token, new_access_token
    assert_not_equal old_refresh_token, new_refresh_token
    assert_not_equal old_user_jti, new_user_jti

    # user.refresh_jtiは最新のjtiと一致しているか
    payload = User.decode_refresh_token(new_refresh_token).payload
    assert_equal payload['jti'], new_user_jti
  end

  # 無効なリフレッシュ
  test 'invalid_refresh_from_refresh_action' do
    # refresh_tokenが存在しない場合はアクセスできないか
    refresh_api
    response_check_of_invalid_request 401

    ## ユーザーが2回のログインを行なった場合
    # 1つ目のブラウザでログイン
    login(@params)
    assert_response 200
    old_refresh_token = cookies[@session_key]

    # 2つ目のブラウザでログイン
    login(@params)
    assert_response 200
    new_refresh_token = cookies[@session_key]

    # cookieに古いrefresh_tokenをセット
    cookies[@session_key] = old_refresh_token
    assert_not cookies[@session_key].blank?

    # 1つ目のブラウザ(古いrefresh_token)でアクセスするとエラーを吐いているか
    refresh_api
    assert_response 401

    # cookieは削除されているか
    assert cookies[@session_key].blank?

    # jtiエラーはカスタムメッセージを吐いているか
    assert_equal 'Invalid jti for refresh token', res_body['error']

    # 有効期限後はアクセスできないか
    travel_to (@refresh_lifetime.from_now) do
      refresh_api
      assert_response 401
      assert_not response.body.present?
    end
  end

  # ログアウト
  test 'destroy_action' do
    # 有効なログイン
    login(@params)
    assert_response 200
    @user.reload
    assert_not_nil @user.refresh_jti
    assert_not_nil @request.cookie_jar[@session_key]

    # 有効なログアウト
    assert_not cookies[@session_key].blank?
    logout
    assert_response 200

    # cookieは削除されているか
    assert cookies[@session_key].blank?

    # userのjtiは削除できているか
    @user.reload
    assert_nil @user.refresh_jti

    # sessionがない状態でログアウトしたらエラーは返ってくるか
    cookies[@session_key] = nil
    logout
    response_check_of_invalid_request 401

    # 有効なログイン
    login(@params)
    assert_response 200
    assert_not cookies[@session_key].blank?

    # session有効期限後にログアウトしたらエラーは返ってくるか
    travel_to (@refresh_lifetime.from_now) do
      logout
      assert_response 401
      assert cookies[@session_key].blank? # cookieは削除されているか
    end
  end
end
```

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

    # 追加
    # $LOAD_PATHにautoload pathを追加しない(Zeitwerk有効時false推奨)
    config.add_autoload_paths_to_load_path = false

    # Cookieを処理するmeddleware
    config.middleware.use ActionDispatch::Cookies

    config.api_only = true
  end
end
```

- `root $ docker compose run --rm api rails t`を実行<br>

```
users...---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=] 0% Time: 00:00:00,  ETA: ??:??:??
users...
users = 10
users = 10
  18/18: [====================================================================================================================================================================================================] 100% Time: 00:00:05, Time: 00:00:05

Finished in 5.47185s
18 tests, 155 assertions, 0 failures, 0 errors, 0 skips
```
