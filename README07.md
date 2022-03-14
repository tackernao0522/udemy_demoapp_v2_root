## 40 email カラムのカスタムバリデーション設定

1. 入力必須 (validates Method)<br>
2. 文字数制限（EachVlidator Class）<br>
3. 書式チェック（EachVlidator Class）<br>
4. 一意性制約（EachValidator Class）<br>

- `root $ mkdir api/lib/validator && touch $_/email_validator.rb`を実行<br>

* `api/lib/validator/email_validator.rb`を編集<br>

```rb:email_validator.rb
class EmailValidator < ActiveModel::EachValidator

end
```

- `api/app/models/user.rb`を編集<br>

```rb:user.rb
require "validator/email_validator" # 追加

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
  # 追加
  validates :email, presence: true,
                    email: { allow_blank: true }
  # ここまで

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
                        message: :invalid_password,
                        allow_blank: true
                      },
                      allow_nil: true # 空パスワードのアップデートを許容する。(Null(nil)の場合スキップ)
end
```

- `api/lib/validator/email_validator.rb`を編集<br>

```rb:email_validator.rb
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    byebug
  end
end
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> User.create(email: "a@a.a")`を実行<br>

```:terminal
irb(main):001:0> User.create(email: "a@a.a")
Return value is: nil

[1, 5] in /app/lib/validator/email_validator.rb
   1: class EmailValidator < ActiveModel::EachValidator
   2:   def validate_each(record, attribute, value)
   3:     byebug
=> 4:   end
   5: end
(byebug)
```

- `(byebug) record`を実行<br>

```:terminal
(byebug) record
#<User id: nil, name: nil, email: "a@a.a", password_digest: nil, activated: false, admin: false, created_at: nil, updated_at: nil>
(byebug)
```

- `(byebug) pp record`を実行<br>

```:terminal
(byebug) pp record
#<User:0x00007f4f9a5985a8
 id: nil,
 name: nil,
 email: "a@a.a",
 password_digest: nil,
 activated: false,
 admin: false,
 created_at: nil,
 updated_at: nil>
#<User id: nil, name: nil, email: "a@a.a", password_digest: nil, activated: false, admin: false, created_at: nil, updated_at: nil>
(byebug)
```

- `(byebug) attribute`を実行<br>

```:terminal
(byebug) attribute
:email
(byebug)
```

- `(byebug) value`を実行<br>

```:terminal
(byebug) value
"a@a.a"
(byebug)
```

- `Ctrl + D`でコンソールを抜ける<br>

```:terminal
+----+------+-------+-----------------+-----------+-------+------------+------------+
| id | name | email | password_digest | activated | admin | created_at | updated_at |
+----+------+-------+-----------------+-----------+-------+------------+------------+
|    |      | a@a.a |                 | false     | false |            |            |
+----+------+-------+-----------------+-----------+-------+------------+------------+
1 row in set
irb(main):002:0>
```

- `irb(main):002:0> exit`で抜ける<br>

* `api/lib/validator/email_validator.rb`を編集<br>

```rb:email_validator.rb
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
   # text length
    max = 255
    record.errors.add(attribute, :too_long, count: max) if value.length > max

    # format
    format = /\A\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*\z/
    record.errors.add(attribute, :invalid) unless format =~ value # =~ ・・・ 文字列が正規表現と一致しているかを判定する演算子 一致していない => nil, 一致している => 0 (正確にはマッチした位置のインデックス)

    # uniqueness
    record.errors.add(attribute, :taken) if record.email_activated?
  end
end
```

- `api/app/models/user.rb`を編集<br>

```rb:user.rb
require "validator/email_validator"

class User < ApplicationRecord
  # Userクラスの一番上に追加
  # バリデーション直前
  before_validation :downcase_email # 追加

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
  validates :email, presence: true,
                    email: { allow_blank: true }

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
                        message: :invalid_password,
                        allow_blank: true
                      },
                      allow_nil: true # 空パスワードのアップデートを許容する。(Null(nil)の場合スキップ)
  # 追加
  ## methods
  # class method  ###########################
  class << self
    # emailからアクティブなユーザーを返す
    def find_by_activated(email)
      find_by(email: email, activated: true)
    end
  end
  # class method end #########################

  # 自分以外の同じemailのアクティブなユーザーがいる場合にtrueを返す
  def email_activated?
    users = User.where.not(id: id)
    users.find_by_activated(email).present?
  end

  private

    # email小文字化
    def downcase_email
      self.email.downcase! if email
    end
  # ここまで
end
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> domain = "@example.com"`を実行<br>

- `irb(main):002:0> email = "a" * (256 - domain.length) + domain`を実行<br>

* `irb(main):003:0> email`を実行<br>

```terminal:
irb(main):003:0> email
=> "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@example.com"
irb(main):004:0>
```

- `irb(main):004:0> user = User.new(name: "test", email: email, password: "password")`を実行<br>

* `irb(main):005:0> user`を実行<br>

```:terminal
irb(main):005:0> user
+----+------+-------------------------------------------------------------------------------------+--------------------------------------------------------------+-----------+-------+------------+------------+
| id | name | email                                                                               | password_digest                                              | activated | admin | created_at | updated_at |
+----+------+-------------------------------------------------------------------------------------+--------------------------------------------------------------+-----------+-------+------------+------------+
|    | test | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa... | $2a$12$UmxEFKOUVX/MWcCwOj3KbebRC1l0Pqp29Puoh8lg4YU/gzA2XMYTi | false     | false |            |            |
+----+------+-------------------------------------------------------------------------------------+--------------------------------------------------------------+-----------+-------+------------+------------+
1 row in set
irb(main):006:0>
```

- `irb(main):006:0> user.save`を実行<br>

```:terminal
   (0.5ms)  BEGIN
  User Load (19.0ms)  SELECT "users".* FROM "users" WHERE "users"."id" IS NOT NULL AND "users"."email" = $1 AND "users"."activated" = $2 LIMIT $3  [["email", "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@example.com"], ["activated", true], ["LIMIT", 1]]
   (1.2ms)  ROLLBACK
=> false
irb(main):007:0>
```

- `irb(main):007:0> user.errors.full_messages`を実行<br>

```:terminal
irb(main):007:0> user.errors.full_messages
=> ["メールアドレスは255文字以内で入力してください"]
irb(main):008:0>
```

- `irb(main):008:0> email = "a@a@a.com"`を実行<br>

* `irb(main):009:0> user.email = email`を実行<br>

- `irb(main):010:0> user.email`を実行<br>

```terminal:
irb(main):010:0> user.email
=> "a@a@a.com"
irb(main):011:0>
```

- `irb(main):011:0> user.save`を実行<br>

```:terminal
irb(main):011:0> user.save
   (0.5ms)  BEGIN
  User Load (1.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" IS NOT NULL AND "users"."email" = $1 AND "users"."activated" = $2 LIMIT $3  [["email", "a@a@a.com"], ["activated", true], ["LIMIT", 1]]
   (1.0ms)  ROLLBACK
=> false
irb(main):012:0>
```

- `irb(main):012:0> user.errors.full_messages`を実行<br>

```:terminal
irb(main):012:0> user.errors.full_messages
=> ["メールアドレスは不正な値です"]
irb(main):013:0>
```

- `irb(main):013:0> email = "a@a.a"`を実行<br>

* `irb(main):014:0> User.create(name: "test", email: email, password: "password")`を実行<br>

```:terminal
irb(main):014:0> User.create(name: "test", email: email, password: "password")
   (0.6ms)  BEGIN
  User Load (2.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" IS NOT NULL AND "users"."email" = $1 AND "users"."activated" = $2 LIMIT $3  [["email", "a@a.a"], ["activated", true], ["LIMIT", 1]]
  User Create (65.5ms)  INSERT INTO "users" ("name", "email", "password_digest", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["name", "test"], ["email", "a@a.a"], ["password_digest", "$2a$12$TjVwJLBxcBNyKQjAsvBrW.0j2j4T7DpXjkhbEK/neW/C3y8WoH2Ue"], ["created_at", "2022-03-14 10:46:16.865872"], ["updated_at", "2022-03-14 10:46:16.865872"]]
   (15.4ms)  COMMIT
+----+------+-------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
| id | name | email | password_digest                                              | activated | admin | created_at                | updated_at                |
+----+------+-------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
| 1  | test | a@a.a | $2a$12$TjVwJLBxcBNyKQjAsvBrW.0j2j4T7DpXjkhbEK/neW/C3y8WoH2Ue | false     | false | 2022-03-14 19:46:16 +0900 | 2022-03-14 19:46:16 +0900 |
+----+------+-------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
1 row in set
irb(main):015:0>
```

- `User.last.update(activated: true)`を実行<br>

```terminal:
irb(main):017:0> User.last.update(activated: true)
  User Load (7.5ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" DESC LIMIT $1  [["LIMIT", 1]]
   (0.6ms)  BEGIN
  User Load (3.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" != $1 AND "users"."email" = $2 AND "users"."activated" = $3 LIMIT $4  [["id", 3], ["email", "a@a.a"], ["activated", true], ["LIMIT", 1]]
  User Update (3.6ms)  UPDATE "users" SET "activated" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["activated", true], ["updated_at", "2022-03-14 10:50:13.569530"], ["id", 3]]
   (4.2ms)  COMMIT
=> true
irb(main):018:0>
```

- `irb(main):018:0> user = User.new(name: "test", email: email, password: "password")`を実行<br>

```:terminal
irb(main):018:0> user = User.new(name: "test", email: email, password: "password")
Traceback (most recent call last):
ArgumentError (wrong number of arguments (given 1, expected 0))
irb(main):019:0>
```

- `irb(main):019:0> user`を実行<br>

```:terminal
irb(main):019:0> user
+----+------+-------+--------------------------------------------------------------+-----------+-------+------------+------------+
| id | name | email | password_digest                                              | activated | admin | created_at | updated_at |
+----+------+-------+--------------------------------------------------------------+-----------+-------+------------+------------+
|    | test | a@a.a | $2a$12$ve8RuWatdgD9kOgap8ZmBeu9KQrGROErcT.5.3mSS1bX3gGW3thCW | false     | false |            |            |
+----+------+-------+--------------------------------------------------------------+-----------+-------+------------+------------+
1 row in set
irb(main):020:0>
```

- `irb(main):020:0> user.save`を実行<br>

```:terminal
irb(main):020:0> user.save
   (0.6ms)  BEGIN
  User Load (0.7ms)  SELECT "users".* FROM "users" WHERE "users"."id" IS NOT NULL AND "users"."email" = $1 AND "users"."activated" = $2 LIMIT $3  [["email", "a@a.a"], ["activated", true], ["LIMIT", 1]]
   (1.3ms)  ROLLBACK
=> false
irb(main):021:0>
```

- `irb(main):021:0> user.errors.full_messages`を実行<br>

```:terminal
irb(main):021:0> user.errors.full_messages
=> ["メールアドレスはすでに存在します"]
irb(main):022:0>
```

- `irb(main):022:0> User.all`を実行して確認(activated が true になっているメールアドレスは登録できない)<br>

```:terminal
irb(main):022:0> User.all
  User Load (3.0ms)  SELECT "users".* FROM "users"
+----+------+-------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
| id | name | email | password_digest                                              | activated | admin | created_at                | updated_at                |
+----+------+-------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
| 1  | test | a@a.a | $2a$12$TjVwJLBxcBNyKQjAsvBrW.0j2j4T7DpXjkhbEK/neW/C3y8WoH2Ue | false     | false | 2022-03-14 19:46:16 +0900 | 2022-03-14 19:46:16 +0900 |
| 2  | test | a@a.a | $2a$12$x/44paDIXZjYWSS7CuDPmu7H71kYPVlcisUWv/9Acia0BTMw55mpu | false     | false | 2022-03-14 19:49:09 +0900 | 2022-03-14 19:49:09 +0900 |
| 3  | test | a@a.a | $2a$12$SiY421T3r21Y8j67gX3ZHumzWm75rMC8pFx07G9cYy0EgpdKpbHWu | true      | false | 2022-03-14 19:49:12 +0900 | 2022-03-14 19:50:13 +0900 |
+----+------+-------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
3 rows in set
irb(main):023:0>
```

- `irb(main):023:0> email = "A@A.A"`を実行<br>

* `irb(main):024:0> user = User.new(name: "test", email: email, password: "password")`を実行<br>

- `irb(main):025:0> user.save`を実行<br>

```:terminal
irb(main):025:0> user.save
   (0.8ms)  BEGIN
  User Load (1.6ms)  SELECT "users".* FROM "users" WHERE "users"."id" IS NOT NULL AND "users"."email" = $1 AND "users"."activated" = $2 LIMIT $3  [["email", "a@a.a"], ["activated", true], ["LIMIT", 1]]
   (1.7ms)  ROLLBACK
=> false
irb(main):026:0>
```

- `irb(main):026:0> user.email`を実行(小文字になっていれば OK)<br>

```:terminal
irb(main):026:0> user.email
=> "a@a.a"
irb(main):027:0>
```
