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

* `front/components/BeforeLogin/BeforeLoginAppBar.vue`を編集<br>

```vue:BeforeLoginAppBar.vue
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
