## 48 nuxt-i18n を導入する

- `root $ docker compose run --rm front yarn add @nuxtjs/i18n`を実行<br>

* `root $ docker compose run --rm front yarn list --pattern @nuxtjs/i18n`を実行して確認<br>

```:terminal
yarn list v1.22.15
└─ @nuxtjs/i18n@7.2.0
Done in 4.02s.
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
    // 追加
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
  // 追加
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

- `root $ mkdir front/locales && touch $_/{ja.json,en.json}`を実行<br>

- `front/locales/en.json`を編集<br>

```json:en.json
{}
```

- `front/locales/ja.json`を編集<br>

```json:ja.json
{
  "title": {
    "signup": "会員登録",
    "login": "ログイン"
  }
}
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-container fluid>
    <v-card flat tile color="transparent">
      <v-card-title>
        Usersテーブルの取得
      </v-card-title>
      <v-card-text>
        <v-simple-table dense>
          <template v-if="users.length" #default>
            <thead>
              <tr>
                <th v-for="(key, i) in userKeys" :key="`key-${i}`">
                  {{ key }}
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(user, i) in users" :key="`user-${i}`">
                <td>{{ user.id }}</td>
                <td>{{ user.name }}</td>
                <td>{{ user.email }}</td>
                <td>{{ dateFormat(user.created_at) }}</td>
              </tr>
            </tbody>
          </template>
          <template v-else>
            ユーザーが存在しません
          </template>
        </v-simple-table>
      </v-card-text>
      <v-card-title>
        Vuetifyの導入（オリジナルカラーの確認）
      </v-card-title>
      <v-card-text>
        <v-btn
          v-for="(color, i) in colors"
          :key="`color-${i}`"
          :color="color"
          class="mr-2"
        >
          {{ color }}
        </v-btn>
      </v-card-text>

      <v-card-title>
        VuetifyカスタムCSSの検証
      </v-card-title>
      <v-card-text>
        ipad（768px）とmobile（426px）で表示・非表示
      </v-card-text>
      <v-card-text>
        <v-card
          v-for="(cls, i) in customClass"
          :key="`cls-${i}`"
          :color="cls.color"
          :class="cls.name"
        >
          <v-card-text>
            {{ cls.des }}
          </v-card-text>
        </v-card>
      </v-card-text>
    </v-card>

    <!-- 追加 -->
    <v-card-title>
      nuxt-i18nの検証
    </v-card-title>
    <v-card-text>
      <v-simple-table dense>
        <template #default>
          <thead>
            <tr>
              <th>en</th>
              <th>ja</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(path, i) in ['signup', 'login']" :key="`path-${i}`">
              <td>{{ path }}</td>
              <td>{{ $t(`title.${path}`) }}</td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>
    </v-card-text>
    <!-- ここまで -->
  </v-container>
</template>

<script>
export default {
  async asyncData({ $axios }) {
    let users = []
    await $axios.$get('/api/v1/users').then((res) => (users = res))
    const userKeys = Object.keys(users[0] || {}) // 追加
    return { users, userKeys }
  },
  // data () 追加
  data() {
    return {
      colors: ['primary', 'info', 'success', 'warning', 'error', 'background'],
      customClass: [
        { name: 'hidden-ipad-and-down', color: 'error', des: 'ipad未満で隠す' },
        { name: 'hidden-ipad-and-up', color: 'info', des: 'ipad以上で隠す' },
        {
          name: 'hidden-mobile-and-down',
          color: 'success',
          des: 'mobile未満で隠す',
        },
        {
          name: 'hidden-mobile-and-up',
          color: 'warning',
          des: 'mobile以上で隠す',
        },
      ],
    }
  },
  computed: {
    dateFormat() {
      return (date) => {
        const dateTimeFormat = new Intl.DateTimeFormat('ja', {
          dateStyle: 'medium',
          timeStyle: 'short',
        })
        return dateTimeFormat.format(new Date(date))
      }
    },
  },
}
</script>
```

# セクション 10: ログイン前のレイアウト構築

## 49 Nuxt.js のレイアウト・ページ・コンポーネントの役割を理解する

### 作成するページ

1. ウェルカムページ<br>
2. 会員登録ページ<br>
3. ログインページ<br>

### レイアウトからページの呼び出し

```html:default.vue
<!-- layoutd/default.vue -->
<template>
  <v-app>
    <header />
    <!-- 共通ヘッダー -->
    <Nuxt />
    <!-- ページファイルの呼び出し -->
    <footer />
    <!-- 共通フッター -->
  </v-app>
</template>
```

### ページからレイアウトを指定する

```js:index.js
// JavaScript
export default {
  layout() {
    return 'welcome'
  }

  // もしくは
  layout: 'welcome'

  // layoutの指定なし => default.vue
}
```

### ページとコンポーネントの違い

<データを非同期で取得する場合><br>

`page`<br>

```js:index.js
async asyncData ({ $axios }) {
  let users = []
  await $axios.$get('/api/v1/users')
    .then(res => (users = res))
  return { users }
}
```

`layout・component・page`<br>

```js:sample.js
async fetch () {
  await this.$axios.$get('/api/v1/users')
    .then(res => (this.users = res))
},
data () {
  return {
    users: []
  }
}
```

### コンポーネントファイルの命名規則

DOM テンプレート内に埋め込むコンポーネントファイル、<br>
components ディレクトリ内で管理する Vue ファイル<br>

|            |   ファイル名   |           DOM テンプレート            |
| :--------: | :------------: | :-----------------------------------: |
| PascasCase | AppButton.vue  | `<app-button>`<br>or<br>`<AppButton>` |
| kebab-case | app-button.vue |                 同上                  |

### 今回のファイル命名ルール

- composnents => PascalCase<br>

* その他 => kebab-case<br>

- DOM テンプレート => <kebab-case> <br>

### まとめ

1. 役割）レイアウト=土台、ページ=URL 表示、コンポーネント=部品<br>

2. 呼び出し）レイアウト => <Nuxt />, ページ => layout ()<br>

3. パス）ページファイルパスが URL になる<br>

4. 違い）ページ => asyncData の使用可, 他 => fetch<br>

5. 規則）コンポーネント名 => PascalCase<br>

## 49 [ウェルカムページ 1/4] コンポーネントファイル群を作成

- `root $ mkdir front/components && mkdir $_/Home && touch $_/{HomeAppBar.vue,HomeAbout.vue,HomeProducts.vue,HomePrice.vue,HomeConatact.vue,HomeCompany.vue}`を実行<br>

* `front/components/Home/HomeAbout.vue`を編集<br>

```vue:HomeAbout.vue
<template>
  <div>
    HomeAbout.vue
  </div>
</template>

<script>
export default {}
</script>
```

- `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <div>
    HomeAppBar.vue
  </div>
</template>

<script>
export default {}
</script>
```

- `front/components/Home/HomeCompany.vue`を編集<br>

```vue:HomeCompany.vue
<template>
  <div>
    HomeCompany.vue
  </div>
</template>

<script>
export default {}
</script>
```

- `front/components/Home/HomeContact.vue`を編集<br>

```vue:HomeContact.vue
<template>
  <div>
    HomeContact.vue
  </div>
</template>

<script>
export default {}
</script>
```

- `front/components/Home/HomePrice.vue`を編集<br>

```vue:HomePrice.vue
<template>
  <div>
    HomePrice.vue
  </div>
</template>

<script>
export default {}
</script>
```

- `front/components/Home/HomeProducts.vue`を編集<br>

```vue:HomeProducts.vue
<template>
  <div>
    HomeProducts.vue
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <home-app-bar />
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
  </v-app>
</template>

<script>
// isを使ったときはimport文が必要になる
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
  },
  data() {
    return {
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

- `root $ mkdir front/components/App && touch $_/AppFooter.vue`を実行<br>

- `front/componsnts/App/AppFooter.vue`を編集<br>

```vue:AppFooter.vue
<template>
  <div>
    <v-footer absolute dark color="black">
      AppFooter.vue
    </v-footer>
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <home-app-bar />
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

- 反映されない場合は `root $ docker compose restart front`を実行してみる<br>
