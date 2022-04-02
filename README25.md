## 78 アクセストークンクラスを作成する

- `root $ touch api/app/services/user_auth/access_token.rb`を実行<br>

* `api/app/services/user_auth/access_token.rb`を編集<br>

```rb:access_token.rb
require 'jwt'

module UserAuth
  class AccessToken
    include TokenCommons

    attr_reader :user_id, :payload, :lifetime, :token, :options

    def initialize(user_id: nil, payload: {}, token: nil, options: {})
      if token.present?
        # decode
        @token =
          token
        @options = options
        @payload =
          JWT.decode(
            @token.to_s,
            decode_key,
            true,
            verify_claims.merge(@options)
          ).first
        @user_id = get_user_id_from(@payload)
      else
        # encode
        @user_id = encrypt_for(user_id)
        @lifetime = payload[:lifetime] || UserAuth.access_token_lifetime
        @payload = claims.merge(payload.except(:lifetime))
        @token = JWT.encode(@payload, secret_key, algorithm, header_fields)
      end
    end

    # 暗号化された@user_idからユーザーを取得する
    def entity_for_user
      User.find(decrypt_for(@user_id))
    end

    # @lifetimeの日本語テキストを返す
    def lifetime_text
      time, period = @lifetime.inspect.sub(/s\z/, '').split
      time + I18n.t("datetime.periods.#{period}", default: '')
    end

    private

    ## エンコードメソッド

    # issuerの値がある場合にtrueを返す
    def verify_issuer?
      UserAuth.token_issuer.present?
    end

    # issuerを返す
    def token_issuer
      verify_issuer? && UserAuth.token_issuer
    end

    # audienceの値がある場合にtrueを返す
    def verify_audience?
      UserAuth.token_audience.present?
    end

    # audienceを返す
    def token_audience
      verify_audience? && UserAuth.token_audience
    end

    # user_idの値がある場合にtrueを返す
    def verify_user_id?
      @user_id.present?
    end

    # 有効期限をUnixtimeで返す(必須)
    def token_expiration
      @lifetime.from_now.to_i
    end

    # エンコード時のデフォルトクレーム
    def claims
      _claims = {}
      _claims[:exp] = token_expiration
      _claims[user_claim] = @user_id if verify_user_id?
      _claims[:iss] = token_issuer if verify_issuer?
      _claims[:aud] = token_audience if verify_audience?
      _claims
    end

    ## デコードメソッド

    # @optionsにsubjectがある場合にtrueを返す
    def verify_subject?
      @options.has_key?(:sub)
    end

    # @optionsのsubの値を返す
    def token_subject
      verify_subject? && @options[:sub]
    end

    # デコード時のデフォルトオプション
    # Doc: https://github.com/jwt/ruby-jwt
    # default: https://www.rubydoc.info/github/jwt/ruby-jwt/master/JWT/DefaultOptions
    def verify_claims
      {
        iss: token_issuer,
        aud: token_audience,
        sub: token_subject,
        verify_expiration: true,
        verify_iss:
          # 有効期限の検証するか(必須)
          verify_issuer?,
        verify_aud:
          # payloadとissの一致を検証するか
          verify_audience?,
        verify_sub:
          # payloadとaudの一致を検証するか
          verify_subject?,
        # payloadとsubの一致を検証するか
        algorithm: algorithm # decode時のアルゴリズム
      }
    end
  end
end
```

- `api/config/locales/ja.yml`を編集<br>

```yml:ja.yml
ja:
  activerecord:
    attributes:
      user:
        name: 名前
        email: メールアドレス
        password: パスワード
        activated: アクティブフラグ
        admin: 管理者フラグ
    errors:
      messages:
        record_invalid: 'バリデーションに失敗しました: %{errors}'
        restrict_dependent_destroy:
          has_one: '%{record}が存在しているので削除できません'
          has_many: '%{record}が存在しているので削除できません'
  date:
    abbr_day_names:
      - 日
      - 月
      - 火
      - 水
      - 木
      - 金
      - 土
    abbr_month_names:
      -
      - 1月
      - 2月
      - 3月
      - 4月
      - 5月
      - 6月
      - 7月
      - 8月
      - 9月
      - 10月
      - 11月
      - 12月
    day_names:
      - 日曜日
      - 月曜日
      - 火曜日
      - 水曜日
      - 木曜日
      - 金曜日
      - 土曜日
    formats:
      default: '%Y/%m/%d'
      long: '%Y年%m月%d日(%a)'
      short: '%m/%d'
    month_names:
      -
      - 1月
      - 2月
      - 3月
      - 4月
      - 5月
      - 6月
      - 7月
      - 8月
      - 9月
      - 10月
      - 11月
      - 12月
    order:
      - :year
      - :month
      - :day
  datetime:
    distance_in_words:
      about_x_hours:
        one: 約1時間
        other: 約%{count}時間
      about_x_months:
        one: 約1ヶ月
        other: 約%{count}ヶ月
      about_x_years:
        one: 約1年
        other: 約%{count}年
      almost_x_years:
        one: 1年弱
        other: '%{count}年弱'
      half_a_minute: 30秒前後
      less_than_x_seconds:
        one: 1秒以内
        other: '%{count}秒未満'
      less_than_x_minutes:
        one: 1分以内
        other: '%{count}分未満'
      over_x_years:
        one: 1年以上
        other: '%{count}年以上'
      x_seconds:
        one: 1秒
        other: '%{count}秒'
      x_minutes:
        one: 1分
        other: '%{count}分'
      x_days:
        one: 1日
        other: '%{count}日'
      x_months:
        one: 1ヶ月
        other: '%{count}ヶ月'
      x_years:
        one: 1年
        other: '%{count}年'
    prompts:
      second: 秒
      minute: 分
      hour: 時
      day: 日
      month: 月
      year: 年
      # 追加
    periods:
      second: 秒
      minute: 分
      hour: 時間
      day: 日
      week: 週間
      month: ヶ月
      year: 年
      # ここまで
  errors:
    format: '%{attribute}%{message}'
    messages:
      invalid_password: は半角英数字・ハイフン・アンダーバーが使えます
      accepted: を受諾してください
      blank: を入力してください
      confirmation: と%{attribute}の入力が一致しません
      empty: を入力してください
      equal_to: は%{count}にしてください
      even: は偶数にしてください
      exclusion: は予約されています
      greater_than: は%{count}より大きい値にしてください
      greater_than_or_equal_to: は%{count}以上の値にしてください
      inclusion: は一覧にありません
      invalid: は不正な値です
      less_than: は%{count}より小さい値にしてください
      less_than_or_equal_to: は%{count}以下の値にしてください
      model_invalid: 'バリデーションに失敗しました: %{errors}'
      not_a_number: は数値で入力してください
      not_an_integer: は整数で入力してください
      odd: は奇数にしてください
      other_than: は%{count}以外の値にしてください
      present: は入力しないでください
      required: を入力してください
      taken: はすでに存在します
      too_long: は%{count}文字以内で入力してください
      too_short: は%{count}文字以上で入力してください
      wrong_length: は%{count}文字で入力してください
    template:
      body: 次の項目を確認してください
      header:
        one: '%{model}にエラーが発生しました'
        other: '%{model}に%{count}個のエラーが発生しました'
  helpers:
    select:
      prompt: 選択してください
    submit:
      create: 登録する
      submit: 保存する
      update: 更新する
  number:
    currency:
      format:
        delimiter: ','
        format: '%n%u'
        precision: 0
        separator: '.'
        significant: false
        strip_insignificant_zeros: false
        unit: 円
    format:
      delimiter: ','
      precision: 3
      separator: '.'
      significant: false
      strip_insignificant_zeros: false
    human:
      decimal_units:
        format: '%n %u'
        units:
          billion: 十億
          million: 百万
          quadrillion: 千兆
          thousand: 千
          trillion: 兆
          unit: ''
      format:
        delimiter: ''
        precision: 3
        significant: true
        strip_insignificant_zeros: true
      storage_units:
        format: '%n%u'
        units:
          byte: バイト
          eb: EB
          gb: GB
          kb: KB
          mb: MB
          pb: PB
          tb: TB
    percentage:
      format:
        delimiter: ''
        format: '%n%'
    precision:
      format:
        delimiter: ''
  support:
    array:
      last_word_connector: '、'
      two_words_connector: '、'
      words_connector: '、'
  time:
    am: 午前
    formats:
      default: '%Y年%m月%d日(%a) %H時%M分%S秒 %z'
      long: '%Y/%m/%d %H:%M'
      short: '%m/%d %H:%M'
    pm: 午後
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):002:0> Hirb.disable`を実行<br>

```:terminal
=> false
```

- `irb(main):003:0> user = User.first`を実行<br>

```
  User Load (6.5ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> #<User id: 1, name: "user0", email: "user0@example.com", password_digest: [FILTERED], activated: true, admin: false, created_at: "2022-03-16 02:15:58", updated_at: "2022-04-01 10:32:16", refresh_jti: "6a141be...
```

- `irb(main):004:0> user`を実行<br>

```
=> #<User id: 1, name: "user0", email: "user0@example.com", password_digest: [FILTERED], activated: true, admin: false, created_at: "2022-03-16 02:15:58", updated_at: "2022-04-01 10:32:16", refresh_jti: "6a141bee4e2365f8f8e7938054038442">
```

- `irb(main):005:0> token = UserAuth::AccessToken.new(user_id: user.id)`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007f5134606e18 @user_id="S5b8HqTrvmbWR9fkzKuPGdkzuPZ2HwrAjVLvdb5kCXgXz7OEhgwwTxanSXRsvME69Ha7tf1WxMXX7Uq6scbv6qyRPu1pvZ7jTEk=--j28MMAjuMa8MBCpW--hPUgCClw9eIqDAED+5xQrg==", @lifetime=3...
```

- `irb(main):006:0> token`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007f5134606e18 @user_id="S5b8HqTrvmbWR9fkzKuPGdkzuPZ2HwrAjVLvdb5kCXgXz7OEhgwwTxanSXRsvME69Ha7tf1WxMXX7Uq6scbv6qyRPu1pvZ7jTEk=--j28MMAjuMa8MBCpW--hPUgCClw9eIqDAED+5xQrg==", @lifetime=30 minutes, @payload={:exp=>1648886667, :sub=>"S5b8HqTrvmbWR9fkzKuPGdkzuPZ2HwrAjVLvdb5kCXgXz7OEhgwwTxanSXRsvME69Ha7tf1WxMXX7Uq6scbv6qyRPu1pvZ7jTEk=--j28MMAjuMa8MBCpW--hPUgCClw9eIqDAED+5xQrg==", :iss=>"http://localhost:3000", :aud=>"http://localhost:3000"}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg4ODY2NjcsInN1YiI6IlM1YjhIcVRydm1iV1I5Zmt6S3VQR2RrenVQWjJId3JBalZMdmRiNWtDWGdYejdPRWhnd3dUeGFuU1hSc3ZNRTY5SGE3dGYxV3hNWFg3VXE2c2NidjZxeVJQdTFwdlo3alRFaz0tLWoyOE1NQWp1TWE4TUJDcFctLWhQVWdDQ2x3OWVJcURBRUQrNXhRcmc9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.HivbNjGUgvGFxPbvTAFbWLTGfJ0qVAAouHyLA-Pddp8">˝
```

- `irb(main):007:0> UserAuth::AccessToken.new(token: token.token)`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007f5134a70d28 @user_id="S5b8HqTrvmbWR9fkzKuPGdkzuPZ2HwrAjVLvdb5kCXgXz7OEhgwwTxanSXRsvME69Ha7tf1WxMXX7Uq6scbv6qyRPu1pvZ7jTEk=--j28MMAjuMa8MBCpW--hPUgCClw9eIqDAED+5xQrg==", @payload={"exp"=>1648886667, "sub"=>"S5b8HqTrvmbWR9fkzKuPGdkzuPZ2HwrAjVLvdb5kCXgXz7OEhgwwTxanSXRsvME69Ha7tf1WxMXX7Uq6scbv6qyRPu1pvZ7jTEk=--j28MMAjuMa8MBCpW--hPUgCClw9eIqDAED+5xQrg==", "iss"=>"http://localhost:3000", "aud"=>"http://localhost:3000"}, @token="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDg4ODY2NjcsInN1YiI6IlM1YjhIcVRydm1iV1I5Zmt6S3VQR2RrenVQWjJId3JBalZMdmRiNWtDWGdYejdPRWhnd3dUeGFuU1hSc3ZNRTY5SGE3dGYxV3hNWFg3VXE2c2NidjZxeVJQdTFwdlo3alRFaz0tLWoyOE1NQWp1TWE4TUJDcFctLWhQVWdDQ2x3OWVJcURBRUQrNXhRcmc9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.HivbNjGUgvGFxPbvTAFbWLTGfJ0qVAAouHyLA-Pddp8", @options={}>
```

- `irb(main):008:0> UserAuth::AccessToken.new(token: token.token).entity_for_user`を実行<br>

```
  User Load (6.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
=> #<User id: 1, name: "user0", email: "user0@example.com", password_digest: [FILTERED], activated: true, admin: false, created_at: "2022-03-16 02:15:58", updated_at: "2022-04-01 10:32:16", refresh_jti: "6a141bee4e2365f8f8e7938054038442">
```

- `irb(main):009:0> Time.at(token.payload[:exp])`を実行<br>

```
=> 2022-04-02 17:04:27 +0900
```

- `irb(main):010:0> token = UserAuth::AccessToken.new(user_id: user.id, payload: { lifetime: 1.hours })`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007f51345a7990 @user_id="ZkkkYB1Z2DoGIafiu2qB0ya/5y0SeJuXw8nsWws1btJrOEkMaD2o3JpRPX+GnWDC5V5W0MKlxm64EKmkYOm7G40le4r4VbCnuVU=--v81qVllKDSFJ5wKG--m+ir9nfMEXI46O/ZCVc3eg==", @lifetime=1...
```

- `irb(main):011:0> Time.at(token.payload[:exp])`を実行<br>

```
=> 2022-04-02 17:50:10 +0900
```

- `irb(main):012:0> token.lifetime_text`を実行<br>

```
=> "1時間"
```

- `irb(main):013:0> token = UserAuth::AccessToken.new(user_id: user.id, payload: { lifetime: 1.day })`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007f51342b7d20 @user_id="PWOxvWimWVcd26tZXUXL9GtK8DRlRgJcBuN3zJrq+aI4eAaCORMHe2XQw2/KNTDl8xhLGB3EBqz3as9AHqcREoCheo9SdxPQX6I=--4s35iIiihUSqb8Gj--Am+aiTMPCWZpTUQ3dUJ1XA==", @lifetime=1...
```

- `irb(main):014:0> token.lifetime_text`を実行<br>

```
=> "1日"
```

- `irb(main):016:0> token = UserAuth::AccessToken.new(token: token.token, options: {sub: "1"})`を実行<br>

```
Traceback (most recent call last):
        4: from (irb):15
        3: from (irb):16:in `rescue in irb_binding'
        2: from (irb):16:in `new'
        1: from app/services/user_auth/access_token.rb:14:in `initialize'
JWT::InvalidSubError (Invalid subject. Expected 1, received PWOxvWimWVcd26tZXUXL9GtK8DRlRgJcBuN3zJrq+aI4eAaCORMHe2XQw2/KNTDl8xhLGB3EBqz3as9AHqcREoCheo9SdxPQX6I=--4s35iIiihUSqb8Gj--Am+aiTMPCWZpTUQ3dUJ1XA==)
```

- `irb(main):017:0> token.payload[:sub]`を実行<br>

```
=> "PWOxvWimWVcd26tZXUXL9GtK8DRlRgJcBuN3zJrq+aI4eAaCORMHe2XQw2/KNTDl8xhLGB3EBqz3as9AHqcREoCheo9SdxPQX6I=--4s35iIiihUSqb8Gj--Am+aiTMPCWZpTUQ3dUJ1XA=="
```

- `irb(main):018:0> sub = token.payload[:sub]`を実行<br>

```
=> "PWOxvWimWVcd26tZXUXL9GtK8DRlRgJcBuN3zJrq+aI4eAaCORMHe2XQw2/KNTDl8xhLGB3EBqz3as9AHqcREoCheo9SdxPQX6I=--4s35iIiihUSqb8Gj--Am+aiTMPCWZpTUQ3dUJ1XA=="
```

- `irb(main):019:0> token = UserAuth::AccessToken.new(token: token.token, options: {sub: sub})`を実行<br>

```
=> #<UserAuth::AccessToken:0x00007f51360d7e18 @user_id="PWOxvWimWVcd26tZXUXL9GtK8DRlRgJcBuN3zJrq+aI4eAaCORMHe2XQw2/KNTDl8xhLGB3EBqz3as9AHqcREoCheo9SdxPQX6I=--4s35iIiihUSqb8Gj--Am+aiTMPCWZpTUQ3dUJ1XA==", @payload={"...
```

- `irb(main):022:0> token = UserAuth::AccessToken.new(token: token.token, options: {sub: sub}).entity_for_user`を実行<br>

```
  User Load (1.9ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
=> #<User id: 1, name: "user0", email: "user0@example.com", password_digest: [FILTERED], activated: true, admin: false, created_at: "2022-03-16 02:15:58", updated_at: "2022-04-01 10:32:16", refresh_jti: "6a141bee4e...
```

- `root $ docker compose run --rm api rails g integration_test AccessToken`を実行<br>

* `api/test/integration/access_token_test.rb`を編集<br>

```rb:access_token_test.rb
require 'test_helper'

class AccessTokenTest < ActionDispatch::IntegrationTest
  def setup
    @user = active_user
    @encode = UserAuth::AccessToken.new(user_id: @user.id)
    @lifetime = UserAuth.access_token_lifetime
  end

  # 共通メソッド
  test 'auth_token_methods' do
    # 初期設定値は想定通りか
    assert_equal 'HS256', @encode.send(:algorithm)
    assert_equal @encode.send(:secret_key), @encode.send(:decode_key)
    assert_equal :sub, @encode.send(:user_claim)
    assert_equal({ typ: 'JWT', alg: 'HS256' }, @encode.send(:header_fields))

    # user_idがnilの場合、暗号メソッドはnilを返しているか
    user_id = nil
    assert_nil @encode.send(:encrypt_for, user_id)

    # user_idがnilの場合、複合メソッドはnilを返しているか
    assert_nil @encode.send(:decrypt_for, user_id)

    # user_idが不正な場合、複合メソッドはnilを返しているか
    user_id = 'aaaaaa'
    assert_nil @encode.send(:decrypt_for, user_id)
  end

  # エンコード検証
  test 'encode_token' do
    # トークン有効期限は期待される時間と同じか(1秒許容)
    expect_lifetime = @lifetime.from_now.to_i
    assert_in_delta expect_lifetime, @encode.send(:token_expiration), 1.second

    # 発行時の@payloadは想定通りか
    payload = @encode.payload
    user_claim = @encode.send(:user_claim)
    assert_equal expect_lifetime, payload[:exp]
    assert_equal @encode.user_id, payload[user_claim]
    assert_equal UserAuth.token_issuer, payload[:iss]
    assert_equal UserAuth.token_audience, payload[:aud]

    # lifetime_textメソッドは想定通りか
    assert_equal '30分', @encode.lifetime_text

    # lifetimeキーがある場合、claimsの値が変わっているか
    time = 1
    lifetime = time.hour
    payload = { lifetime: lifetime }
    encode = UserAuth::AccessToken.new(user_id: @user.id, payload: payload)
    claims = encode.send(:claims)
    expect_lifetime = lifetime.from_now.to_i
    assert_equal expect_lifetime, claims[:exp]

    # lifetime_textは想定通りか
    assert_equal "#{time}時間", encode.lifetime_text
  end

  # デコード検証
  test 'decode_token' do
    decode = UserAuth::AccessToken.new(token: @encode.token)
    payload = decode.payload

    # デコードユーザーは一致しているか
    token_user = decode.entity_for_user
    assert_equal @user, token_user

    # verify_claimsは想定通りか
    verify_claims = decode.send(:verify_claims)
    assert_equal UserAuth.token_issuer, verify_claims[:iss]
    assert_equal UserAuth.token_audience, verify_claims[:aud]
    assert_equal UserAuth.token_signature_algorithm, verify_claims[:algorithm]
    assert verify_claims[:verify_expiration]
    assert verify_claims[:verify_iss]
    assert verify_claims[:verify_aud]
    assert_not verify_claims[:sub]
    assert_not verify_claims[:verify_sub]

    # 有効期限内はエラーを吐いていないか
    travel_to (@lifetime.from_now - 1.second) do
      assert UserAuth::AccessToken.new(token: @encode.token)
    end

    # 有効期限後トークンはエラーを吐いているか
    travel_to (@lifetime.from_now) do
      assert_raises JWT::ExpiredSignature do
        UserAuth::AccessToken.new(token: @encode.token)
      end
    end

    # トークンが書き換えられた場合エラーを吐いているか
    invalid_token = @encode.token + 'a'
    assert_raises JWT::VerificationError do
      UserAuth::AccessToken.new(token: invalid_token)
    end

    # issuerが改ざんされたtokenはエラーを吐いているか
    invalid_token = UserAuth::AccessToken.new(payload: { iss: 'invalid' }).token
    assert_raises JWT::InvalidIssuerError do
      UserAuth::AccessToken.new(token: invalid_token)
    end

    # audienceが改ざんされたtokenはエラーを吐いているか
    invalid_token = UserAuth::AccessToken.new(payload: { aud: 'invalid' }).token
    assert_raises JWT::InvalidAudError do
      UserAuth::AccessToken.new(token: invalid_token)
    end
  end

  # デコードオプション
  test 'verify_claims' do
    # subオプションは有効か
    sub = @encode.user_id
    options = { sub: sub }
    decode = UserAuth::AccessToken.new(token: @encode.token, options: options)
    verify_claims = decode.send(:verify_claims)
    assert_equal sub, verify_claims[:sub]
    assert verify_claims[:verify_sub]

    # subオプションで有効なユーザーは返ってきているか
    token_user = decode.entity_for_user
    assert_equal @user, token_user

    # 不正なsubにはエラーを吐いているか
    options = { sub: 'invalid' }
    assert_raises JWT::InvalidSubError do
      UserAuth::AccessToken.new(token: @encode.token, options: options)
    end
  end

  # not activeユーザーの挙動
  test 'not_active_user' do
    not_active = User.create(name: 'a', email: 'a@a.a', password: 'password')
    assert_not not_active.activated
    encode_token = UserAuth::AccessToken.new(user_id: not_active.id).token

    # アクティブではないユーザも取得できているか
    decode_token_user =
      UserAuth::AccessToken.new(token: encode_token).entity_for_user

    assert_equal not_active, decode_token_user
  end
end
```

- `root $ docker compose run --rm api rails t test/integration`を実行<br>

```
users...---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=] 0% Time: 00:00:00,  ETA: ??:??:??
users...
users = 10
users = 10
  8/8: [==============================================================================================================================================================================] 100% Time: 00:00:04, Time: 00:00:04

Finished in 4.77928s
8 tests, 49 assertions, 0 failures, 0 errors, 0 skips
```
