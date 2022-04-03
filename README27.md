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
