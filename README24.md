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
