# セクション 11: ログイン後のレイアウト構築

## 60 ログイン後のツールバーを作成する 1/2 〜下準備編〜

### ログイン後のページ設計

`ログイン => /login` -リダイレクト-> `プロジェクト一覧 => /projects` -選択-> `個別プロジェクト => /project/id/dashbord`<br>

`アカウント => account/settings`<br>

`ログアウト => /logout`<br>

### ハンズオン

- `front $ touch layouts/logged-in.vue`を実行<br>

- `front $ mkdir components/LoggedIn && touch $_/{LoggedInAppBar.vue,LoggedInAppBarAccountMenu.vue}`を実行<br>

* `front $ touch pages/projects.vue`を実行<br>

* `front/layouts/logged-in.vue`を編集<br>

```vue:logged-in.vue
<template>
  <v-app>
    <logged-in-app-bar />
    <v-main>
      <Nuxt />
    </v-main>
    <app-footer />
  </v-app>
</template>

<script>
export default {}
</script>
```

- `front/components/LoggedIn/LoggedInAppBar.vue`を編集<br>

```vue:LoggedInAppBar.vue
<template>
  <v-app-bar app dense elevation="1" color="white">
    <nuxt-link :to="redirectPath" class="text-decoration-none">
      <app-logo />
    </nuxt-link>

    <app-title />

    <v-spacer />

    <logged-in-app-bar-account-menu />
  </v-app-bar>
</template>

<script>
export default {}
</script>
```

- `front/components/LoggedIn/LoggedInAppBarAccountMenu.vue`を編集<br>

```vue:LoggedInAppBarAccountMenu.vue
<template>
  <div>
    LoggedInAppBarAccountMenu.vue
  </div>
</template>

<script>
export default {}
</script>
```

- `front/components/LoggedIn/LoggedInAppBar.vue`を編集<br>

```vue:LoggedInAppBar.vue
<template>
  <v-app-bar app dense elevation="1" color="white">
    <nuxt-link :to="redirectPath" class="text-decoration-none">
      <app-logo />
    </nuxt-link>

    <app-title />

    <v-spacer />

    <logged-in-app-bar-account-menu />
  </v-app-bar>
</template>

<script>
export default {
  <!-- 追加 -->
  data({ $store }) {
    return {
      redirectPath: $store.state.loggedIn.redirectPath,
    }
  },
}
</script>
```

- `front/store/index.js`を編集<br>

```js:index.js
// 追加
const redirectPath = 'projects'

// 変数
export const state = () => ({
  styles: {
    homeAppBarHeight: 56,
  },
  // 追加
  loggedIn: {
    redirectPath: {
      name: redirectPath,
    },
  },
})

// 算出プロパティ
export const getters = {}

// stateの値を変更する場所
export const mutations = {}

// メソッド
export const actions = {}
```

- `front/pages/projects.vue`を編集<br>

```vue:projects.vue
<template>
  <div>
    projects.vue
  </div>
</template>

<script>
export default {
  layoute: 'logged-in',
}
</script>
```

- http://localhost:8080/projects にアクセスしてみる<br>

* `root $ docker compose run --rm front yarn lint --fix`を実行すると ESLint の警告を自動で直してくれたりする<br>

- `front $ mv components/App/AppLoginButton.vue components/BeforeLogin/BeforLoginAppBarLoginButton.vue`を実行<br>

- `front $ mv components/App/AppSignupButton.vue components/BeforeLogin/BeforLoginAppBarSignupButton.vue`を実行<br>

* `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar
    app
    :dark="!isScrollPoint"
    :height="homeAppBarHeight"
    :color="toolbarStyle.color"
    :elevation="toolbarStyle.elevation"
  >
    <app-logo @click.native="$vuetify.goTo('#scroll-top')" />

    <app-title class="hidden-mobile-and-down" />

    <v-spacer />

    <v-toolbar-items class="ml-2 hidden-ipad-and-down">
      <v-btn
        v-for="(menu, i) in menus"
        :key="`menu-btn-${i}`"
        :class="{ 'hidden-sm-and-down': menu.title === 'about' }"
        text
        @click="$vuetify.goTo(`#${menu.title}`)"
      >
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>

    <!-- 編集 -->
    <befor-login-app-bar-signup-button />
    <befor-login-app-bar-login-button />

    <v-menu bottom nudge-left="110" nudge-width="100">
      <template #activator="{ on }">
        <v-app-bar-nav-icon class="hidden-ipad-and-up" v-on="on" />
      </template>
      <v-list dense class="hidden-ipad-and-up">
        <v-list-item
          v-for="(menu, i) in menus"
          :key="`menu-list-${i}`"
          exact
          @click="$vuetify.goTo(`#${menu.title}`)"
        >
          <v-list-item-title>
            {{ $t(`menus.${menu.title}`) }}
          </v-list-item-title>
        </v-list-item>
      </v-list>
    </v-menu>
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
import AppTitle from '../App/AppTitle.vue'
<!-- 編集 -->
import BeforLoginAppBarLoginButton from '../BeforeLogin/BeforLoginAppBarLoginButton.vue'
import BeforLoginAppBarSignupButton from '../BeforeLogin/BeforLoginAppBarSignupButton.vue'

export default {
  <!-- 編集 -->
  components: {
    AppLogo,
    AppTitle,
    BeforLoginAppBarSignupButton,
    BeforLoginAppBarLoginButton,
  },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
    imgHeight: {
      type: Number,
      default: 0,
    },
  },
  data({ $store }) {
    return {
      scrollY: 0,
      homeAppBarHeight: $store.state.styles.homeAppBarHeight,
    }
  },
  computed: {
    isScrollPoint() {
      // 500 - 56 = 444px超の場合、trueを返す
      return this.scrollY > this.imgHeight - this.homeAppBarHeight
    },
    toolbarStyle() {
      const color = this.isScrollPoint ? 'white' : 'transparent'
      const elevation = this.isScrollPoint ? 4 : 0
      return { color, elevation }
    },
  },
  // Vue.new() => Vueインスタンス
  // マウント => Vueの実行準備が完全に整った後
  mounted() {
    window.addEventListener('scroll', this.onScroll)
  },
  // Vueインスタンスが破壊される前に実行される
  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
  },
  methods: {
    onScroll() {
      this.scrollY = window.scrollY
    },
  },
}
</script>
```

- `front/components/BeforeLogin/BeforeLoginAppBar.vue`を編集<br>

```vue:BeforeLoginAppBar.vue
<template>
  <v-app-bar app :height="homeAppBarHeight" color="white">
    <nuxt-link to="/" class="text-decoration-none">
      <app-logo />
    </nuxt-link>

    <app-title class="hidden-mobile-and-down" />

    <v-spacer />

    <!-- 編集 -->
    <befor-login-app-bar-signup-button />
    <befor-login-app-bar-login-button />
  </v-app-bar>
</template>

<script>
<!-- 編集 -->
import AppLogo from '../App/AppLogo.vue'
import AppTitle from '../App/AppTitle.vue'
import BeforLoginAppBarLoginButton from './BeforLoginAppBarLoginButton.vue'
import BeforLoginAppBarSignupButton from './BeforLoginAppBarSignupButton.vue'

export default {
  <!-- 編集 -->
  components: {
    BeforLoginAppBarSignupButton,
    BeforLoginAppBarLoginButton,
    AppLogo,
    AppTitle,
  },

  data({ $store }) {
    return {
      homeAppBarHeight: $store.state.styles.homeAppBarHeight,
    }
  },
}
</script>
```

## 61 ログイン後のツールバーを作成する 〜inject 編〜

- `front/locales/ja.json`を編集<br>

```json:ja.json
{
  "menus": {
    "about": "サイトについて",
    "products": "製品",
    "price": "価格",
    "contact": "お問い合わせ",
    "company": "会社情報",
    "payments": {
      "month": "月払い",
      "year": "年払い"
    }
  },
  "pages": {
    "signup": "会員登録",
    "login": "ログイン",
    "logout": "ログアウト",
    "account": {
      "settings": "アカウント設定",
      "password": "パスワード変更"
    }
  }
}
```

- `root $ touch front/plugins/my-inject.js`を実行<br>

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
  // 編集
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
    // 環境変数API_URLが優先される
    // baseURL: '/'
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

- `front/plugins/my-inject.js`を編集<br>

```js:my-inject.js
class MyInject {
  // Nuxtのcontextを使用するには、constructor内で初期化する
  constructor(ctx) {
    // ctx => { app }
    this.app = ctx.app
  }

  pageTitle(routeName) {
    // jsonPath => 'account-settings'
    const jsonPath = `pages.${routeName.replace(/-/g, '.')}`
    const title = this.app.i18n.t(jsonPath)
    return title
  }
}

// inject => オリジナルクラスを追加することができる
// export default (context, inject)
export default ({ app }, inject) => {
  // inject('呼び出し名', クラスのインスタンス(context))
  // 'my' => $my
  inject('my', new MyInject({ app }))
}
```

- `front/components/LoggedIn/LoggedInAppBarAccountMenu.vue`を編集<br>

```vue:LoggedInAppBarAccountMenu.vue
<template>
  <v-menu app offset-x offset-y max-width="200">
    <template #activator="{ on }">
      <v-btn icon v-on="on">
        <v-icon>mdi-account-circle</v-icon>
      </v-btn>
    </template>
    <v-list dense>
      <v-subheader>ログイン中のユーザー</v-subheader>

      <v-list-item>
        <v-list-item-content>
          <v-list-item-subtitle>
            <!-- TODO -->
            ユーザー名が表示されます
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-divider />

      <v-subheader>アカウント</v-subheader>

      <template v-for="(menu, i) in menus">
        <!-- <hr>タグになる -->
        <v-divider v-if="menu.divider" :key="`menu-divider-${i}`" />

        <v-list-item :key="`menu-list-${i}`" :to="{ name: menu.name }">
          <v-list-item-icon class="mr-2">
            <v-icon size="22" v-text="menu.icon" />
          </v-list-item-icon>
          <v-list-item-title>
            <!-- pages.account.settings -->
            {{ $my.pageTitle(menu.name) }}
          </v-list-item-title>
        </v-list-item>
      </template>
    </v-list>
  </v-menu>
</template>

<script>
export default {
  data() {
    return {
      menus: [
        { name: 'account-settings', icon: 'mdi-account-cog' },
        { name: 'account-password', icon: 'mdi-lock-outline' },
        { name: 'logout', icon: 'mdi-logout-variant', divider: true },
      ],
    }
  },
}
</script>
```

- `front/components/BeforeLogin/BeforeLoginAppBarLoginButton.vue`を編集<br>

```vue:BeforeLoginAppBarLoginButton.vue
<template>
  <v-btn text class="ml-2 font-weight-bold" color="black" to="/login">
    <!-- 編集 -->
    {{ $my.pageTitle('login') }}
  </v-btn>
</template>

<script>
export default {}
</script>
```

- `front/components/BeforeLogin/BeforeLoginAppBarSignupButton.vue`を編集<br>

```vue:BeforeLoginAppBarSignupButton.vue
<template>
  <v-btn outlined color="appblue" class="ml-2 font-weight-bold" to="/signup">
    <!-- 編集 -->
    {{ $my.pageTitle('signup') }}
  </v-btn>
</template>

<script>
export default {}
</script>
```

- `front/components/User/UserFormCard.vue`を編集<br>

```vue:UserFormCard.vue
<template>
  <v-container fluid>
    <v-row align="center" justify="center">
      <v-col cols="12" class="my-8 text-center">
        <h1 class="text-h5 font-weight-bold">{{ appName }}に{{ pageTitle }}</h1>
      </v-col>

      <v-card flat width="80%" max-width="320" color="transparent">
        <!-- コンテンツを差し込むスロット -->
        <slot name="user-form-card-content" />
      </v-card>
    </v-row>
  </v-container>
</template>

<script>
export default {
  <!-- 編集 -->
  data({ $route, $config: { appName }, $my }) {
    return {
      appName,
      // $route.name => /signup = name: signup
      // $route.name => /account/settings = name: account-settings
      // `pages.${$route.name}` => pages.signup
      <!-- 編集 -->
      pageTitle: $my.pageTitle($route.name),
    }
  },
}
</script>
```
