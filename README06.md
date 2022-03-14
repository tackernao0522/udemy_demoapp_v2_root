## 39 バリデーションメッセージの日本語化

- `api/config/lacales/ja.yml`ファイルを作成<br>

* `root $ curl https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml`実行<br>

- `root $ curl https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml | pbcopy |pbpaste > api/config/locales/ja.yml`を実行<br>

- `root $ +`root \$ curl https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml | pbcopy |pbpaste > api/config/locales/ja.yml`を再実行(下記のようになる)<br>

```yml:ja.yml
ja:
  activerecord:
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
  errors:
    format: '%{attribute}%{message}'
    messages:
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

- `api/config/locales/ja.yml`を編集<br>

```yml:ja.yml
ja:
  activerecord:
    # 追加
    attributes:
      user:
        name: 名前
        email: メールアドレス
        password: パスワード
        activated: アクティブフラグ
        admin: 管理者フラグ
    # ここまで
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
  errors:
    format: '%{attribute}%{message}'
    messages:
      invalid_password: は半角英数字・ハイフン・アンダーバーが使えます # 追加
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

- api/app/models/user.rb`を編集<br>

```rb:user.rb
class User < ApplicationRecord # 7. User.create() => 入力必須バリデーション、User.update() => x # 6. 最大文字数 72文字まで # 5. authenticate() # 4. 一致のバリデーション追加 # 3. password_confirmation => パスワードの一致確認 # 2. password_digest => password # 1. passwordを暗号化する # gem bcrypt
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
              # 追加
              allow_blank: true
            },
            # 空パスワードのアップデートを許容する。(Null(nil)の場合スキップ)
            allow_nil: true
end
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):002:0> user = User.new`を実行<br>

- `irb(main):003:0> user.save`を実行<br>

```:terminal
irb(main):003:0> user.save
=> false
```

- `irb(main):004:0> user.errors.full_messages`を実行<br>

```:terminal
irb(main):004:0> user.errors.full_messages
=> ["パスワードを入力してください", "名前を入力してください"]
```

- `irb(main):005:0> I18n.t("activerecord.attributes.user")`を実行<br>

```terminal
irb(main):005:0> I18n.t("activerecord.attributes.user")
=> {:name=>"名前", :email=>"メールアドレス", :password=>"パスワード", :activated=>"アクティブフラグ", :admin=>"管理者フラグ"}
```

- `irb(main):006:0> user.password ="あああああああああ"`を実行<br>

- `irb(main):007:0> user.password`を実行<br>

```terminal:
irb(main):007:0> user.password
=> "あああああああああ"
```

- `irb(main):008:0> user.save`を実行<br>

```terminal:
irb(main):008:0> user.save
=> false
```

- `irb(main):009:0> user.errors.full_messages`を実行<br>

```:terminal
irb(main):009:0> user.errors.full_messages
=> ["名前を入力してください", "パスワードは半角英数字・ハイフン・アンダーバーが使えます"]
```
