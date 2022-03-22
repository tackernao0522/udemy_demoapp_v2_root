## 53 [ウェルカムページ 4/4] レスポンシブデザインに対応させる

- `root $ touch front/components/App/{AppLoginButton.vue,AppSignupButton.vue}`を実行<br>

- `front/components/App/AppLoginButton.vue`を編集<br>

```vue:AppLoginButton.vue
<template>
  <v-btn text class="ml-2 font-weight-bold" color="black" to="/login">
    {{ $t('pages.login') }}
  </v-btn>
</template>

<script>
export default {}
</script>
```

- `front/components/App/AppSignupButton.vue`を編集<br>

```vue:AppSignupButton.vue
<template>
  <v-btn text color="appblue" class="ml-2 font-weight-bold" to="/signup">
    {{ $t('pages.signup') }}
  </v-btn>
</template>

<script>
export default {}
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
          // 追加
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
  <v-app-bar
    app
    :dark="!isScrollPoint"
    :height="homeAppBarHeight"
    :color="toolbarStyle.color"
    :elevation="toolbarStyle.elevation"
  >
    <app-logo @click.native="$vuetify.goTo('#scroll-top')" />
    <v-toolbar-title />
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn
        v-for="(menu, i) in menus"
        :key="`menu-btn-${i}`"
        text
        @click="$vuetify.goTo(`#${menu.title}`)"
      >
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
    <!-- 追加 -->
    <app-signup-button />
    <app-login-button />
    <!-- ここまで -->
  </v-app-bar>
</template>

<script>
import AppLoginButton from '../App/AppLoginButton.vue'
import AppLogo from '../App/AppLogo.vue'
import AppSignupButton from '../App/AppSignupButton.vue'
export default {
  components: { AppLogo, AppSignupButton, AppLoginButton },
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
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName }, $store }) {
    return {
      appName,
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

- ブラウザで確認してみる<br>

* `front/components/App/AppSignupButton.vue`を編集<br>

```vue:AppSignupButton.vue
<template>
  <!-- 編集 -->
  <v-btn outlined color="appblue" class="ml-2 font-weight-bold" to="/signup">
    {{ $t('pages.signup') }}
  </v-btn>
</template>

<script>
export default {}
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
    <v-toolbar-title class="hidden-mobile-and-down">
      {{ appName }}
    </v-toolbar-title>
    <v-spacer />

    <v-toolbar-items class="ml-2 hidden-ipad-and-down">
      <!-- 編集 -->
      <v-btn
        v-for="(menu, i) in menus"
        :key="`menu-btn-${i}`"
        text
        @click="$vuetify.goTo(`#${menu.title}`)"
      >
        <!-- ここまで -->
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>

    <app-signup-button />
    <app-login-button />

    <!-- 追加 -->
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
    <!-- ここまで -->
  </v-app-bar>
</template>

<script>
import AppLoginButton from '../App/AppLoginButton.vue'
import AppLogo from '../App/AppLogo.vue'
import AppSignupButton from '../App/AppSignupButton.vue'
export default {
  components: { AppLogo, AppSignupButton, AppLoginButton },
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
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName }, $store }) {
    return {
      appName,
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
    <v-toolbar-title class="hidden-mobile-and-down">
      {{ appName }}
    </v-toolbar-title>
    <v-spacer />

    <v-toolbar-items class="ml-2 hidden-ipad-and-down">
      <v-btn v-for="(menu, i) in menus" :key="`menu-btn-${i}`" <!-- 追加 -->
        :class="{ 'hidden-sm-and-down': menu.title === 'about' }" text
        @click="$vuetify.goTo(`#${menu.title}`)" >
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
export default {
  components: { AppLogo, AppSignupButton, AppLoginButton },
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
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName }, $store }) {
    return {
      appName,
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

- `front/components/App/AppFooter.vue`を編集<br>

```vue:AppFooter.vue
<template>
  <div :style="{ marginTop: `${height}px` }">
    <v-footer absolute dark color="black" :height="height">
      <v-col cols="12" class="py-0">
        <div class="text-center text-body-2">
          &copy;{{ copyRightYear }}
          <strong>{{ appName }}</strong>
        </div>
      </v-col>
    </v-footer>
  </div>
</template>

<script>
export default {
  data({ $config: { appName } }) {
    return {
      appName,
      height: 32,
    }
  },
  computed: {
    copyRightYear() {
      const beginningYear = 2022
      const thisYear = new Date().getFullYear()
      // 2022 < 2022
      return beginningYear < thisYear
        ? `${beginningYear} - ${thisYear}`
        : beginningYear
    },
  },
}
</script>
```

- `front/components/Home/HomeAbout.vue`を編集<br>

```vue:HomeAbout.vue
<template>
  <div>
    <v-card-title class="pb-8 text-subtitle-2 justify-center">
      このアプリケーションの作り方は、下記URLに公開されています。
    </v-card-title>
    <v-card-title class="text-subtitle-2 justify-center">
      「Rails6とNuxt.jsで作るユーザーJWT認証付きシングルページアプリケーション」
    </v-card-title>
    <v-card-text class="text-center">
      <a
        :href="blogUrl"
        rel="nofollow"
        target="_blank"
        class="text-decoration-none"
      >
        {{ blogUrl }}
      </a>
    </v-card-text>
    <v-card-title class="text-subtitle-2 justify-center">
      採用している技術
    </v-card-title>
    <v-container fluid :style="{ maxWidth: '960px' }">
      <v-row justify="space-around">
        <div
          v-for="(tec, i) in technologies"
          :key="`tec-${i}`"
          class="text-center pa-2"
        >
          <v-avatar :color="tec.color" size="80">
            <span class="white--text">
              {{ tec.name }}
            </span>
          </v-avatar>
          <div class="pt-2 text-body-2 my-grey">
            {{ tec.use }}
          </div>
          <div class="pt-2 text-body-2 my-grey">
            {{ tec.v }}
          </div>
        </div>
      </v-row>
    </v-container>
  </div>
</template>

<script>
export default {
  data() {
    return {
      blogUrl: 'https://blog.cloud-acct.com/categories/udemy',
      technologies: [
        { name: 'Docker', v: 'v19.03+', use: '開発環境', color: '#2496ED' },
        {
          name: 'Rails api',
          v: 'v6.0+',
          use: 'サーバーサイド',
          color: '#CC0000',
        },
        { name: 'Postgre SQL', v: '', use: 'データベース', color: '#336791' },
        {
          name: 'Nuxt.js spa',
          v: 'v2.13+',
          use: 'フロントエンド',
          color: '#00C48D',
        },
        { name: 'Heroku', v: '', use: 'ホスティング', color: '#6762A6' },
        {
          name: 'Vuetify',
          v: 'v2.3+',
          use: 'CSSフレームワーク',
          color: '#1867C0',
        },
      ],
    }
  },
}
</script>
```

- `front/components/Home/HomeProducts.vue`を編集<br>

```vue:HomeProducts.vue
<template>
  <v-row>
    <v-col cols="12" sm="6">
      <v-list flat>
        <v-list-item v-for="(point, i) in points" :key="`point-${i}`">
          <v-list-item-icon>
            <v-icon size="30" :color="point.color" v-text="point.icon" />
          </v-list-item-icon>
          <v-list-item-content>
            <div class="text-subtitle-1" v-text="point.text" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-col>
    <v-col cols="12" sm="6">
      <v-sparkline
        :value="sparkline.value"
        :gradient="sparkline.gradient"
        :smooth="sparkline.radius || false"
        :padding="sparkline.padding"
        :line-width="sparkline.width"
        :stroke-linecap="sparkline.lineCap"
        :gradient-direction="sparkline.gradientDirection"
        :fill="sparkline.fill"
        :type="sparkline.type"
        :auto-line-width="sparkline.autoLineWidth"
        auto-draw
      />
    </v-col>
  </v-row>
</template>

<script>
export default {
  data() {
    const gradients = [
      ['#222'],
      ['#42b3f4'],
      ['red', 'orange', 'yellow'],
      ['purple', 'violet'],
      ['#00c6ff', '#F0F', '#FF0'],
    ]
    return {
      points: [
        {
          icon: 'mdi-file-table-box-multiple-outline',
          color: 'blue',
          text: '直感的な操作で快適に事業計画書を作成',
        },
        {
          icon: 'mdi-chart-bar',
          color: 'green accent-4',
          text: 'KPIから考えた夢物語ではない経営計画値を算出',
        },
        {
          icon: 'mdi-chart-arc',
          color: 'deep-orange',
          text: 'ビジュアライズに優れたグラフツールで経営を可視化',
        },
      ],
      sparkline: {
        width: 4,
        radius: 10,
        padding: 4,
        lineCap: 'round',
        gradient: gradients[4],
        value: [0, 2, 5, 9, 5, 10, 8, 2, 9, 20],
        gradientDirection: 'right',
        gradients,
        fill: false,
        type: 'trend',
        autoLineWidth: true,
      },
    }
  },
}
</script>
```

- `front/components/Home/HomePrice.vue`を編集<br>

```vue:HomePrice.vue
<template>
  <v-row>
    <v-col cols="12" class="py-0">
      <v-card-actions class="py-0">
        <v-spacer />
        <v-radio-group v-model="payment" row>
          <v-radio
            v-for="(pay, i) in payments"
            :key="`pay-${i}`"
            :label="$t(`menus.payments.${pay.label}`)"
            :value="pay.label"
            :color="pay.color"
          />
        </v-radio-group>
      </v-card-actions>
    </v-col>
    <v-col
      v-for="(plan, i) in plans"
      :key="`plan-${i}`"
      cols="12"
      :sm="12 / plans.length"
    >
      <v-card max-width="402" class="mx-auto">
        <v-card-title :class="['white--text', plan.color]">
          {{ plan.name }}
        </v-card-title>

        <v-card-title class="justify-center">
          {{ plan.exp }}
        </v-card-title>

        <v-divider />

        <v-card-title class="justify-center">
          メンバー
          {{ plan.member }}
        </v-card-title>

        <v-divider />

        <v-card-actions class="justify-center align-baseline">
          月
          <span class="px-2 display-1 font-weight-bold">
            {{ yen(plan.price[payment]) }}
          </span>
          円
        </v-card-actions>
        <v-card-actions class="justify-center align-baseline">
          年間 {{ yen(plan.price[payment] * 12) }} 円
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  data() {
    const payments = [
      { label: 'month', color: 'indigo' },
      { label: 'year', color: 'myblue' },
    ]
    return {
      payments,
      payment: payments[1].label,
      plans: [
        {
          name: 'Only',
          color: 'info',
          exp: '経営者1人のためのプラン',
          member: '1人',
          price: {
            month: 1200,
            year: 800,
          },
        },
        {
          name: 'Small',
          color: 'primary',
          exp: '小規模事業に特化した1チーム専用',
          member: '3人まで',
          price: {
            month: 2400,
            year: 1800,
          },
        },
        {
          name: 'Business',
          color: 'indigo',
          exp: '大規模なチームに戦略的経営を導入',
          member: '10人まで',
          price: {
            month: 5000,
            year: 4000,
          },
        },
      ],
    }
  },
  computed: {
    yen() {
      return (val) => {
        return String(val).replace(/(\d)(?=(\d\d\d)+(?!\d))/g, '$1,')
      }
    },
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
    "company": "会社情報",
    // 追加
    "payments": {
      "month": "月払い",
      "year": "年払い"
    }
    // ここまで
  },
  "pages": {
    "signup": "会員登録",
    "login": "ログイン"
  }
}
```

- `front/components/Home/HomeContact.vue`を編集<br>

```vue:HomeContact.vue
<template>
  <v-row justify="center">
    <v-col cols="12" sm="10" md="8">
      <v-form ref="contact" v-model="isValid">
        <v-container>
          <v-row>
            <v-col cols="12" sm="6">
              <v-text-field
                v-model="name"
                :rules="nameRules"
                :disabled="sentIt"
                label="名前(必須)"
                outlined
              />
            </v-col>
            <v-col cols="12" sm="6">
              <v-text-field
                v-model="email"
                :rules="emailRules"
                :disabled="sentIt"
                label="メールアドレス(必須)"
                outlined
                validate-on-blur
              />
            </v-col>
          </v-row>

          <v-textarea
            v-model="contents"
            :rules="contentRules"
            :disabled="sentIt"
            label="お問合せの内容をお聞かせください(必須)"
            rows="5"
            outlined
            auto-grow
          />

          <v-btn
            :disabled="!isValid || loading || sentIt"
            :loading="loading"
            color="primary"
            class="mr-2"
            @click="onSend"
          >
            送信する
          </v-btn>

          <v-btn text @click="formReset">
            キャンセル
          </v-btn>
          <div class="grey--text">
            <small>実際には送信されません</small>
          </div>
        </v-container>
      </v-form>
    </v-col>
    <v-snackbar v-model="sentIt" timeout="-1" color="primary">
      お問合せ内容が送信されました。メールアドレスへ担当者よりご連絡いたします。
      <template #action="{ attrs }">
        <v-btn color="white" text v-bind="attrs" @click="formReset">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-row>
</template>

<script>
export default {
  data() {
    return {
      isValid: false,
      name: '',
      nameRules: [(v) => !!v || '名前を入力してください'],
      email: '',
      emailRules: [
        (v) => !!v || 'メールアドレスを入力してください',
        (v) => /.+@.+\..+/.test(v) || 'メールアドレスが正しくありません',
      ],
      contents: '',
      contentRules: [(v) => !!v || 'お問合せ内容を入力してください'],
      loading: false,
      sentIt: false,
    }
  },
  methods: {
    onSend() {
      this.loading = true
      setTimeout(() => {
        this.loading = false
        this.sentIt = true
      }, 1500)
    },
    formReset() {
      this.sentIt = false
      this.$refs.contact.reset()
    },
  },
}
</script>
```

- `front/assets/images`ディレクトリを作成<br>

- `front/assets/images`ディレクトリに`member1.png, member2.png, membere.png`ファイルを配置<br>

- `front/components/Home/HomeCompany.vue`を編集<br>

```vue:HomeCompany.vue
<template>
  <div>
    <v-card-title class="font-weight-bold justify-center">
      メンバー
    </v-card-title>
    <v-row justify="space-around">
      <v-col v-for="(member, i) in members" :key="`member-${i}`">
        <v-list-item>
          <v-list-item-icon>
            <img
              :src="member.img"
              :alt="member.nickname"
              width="50"
              height="50"
            />
          </v-list-item-icon>
          <v-list-item-content>
            <div>
              {{ member.name }}
            </div>
            <v-list-item-action-text>
              {{ member.position }}
            </v-list-item-action-text>
            <v-list-item-action-text>
              <v-btn
                v-if="member.twitter"
                :href="member.twitter"
                target="_blank"
                rel="noopener noreferrer"
                small
                icon
              >
                <v-icon size="18">
                  mdi-twitter
                </v-icon>
              </v-btn>
              <v-btn
                v-if="member.slack"
                :href="member.slack"
                target="_blank"
                rel="noopener noreferrer"
                small
                icon
              >
                <v-icon size="18">
                  mdi-slack
                </v-icon>
              </v-btn>
            </v-list-item-action-text>
          </v-list-item-content>
        </v-list-item>
      </v-col>
    </v-row>
    <v-card-title class="font-weight-bold justify-center">
      会社情報
    </v-card-title>
    <v-row justify="center">
      <v-col cols="12" sm="10" md="8">
        <v-list flat dense>
          <v-divider />
          <template v-for="(info, i) in infomations">
            <v-list-item :key="`info-list-${i}`">
              <v-list-item-icon>
                <v-icon v-text="info.icon" />
              </v-list-item-icon>
              <v-list-item-content>
                <a
                  v-if="info.link"
                  :href="info.link"
                  rel="nofollow"
                  target="_blank"
                  class="text-decoration-none"
                >
                  {{ info.link }}
                </a>
                <div v-else class="text-subtitle-1" v-text="info.text" />
              </v-list-item-content>
            </v-list-item>
            <v-divider :key="`info-divider-${i}`" />
          </template>
        </v-list>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import member1 from '~/assets/images/member1.png'
import member2 from '~/assets/images/member2.png'
import member3 from '~/assets/images/member3.png'

export default {
  data() {
    const twitter = 'https://twitter.com/esegrammer'
    const slack =
      'https://join.slack.com/t/dokugaku-kai/shared_invite/zt-a5j1suoh-Y0fspHbo1fb0Wj6YTpDdXA'
    const companyUrl = 'http://blog.cloud-acct.com'
    return {
      members: [
        { name: 'あんどう', position: '代表', img: member1, twitter, slack },
        { name: 'アローン', position: 'エンジニア', img: member2 },
        { name: 'カール', position: 'エンジニア', img: member3 },
      ],
      infomations: [
        { icon: 'mdi-domain', text: 'BizPlanner株式会社' },
        { icon: 'mdi-link-variant', link: companyUrl },
        { icon: 'mdi-flag', text: '2020年7月に設立' },
        { icon: 'mdi-account-multiple', text: '3人のメンバー' },
        { icon: 'mdi-map-marker', text: '東京都港区虎ノ門一丁目17番1号' },
        { icon: 'mdi-handshake', text: 'Webアプリ開発・経営コンサルティング' },
      ],
    }
  },
}
</script>
```
