## 90 リフレッシュトークンを使ってログイン状態を維持する

### ログインしたままにする業務

ユーザーがアプリに訪れた最初に一回 => plugins<br>

`Nuxt` --refresh_token--> `Rails`<br>
↓<br>
`Nuxt ($auth.login())` <--トークンが有効-- `Rails`<br>
`Nuxt (何もしない)` <--トークンが無効-- `Rails`<br>

### ハンズオン

- `root $ touch front/plugins/nuxt-client-init.js`を実行<br>

* `front/nuxt.config.js`を編集<br>

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
    // 追加
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

- `front/plugins/nuxt-client-init.js`を編集<br>

```js:nuxt-client-init.js
export default async ({ $auth, $axios }) => {
  await $axios
    .$post(
      '/api/v1/auth_token/refresh',
      {},
      { validateStatus: (status) => $auth.resolveUnauthorized(status) },
    )
    .then((response) => $auth.login(response))
}
```

- `front/components/LoggedIn/LoggedInAppBarBreadcrumbs.vue`を編集<br>

```vue:LoggedInAppBarBreadcrumbs.vue
<template>
  <v-breadcrumbs :items="items" class="d-block text-truncate">
    <template #item="{ item }">
      <v-breadcrumbs-item>
        <div class="text-truncate" :style="{ maxWidth: '120px' }">
          {{ item.text }}
        </div>
      </v-breadcrumbs-item>
    </template>
    <template #divider>
      <v-icon>
        mdi-chevron-right
      </v-icon>
    </template>
  </v-breadcrumbs>
</template>

<script>
export default {
  computed: {
    items() {
      const currentRouteName = this.$route.name
      const items = [{ text: this.$my.pageTitle(currentRouteName) }]
      <!-- 修正 -->
      const currentProject = this.$store.state.project.current
      // breakpoint.xs => 600未満の場合にtrueを返す
      if (
        currentProject &&
        currentRouteName.match(/project/) &&
        !this.$vuetify.breakpoint.xs
      ) {
        // プロジェクト名を表示する
        items.unshift({ text: currentProject.name })
      }
      <!-- ここまで -->
      return items
    },
  },
}
</script>
```

## 91 サイレントレフレッシュでアクセストークンを更新する

### フロントエンドのログイン判定

`Vuex`<br>

`user`が存在する && `auth.expires`が有効期限内<br>
↓<br>
`loggedIn: true`<br>

### サイレントリフレッシュのタイミング

|    user    | expires  |             access token             |    refresh token     |       logged in       |      redirect      |
| :--------: | :------: | :----------------------------------: | :------------------: | :-------------------: | :----------------: |
|  存在する  |  期限内  |                  -                   |          -           |         TRUE          |         -          |
|  存在する  | 期限切れ | 更新処理<br>(サイレントリフレッシュ) | 有効<br>----<br>無効 | TRUE<br>----<br>FALSE | 「セッション切れ」 |
| 存在しない |    -     |                  -                   |          -           |         FALSE         |  「ログインして」  |

### ハンズオン

- `front/plugins/auth.js`を編集<br>

```js:auth.js
import jwtDecode from 'jwt-decode'

class Authentication {
  constructor(ctx) {
    this.store = ctx.store
    this.$axios = ctx.$axios
  }

  get token() {
    return this.store.state.auth.token
  }

  get expires() {
    return this.store.state.auth.expires
  }

  get payload() {
    return this.store.state.auth.payload
  }

  get user() {
    return this.store.state.user.current || {}
  }

  // 認証情報をVuexに保存する
  setAuth({ token, expires, user }) {
    const exp = expires * 1000
    const jwtPayload = token ? jwtDecode(token) : {}

    this.store.dispatch('getAuthToken', token)
    this.store.dispatch('getAuthExpires', exp)
    this.store.dispatch('getCurrentUser', user)
    this.store.dispatch('getAuthPayload', jwtPayload)
  }

  // ログイン業務
  login(response) {
    this.setAuth(response)
  }

  // Vuexの値を初期値に戻す
  resetVuex() {
    this.setAuth({ token: null, expires: 0, user: null })
    this.store.dispatch('getCurrentProject', null)
    this.store.dispatch('getProjectList', [])
  }

  // axiosのレスポンス401を許容する
  // Doc: https://github.com/axios/axios#request-config
  resolveUnauthorized(status) {
    return (status >= 200 && status < 300) || status === 401
  }

  // ログアウト業務
  async logout() {
    await this.$axios.$delete('/api/v1/auth_token', {
      validateStatus: (status) => this.resolveUnauthorized(status),
    })
    this.resetVuex()
  }

  // 追加
  // 有効期限内にtrueを返す
  isAuthenticated() {
    return new Date().getTime() < this.expires
  }

  // ユーザーが存在している場合はtrueを返す
  isExistUser() {
    return (
      this.user.sub && this.payload.sub && this.user.sub === this.payload.sub
    )
  }

  // ユーザーが存在し、かつ有効期限切れの場合にtrueを返す
  isExistUserAndExpired() {
    return this.isExistUser() && !this.isAuthenticated()
  }

  // ユーザーが存在し、かつ有効期限内の場合にtrueを返す
  loggedIn() {
    return this.isExistUser() && this.isAuthenticated()
  }
  // ここまで
}

export default ({ store, $axios }, inject) => {
  inject('auth', new Authentication({ store, $axios }))
}
```

- `root $ touch front/middleware/silent-refresh-token.js`を実行<br>

* `front/middleware/silent-refresh-token.js`を編集<br>

```js:silent-refresh-token.js
export default async ({ $auth, $axios, store, route, redirect, isDev }) => {
  if ($auth.isExistUserAndExpired()) {
    if (isDev) {
      // eslint-disable-next-line no-console
      console.log('Execute silent refresh!!')
    }
    await $axios
      .$post('/api/v1/auth_token/refresh')
      .then((response) => $auth.login(response))
      .catch(() => {
        const msg =
          'セッションの有効期限が切れました。' + 'もう一度ログインしてください'
        // TODO test
        // eslint-disable-next-line no-console
        console.log(msg)
        // TODO トースター出力
        // store.dispatch('getToast', { msg })
        // TODO アクセスルート記憶
        // store.dispatch('getRememberPath', route)
        // Vuexの初期化(セッションはサーバで削除済み)
        $auth.resetVuex()
        return redirect('/login')
      })
  }
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

  // 追加
  router: {
    middleware: ['silent-refresh-token'],
  },

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

- `api/config/initializers/user_auth.rb`を編集<br>

```rb:user_auth.rb
module UserAuth # access tokenの有効期限
  mattr_accessor :access_token_lifetime
  # 仮にテストの為10秒にしておく(確認後に削除)
  self.access_token_lifetime = 10.second # self.access_token_lifetime = 30.minute

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

参考: https://stackoverflow.com/questions/6346849/what-happens-to-settimeout-when-the-computer-goes-to-sleep <br>

- ブラウザーでログインした後に localhost:8080/projects からダッシュボードに遷移してコンソールを確認してみる<br>

```
{url: '/api/v1/auth_token/refresh', method: 'post', headers: {…}, baseURL: 'http://localhost:3000', transformRequest: Array(1), …}
axios.js?dcce:14 {data: {…}, status: 200, statusText: 'OK', headers: {…}, config: {…}, …}
silent-refresh-token.js?507b:5 Execute silent refresh!!
```
