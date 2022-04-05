# セクション 13: フロントエンドのログインリクエストを送る

## 86 Nuxt.js から Rails へログインリクエストを送る

- `front/pages/login.vue`を編集<br>

```vue:login.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <v-form ref="form" v-model="isValid" @submit.prevent="login">
        <!-- 編集 userをauthに変更 -->
        <user-form-email :email.sync="params.auth.email" />
        <!-- 編集 userをauthに変更 -->
        <user-form-password :password.sync="params.auth.password" />
        <v-card-actions>
          <nuxt-link to="#" class="body-2 text-decoration-none">
            パスワードを忘れた？
          </nuxt-link>
        </v-card-actions>
        <v-card-text class="px-0">
          <!-- disabled=true => ボタンクリックを無効にする -->
          <v-btn
            type="submit"
            :disabled="!isValid || loading"
            :loading="loading"
            block
            color="appblue"
            class="white--text"
          >
            ログインする
          </v-btn>
        </v-card-text>
      </v-form>
    </template>
  </user-form-card>
</template>

<script>
import UserFormCard from '../components/User/UserFormCard.vue'
import UserFormEmail from '../components/User/UserFormEmail.vue'
import UserFormPassword from '../components/User/UserFormPassword.vue'

export default {
  components: { UserFormCard, UserFormEmail, UserFormPassword },
  layout: 'before-login',
  data({ $store }) {
    return {
      name: '',
      isValid: false,
      loading: false,
      // TODO 削除する
      <!-- 編集 userをauthに変更 -->
      params: { auth: { email: 'user0@example.com', password: 'password' } },
      redirectPath: $store.state.loggedIn.homePath,
    }
  },
  methods: {
    <!-- asyncを追記 -->
    async login() {
      this.loading = true
      <!-- 追記 -->
      if (this.isValid) {
        await this.$axios
          .$post('/api/v1/auth_token', this.params)
          .then((response) => this.authSuccessful(response))
          .catch((error) => this.authFailure(error))
      }
      this.loading = false
      <!-- ここまで -->
      this.$router.push(this.redirectPath)
    },
    <!-- 追記 -->
    authSuccessful(response) {
      // eslint-disable-next-line no-console
      console.log('authSuccessful', response)
      // TODO ログイン処理
      // TODO 記憶ルートリダイレクト
      this.$router.push(this.redirectPath)
    },
    authFailure({ response }) {
      if (response && response.status === 404) {
        // TODO トースター出力
      }
      // TODO エラー処理
    },
    <!-- ここまで -->
  },
}
</script>
```

- localhost:8080 にアクセスしてログインしてみると コンソールに`403 (Forbidden)`が返ってきている<br>

* `front/plugins/axios.js`を編集<br>

```js:axios.js
export default ({ $axios }) => {
  // リクエストログ
  $axios.onRequest((config) => {
    // 追記
    config.headers.common['X-Requested-With'] = 'XMLHttpRequest'
    // eslint-disable-next-line no-console
    console.log(config)
  })
  // レスポンスログ
  $axios.onResponse((config) => {
    // eslint-disable-next-line no-console
    console.log(config)
  })
  // エラーログ
  $axios.onError((e) => {
    // eslint-disable-next-line no-console
    console.log(e.response)
  })
}
```

- 再度ログインしてみる<br>

```
{url: '/api/v1/auth_token', method: 'post', data: {…}, headers: {…}, baseURL: 'http://localhost:3000', …}
axios.js?dcce:11
{data: {…}, status: 200, statusText: 'OK', headers: {…}, config: {…}, …}
login.vue?8d26:74 authSuccessful
{token: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2N…wMCJ9.ypuKUZTS6FXSpOEVLynnwKS9QBL55IOT-_pSrehprnI', expires: 1649135125, user: {…}}
expires: 1649135125
token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkxMzUxMjUsInN1YiI6IldpTFdxTkpqQkRUT3BhaWt1RGdrbm8wRlE3a21UZEZlSlFCQk9oSTlHSUhJOXNzYm40RXhvOHpxT1MrUUthOEJxWGRIZ085UmxkZUllZ3g0S1hMbFVpRjdHRE1aTzlNTjdTOD0tLWFObEhYSkhUbHg3Mi9MNmEtLUxwUllLVngzNE1xazkyZk9qRHg4Z3c9PSIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCJ9.ypuKUZTS6FXSpOEVLynnwKS9QBL55IOT-_pSrehprnI"
user:
id: 1
name: "user0"
sub: "WiLWqNJjBDTOpaikuDgkno0FQ7kmTdFeJQBBOhI9GIHI9ssbn4Exo8zqOS+QKa8BqXdHgO9RldeIegx4KXLlUiF7GDMZO9MN7S8=--aNlHXJHTlx72/L6a--LpRYKVx34Mqk92fOjDx8gw=="
[[Prototype]]: Object
[[Prototype]]: Object
```

参考: https://axios.nuxtjs.org/options/#credentials <br>

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
  plugins: ['plugins/axios', 'plugins/my-inject'],

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
    // 編集
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

- `再度ログインする`<br>

- コンソールの Application の Cookies に refresh_token が保存されていれば OK<br>
