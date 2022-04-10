# セクション 14: 本番環境への対応

## 98 ブラウザのサードパーティ Cookie 拒否に対応する

### 現状の問題点

Safari でリロードするとログアウトする<br>

原因：refresh_token が Cookie に保存されていない<br>

サードパーティの Cookie を拒否する仕様のため<br>

2022 年までに chrome も上記仕様になる予定<br>

### Cookie 拒否の解決方法

同一サイトとみなされれば Cookie は拒否されない<br>

同一サイトの定義は => 同じドメインのサブドメイン同士<br>

`Nuxt`udemy-v2.example.com -- `DNS` example.com -- `Rails`api.udemy-v2.example.com <br>

https://web.dev/same-site-same-origin/ <br>

### このチャプターで達成すること

| Nuxt.js（フロント）                 | Rails（サーバ）                      |
| ----------------------------------- | ------------------------------------ |
| 1 □ Heroku にカスタムドメイン設定   | 6 □ Heroku にカスタムドメイン設定    |
| 2 □ ドメインの DNS 設定             | 7 □ ドメインの DNS 設定              |
| 3 □ SSL 化                          | 8 □ SSL 化                           |
| 4 □ Rails の API_DOMAIN の値の変更  | 9 □ Nuxt.js の API_URL の値を変更    |
| 5 □ 常時 SSL 化（リダイレクト処理） | 10 □ 常時 SSL 化（リダイレクト処理） |

### このチャプターを始める前に

- カスタムドメインを取得済みであること<br>

* Heroku にクレジットカードを登録済みであること<br>

- SSL 化には、\$7 x 2app/月 の料金がかかること<br>

https://jp.heroku.com/pricing <br>

(2021 年時点)<br>

### 99 Nuxt.js にカスタムドメインを設定して SSL 化する

- `front $ heroku login`を実行<br>

* `front $ heroku domains:add www.takapro-tech.site`を実行<br>

- `front $ heroku domains`を実行して確認<br>

```
=== tk-railsnuxtv1-front Heroku Domain
tk-railsnuxtv1-front.herokuapp.com

=== tk-railsnuxtv1-front Custom Domains
Domain Name           DNS Record Type DNS Target                                              SNI Endpoint
www.takapro-tech.site CNAME           fathomless-taiga-o7ceundax1gabha0ex721nu4.herokudns.com pteranodon-17263
```

taka-project777@taka.com <br>

https://qiita.com/tksh8/items/ab3748bcb01316461abe

https://qiita.com/ozin/items/62bc7ef1dd3c827177fb <br>

## 100 Nuxt.js に SSL リダイレクト処理を行う

https://nuxtjs.org/ja/docs/configuration-glossary/configuration-servermiddleware/ <br>

https://nuxtjs.org/ja/docs/concepts/nuxt-lifecycle/ <br>

https://github.com/unjs/redirect-ssl/blob/e393ad6b11fe76e43934f9737fdc5c1696e0806c/src/index.ts#L43 <br>

### 現状の Nuxt アプリドメイン

1. Heoku Domain https://tk-railsnuxtv1-front.herokuapp.com <br>

2. Custom Domain http://www.takapro-tech.site <br>

3. SSL Custom Domain https://www.takapro-tech.site <br>

### ハンズオン

- `api $ heroku config:get API_DOMAIN`を実行<br>

```
tk-railsnuxtv1-front.herokuapp.com
```

- `api $ heroku config:set API_DOMAIN=www.takapro-tech.site`を実行<br>

- `api $ heroku config:get API_DOMAIN`を実行<br>

```
www.takapro-tech.site
```

- `front $ mkdir server && touch $_/redirect-ssl.js`を実行<br>

* `front/server/redirect-ssl.js`を編集<br>

```js:redirect-ssl.js
// req、res => https://nuxtjs.org/docs/2.x/internals-glossary/nuxt-render
export default (req, res, next) => {
  const { NODE_ENV, BASE_URL } = process.env

  // 本番環境 && BASE_URLが存在する場合
  if (NODE_ENV === 'production' && !!BASE_URL) {
    const redirectDomain = /herokuapp.com/
    const reqHost = req.headers.host
    // x-forwarded-proto(Herokuヘッダー): https://devcenter.heroku.com/articles/http-routing#heroku-headers
    const reqProtocol = req.headers['x-forwarded-proto']

    // SSLではない || リクエストホストにredirectDomainを含む場合
    if (reqProtocol === 'http' || redirectDomain.test(reqHost)) {
      const redirectTargetUrl = BASE_URL + req.url

      // リクエストに301(恒久的な転送)レスポンスヘッダーを送信する
      // { Location: 'リダイレクト先URL' }
      // Doc: https://nodejs.org/api/http.html#http_response_writehead_statuscode_statusmessage_headers
      res.writeHead(301, { Location: redirectTargetUrl })

      // リクエストの送信を終了する
      // end(data)が指定されている場合は、request.write(data, encoding)の後にrequest.end(callback)を呼び出したのと同じです。callbackを指定した場合は、リクエストストリームが終了したときに呼び出されます。
      // Doc: https://nodejs.org/api/http.html#http_response_end_data_encoding_callback
      return res.end(redirectTargetUrl)
    }
  }
  return next()
}
```

- `front/nuxt.config.js`を編集<br>

```js:nuxt.config.js
export default {
  // Disable server-side rendering: https://go.nuxtjs.dev/ssr-mode
  ssr: false,

  // Global page headers: https://go.nuxtjs.dev/config-head
  head: {
    title: 'app',
    htmlAttrs: {
      lang: 'en',
    },
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: '' },
      { name: 'format-detection', content: 'telephone=no' },
    ],
    link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }],
  },

  // Global CSS: https://go.nuxtjs.dev/config-css
  css: ['~/assets/sass/main.scss'],

  // Plugins to run before rendering page: https://go.nuxtjs.dev/config-plugins
  plugins: [
    'plugins/auth',
    'plugins/axios',
    'plugins/my-inject',
    'plugins/nuxt-client-init',
  ],

  // Auto import components: https://go.nuxtjs.dev/config-components
  components: true,

  // Modules for dev and build (recommended): https://go.nuxtjs.dev/config-modules
  buildModules: [
    // https://go.nuxtjs.dev/eslint
    '@nuxtjs/eslint-module',
    // Doc: https://www.npmjs.com/package/@nuxtjs/vuetify
    '@nuxtjs/vuetify',
  ],

  // Modules: https://go.nuxtjs.dev/config-modules
  modules: [
    // https://go.nuxtjs.dev/axios
    '@nuxtjs/axios',
    // Doc: https://www.npmjs.com/package/@nuxtjs/i18n
    '@nuxtjs/i18n',
  ],

  publicRuntimeConfig: {
    appName: process.env.APP_NAME,
  },

  router: {
    middleware: ['silent-refresh-token'],
  },

  // 追加
  // Doc: https://nuxtjs.org/ja/docs/configuration-glossary/configuration-servermiddleware/
  serverMiddleware: ['~/server/redirect-ssl'],

  // Axios module configuration: https://go.nuxtjs.dev/config-axios
  axios: {
    // 環境変数API_URLが優先される
    // baseURL: 'http://localhost:3000'
    // クロスドメインで認証情報を共有する
    // DOC: https://axios.nuxtjs.org/options/#credentials
    credentials: true,
  },
  vuetify: {
    // カスタムCSSファイルパス
    customVariables: ['~/assets/sass/valiables.scss'],
    // カスタムCSSを有効にするフラグ
    // Doc: https://vuetifyjs.com/en/features/sass-variables/#nuxt-install
    treeShake: true,
    theme: {
      themes: {
        light: {
          primary: '4080BE',
          info: '4FC1E9',
          success: '44D69E',
          warning: 'FEB65E',
          error: 'FB8678',
          background: 'f6f6f4',
          appblue: '1867C0',
        },
      },
    },
  },
  // Doc: https://nuxt-community.github.io/nuxt-i18n/basic-usage.html#nuxt-link
  i18n: {
    // 対応言語
    locales: ['ja', 'en'],
    // デフォルトで使用する言語を指定
    defaultLocale: 'ja',
    // no_prefix => ルート名に__jaを追加しない
    strategy: 'no_prefix',
    // Doc: https://kazupon.github.io/vue-i18n/api/#properties
    vueI18n: {
      // 翻訳対象のキーがない場合に参照される言語
      // "login": "ログイン"
      fallbackLocale: 'ja',
      // true => i18nの警告を完全に表示しない(default: false)
      // silentTranslationWarn: true,
      // フォールバック時に警告を発生させる(default: false)
      // true => 警告を発生させない(翻訳のキーが存在しない場合のみ警告)
      silentFallbackWarn: true,
      // 翻訳データ
      messages: {
        ja: require('./locales/ja.json'),
        en: require('./locales/en.json'),
      },
    },
  },
  // Build Configuration: https://go.nuxtjs.dev/config-build
  build: {},
}
```

- `front $ heroku config:set BASE_URL=https://www.takapro-tech.site`を実行 <br>

* heroku に push する `$ git push heroku`<br>

## 101 Rails にカスタムドメインを設定して SSL 化する<br>

https://devcenter.heroku.com/ja/articles/automated-certificate-management#setup <br>

- `api $ heroku domains:add www.api.takapro-tech.site`を実行<br>

```
tk-railsnuxtv1-api.herokuapp.com

=== tk-railsnuxtv1-api Custom Domains
Domain Name               DNS Record Type DNS Target                                           SNI Endpoint
api.www.takapro-tech.site CNAME           closed-violet-pnm512ykaapch0f89gugoid8.herokudns.com undefined
groovy@groovy-no-MacBook-Pro api % heroku domains:add www.api.takapro-tech.site
Configure your app's DNS provider to point to the DNS Target animated-goldenrod-az4sg9vabl8nfnsdzoek8gdo.herokudns.com.
    For help, see https://devcenter.heroku.com/articles/custom-domains

The domain www.api.takapro-tech.site has been enqueued for addition
Run heroku domains:wait 'www.api.takapro-tech.site' to wait for completion
Adding www.api.takapro-tech.site to ⬢ tk-railsnuxtv1-api... done
```

- `api $ heroku domains`を実行して`target`をコピーしておく<br>

```
=== tk-railsnuxtv1-api Heroku Domain
tk-railsnuxtv1-api.herokuapp.com

=== tk-railsnuxtv1-api Custom Domains
Domain Name               DNS Record Type DNS Target                                                SNI Endpoint
www.api.takapro-tech.site CNAME           animated-goldenrod-az4sg9vabl8nfnsdzoek8gdo.herokudns.com undefined
```

- お名前.com の DSN 設定に追加しておく<br>

* `api $ heroku ps:resize web=hobby`を実行<br>

```
Scaling dynos on ⬢ tk-railsnuxtv1-api... done
=== Dyno Types
type  size   qty  cost/mo
────  ─────  ───  ───────
web   Hobby  1    7
=== Dyno Totals
type   total
─────  ─────
Hobby  1
```

- `api $ heroku ps`を実行<br>

```
=== web (Hobby): /bin/sh -c bundle\ exec\ puma\ -C\ config/puma.rb (1)
web.1: up 2022/04/10 12:20:21 +0900 (~ 41s ago)
```

- `api $ heroku certs:auto`を実行<br>

Status は`Cert issued`になっていれば OK<br>

```
=== Automatic Certificate Management is enabled on tk-railsnuxtv1-api

Certificate details:
Common Name(s): www.api.takapro-tech.site
Domain(s):      7104b1b7-7b4a-4fe9-a55a-15fc0e5870e3
Expires At:     2022-07-09 02:20 UTC
Issuer:         /C=US/O=Let's Encrypt/CN=R3
Starts At:      2022-04-10 02:20 UTC
Subject:        /CN=www.api.takapro-tech.site
SSL certificate is verified by a root authority.

Domain                     Status       Last Updated
─────────────────────────  ───────────  ────────────
www.api.takapro-tech.site  Cert issued  5 minutes
```

- `front/heroku.yml`を編集<br>

```yml:heroku.yml
setup:
  config:
    NODE_ENV: production
build:
  docker:
    web: Dockerfile
  config:
    WORKDIR: app
    # 編集
    API_URL: 'https://www.api.takapro-tech.site'
run:
  web: yarn run start
```

- front を heroku に push しておく(これで safari などでも SSL redeirect する)<br>

## 102 Rails に SSL リダイレクト処理を行う

https://composition-api.nuxtjs.org/ <br>

https://github.com/jtrupiano/rack-rewrite <br>

https://web.dev/samesite-cookies-explained/ <br>

https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Set-Cookie/SameSite <br>

- `api/config/enviroments/production.rb`を編集<br>

```rb:production.rb
Rails.application.configure do
  # Settings specified here will take precedence over those in config/application.rb.

  # Code is not reloaded between requests.
  config.cache_classes = true

  # Eager load code on boot. This eager loads most of Rails and
  # your application in memory, allowing both threaded web servers
  # and those relying on copy on write to perform better.
  # Rake tasks automatically ignore this option for performance.
  config.eager_load = true

  # Full error reports are disabled and caching is turned on.
  config.consider_all_requests_local = false

  # Ensures that a master key has been made available in either ENV["RAILS_MASTER_KEY"]
  # or in config/master.key. This key is used to decrypt credentials (and other encrypted files).
  # config.require_master_key = true

  # Disable serving static files from the `/public` folder by default since
  # Apache or NGINX already handles this.
  config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?

  # Enable serving of images, stylesheets, and JavaScripts from an asset server.
  # config.action_controller.asset_host = 'http://assets.example.com'

  # Specifies the header that your server uses for sending files.
  # config.action_dispatch.x_sendfile_header = 'X-Sendfile' # for Apache
  # config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect' # for NGINX

  # Store uploaded files on the local file system (see config/storage.yml for options).
  config.active_storage.service = :local

  # Mount Action Cable outside main process or domain.
  # config.action_cable.mount_path = nil
  # config.action_cable.url = 'wss://example.com/cable'
  # config.action_cable.allowed_request_origins = [ 'http://example.com', /http:\/\/example.*/ ]

  # Force all access to the app over SSL, use Strict-Transport-Security, and use secure cookies.
  # コメントアウト解除
  config.force_ssl = true

  # Use the lowest log level to ensure availability of diagnostic information
  # when problems arise.
  config.log_level = :debug

  # Prepend all log lines with the following tags.
  config.log_tags = %i[request_id]

  # Use a different cache store in production.
  # config.cache_store = :mem_cache_store

  # Use a real queuing backend for Active Job (and separate queues per environment).
  # config.active_job.queue_adapter     = :resque
  # config.active_job.queue_name_prefix = "app_production"

  config.action_mailer.perform_caching = false

  # Ignore bad email addresses and do not raise email delivery errors.
  # Set this to true and configure the email server for immediate delivery to raise delivery errors.
  # config.action_mailer.raise_delivery_errors = false

  # Enable locale fallbacks for I18n (makes lookups for any locale fall back to
  # the I18n.default_locale when a translation cannot be found).
  config.i18n.fallbacks = true

  # Send deprecation notices to registered listeners.
  config.active_support.deprecation = :notify

  # Use default logging formatter so that PID and timestamp are not suppressed.
  config.log_formatter = ::Logger::Formatter.new

  # Use a different logger for distributed setups.
  # require 'syslog/logger'
  # config.logger = ActiveSupport::TaggedLogging.new(Syslog::Logger.new 'app-name')

  if ENV['RAILS_LOG_TO_STDOUT'].present?
    logger = ActiveSupport::Logger.new(STDOUT)
    logger.formatter = config.log_formatter
    config.logger = ActiveSupport::TaggedLogging.new(logger)
  end

  # Do not dump schema after migrations.
  config.active_record.dump_schema_after_migration = false

  # Inserts middleware to perform automatic connection switching.
  # The `database_selector` hash is used to pass options to the DatabaseSelector
  # middleware. The `delay` is used to determine how long to wait after a write
  # to send a subsequent read to the primary.
  #
  # The `database_resolver` class is used by the middleware to determine which
  # database is appropriate to use based on the time delay.
  #
  # The `database_resolver_context` class is used by the middleware to set
  # timestamps for the last write to the primary. The resolver uses the context
  # class timestamps to determine how long to wait before reading from the
  # replica.
  #
  # By default Rails will store a last write timestamp in the session. The
  # DatabaseSelector middleware is designed as such you can define your own
  # strategy for connection switching and pass that into the middleware through
  # these configuration options.
  # config.active_record.database_selector = { delay: 2.seconds }
  # config.active_record.database_resolver = ActiveRecord::Middleware::DatabaseSelector::Resolver
  # config.active_record.database_resolver_context = ActiveRecord::Middleware::DatabaseSelector::Resolver::Session
end
```

- `api/app/controller/application_controller.rb`を編集<br>

```rb:application_controller.rb
class ApplicationController < ActionController::API # 301リダイレクト(本番環境のみ有効) # 追加
  before_action :moved_permanently, if: :is_redirect # Cookieを扱う
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
  end

  # 追加
  # リダイレクト条件に一致した場合はtrueを返す
  def is_redirect
    redirect_domain = 'herokuapp.com'
    Rails.env.production? && ENV['BASE_URL'] &&
      request.host.include?(redirect_domain)
  end

  # 301リダイレクトを行う
  def moved_permanently
    redirect_to "#{ENV['BASE_URL']}#{request.path}", status: 301
  end # リクエストヘッダ X-Requested-With: 'XMLHttpRequest' の存在を判定
end
```

- `api $ heroku config:get BASE_URL`を実行して確認<br>

```
https://tk-railsnuxtv1-api.herokuapp.com
```

- `api $ heroku config:set BASE_URL=https://www.api.takapro-tech.site`を実行<br>

```
Setting BASE_URL and restarting ⬢ tk-railsnuxtv1-api... done, v28
BASE_URL: https://www.api.takapro-tech.site
```

- `api $ heroku config:get BASE_URL`を実行して確認<br>

```
https://www.api.takapro-tech.site
```

- heroku に push しておく<br>

* `api $ heroku open /api/v1/projects`を実行<br>

- https://www.api.takapro-tech.site/api/v1/projects へリダイレクト処理される<br>

* `api $ heroku open http://www.api.takapro-tech.site`を実行<br>

- https://www.api.takapro-tech.site/ へリダイレクト処理される<br>

* `api $ heroku config:set COOKIES_SAME_SITE=strict`を実行<br>

* `api $ heroku config:get COOKIES_SAME_SITE`を実行<br>

```
strict
```
