## 90 リフレッシュトークンを使ってプロジェクト一覧を取得する

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
