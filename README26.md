## 79 トークンを簡単に扱うためのモジュールを作成する

- `root $ touch api/app/services/token_generate_service.rb`を実行<br>

* `api/app/services/token_generate_service.rb`を編集<br>

```rb:token_generate_service.rb
module TokenGenerateService # include時の初期化処理実行場所(include先のオブジェクト)
  def self.included(base)
    # include時にクラスメソッドを追加する
    base.extend ClassMethods
  end

  ## クラスメソッド
  module ClassMethods # アクセストークンのインスタンス生成(オプション => sub: encrypt user id)
    def decode_access_token(token, options = {})
      UserAuth::AccessToken.new(token: token, options: options)
    end

    # アクセストークンのuserを返す
    def from_access_token(token, options = {})
      decode_access_token(token, options).entity_for_user
    end

    # リフレッシュトークンのインスタンス生成
    def decode_refresh_token(token)
      UserAuth::RefreshToken.new(token: token)
    end

    # リフレッシュトークンのuserを返す
    def from_refresh_token(token)
      decode_refresh_token(token).entity_for_user
    end
  end

  ## インスタンスメソッド

  # アクセストークンのインスタンス生成
  def encode_access_token(payload = {})
    UserAuth::AccessToken.new(user_id: id, payload: payload)
  end

  # アクセストークンを返す(期限変更 => lifetime: 10.minute)
  def to_access_token(payload = {})
    encode_access_token(payload).token
  end

  # リフレッシュトークンのインスタンス生成
  def encode_refresh_token
    UserAuth::RefreshToken.new(user_id: id)
  end

  # リフレッシュトークンを返す
  def to_refresh_token
    encode_refresh_token.token
  end
end
```

- `api/app/models/user.rb`を編集<br>

```rb:user.rb
require 'validator/email_validator'

class User < ApplicationRecord
  include TokenGenerateService # 追加 Token生成モジュール

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

  private

  # email小文字化
  def downcase_email
    self.email.downcase! if email
  end # emailからアクティブなユーザーを返す
end
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> user = User.first`を実行<br>

```
  User Load (10.1ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
```

- `irb(main):003:0> user.encode_access_token`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007faf4d18d658 @user_id="d64oQYsSkT8gfi3JcEI6NvVmVmFW5ZhxioDpIwdUn88eU1hAa/qjQC+QCDShQHW1oZGSB18gOoAVo2rBbJamqTrfUqxyDRDO3jU=--WePNeijCv06fdZj/--GDAqmLQkA+dtTVMhp8I9bQ==", @lifetime=30 minutes, @payload={:exp=>1648890808, :sub=>"d64oQYsSkT8gfi3JcEI6NvVmVmFW5ZhxioDpIwdUn88eU1hAa/qjQC+QCDShQHW1oZGSB18gOoAVo2rBbJamqTrfUqxyDRDO3jU=--WePNeijCv06fdZj/--GDAqmLQkA+dtTVMhp8I9bQ==", :iss=>"http://localhost:3000", :aud=>"http://localhost:3000"}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg4OTA4MDgsInN1YiI6ImQ2NG9RWXNTa1Q4Z2ZpM0pjRUk2TnZWbVZtRlc1Wmh4aW9EcEl3ZFVuODhlVTFoQWEvcWpRQytRQ0RTaFFIVzFvWkdTQjE4Z09vQVZvMnJCYkphbXFUcmZVcXh5RFJETzNqVT0tLVdlUE5laWpDdjA2ZmRaai8tLUdEQXFtTFFrQStkdFRWTWhwOEk5YlE9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.HM6IOnsu8c1O5RJ30NxyCeqzbuPUzrqQX1rJTynUJEI">
```

- `irb(main):005:0> user.to_access_token`を実行<br>

```
=> "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg4OTA4NjIsInN1YiI6ImVVUUE1VHJjRXRQNEIrdEFTOTFaQ3FaWUc4ZVpPaTNRK1Y0cjBJNlZkeFJzdXVzU0p1MCtnbTkyUW9MNXdXZmlnZ2Q4ZW9JelYxUDlwKzUzTU42STRwbmlTOHRJRFF4N3dDdz0tLU5KT053Nmx3RGMrUFRBRW4tLTczSU1WU3drcGgwWDgzbjdwaGZUc3c9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.wsnT6dVVJ92ej8lRKQgpiRcD-oQFqSjC8K-p9nlyaKk"
```

- `irb(main):008:0> user.encode_refresh_token`を実行<br>

```
  User Load (1.6ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
   (1.6ms)  BEGIN
  User Load (1.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
  User Update (36.8ms)  UPDATE "users" SET "refresh_jti" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["refresh_jti", "9cccbcc719edf5e26b31661b3af24585"], ["updated_at", "2022-04-02 08:45:53.016693"], ["id", 1]]
   (5.1ms)  COMMIT
=> #<UserAuth::RefreshToken:0x00007faf4b62c0d0 @user_id="4evynGwDKmYMHkAWezhODlOCojgWQTWe657N+fux0YwIahz3kuPf5QCh5Eb41nmpd87uISLRM8sGvRXFx1ig76MqL3ZtUH1uXhs=--+HqbjVq6K/OCubTR--AX469+RUrtO906U5e5j0lA==", @payload={:sub=>"4evynGwDKmYMHkAWezhODlOCojgWQTWe657N+fux0YwIahz3kuPf5QCh5Eb41nmpd87uISLRM8sGvRXFx1ig76MqL3ZtUH1uXhs=--+HqbjVq6K/OCubTR--AX469+RUrtO906U5e5j0lA==", :jti=>"9cccbcc719edf5e26b31661b3af24585", :exp=>1648975552}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI0ZXZ5bkd3REttWU1Ia0FXZXpoT0RsT0NvamdXUVRXZTY1N04rZnV4MFl3SWFoejNrdVBmNVFDaDVFYjQxbm1wZDg3dUlTTFJNOHNHdlJYRngxaWc3Nk1xTDNadFVIMXVYaHM9LS0rSHFialZxNksvT0N1YlRSLS1BWDQ2OStSVXJ0TzkwNlU1ZTVqMGxBPT0iLCJqdGkiOiI5Y2NjYmNjNzE5ZWRmNWUyNmIzMTY2MWIzYWYyNDU4NSIsImV4cCI6MTY0ODk3NTU1Mn0.Ni_9sP6pvd9rgQ-e9itdXbwPCNcOmQrvCOlFRFboL7Q">
```

- `irb(main):011:0> user.to_refresh_token`を実行<br>

```
  User Load (12.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
   (0.5ms)  BEGIN
  User Load (0.7ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
  User Update (1.4ms)  UPDATE "users" SET "refresh_jti" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["refresh_jti", "23c066bd5da6f1bbd2c02464c13229bd"], ["updated_at", "2022-04-02 08:46:58.949135"], ["id", 1]]
   (22.5ms)  COMMIT
=> "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJJM3R5TXhGS2crZVhjU2U3TnhCbUNhYWF6b2FEdllYUEZtdThzbWxaY3REOHZtZDEyY1dMTDdRUURJdFNuNzh4UEdIVmorUi9QKzdoczBRNlhWRkRkNVdlbTA4eXNIeVpKNnM9LS1SbDhJaklRN1RtZFJlYTBQLS1PS3BMWUlPT3ZIaUdPcFR6Z2t5N25nPT0iLCJqdGkiOiIyM2MwNjZiZDVkYTZmMWJiZDJjMDI0NjRjMTMyMjliZCIsImV4cCI6MTY0ODk3NTYxOH0.QMU4Az_QFRiWj3DcJZ-JUuQoV-7v_kgr2K7dNSxxHVA"
```

- `irb(main):014:0> encode = user.encode_refresh_token`を実行<br>

```
  User Load (1.7ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
   (2.3ms)  BEGIN
  User Load (2.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 1], ["email", "user0@example.com"], ["activated", true], ["LIMIT", 1]]
  User Update (3.3ms)  UPDATE "users" SET "refresh_jti" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["refresh_jti", "2b1fa6be5cf69bd80a67577d2e61e8c9"], ["updated_at", "2022-04-02 08:48:02.158010"], ["id", 1]]
   (18.8ms)  COMMIT
```

- `irb(main):017:0> encode`を実行<br>

```
=> #<UserAuth::RefreshToken:0x00007faf4b46a0f8 @user_id="1+kB9nLAHHWb7g+ziHxyKJJc1o3vkQ0+hGYZLJayq16NH1YuQ5b88QHxoiYp+5RJ34ULVy6sAwFDrTmhYdgvcgas66IeItFQgCs=--6oPsyZxqe56vQZQy--MnVZXtjXtFrpgjSMmwcF+A==", @payload={:sub=>"1+kB9nLAHHWb7g+ziHxyKJJc1o3vkQ0+hGYZLJayq16NH1YuQ5b88QHxoiYp+5RJ34ULVy6sAwFDrTmhYdgvcgas66IeItFQgCs=--6oPsyZxqe56vQZQy--MnVZXtjXtFrpgjSMmwcF+A==", :jti=>"2b1fa6be5cf69bd80a67577d2e61e8c9", :exp=>1648975682}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxK2tCOW5MQUhIV2I3Zyt6aUh4eUtKSmMxbzN2a1EwK2hHWVpMSmF5cTE2TkgxWXVRNWI4OFFIeG9pWXArNVJKMzRVTFZ5NnNBd0ZEclRtaFlkZ3ZjZ2FzNjZJZUl0RlFnQ3M9LS02b1BzeVp4cWU1NnZRWlF5LS1NblZaWHRqWHRGcnBnalNNbXdjRitBPT0iLCJqdGkiOiIyYjFmYTZiZTVjZjY5YmQ4MGE2NzU3N2QyZTYxZThjOSIsImV4cCI6MTY0ODk3NTY4Mn0.J7HPK3MOY2_0kVCBHeTdmqw11kPe1BXtBlGRNOHHjbQ">
```

- `irb(main):020:0> User.from_refresh_token(encode.token)`を実行<br>

```
  User Load (5.5ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  User Load (4.5ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti                      |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-02 17:48:02 +0900 | 2b1fa6be5cf69bd80a67577d2e61e8c9 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
1 row in set
```

- `irb(main):023:0> user.reload`を実行<br>

```
  User Load (1.6ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti                      |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-02 17:48:02 +0900 | 2b1fa6be5cf69bd80a67577d2e61e8c9 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
1 row in set
```

- `irb(main):024:0> user`を実行<br>

```
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti                      |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-02 17:48:02 +0900 | 2b1fa6be5cf69bd80a67577d2e61e8c9 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
1 row in set
```

- `irb(main):026:0> access_token = user.to_access_token`を実行<br>

* `irb(main):027:0> access_token`を実行<br>

```
=> "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg4OTE0MDMsInN1YiI6IngzVTg1RURhZWdpL2t2VlJFMUZGSzg1S2xXTm1ITTRLT2h4VUFWblhIWGZUYklDVjVHRTFoUXJjeUtjekNJZnNZeVpmUm12cUJtMEZaVEg2bDVNMXJHZVBtaTVKSTZRaHRkdz0tLTY1c1FLUWczNGZCT3VIWGEtLURtSndMR1lNTm9wZXVvNnE5WTdDNlE9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.7t35r59Jkeja1Hs_aD00cxhy8Y7EvhLnTAj1jA8lPsA"
```

- `irb(main):029:0> User.from_access_token(access_token)`を実行<br>

```
  User Load (1.8ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                | refresh_jti                      |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-04-02 17:48:02 +0900 | 2b1fa6be5cf69bd80a67577d2e61e8c9 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+----------------------------------+
1 row in set
```

- `irb(main):030:0> User.decode_access_token(access_token)`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007faf4b5fef90 @user_id="x3U85EDaegi/kvVRE1FFK85KlWNmHM4KOhxUAVnXHXfTbICV5GE1hQrcyKczCIfsYyZfRmvqBm0FZTH6l5M1rGePmi5JI6Qhtdw=--65sQKQg34fBOuHXa--DmJwLGYMNopeuo6q9Y7C6Q==", @payload={"exp"=>1648891403, "sub"=>"x3U85EDaegi/kvVRE1FFK85KlWNmHM4KOhxUAVnXHXfTbICV5GE1hQrcyKczCIfsYyZfRmvqBm0FZTH6l5M1rGePmi5JI6Qhtdw=--65sQKQg34fBOuHXa--DmJwLGYMNopeuo6q9Y7C6Q==", "iss"=>"http://localhost:3000", "aud"=>"http://localhost:3000"}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg4OTE0MDMsInN1YiI6IngzVTg1RURhZWdpL2t2VlJFMUZGSzg1S2xXTm1ITTRLT2h4VUFWblhIWGZUYklDVjVHRTFoUXJjeUtjekNJZnNZeVpmUm12cUJtMEZaVEg2bDVNMXJHZVBtaTVKSTZRaHRkdz0tLTY1c1FLUWczNGZCT3VIWGEtLURtSndMR1lNTm9wZXVvNnE5WTdDNlE9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.7t35r59Jkeja1Hs_aD00cxhy8Y7EvhLnTAj1jA8lPsA", @options={}>
```

## 80 アクセストークンから current_user を取得する

- `root $ touch api/app/services/user_authenticate_service.rb`を実行<br>

`api/app/services/user_authenticate_service.rb`を編集<br>

```rb:user_authenticate_service.rb
module UserAuthenticateService # 認証済みのユーザーが居ればtrue、存在しない場合は401を返す # 認証（トークンの持ち主を判定）
  def authenticate_user
    current_user.present? || unauthorized_user
  end

  # 2021.12.20 追加
  # 保護リソースには認証・認可を行うこちらのメソッドを使用してください。
  # 認証 & 認可（トークンの持ち主 && メール認証ユーザーを判定）
  # 認証済み && メール認証済みのユーザーが居ればtrue、存在しない場合は401を返す
  def authenticate_active_user
    (current_user.present? && current_user.activated?) || unauthorized_user
  end

  private

  # リクエストヘッダートークンを取得する
  def token_from_request_headers
    request.headers['Authorization']&.split&.last
  end

  # access_tokenから有効なユーザーを取得する
  def fetch_user_from_access_token
    User.from_access_token(token_from_request_headers)
  rescue UserAuth.not_found_exception_class, JWT::DecodeError, JWT::EncodeError
    nil
  end

  # tokenのユーザーを返す
  def current_user
    return nil unless token_from_request_headers
    @_current_user ||= fetch_user_from_access_token
  end

  # 認証エラー
  def unauthorized_user
    cookies.delete(UserAuth.session_key)
    head(:unauthorized)
  end
end
```

- `api/app/controllers/application_controller.rb`を編集<br>

```rb:application_controller.rb
class ApplicationController < ActionController::API
  include ActionController.Cookies # Cookieを扱う
  include UserAuthenticateService # 認可を行う
end
```

- `api/app/controllers/api/v1/users_controller.rb`を編集<br>

```rb:users_controller.rb
class Api::V1::UsersController < ApplicationController
  before_action :authenticate_active_user # 追加

  def index
    users = User.all # as_json => ハッシュの形でSONデータを返す { "id" => 1, "name" => "test", ... }
    render json: users.as_json(only: %i[id name email created_at])
  end
end
```

- `root $ docker compose down`を実行<br>

* `root $ docker compose up api`を実行<br>

- `root $ curl http://localhost:3000/api/v1/users`を実行<br>

```
Completed 401 Unauthorized in 7ms (ActiveRecord: 0.0ms | Allocations: 152) // 401エラーが返される
```

- `api/app/controllers/api/v1/users_controller.rb`を編集<br>

```rb:users_controller.rb
class Api::V1::UsersController < ApplicationController # before_action :authenticate_active_user # コメントアウトする 確認後戻す
  def index
    users = User.all # as_json => ハッシュの形でSONデータを返す { "id" => 1, "name" => "test", ... }
    render json: users.as_json(only: %i[id name email created_at])
  end
end
```

- `root $ curl http://localhost:3000/api/v1/users`を実行<br>

```
[{"id":2,"name":"user1","email":"user1@example.com","created_at":"2022-03-16T11:15:58.486+09:00"},{"id":3,"name":"user2","email":"user2@example.com","created_at":"2022-03-16T11:15:58.860+09:00"},{"id":4,"name":"user3","email":"user3@example.com","created_at":"2022-03-16T11:15:59.199+09:00"},{"id":5,"name":"user4","email":"user4@example.com","created_at":"2022-03-16T11:15:59.541+09:00"},{"id":6,"name":"user5","email":"user5@example.com","created_at":"2022-03-16T11:15:59.880+09:00"},{"id":7,"name":"user6","email":"user6@example.com","created_at":"2022-03-16T11:16:00.219+09:00"},{"id":8,"name":"user7","email":"user7@example.com","created_at":"2022-03-16T11:16:00.554+09:00"},{"id":9,"name":"user8","email":"user8@example.com","created_at":"2022-03-16T11:16:00.898+09:00"},{"id":10,"name":"user9","email":"user9@example.com","created_at":"2022-03-16T11:16:01.241+09:00"},{"id":1,"name":"user0","email":"user0@example.com","created_at":"2022-03-16T11:15:58.112+09:00"}]%
```

- `api/app/controllers/api/v1/users_controller.rb`を編集<br>

```rb:users_controller.rb
class Api::V1::UsersController < ApplicationController
  before_action :authenticate_active_user

  def index
    render json: current_user.as_json(only: %i[id name email created_at]) # 編集
  end # as_json => ハッシュの形でSONデータを返す { "id" => 1, "name" => "test", ... }
end
```

- `root $ docker compose run --rm api rails r 'puts User.last.to_access_token' | pbcopy`を実行<br>

- `root $ curl http://localhost:3000/api/v1/users \ -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg4OTU2MTQsInN1YiI6IktOZG5FQTVQYy9INEdwZU9ObkNlc0diaitKRVB3RDRVaFRxaVlEMDhqbThiSE9xOUx1SFNlcC85ZURod255TGlBVkc5SExIQlRuU1ptYXdGaUd0YStoZlcwdmdDWjMwOE42az0tLUU3YytVeTNtWUpWaVpVTnEtLURzNTIyUXlydDRGTEFDd3c1K09FK1E9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.5qCfm2e0QNPfCh_z4gcADhTueFfGXjzirrNUwY3M2M0"`を実行<br>

```
{"id":10,"name":"user9","email":"user9@example.com","created_at":"2022-03-16T11:16:01.241+09:00"}%
```

## 81 リフレッシュトークンからsession_userを取得する

+ `root $ touch api/app/services/user_sessionize_service.rb`を実行<br>

+ `api/app/services/user_sessionize_service.rb`を編集<br>

```rb:user_sessionize_service.rb
module UserSessionizeService

  # セッションユーザーが居ればtrue、存在しない場合は401を返す
  def sessionize_user
    session_user.present? || unauthorized_user
  end

  # セッションキー
  def session_key
    UserAuth.session_key
  end

  # セッションcookieを削除する
  def delete_session
    cookies.delete(session_key)
  end

  private

    # cookieのtokenを取得
    def token_from_cookies
      cookies[session_key]
    end

    # refresh_tokenから有効なユーザーを取得する
    def fetch_user_from_refresh_token
      User.from_refresh_token(token_from_cookies)
    rescue JWT::InvalidJtiError
      # jtiエラーの場合はcontrollerに処理を委任
      catch_invalid_jti
    rescue UserAuth.not_found_exception_class,
          JWT::DecodeError, JWT::EncodeError
      nil
    end

    # refresh_tokenのユーザーを返す
    def session_user
      return nil unless token_from_cookies
      @_session_user ||= fetch_user_from_refresh_token
    end

    # jtiエラーの処理
    def catch_invalid_jti
      delete_session
      raise JWT::InvalidJtiError
    end

    # 認証エラー
    def unauthorized_user
      delete_session
      head(:unauthorized)
    end
end
```
