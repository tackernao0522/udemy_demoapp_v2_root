## 75 JWT の初期設定ファイルを作成する

- `root $ touch api/config/initializers/user_auth.rb`を実行<br>

* `api/config/initializers/user_auth.rb`を編集<br>

```rb:user_auth.rb
module UserAuth # access tokenの有効期限
  mattr_accessor :access_token_lifetime
  self.access_token_lifetime = 30.minute

  # refresh tokenの有効期限
  mattr_accessor :refresh_token_lifetime
  self.refresh_token_lifetime = 1.day

  # cookieからrefresh tokenを取得する際のキー
  mattr_accessor :session_key
  self.session_key = :refresh_token

  # userを識別するクレーム名
  mattr_accessor :user_claim
  self.user_claim = :sub

  # JWTの発行者を識別する文字列(認可サーバーURL)
  mattr_accessor :token_issuer
  self.token_issuer = ENV['BASE_URL']

  # JWTの受信者を識別する文字列(保護リソースURL)
  mattr_accessor :token_audience
  self.token_audience = ENV['BASE_URL']

  # JWTの署名アルゴリズム
  mattr_accessor :token_signature_algorithm
  self.token_signature_algorithm = 'HS256'

  # 署名・検証に使用する秘密鍵
  mattr_accessor :token_secret_signature_key
  self.token_secret_signature_key =
    Rails.application.credentials.secret_key_base

  # 署名・検証に使用する公開鍵(RS256)
  mattr_accessor :token_public_key
  self.token_public_key = nil

  # ユーザーが見つからない場合のエラーを指定
  mattr_accessor :not_found_exception_class
  self.not_found_exception_class = ActiveRecord::RecordNotFound
end
```

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
      API_DOMAIN: 'localhost:$FRONT_PORT'
      # 追加
      BASE_URL: 'http://localhost:$API_PORT'
    command: /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
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
        API_URL: 'http://localhost:$API_PORT'
    # コンテナで実行したいコマンド(CMD)
    command: yarn run dev
    volumes:
      - './front:/$WORKDIR'
    ports:
      - '$FRONT_PORT:3000'
    depends_on:
      - api
```

`$ api $ heroku info -s | grep web_url`を実行<br>

```:terminal
web_url=https://tk-railsnuxtv1-api.herokuapp.com/
```

- `api $ heroku config:set BASE_URL=https://tk-railsnuxtv1-api.herokuapp.com`を実行<br>

```:terminal
Setting BASE_URL and restarting ⬢ tk-railsnuxtv1-api... done, v11
BASE_URL: https://tk-railsnuxtv1-api.herokuapp.com
```

- `api $ heroku config:get BASE_URL`を実行<br>

```:terminal
https://tk-railsnuxtv1-api.herokuapp.com
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> UserAuth.token_issuer`を実行<br>

```:terminal
=> "http://localhost:3000"
```

- `irb(main):002:0> UserAuth.token_audience`を実行<br>

```:terminal
=> "http://localhost:3000"
```

## 76 トークン発行クラスの共通モジュールを作成

- `root $ mkdir -p api/app/services/user_auth && touch $_/token_commons.rb`を実行<br>

* `api/app/services/user_auth/token_commons.rb`を編集<br>

```rb:token_commons.rb
module UserAuth
  module TokenCommons # エンコードキー
    def secret_key
      UserAuth.token_secret_signature_key
    end

    # デコードキー
    def decode_key
      UserAuth.token_public_key || secret_key
    end

    # アルゴリズム
    def algorithm
      UserAuth.token_signature_algorithm
    end

    # user識別クレーム
    def user_claim
      UserAuth.user_claim
    end

    # キーがハッシュでも文字列でもuser_claimの値を返す
    def get_user_id_from(payload)
      payload.with_indifferent_access[user_claim]
    end

    # 暗号化クラスの生成
    # Doc: https://api.rubyonrails.org/v6.1.0/classes/ActiveSupport/MessageEncryptor.html
    # key_generatorメソッドのsecretはsecret_key_baseが使われる
    # 参考: https://techracho.bpsinc.jp/hachi8833/2017_10_24/46809
    def crypt
      salt = 'signed user id'
      key_length = ActiveSupport::MessageEncryptor.key_len
      secret = Rails.application.key_generator.generate_key(salt, key_length)
      ActiveSupport::MessageEncryptor.new(secret)
    end

    # user_id暗号化
    def encrypt_for(user_id)
      return unless user_id
      crypt.encrypt_and_sign(user_id.to_s, purpose: :authorization)
    end

    # user_id複合化(複合エラーの場合はnilを返す)
    def decrypt_for(user_id)
      return unless user_id
      crypt.decrypt_and_verify(user_id.to_s, purpose: :authorization)
    rescue StandardError
      nil
    end

    # エンコード時のヘッダー
    # Doc: https://openid-foundation-japan.github.io/draft-ietf-oauth-json-web-token-11.ja.html#typHdrDef
    def header_fields
      { typ: 'JWT', alg: algorithm }
    end
  end
end
```

## 77 リフレッシュトークンクラスを作成する

- `root $ touch api/app/services/user_auth/refresh_token.rb`を実行<br>

* `api/app/services/user_auth/refresh_token.rb`を編集<br>

```rb:refresh_token.rb
require 'jwt'

module UserAuth
  class RefreshToken
    include TokenCommons

    attr_reader :user_id, :payload, :token

    def initialize(user_id: nil, token: nil)
      if token.present?
        # decode
        @token =
          token
        @payload =
          JWT.decode(@token.to_s, decode_key, true, verify_claims).first
        @user_id = get_user_id_from(@payload)
      else
        # encode
        @user_id = encrypt_for(user_id)
        @payload = claims
        @token = JWT.encode(@payload, secret_key, algorithm, header_fields)
        remember_jti(user_id)
      end
    end

    # 暗号化されたuserIDからユーザーを取得する
    def entity_for_user(id = nil)
      id ||= @user_id
      User.find(decrypt_for(id))
    end

    private

    # リフレッシュトークンの有効期限
    def token_lifetime
      UserAuth.refresh_token_lifetime
    end

    # 有効期限をUnixtimeで返す(必須)
    def token_expiration
      token_lifetime.from_now.to_i
    end

    # jwt_idの生成(必須)
    # Digest::MD5.hexdigest => 複合不可のハッシュを返す
    # SecureRandom.uuid => 一意性の値を返す
    def jwt_id
      Digest::MD5.hexdigest(SecureRandom.uuid)
    end

    # エンコード時のデフォルトクレーム
    def claims
      { user_claim => @user_id, jti: jwt_id, exp: token_expiration }
    end

    # @payloadのjtiを返す
    def payload_jti
      @payload.with_indifferent_access[:jti]
    end

    # jtiをUsersテーブルに保存する
    def remember_jti(user_id)
      User.find(user_id).remember(payload_jti)
    end

    ##  デコードメソッド

    # デコード時のjwt_idを検証する(エラーはJWT::DecodeErrorに委託する)
    def verify_jti?(jti, payload)
      user_id = get_user_id_from(payload)
      decode_user = entity_for_user(user_id)
      decode_user.refresh_jti == jti
    rescue UserAuth.not_found_exception_class
      false
    end

    # デコード時のデフォルトオプション
    # Doc: https://github.com/jwt/ruby-jwt
    # default: https://www.rubydoc.info/github/jwt/ruby-jwt/master/JWT/DefaultOptions
    def verify_claims
      {
        verify_expiration: true,
        verify_jti:
          proc do |# 有効期限の検証するか(必須)
          jti, payload|
            verify_jti?(
              # jtiとセッションIDの検証
              jti,
              payload
            )
          end,
        algorithm: algorithm # decode時のアルゴリズム
      }
    end
  end
end
```

- `root $ docker compose run --rm api rails g migration add_refresh_jti_to_users refresh_jti:string`を実行<br>

```:terminal
[+] Running 1/0
 ⠿ Container udemy-rails-nuxt-db-1  Running                                                                                                                                                                        0.0s
      invoke  active_record
      create    db/migrate/20220401100440_add_refresh_jti_to_users.rb
```

- `root $ docker compose run --rm api rails db:migrate`を実行<br>

```:terminal
[+] Running 1/0
 ⠿ Container udemy-rails-nuxt-db-1  Running                                                                                                                                                                        0.0s
== 20220401100440 AddRefreshJtiToUsers: migrating =============================
-- add_column(:users, :refresh_jti, :string)
   -> 0.0770s
== 20220401100440 AddRefreshJtiToUsers: migrated (0.0772s) ====================
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> User.first`を実行<br>

```:terminal
  User Load (9.1ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-03-16 11:15:58 +0900 |             |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
1 row in set
```

- `api/app/models/user.rb`を編集<br>

```rb:user.rb
require 'validator/email_validator'

class User < ApplicationRecord # バリデーション直前 # Userクラスの一番上に追加
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

  # 追加
  # リフレッシュトークンのJWT IDを記憶する
  def remember(jti)
    update!(refresh_jti: jti)
  end

  # 追加
  # リフレッシュトークンのJWT IDを削除する
  def forget
    update!(refresh_jti: nil)
  end

  private

  # email小文字化
  def downcase_email
    self.email.downcase! if email
  end # emailからアクティブなユーザーを返す
end
```

- `irb(main):002:0> reload!`を実行<br>

- `irb(main):003:0> user = User.first`を実行<br>

```:terminal
  User Load (0.8ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
Traceback (most recent call last):
ArgumentError (wrong number of arguments (given 1, expected 0))
```

- `irb(main):004:0> user`を実行<br>

```:terminal
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-03-16 11:15:58 +0900 |             |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
1 row in set
```

- `irb(main):006:0> user.remember("a")`を実行<br>

```:terminal
   (1.3ms)  BEGIN
  User Load (1.7ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
  User Update (12.6ms)  UPDATE "users" SET "updated_at" = $1, "refresh_jti" = $2 WHERE "users"."id" = $3  [["updated_at", "2022-04-01 10:18:54.792764"], ["refresh_jti", "a"], ["id", 1]]
   (18.1ms)  COMMIT
=> true
```

- `irb(main):007:0> user`を実行<br>

```:terminal
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-01 19:18:54 +0900 | a           |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
1 row in set
```

- `irb(main):008:0> user.forget`を実行<br>

```:terminal
   (0.8ms)  BEGIN
  User Load (1.0ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
  User Update (2.5ms)  UPDATE "users" SET "updated_at" = $1, "refresh_jti" = $2 WHERE "users"."id" = $3  [["updated_at", "2022-04-01 10:20:46.198516"], ["refresh_jti", nil], ["id", 1]]
   (8.7ms)  COMMIT
=> true
```

- `irb(main):009:0> user`を実行<br>

```:terminal
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-01 19:20:46 +0900 |             |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+-------------+
1 row in set
```

- `irb(main):010:0> token1 = UserAuth::RefreshToken.new(user_id: user.id)`を実行<br>

```:terminal
  User Load (5.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
   (20.5ms)  BEGIN
  User Load (5.4ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
  User Update (3.2ms)  UPDATE "users" SET "refresh_jti" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["refresh_jti", "14d466c61fe3a179cfe85d0365769df2"], ["updated_at", "2022-04-01 10:23:06.884959"], ["id", 1]]
   (3.2ms)  COMMIT
Traceback (most recent call last):
ArgumentError (wrong number of arguments (given 1, expected 0))
```

- `irb(main):011:0> token1`を実行<br>

```:terminal
=> #<UserAuth::RefreshToken:0x00007f3d90bb1640 @user_id="YD/jdVcJxxm/iMoVIQxsK3eXGdSbYUgTdQkSesYVQO5GMmJgZY0AvKVvgqJpx22+ozEtZCvMf8pf9QaNt5OYrn9V8OMgmLB/jW8=--0SH5dc3Y/OPabQ38--WAJ+eNPxbe74V1bbkQ2Jog==", @payload={:sub=>"YD/jdVcJxxm/iMoVIQxsK3eXGdSbYUgTdQkSesYVQO5GMmJgZY0AvKVvgqJpx22+ozEtZCvMf8pf9QaNt5OYrn9V8OMgmLB/jW8=--0SH5dc3Y/OPabQ38--WAJ+eNPxbe74V1bbkQ2Jog==", :jti=>"14d466c61fe3a179cfe85d0365769df2", :exp=>1648894986}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJZRC9qZFZjSnh4bS9pTW9WSVF4c0szZVhHZFNiWVVnVGRRa1Nlc1lWUU81R01tSmdaWTBBdktWdmdxSnB4MjIrb3pFdFpDdk1mOHBmOVFhTnQ1T1lybjlWOE9NZ21MQi9qVzg9LS0wU0g1ZGMzWS9PUGFiUTM4LS1XQUorZU5QeGJlNzRWMWJia1EySm9nPT0iLCJqdGkiOiIxNGQ0NjZjNjFmZTNhMTc5Y2ZlODVkMDM2NTc2OWRmMiIsImV4cCI6MTY0ODg5NDk4Nn0.cDnGsgMsx6duF6dB4O5j1JmvRmzHyxymrHm6JpmYAEY">
```

- `irb(main):013:0> token1.payload[:jti]`を実行<br>

```:terminal
=> "14d466c61fe3a179cfe85d0365769df2"
```

- `irb(main):014:0> user.reload`を実行<br>

```:terminal
  User Load (1.4ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti                      |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-01 19:23:06 +0900 | 14d466c61fe3a179cfe85d0365769df2 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
1 row in set
```

- `irb(main):015:0> user.refresh_jti == token1.payload[:jti]`を実行<br>

```:termianl
=> true
```

- `irb(main):016:0> UserAuth::RefreshToken.new(token: token1.token)`を実行<br>

```:terminal
  User Load (5.6ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
=> #<UserAuth::RefreshToken:0x00007f3d90626748 @user_id="YD/jdVcJxxm/iMoVIQxsK3eXGdSbYUgTdQkSesYVQO5GMmJgZY0AvKVvgqJpx22+ozEtZCvMf8pf9QaNt5OYrn9V8OMgmLB/jW8=--0SH5dc3Y/OPabQ38--WAJ+eNPxbe74V1bbkQ2Jog==", @payload={"sub"=>"YD/jdVcJxxm/iMoVIQxsK3eXGdSbYUgTdQkSesYVQO5GMmJgZY0AvKVvgqJpx22+ozEtZCvMf8pf9QaNt5OYrn9V8OMgmLB/jW8=--0SH5dc3Y/OPabQ38--WAJ+eNPxbe74V1bbkQ2Jog==", "jti"=>"14d466c61fe3a179cfe85d0365769df2", "exp"=>1648894986}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJZRC9qZFZjSnh4bS9pTW9WSVF4c0szZVhHZFNiWVVnVGRRa1Nlc1lWUU81R01tSmdaWTBBdktWdmdxSnB4MjIrb3pFdFpDdk1mOHBmOVFhTnQ1T1lybjlWOE9NZ21MQi9qVzg9LS0wU0g1ZGMzWS9PUGFiUTM4LS1XQUorZU5QeGJlNzRWMWJia1EySm9nPT0iLCJqdGkiOiIxNGQ0NjZjNjFmZTNhMTc5Y2ZlODVkMDM2NTc2OWRmMiIsImV4cCI6MTY0ODg5NDk4Nn0.cDnGsgMsx6duF6dB4O5j1JmvRmzHyxymrHm6JpmYAEY">
```

- `irb(main):017:0> UserAuth::RefreshToken.new(token: token1.token).entity_for_user`を実行<br>

```:terminal
  User Load (2.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  User Load (4.0ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti                      |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-01 19:23:06 +0900 | 14d466c61fe3a179cfe85d0365769df2 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
1 row in set
```

- `irb(main):018:0> UserAuth::RefreshToken.new(user_id: user.id)`を実行<br>

```:terminal
  User Load (1.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
   (0.5ms)  BEGIN
  User Load (1.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
  User Update (2.3ms)  UPDATE "users" SET "refresh_jti" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["refresh_jti", "6a141bee4e2365f8f8e7938054038442"], ["updated_at", "2022-04-01 10:32:16.598484"], ["id", 1]]
   (10.9ms)  COMMIT
=> #<UserAuth::RefreshToken:0x00007f3d92dcecf0 @user_id="aJxwc6N5a3CwJViU5L1LyMTq6Vi8J2sz+EnahWeK0KtENRp0t/o14MCYcKLNxCVk9ABXPMEOyo++di+AJrp/StwuC1pdinGsbZA=--VyR5ypMbKrN43+vN--mtWrE2Y9TahKqXg4ZDUDUQ==", @payload={:sub=>"aJxwc6N5a3CwJViU5L1LyMTq6Vi8J2sz+EnahWeK0KtENRp0t/o14MCYcKLNxCVk9ABXPMEOyo++di+AJrp/StwuC1pdinGsbZA=--VyR5ypMbKrN43+vN--mtWrE2Y9TahKqXg4ZDUDUQ==", :jti=>"6a141bee4e2365f8f8e7938054038442", :exp=>1648895536}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhSnh3YzZONWEzQ3dKVmlVNUwxTHlNVHE2Vmk4SjJzeitFbmFoV2VLMEt0RU5ScDB0L28xNE1DWWNLTE54Q1ZrOUFCWFBNRU95bysrZGkrQUpycC9TdHd1QzFwZGluR3NiWkE9LS1WeVI1eXBNYktyTjQzK3ZOLS1tdFdyRTJZOVRhaEtxWGc0WkRVRFVRPT0iLCJqdGkiOiI2YTE0MWJlZTRlMjM2NWY4ZjhlNzkzODA1NDAzODQ0MiIsImV4cCI6MTY0ODg5NTUzNn0.tuPPAr8lbixgZx4EaLtSCFgU8_Arjfhe3ldNsQNxaBk">
```

- `irb(main):020:0> user.reload`を実行<br>

```:terminal
  User Load (2.0ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti                      |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-01 19:32:16 +0900 | 6a141bee4e2365f8f8e7938054038442 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
1 row in set
```

- `irb(main):021:0> user.refresh_jti == token1.payload[:jti]`を実行<br>

```:terminal
=> false
```

- `irb(main):022:0> UserAuth::RefreshToken.new(token: token1.token).entity_for_user`を実行<br>

```:terminal
  User Load (2.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
Traceback (most recent call last):
        3: from (irb):22
        2: from (irb):22:in `new'
        1: from app/services/user_auth/refresh_token.rb:13:in `initialize'
JWT::InvalidJtiError (Invalid jti)
```

- `root $ docker compose run --rm api rails g integration_test RefreshToken`を実行<br>

* `api/test/integration/refresh_token_test.rb`を編集<br>

```rb:refresh_token_test.rb
require 'test_helper'

class RefreshTokenTest < ActionDispatch::IntegrationTest
  def setup
    @user = active_user
    @encode = UserAuth::RefreshToken.new(user_id: @user.id)
    @lifetime = UserAuth.refresh_token_lifetime
  end

  # エンコード
  test 'encode_token' do
    # payload[:exp]の値は想定通りか(1秒許容)
    payload = @encode.payload
    expect_lifetime = @lifetime.from_now.to_i
    assert_in_delta expect_lifetime, payload[:exp], 1.second

    # payload[:jti]の値は想定通りか
    encode_user = @encode.entity_for_user
    expect_jti = encode_user.refresh_jti
    assert_equal expect_jti, payload[:jti]

    # payload[:sub]の値は想定通りか
    user_claim = @encode.send(:user_claim)
    assert_equal @encode.user_id, payload[user_claim]
  end

  # デコード
  test 'decode_token' do
    decode = UserAuth::RefreshToken.new(token: @encode.token)
    payload = decode.payload

    # デコードユーザーは一致しているか
    token_user = decode.entity_for_user
    assert_equal @user, token_user

    # verify_claimsは想定通りか
    verify_claims = decode.send(:verify_claims)
    assert verify_claims[:verify_expiration]
    assert_equal UserAuth.token_signature_algorithm, verify_claims[:algorithm]

    # 有効期限後トークンはエラーを吐いているか
    travel_to (@lifetime.from_now) do
      assert_raises JWT::ExpiredSignature do
        UserAuth::RefreshToken.new(token: @encode.token)
      end
    end

    # トークンが書き換えられた場合エラーを吐いているか
    invalid_token = @encode.token + 'a'
    assert_raises JWT::VerificationError do
      UserAuth::RefreshToken.new(token: invalid_token)
    end
  end

  # デコードオプション
  test 'verify_claims' do
    @user.reload
    assert_equal @user.refresh_jti, @encode.payload[:jti]

    # userのjtiが正常の場合
    assert UserAuth::RefreshToken.new(token: @encode.token)

    # userのjtiが不正な場合
    @user.remember('invalid')
    e =
      assert_raises JWT::InvalidJtiError do
        UserAuth::RefreshToken.new(token: @encode.token)
      end # エラーメッセージをtestする場合
    assert_equal 'Invalid jti', e.message

    # userにjtiが存在しない場合
    @user.forget
    @user.reload
    assert_nil @user.refresh_jti
    assert_raises JWT::InvalidJtiError do
      UserAuth::RefreshToken.new(token: @encode.token)
    end
  end
end
```

- `root $ dokcer compose run --rm api rails t`を実行<br>

```:termianal
[+] Running 1/0
 ⠿ Container udemy-rails-nuxt-db-1  Running                                                                                                                                                                        0.0s
/usr/local/bundle/gems/activerecord-6.0.4.6/lib/active_record/connection_adapters/postgresql_adapter.rb:46: warning: Using the last argument as keyword parameters is deprecated; maybe ** should be added to the call
/usr/local/bundle/gems/pg-1.3.3/lib/pg.rb:68: warning: The called method `connect' is defined here
Started with run options --seed 22484

users...                                                                                                                                                                             ] 0% Time: 00:00:00,  ETA: ??:??:??
users...
users = 10
users = 10
  8/8: [===========================================================================================================================================================================] 100% Time: 00:00:22, Time: 00:00:22

Finished in 22.97821s
8 tests, 61 assertions, 0 failures, 0 errors, 0 skips
```
