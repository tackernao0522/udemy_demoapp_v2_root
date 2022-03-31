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

## 73 JWT の３つのメリットと注意点を理解する

- `root $ docker compose run --rm api rails c`を実行<br>

- `irb(main):002:0> payload = {sub: 1}`を実行<br>

```
=> {:sub=>1}
```

- `irb(main):003:0> secret_key = Rails.application.credentials.secret_key_base`を実行<br>

```
=> "705f6bc32e0db87bbff3d231b43e6678b3d54e1db609fe4d81bb1e64ae16a41a092da66ca0a57e7eee339d31bdc2f4e900912af00f9e3dcc47501c0709216db6"
```

- `irb(main):004:0> token = JWT.encode(payload, secret_key)`を実行<br>

```
=> "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9.-4ku4NGdeQSe5iCxLcUrG4jjZTtdjOKyHc7PvmfMu9k"
```

1. 情報が改ざんできない<br>

- `irb(main):005:0> JWT.decode(token, secret_key)`を実行<br>

```
=> [{"sub"=>1}, {"alg"=>"HS256"}]
```

- https://jwt.io/ の Encoded の枠に`eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9.-4ku4NGdeQSe5iCxLcUrG4jjZTtdjOKyHc7PvmfMu9k`を貼り付けると Decode された状態が右枠に確認できる<br>

* Decode の sub を 2 に変えると Encoded の文字列が変わっているのがわかる(改ざんされた状態)<br>

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjJ9.JM1y2F5gk72w5e7Djb3apcUqgJrmT8yR5eXirQjvXiQ
```

- `irb(main):006:0> token2 = "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjJ9.JM1y2F5gk72w5e7Djb3apcUqgJrmT8yR5eXirQjvXiQ" 改ざんされた token を入れて実行<br>

```
=> "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjJ9.JM1y2F5gk72w5e7Djb3apcUqgJrmT8yR5eXirQjvXiQ"
```

- `irb(main):007:0> JWT.decode(token2, secret_key)`を実行(通らないエラーになる)<br>

```
Traceback (most recent call last):
        1: from (irb):7
JWT::VerificationError (Signature verification raised)
```

2. ユーザーテーブルが簡潔になる<br>

参考: https://railstutorial.jp/chapters/account_activation_password_reset?version=4.2#cha-account_activation_and_password_reset <br>

3. 発行者が担保される<br>

- `irb(main):008:0> key = "aaaaaaaaa"`を実行<br>

```
=> "aaaaaaaaa"
```

- `irb(main):009:0> token = JWT.encode(payload, key)`を実行<br>

```
=> "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9._kSU33Ae3KEXpmcFToecTds0IUGPQWROsj6onLZDhG8"
```

- `irb(main):010:0> JWT.decode(token, key)`を実行<br>

```
=> [{"sub"=>1}, {"alg"=>"HS256"}]
```

- `irb(main):011:0> JWT.decode(token, key+"a")`を実行<br>

```
Traceback (most recent call last):
        1: from (irb):11
JWT::VerificationError (Signature verification raised)
```

- `irb(main):012:0> JWT.decode(token, nil)`を実行<br>

```
Traceback (most recent call last):
        2: from (irb):11
        1: from (irb):12:in `rescue in irb_binding'
JWT::DecodeError (No verification key available)
```

### 署名アルゴリズム

署名時と同じ鍵を使って検証する<br>
HS256<br>

秘密鍵と公開鍵のペアで検証する<br>
RS256<br>

### JWT の注意点

1. 誰でもトークンの内容が確認できる<br>

### まとめ

誰でも観れるけど、改ざんできないし、発行者しかデコードできないトークン<br>

## 74 JWT を使ったログイン認証の流れを理解する

参考: https://www.youtube.com/watch?v=4JREwhSC2dQ <br>

### 認証に使用する 2 つの JWT

`refresh_token` `access_token`<br>

### refresh_token の役割

|          |                                       |                    意図                     |
| :------: | :-----------------------------------: | :-----------------------------------------: |
|   目的   | access_token を<br>発行するための JWT | 不正なリクエストから<br>ユーザー情報を守る  |
|  保管先  |           Cookie(HttpOnly)            |     外部 JS を遮断<br>盗聴リスクを軽滅      |
| 有効期限 |                24 時間                | 業務用アプリを前提<br>1 日 1 回の訪問を想定 |

### access_token の役割

|          |                                          |                     意図                     |
| :------: | :--------------------------------------: | :------------------------------------------: |
|   目的   | 保護リソースに<br>アクセスするための JWT |  不正なリクエストから<br>ユーザー情報を守る  |
|  保管先  |                   Vuex                   | 保管されっぱなしを回避し<br>盗聴リスクを軽滅 |
| 有効期限 |                  30 分                   |   盗聴された場合の被害を<br>最小限に抑える   |

### トークンの役割のまとめ

|          |             refresh_token             |               access_token               |
| :------: | :-----------------------------------: | :--------------------------------------: |
|   目的   | access_token を<br>発行するための JWT | 保護リソースに<br>アクセスするための JWT |
|  保管先  |           Cookie(HttpOnly)            |                   Vuex                   |
| 有効期限 |                24 時間                |                  30 分                   |

### ログイン認証の流れ

`Nuxt`-(email, passowrd)->`Rails`(テーブルからうユーザーを取得)-`Rails`(ユーザーがいない場合)-(404 Not Found)->`Nuxt`-> `Vuex`に保管<br>

`Nuxt`-(email, passowrd)->`Rails`(テーブルからうユーザーを取得)-`Rails`(refresh, access の発行)(refresh ID を Users テーブルに保存)(refresh を Cookie に保存)-(access, expires, user)->`Nuxt`->`Vuex`に保管<br>

### サイレントリフレッシュの流れ

`Nuxt`(expires の有効期限が切れた)-リクエスト/refresh `Cookie = refresh`->`Rails`(refresh が有効か確認)無効だった場合-401 Unauthorized->`Nuxt`->セッションが切れたという事をユーザーに通知しログインページにリダイレクトする`redirect /login`<br>

`Nuxt`(expires の有効期限が切れた)-リクエスト/refresh `Cookie = refresh`->`Rails`(refresh が有効か確認)有効だった場合(新しい refresh, access を発行)(Cookie 保存, refresh ID 保存)-access, expires, user->`Nuxt`->`Vuex`に保管<br>

### ログインしたままにする実装

`Nuxt`-リクエスト /refresh(Cookie の有無に関わらず)->`Rails(refreshが有効か確認)(無効だった場合、無い場合)`-401 Unauthorized->`Nuxt`(何もしない)<br>

`Nuxt`-リクエスト /refresh(Cookie の有無に関わらず)->`Rails(refreshが有効か確認)(有効だった場合)(新しいrefresh, accessを発行)(Cookie保存, refresh ID保存)`-(access, expires, user)->`Nuxt`->`Vuex`に保管->redirec /project<br>

### 対 CSRF

クロスサイト・リクエストフォージェリ<br>

リクエスト時に Cookie が自動送信される仕組みを悪用した攻撃<br>

`ログイン済みユーザ`->`罠サイト(CSRF攻撃)(別オリジンからの攻撃となる)`-不正な api リクエスト(今回は Cookie = refresh のみ)->`Rails`(ユーザー DB の書き換えがされてしまう)<br>

refresh => access を発行することしかできない<br>
よってユーザー DB が書き換えられることはない<br>

access を受け取ったらヤバいのでは？<br>
下記のように CORS を正しく設定することで、リクエストできない。よってレスポンスも受け取れない。<br>

### CORS とは

クロスオリジン・リソース・シェアリング<br>
別オリジン間で通信を行う際の HTTP 通信ルール<br>

`Nuxt(http://loccalhost:3000)` <-同一オリジン間の通信しか許容しないよ！-> `Rails(http://localhost:3000)`<br>

`Nuxt(http://localhost:8080)` <-CORS 設定-> `Rails(http://localhost:3000)`<br>
ルールの中であれば、別オリジンの通信も許容するよ<br>

注意<br>

### CSRF 対策 == CORS 設定ではない

CORS = ルールであって、セキュリティではない<br>

- ブラウザの互換性による<br>

* シンプルなリクエストにはルール適用外 (`<img src="">`)<br>

今回のアプリの場合は<br>
カスタムヘッダを使用<br>

`Nuxt`<--X-Requested-With: XMLHttpRequest(対 CSRF)-->(request.xhr?) CORS しかアクセス不可`Rails`<br>

### 対 XSS

クロスサイト・スクリプティング<br>
外部パラメータに応じて表示を変化させる場所への脆弱性をついた攻撃<br>

http://example.com?q=<script>alert(document.cookie)</script> <br>
Cookie、 Web ストレージ、変数値の盗み出し<br>

Cookie(HttpOnly) => JS のアクセスを遮断<br>
access を変数に保存 => 保存され歯内の状態を回避<br>

### ただし、完璧ではない

対 XSS => Cookie(HttpOnly)でも盗まれる<br>
同一オリジン、同一 JS で実行されるため 変数の access も送信される （参照) https://www.youtube.com/watch?v=4JREwhSC2dQ <br>

対 CSRF => CORS 設定が完璧な前提での話<br>
ここが不完全だとレスポンストークンが受け取れる<br>

脆弱性を一つ一つ潰して、そもそお攻撃させないことが大事<br>

### Web アプリのセキュリティについて

おすすめ書籍<br>
`体系的に学ぶ 安全なWebアプリケーションの作り方 第2版`<br>
CSRF CORS XSS ..<br>

### ログイン認証の流れまとめ

- refresh_token => access を発行するため<br>

* access_token => 保護リソースにアクセスする<br>

- access の有効期限が切れた => サイレントリフレッシュ<br>

* ログインしたままの実装 => refresh をサーバで検証<br>

- 対 CSRF => CORS 設定をまず確認<br>

* 対 XSS => 脆弱性を潰す他ない<br>
