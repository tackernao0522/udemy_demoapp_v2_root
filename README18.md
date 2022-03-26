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
