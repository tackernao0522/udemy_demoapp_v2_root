## 54 ログイン前のレイアアウトファイルを作成する

- heroku に環境変数を渡す<br>

* `front $ heroku config:set APP_NAME=BizPlanner`を実行<br>

* `front $ heroku config`を実行して確認<br>

```:terminal
=== tk-railsnuxtv1-front Config Vars
APP_NAME: BizPlanner
NODE_ENV: production
```

- `root $ touch front/layouts/before-login.vue`を実行<br>

* `front/layouts/before-login.vue`を編集<br>

```vue:before-login.vue
<template>
  <div>
    before-login.vue
    <Nuxt />
  </div>
</template>

<script>
export default {}
</script>
```

- `root $ touch front/pages/{signup.vue,login.vue}`を実行<br>

* `front/pages/login.vue`を編集<br>

```vue:login.vue
<template>
  <div>login.vue</div>
</template>

<script>
export default {
  layout: 'before-login',
}
</script>
```

- `front/pages/signup.vue`を編集<br>

```vue:signup.vue
<template>
  <div>signup.vue</div>
</template>

<script>
export default {
  layout: 'before-login',
}
</script>
```

- `root $ touch front/components/App/AppTitle.vue`を実行<br>

* `front/components/App/AppTitle.vue`を編集<br>

```vue:AppTitle.vue
<template>
  <v-toolbar-title>
    {{ appName }}
  </v-toolbar-title>
</template>

<script>
export default {
  data({ $config: { appName } }) {
    return {
      appName,
    }
  },
}
</script>
```

- `front/components/Home/HomeAppBar.vue`を編集<br>

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

    <!-- 編集 -->
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

    <app-signup-button />
    <app-login-button />

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
import AppLoginButton from '../App/AppLoginButton.vue'
import AppLogo from '../App/AppLogo.vue'
import AppSignupButton from '../App/AppSignupButton.vue'
import AppTitle from '../App/AppTitle.vue'
export default {
  components: { AppLogo, AppSignupButton, AppLoginButton, AppTitle },
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
  <!-- 編集 -->
  data({ $store }) {
    return {
      scrollY: 0,
      homeAppBarHeight: $store.state.styles.homeAppBarHeight,
    }
  },
  <!-- ここまで -->
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

- `root $ mkdir front/components/BeforeLogin && touch $_/BeforeLoginAppBar.vue`を実行<br>

* `front layouts/before-login.vue`を編集<br>

```vue:before-login.vue
<template>
  <v-app>
    <before-login-app-bar />
    <v-main>
      <Nuxt />
    </v-main>
    <app-footer />
  </v-app>
</template>

<script>
import AppFooter from '../components/App/AppFooter.vue'
import BeforeLoginAppBar from '../components/BeforeLogin/BeforeLoginAppBar.vue'
export default {
  components: { BeforeLoginAppBar, AppFooter },
}
</script>
```

## 55 会員登録フォームを作成する

- `root $ mkdir front/components/User && touch $_/{UserFormCard.vue,UserFormName.vue,UserFormEmail.vue,UserFormPassword.vue}`を実行<br>

* `front/components/User/UserFormCard.vue`を編集<br>

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
  data({ $route, $config: { appName } }) {
    return {
      appName,
      // $route.name => /signup = name: signup
      // $route.name => /account/settings = name: account-settings
      // `pages.${$route.name}` => pages.signup
      pageTitle: this.$t(`pages.${$route.name}`),
    }
  },
}
</script>
```

- `front components/User/UserFormName.vue`を編集<br>

```vue:UserFormName.vue
<template>
  <v-text-field
    label="ユーザー名を入力"
    placeholder="あなたの表示名"
    outlined
  />
</template>

<script>
export default {}
</script>
```

- `front/components/User/UserFormEmail.vue`を編集<br>

```vue:UserFormEmail.vue
<template>
  <v-text-field
    label="メールアドレスを入力"
    placeholder="your@email.com"
    outlined
  />
</template>

<script>
export default {}
</script>
```

- `front/components/User/UserFormPassword.vue`を編集<br>

```vue:UserFormPassword.vue
<template>
  <v-text-field label="パスワードを入力" placeholder="8文字以上" outlined />
</template>

<script>
export default {}
</script>
```

- `front/pages/signup.vue`を編集<br>

```vue:signup.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <user-form-name />
      <user-form-email />
      <user-form-password />
    </template>
  </user-form-card>
</template>

<script>
import UserFormCard from '../components/User/UserFormCard.vue'
import UserFormEmail from '../components/User/UserFormEmail.vue'
import UserFormName from '../components/User/UserFormName.vue'
import UserFormPassword from '../components/User/UserFormPassword.vue'

export default {
  components: { UserFormCard, UserFormName, UserFormEmail, UserFormPassword },
  layout: 'before-login',
}
</script>
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
  plugins: ['plugins/axios'],

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
    // 追加
    // no_prefix => ルート名に__jaを追加しない
    strategy: 'no_prefix',
    // Doc: https://kazupon.github.io/vue-i18n/api/#properties
    // ここまで
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

- `front/pages/signup.vue`を編集<br>

```vue:signup.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <!-- 編集 -->
      <v-form v-model="isValid">
        <user-form-name />
        <user-form-email />
        <user-form-password />
        <!-- disabled=true => ボタンクリックを無効にする -->
        <v-btn :disabled="!isValid" block color="appblue" class="white--text">
          登録する
        </v-btn>
      </v-form>
      <!-- ここまで -->
    </template>
  </user-form-card>
</template>

<script>
import UserFormCard from '../components/User/UserFormCard.vue'
import UserFormEmail from '../components/User/UserFormEmail.vue'
import UserFormName from '../components/User/UserFormName.vue'
import UserFormPassword from '../components/User/UserFormPassword.vue'

export default {
  components: { UserFormCard, UserFormName, UserFormEmail, UserFormPassword },
  layout: 'before-login',
  <!-- 追加 -->
  data() {
    return {
      isValid: false,
    }
  },
  <!-- ここまで -->
}
</script>
```
