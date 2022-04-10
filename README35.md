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
