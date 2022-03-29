## 65 動的なルートに表示する個別プロジェクトページを作成する

### Vue.js で動的なルーティングを生成するには？

アンダーバーを付けた Vue ファイル、もしくはディレクトリを用意する<br>

`_id.vue` or `_id`<br>

名前は params のキーと一致させる<br>
`params: { id: 1 }<br>

### ハンズオン

- `root $ mkdir -p front/pages/project/_id && touch $_/{dashboard.vue,layouts.vue,pages.vue,components.vue,settings.vue,help.vue}`を実行<br>

- `root $ tree front/pages/project`を実行して確認<br>

```:terminal
front/pages/project
└── _id
    ├── components.vue
    ├── dashboard.vue
    ├── help.vue
    ├── layouts.vue
    ├── pages.vue
    └── settings.vue
```

- `root $ touch front/layouts/project.vue`を実行<br>

- `root $ touch front/pages/project.vue`を実行<br>

* `front/layouts/project.vue`を編集<br>

```vue:project.vue
<template>
  <v-app>
    <logged-in-app-bar />
    <v-main>
      project.vue
      <Nuxt />
    </v-main>
  </v-app>
</template>

<script>
import LoggedInAppBar from '../components/LoggedIn/LoggedInAppBar.vue'
export default {
  components: { LoggedInAppBar },
}
</script>
```

- `front/pages/project.vue`を編集<br>

```vue:project.vue
<template>
  <v-container>
    <nuxt-child />
  </v-container>
</template>

<script>
export default {
  latyout: 'project',
  validate({ route }) {
    return route.name !== 'project'
  },
}
</script>
```

- `front/pages/project/_id/components.vue`を編集<br>

```vue:components.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project_id/dashboard.vue`を編集<br>

```vue:dashboard.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/help.vue`を編集<br>

```vue:help.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/layouts.vue`を編集<br>

```vue:layuts.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/pages.vue`を編集<br>

```vue:pages.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/settings.vue`を編集<br>

```vue:settings.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

## 66 ナビゲーションドロワーを作成する

### 無ビゲーションドロワーの設計

`layouts/project.vue` => 開閉フラグ<br>

`components/LoggedIn/LoggedInAppBar.vue` => 開閉ボタン<br>

`components/Project/ProjectNavigationDrawer.vue` => ドロワー表示/フラグの切り替え<br>

- `root $ mkdir front/components/Project && touch $_/ProjectNavigationDrawer.vue`を実行<br>

* `front/components/LoggedIn/LoggedInAppBar.vue`を編集<br>

```vue:LoggedInAppBar.vue
<template>
  <v-app-bar app dense elevation="1" color="white">
    <!-- 追加 -->
    <slot name="navigation-toggle-button" />

    <nuxt-link :to="homePath" class="text-decoration-none">
      <app-logo />
    </nuxt-link>

    <app-title />

    <v-spacer />

    <logged-in-app-bar-account-menu />
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
import AppTitle from '../App/AppTitle.vue'
import LoggedInAppBarAccountMenu from './LoggedInAppBarAccountMenu.vue'
export default {
  components: { AppLogo, AppTitle, LoggedInAppBarAccountMenu },
  data({ $store }) {
    return {
      homePath: $store.state.loggedIn.homePath,
    }
  },
}
</script>
```

- `front/layouts/project.vue`を編集<br>

```vue:project.vue
<template>
  <v-app>
    <!-- toolbar -->
    <logged-in-app-bar>
      <template #navigation-toggle-button>
        <v-app-bar-nav-icon @click="drawer = !drawer" />
      </template>
    </logged-in-app-bar>

    <!-- navigation drawer -->
    <project-navigation-drawer />

    <!-- main content -->
    <v-main>
      <Nuxt />
    </v-main>
  </v-app>
</template>

<script>
import LoggedInAppBar from '../components/LoggedIn/LoggedInAppBar.vue'
export default {
  components: { LoggedInAppBar },
  data() {
    return {
      drawer: null,
    }
  },
}
</script>
```

- `front/components/Project/ProjectNavigationDrawer.vue`を編集<br>

```vue:ProjectNavigationDrawer.vue
<template>
  <v-navigation-drawer app clipped :mobile-breakpoint="mobileBreakpoint">
    <v-list>
      <v-divider />

      <v-list-item
        href="https://blog.cloud-acct.com/categories/udemy"
        target="_blank"
      >
        <v-list-item-icon>
          <v-icon>
            mdi-open-in-new
          </v-icon>
        </v-list-item-icon>

        <v-list-item-content>
          <v-list-item-title>
            このアプリの作り方
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </v-navigation-drawer>
</template>

<script>
export default {
  data() {
    return {
      mobileBreakpoint: 960,
    }
  },
}
</script>
```

- `front/components/LoggenIn/LoggedInAppBar.vue`を編集<br>

```vue:LoggedInAppBar.vue
<template>
  <v-app-bar app dense elevation="1" :clipped-left="clippedLeft" color="white">
    <slot name="navigation-toggle-button" />

    <nuxt-link :to="homePath" class="text-decoration-none">
      <app-logo />
    </nuxt-link>

    <app-title />

    <v-spacer />

    <logged-in-app-bar-account-menu />
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
import AppTitle from '../App/AppTitle.vue'
import LoggedInAppBarAccountMenu from './LoggedInAppBarAccountMenu.vue'
export default {
  components: { AppLogo, AppTitle, LoggedInAppBarAccountMenu },
  props: {
    clippedLeft: {
      type: Boolean,
      default: false,
    },
  },
  data({ $store }) {
    return {
      homePath: $store.state.loggedIn.homePath,
    }
  },
}
</script>
```

- `front/layouts/project.vue`を編集<br>

```vue:project.vue
<template>
  <v-app>
    <!-- toolbar -->
    <logged-in-app-bar clipped-left>
      <template #navigation-toggle-button>
        <v-app-bar-nav-icon @click="drawer = !drawer" />
      </template>
    </logged-in-app-bar>

    <!-- navigation drawer -->
    <project-navigation-drawer :drawer.sync="drawer" />
    <!-- main content -->
    <v-main>
      <Nuxt />
    </v-main>
  </v-app>
</template>

<script>
import LoggedInAppBar from '../components/LoggedIn/LoggedInAppBar.vue'
import ProjectNavigationDrawer from '../components/Project/ProjectNavigationDrawer.vue'
export default {
  components: { LoggedInAppBar, ProjectNavigationDrawer },
  data() {
    return {
      drawer: null,
    }
  },
}
</script>
```

- `front/components/Project/ProjectNavigationDrawer.vue`を編集<br>

```vue:ProjectNavigationDrawer.vue
<template>
  <v-navigation-drawer
    v-model="setDrawer"
    app
    clipped
    :mobile-breakpoint="mobileBreakpoint"
  >
    <v-list>
      <v-divider />

      <v-list-item
        href="https://blog.cloud-acct.com/categories/udemy"
        target="_blank"
      >
        <v-list-item-icon>
          <v-icon>
            mdi-open-in-new
          </v-icon>
        </v-list-item-icon>

        <v-list-item-content>
          <v-list-item-title>
            このアプリの作り方
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </v-navigation-drawer>
</template>

<script>
export default {
  props: {
    drawer: {
      type: Boolean,
      default: null,
    },
  },
  data() {
    return {
      mobileBreakpoint: 960,
    }
  },
  computed: {
    setDrawer: {
      get() {
        return this.drawer
      },
      set(newVal) {
        return this.$emit('update:drawer', newVal)
      },
    },
  },
}
</script>
```

- `front/components/Project/ProjectNavigationDrawer.vue`を編集<br>

```vue:ProjectNavigationDrawer.vue
<template>
  <v-navigation-drawer
    v-model="setDrawer"
    app
    clipped
    :mobile-breakpoint="mobileBreakpoint"
  >
    <v-list>
      <v-divider />

      <!-- 編集 -->
      <v-list-item
        v-for="(nav, i) in navMenus"
        :key="`nav-${i}`"
        :to="$my.projectLinkTo($route.params.id, nav.name)"
      >
        <v-list-item-icon>
          <v-icon v-text="nav.icon" />
        </v-list-item-icon>

        <v-list-item-content>
          <v-list-item-title>
            {{ $my.pageTitle(nav.name) }}
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <!-- ここまで -->
    </v-list>
  </v-navigation-drawer>
</template>

<script>
export default {
  props: {
    drawer: {
      type: Boolean,
      default: null,
    },
  },
  data() {
    return {
      mobileBreakpoint: 960,
      <!-- 追加 -->
      navMenus: [
        { name: 'project-id-dashboard', icon: 'mdi-view-dashboard' },
        { name: 'project-id-layouts', icon: 'mdi-view-compact' },
        { name: 'project-id-pages', icon: 'mdi-image' },
        { name: 'project-id-components', icon: 'mdi-view-comfy' },
        { name: 'project-id-settings', icon: 'mdi-cog' },
        { name: 'project-id-help', icon: 'mdi-help-circle' },
      ],
      <!-- ここまで -->
    }
  },
  computed: {
    setDrawer: {
      get() {
        return this.drawer
      },
      set(newVal) {
        return this.$emit('update:drawer', newVal)
      },
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
    },
    // 追加
    "project": {
      "id": {
        "dashboard": "ダッシュボード",
        "layouts": "レイアウト",
        "pages": "ページ",
        "components": "コンポーネント",
        "settings": "設定",
        "help": "ヘルプ"
      }
    }
    // ここまで
  }
}
```

参考: https://vuetifyjs.com/ja/features/icon-fonts/ <br>

- `front/components/Project/ProjectNavigationDrawer.vue`を編集<br>

```vue:ProjectNavigationDrawer.vue
<template>
  <v-navigation-drawer
    v-model="setDrawer"
    app
    clipped
    :mobile-breakpoint="mobileBreakpoint"
  >
    <v-list>
      <!-- close button -->
      <!-- 編集 -->
      <template v-if="isBreakpoint">
        <v-list-item @click="$emit('update:drawer', false)">
          <v-list-item-content class="text-center">
            <v-list-item-acttion-text
              class="d-flex justify-center align-center"
            >
              <v-icon>
                mdi-chevron-double-left
              </v-icon>
              閉じる
            </v-list-item-acttion-text>
          </v-list-item-content>
        </v-list-item>
        <v-divider />
      </template>

      <!-- nav menus -->
      <v-list-item
        v-for="(nav, i) in navMenus"
        :key="`nav-${i}`"
        :to="$my.projectLinkTo($route.params.id, nav.name)"
      >
        <v-list-item-icon>
          <v-icon v-text="nav.icon" />
        </v-list-item-icon>
        <!-- ここまで -->

        <v-list-item-content>
          <v-list-item-title>
            {{ $my.pageTitle(nav.name) }}
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </v-navigation-drawer>
</template>

<script>
export default {
  props: {
    drawer: {
      type: Boolean,
      default: null,
    },
  },
  data() {
    return {
      mobileBreakpoint: 960,
      navMenus: [
        { name: 'project-id-dashboard', icon: 'mdi-view-dashboard' },
        { name: 'project-id-layouts', icon: 'mdi-view-compact' },
        { name: 'project-id-pages', icon: 'mdi-image' },
        { name: 'project-id-components', icon: 'mdi-view-comfy' },
        { name: 'project-id-settings', icon: 'mdi-cog' },
        { name: 'project-id-help', icon: 'mdi-help-circle' },
      ],
    }
  },
  computed: {
    setDrawer: {
      get() {
        return this.drawer
      },
      set(newVal) {
        return this.$emit('update:drawer', newVal)
      },
    },
    <!-- 追加 -->
    isBreakpoint() {
      const windowWidth = this.$vuetify.breakpoint.width
      return this.mobileBreakpoint > windowWidth
    },
    <!-- ここまで -->
  },
}
</script>
```
