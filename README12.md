## 51 [ウェルカムページ 2/4] アイキャッチ画像・アプリ名・メニューを表示しよう

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <home-app-bar />
    <!-- 追加 -->
    <v-img
      dark
      src="https://picsum.photos/id/20/1920/1080?blur=5"
      gradient="to top right, rgba(19,84,122,.6), rgba(128,208,199,.9)"
      :height="imgHeight"
    >
      <v-row
        align="center"
        justify="center"
        :style="{ height: `${imgHeight}px` }"
      >
        <v-col cols="12" class="text-center">
          <h1 class="display-1 mb-4">
            未来を作ろう。ワクワクしよう。
          </h1>
          <h4 class="subheading" :style="{ letterSpacing: '5px' }">
            中小企業に特化した事業計画策定ツール
          </h4>
        </v-col>
      </v-row>
    </v-img>
    <!-- ここまで -->
    <v-sheet>
      <v-container fluid :style="{ maxWidth: '1280px' }">
        <v-row v-for="(menu, i) in menus" :key="`menu-${i}`">
          <v-col cols="12">
            <!-- home-about, home-company ... -->
            <div :is="`Home-${menu.title}`" />
          </v-col>
        </v-row>
      </v-container>
    </v-sheet>
    <app-footer />
  </v-app>
</template>

<script>
// isを使ったときはimport文が必要になる
import AppFooter from '../components/App/AppFooter.vue'
import HomeAbout from '~/components/Home/HomeAbout'
import HomeProducts from '~/components/Home/HomeProducts'
import HomePrice from '~/components/Home/HomePrice'
import HomeContact from '~/components/Home/HomeContact'
import HomeCompany from '~/components/Home/HomeCompany'

export default {
  components: {
    HomeAbout,
    HomeProducts,
    HomePrice,
    HomeContact,
    HomeCompany,
    AppFooter,
  },
  data() {
    return {
      imgHeight: 500, // 追加
      menus: [
        {
          title: 'about',
          subtitle:
            'このサイトはブログ"独学プログラマ"で公開されているチュートリアルのデモアプリケーションです',
        },
        { title: 'products', subtitle: '他にはない優れた機能の数々' },
        { title: 'price', subtitle: '会社の成長に合わせた3つのプラン' },
        { title: 'contact', subtitle: 'お気軽にご連絡を' },
        { title: 'company', subtitle: '私たちの会社' },
      ],
    }
  },
}
</script>
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <home-app-bar />
    <v-img
      dark
      src="https://picsum.photos/id/20/1920/1080?blur=5"
      gradient="to top right, rgba(19,84,122,.6), rgba(128,208,199,.9)"
      :height="imgHeight"
    >
      <v-row
        align="center"
        justify="center"
        :style="{ height: `${imgHeight}px` }"
      >
        <v-col cols="12" class="text-center">
          <h1 class="display-1 mb-4">
            未来を作ろう。ワクワクしよう。
          </h1>
          <h4 class="subheading" :style="{ letterSpacing: '5px' }">
            中小企業に特化した事業計画策定ツール
          </h4>
        </v-col>
      </v-row>
    </v-img>
    <v-sheet>
      <v-container fluid :style="{ maxWidth: '1280px' }">
        <v-row v-for="(menu, i) in menus" :key="`menu-${i}`">
          <!-- 追加 -->
          <v-col cols="12">
            <v-card flat>
              <v-card-title class="justify-center display-1">
                {{ $t(`menus.${menu.title}`) }}
              </v-card-title>
              <v-card-text class="text-center">
                {{ menu.subtitle }}
              </v-card-text>
            </v-card>
          </v-col>
          <!-- ここまで -->
          <v-col cols="12">
            <!-- home-about, home-company ... -->
            <div :is="`Home-${menu.title}`" />
          </v-col>
        </v-row>
      </v-container>
    </v-sheet>
    <app-footer />
  </v-app>
</template>

<script>
// isを使ったときはimport文が必要になる
import AppFooter from '../components/App/AppFooter.vue'
import HomeAbout from '~/components/Home/HomeAbout'
import HomeProducts from '~/components/Home/HomeProducts'
import HomePrice from '~/components/Home/HomePrice'
import HomeContact from '~/components/Home/HomeContact'
import HomeCompany from '~/components/Home/HomeCompany'

export default {
  components: {
    HomeAbout,
    HomeProducts,
    HomePrice,
    HomeContact,
    HomeCompany,
    AppFooter,
  },
  data() {
    return {
      imgHeight: 500,
      menus: [
        {
          title: 'about',
          subtitle:
            'このサイトはブログ"独学プログラマ"で公開されているチュートリアルのデモアプリケーションです',
        },
        { title: 'products', subtitle: '他にはない優れた機能の数々' },
        { title: 'price', subtitle: '会社の成長に合わせた3つのプラン' },
        { title: 'contact', subtitle: 'お気軽にご連絡を' },
        { title: 'company', subtitle: '私たちの会社' },
      ],
    }
  },
}
</script>
```

- `front/locales/ja.json`を編集<br>

```json:ja.json
{
  "menus": {
    "about": "サイトについて",
    "products": "製品",
    "price": "価格",
    "contact": "お問い合わせ",
    "company": "会社情報"
  },
  "pages": {
    "signup": "会員登録",
    "login": "ログイン"
  }
}
```

- 反映されない場合は `root $ docker compose restart front`を実行してみる<br>

* `root $ touch froun/.env`を実行<br>

```:.env
APP_NAME=BizPnanner
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

  // 追加
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

- `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar app dark>
    <v-toolbar-title>
      {{ appName }}
    </v-toolbar-title>
  </v-app-bar>
</template>

<script>
export default {
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName } }) {
    return {
      appName,
    }
  },
}
</script>
```

- `root $ touch front/components/App/AppLogo.vue`を実行<br>

```vue:AppLogo.vue
<template>
  <v-avatar color="black" size="34" class="my-app-log">
    <span class="white--text text-subtitle-2">Biz</span>
  </v-avatar>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
.my-app-log {
  margin-right: 8px;
  cursor: pointer;
}
</style>
```

- `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar app dark>
    <!-- 追加 -->
    <app-logo />
    <v-toolbar-title>
      {{ appName }}
    </v-toolbar-title>
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName } }) {
    return {
      appName,
    }
  },
}
</script>
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <!-- 編集 -->
    <home-app-bar :menus="menus" />
    <v-img
      dark
      src="https://picsum.photos/id/20/1920/1080?blur=5"
      gradient="to top right, rgba(19,84,122,.6), rgba(128,208,199,.9)"
      :height="imgHeight"
    >
      <v-row
        align="center"
        justify="center"
        :style="{ height: `${imgHeight}px` }"
      >
        <v-col cols="12" class="text-center">
          <h1 class="display-1 mb-4">
            未来を作ろう。ワクワクしよう。
          </h1>
          <h4 class="subheading" :style="{ letterSpacing: '5px' }">
            中小企業に特化した事業計画策定ツール
          </h4>
        </v-col>
      </v-row>
    </v-img>
    <v-sheet>
      <v-container fluid :style="{ maxWidth: '1280px' }">
        <v-row v-for="(menu, i) in menus" :key="`menu-${i}`">
          <v-col cols="12">
            <v-card flat>
              <v-card-title class="justify-center display-1">
                {{ $t(`menus.${menu.title}`) }}
              </v-card-title>
              <v-card-text class="text-center">
                {{ menu.subtitle }}
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12">
            <!-- home-about, home-company ... -->
            <div :is="`Home-${menu.title}`" />
          </v-col>
        </v-row>
      </v-container>
    </v-sheet>
    <app-footer />
  </v-app>
</template>

<script>
// isを使ったときはimport文が必要になる
import AppFooter from '../components/App/AppFooter.vue'
import HomeAbout from '~/components/Home/HomeAbout'
import HomeProducts from '~/components/Home/HomeProducts'
import HomePrice from '~/components/Home/HomePrice'
import HomeContact from '~/components/Home/HomeContact'
import HomeCompany from '~/components/Home/HomeCompany'

export default {
  components: {
    HomeAbout,
    HomeProducts,
    HomePrice,
    HomeContact,
    HomeCompany,
    AppFooter,
  },
  data() {
    return {
      imgHeight: 500,
      menus: [
        {
          title: 'about',
          subtitle:
            'このサイトはブログ"独学プログラマ"で公開されているチュートリアルのデモアプリケーションです',
        },
        { title: 'products', subtitle: '他にはない優れた機能の数々' },
        { title: 'price', subtitle: '会社の成長に合わせた3つのプラン' },
        { title: 'contact', subtitle: 'お気軽にご連絡を' },
        { title: 'company', subtitle: '私たちの会社' },
      ],
    }
  },
}
</script>
```

- `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar app dark>
    <app-logo />
    <v-toolbar-title />
    <!-- 追加 -->
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn v-for="(menu, i) in menus" :key="`menu-btn-${i}`" text>
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
    <!-- ここまで -->
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
  },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName } }) {
    return {
      appName,
    }
  },
}
</script>
```
