# セクション 12: サーバーサイドのログイン認証

## 71 JWT とは何か?

- Json Web Token の略<br>

2 つのパーティー間で情報を安全に送信するための方法<br>

### JWT の実態

- JSON オブジェクトをエンコードした文字列<br>

```json:sample.json
{
  "sub": "1234567890",
  ...
}
```

↓

(エンコードされた文字列) = トークンと呼ぶ<br>

### ３つに分かれるトークン

eyJhbDcioiJIUzUlNilsxxxxxxx. => ヘッダ.<br>
eyJhbDcioiJIUzUlNilsxxxxxxx. => ペイロード.<br>
eyJhbDcioiJIUzUlNilsxxxxxxx. => 署名.<br>

### ヘッダの情報

エンコード eyJhbDciOiJIUzU1NilsInR5cCI6IkpXVCJ9.<br>
デコード { "alg": "HS256", "typ": "JWT" }<br>

トークンのタイプ・署名アルゴリズム(手順・計算方法)の情報が入っている<br>

### ペイロードの情報

エンコード eyJhbDciOiJIUzU1NilsInR5cCI6IkpXVCJ9.<br>
デコード { "sub": "1234567890", "name": "John Doe", "iat": 1516239022 }<br>

任意の情報を埋め込むことができる(有効期限・ユーザー識別情報)<br>

### 署名の情報

エンコード eyJhbDciOiJIUzU1NilsInR5cCI6IkpXVCJ9.<br>

- JWT の送信者が本人であること<br>

* JWT が改ざんされてないこと<br>
  を確認するために使用される<br>

  <署名アルゴリズムが HS256 だった場合の署名作成方法><br>

HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), your-256-bit-secret )<br>
↓<br>
エンコードされたヘッダ + "." + エンコードされたペイロード, シークレットキー<br>

## 72 Rails コンソールから JWT を発行してみる

https://jwt.io/libraries (Ruby)星の多いものを選んだ方が賢明<br>

https://rubygems.org/gems/jwt <br>

- `api/Gemfile`を編集<br>

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

# パスワードを暗号化する
gem 'bcrypt', '~> 3.1', '>= 3.1.16'

# 追加
# jwt Doc: https://rubygems.org/gems/jwt
gem 'jwt', '~> 2.3'

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

group :test do
  # テスト結果色付け Doc: https://github.com/kern/minitest-reporters
  gem 'minitest-reporters', '~> 1.1', '>= 1.1.11'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- `root $ docker compose build api`を実行<br>

* `root $ docker compose run --rm api rails c`を実行<br>

- `irb(main):001:0> payload = { sub: 1 }`を実行<br>

* `irb(main):002:0> Hirb.disable`を実行<br>

- `irb(main):003:0> payload`を実行<br>

```terminal
=> {:sub=>1}
```

参考: https://openid-foundation-japan.github.io/draft-ietf-oauth-json-web-token-11.ja.html#ReservedClaimName<br>

- `irb(main):004:0> secret_key = Rails.application.credentials.se`を実行<br>

```terminal
=> nil
```

- `irb(main):005:0> secret_key = Rails.application.credentials.secret_key_base`を実行<br>

```:terminal
=> "705f6bc32e0db87bbff3d231b43e6678b3d54e1db609fe4d81bb1e64ae16a41a092da66ca0a57e7eee339d31bdc2f4e900912af00f9e3dcc47501c0709216db6"
```

- `irb(main):006:0> token = JWT.encode(payload, secret_key)`を実行<br>

```:terminal
=> "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9.-4ku4NGdeQSe5iCxLcUrG4jjZTtdjOKyHc7PvmfMu9k"
```

- `irb(main):007:0> token`を実行<br>

```:terminal
=> "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9.-4ku4NGdeQSe5iCxLcUrG4jjZTtdjOKyHc7PvmfMu9k"
```

参考: https://www.rubydoc.info/github/jwt/ruby-jwt/JWT/DefaultOptions#DEFAULT_OPTIONS-constant<br>

- `irb(main):008:0> JWT.decode(token, secret_key)`を実行<br>

```:terminal
=> [{"sub"=>1}, {"alg"=>"HS256"}]
```
