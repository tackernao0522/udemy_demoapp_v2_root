## 69 パンくずリストの追加

- `root $ touch front/components/LoggedIn/LoggedInAppBarBreadcrumbs.vue`を実行<br>

* `front/components/LoggedIn/LoggedInAppBarBreadcrumbs.vue`を編集<br>

```vue:LoggedInAppBarBreadcrumbs.vue
<template>
  <v-breadcrumbs :items="items" class="d-block text-truncate">
    <template #item="{ item }">
      <v-breadcrumbs-item>
        {{ item.text }}
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
      return [{ text: this.$my.pageTitle(this.$route.name) }]
    },
  },
}
</script>
```

- `front/components/LoggedIn/LoggedInAppBar.vue`を編集<br>

```vue:LoggedInAppBar.vue
<template>
  <v-app-bar app dense elevation="1" :clipped-left="clippedLeft" color="white">
    <slot name="navigation-toggle-button" />

    <nuxt-link :to="homePath" class="text-decoration-none">
      <app-logo />
    </nuxt-link>

    <app-title />

    <!-- 追加 -->
    <!-- page title -->
    <logged-in-app-bar-breadcrumbs />
    <!-- ここまで -->
    <v-spacer />

    <!-- account menu -->
    <logged-in-app-bar-account-menu />
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
import AppTitle from '../App/AppTitle.vue'
import LoggedInAppBarAccountMenu from './LoggedInAppBarAccountMenu.vue'
import LoggedInAppBarBreadcrumbs from './LoggedInAppBarBreadcrumbs.vue'
export default {
  components: {
    AppLogo,
    AppTitle,
    LoggedInAppBarAccountMenu,
    LoggedInAppBarBreadcrumbs,
  },
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

- `front/components/LoggedIn/LoggedInAppBar.vue`を再編集<br>

```vue:LoggedInAppBar.vue
<template>
  <v-app-bar app dense elevation="1" :clipped-left="clippedLeft" color="white">
    <slot name="navigation-toggle-button" />

    <nuxt-link :to="homePath" class="text-decoration-none">
      <app-logo />
    </nuxt-link>

    <!-- 編集 -->
    <app-title :class="{ 'hidden-mobile-and-down': isNotHomePath }" />

    <!-- 編集 -->
    <!-- page title -->
    <logged-in-app-bar-breadcrumbs v-if="isNotHomePath" />
    <v-spacer />

    <!-- account menu -->
    <logged-in-app-bar-account-menu />
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
import AppTitle from '../App/AppTitle.vue'
import LoggedInAppBarAccountMenu from './LoggedInAppBarAccountMenu.vue'
import LoggedInAppBarBreadcrumbs from './LoggedInAppBarBreadcrumbs.vue'
export default {
  components: {
    AppLogo,
    AppTitle,
    LoggedInAppBarAccountMenu,
    LoggedInAppBarBreadcrumbs,
  },
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
  <!-- 追加 -->
  computed: {
    isNotHomePath() {
      return this.$route.name !== this.homePath.name
    },
  },
}
</script>
```

- `front/components/LoggedIn/LoggedInAppBarBreadcrumbs.vue`を編集<br>

```vue:LoggedInAppBarBreadcrumbs.vue
<template>
  <v-breadcrumbs :items="items" class="d-block text-truncate">
    <template #item="{ item }">
      <v-breadcrumbs-item>
        {{ item.text }}
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
  <!-- 編集 -->
  computed: {
    items() {
      const currentRouteName = this.$route.name
      const items = [{ text: this.$my.pageTitle(currentRouteName) }]
      // breakpoint.xs => 600未満の場合にtrueを返す
      if (currentRouteName.match(/project/) && !this.$vuetify.breakpoint.xs) {
        // プロジェクト名を表示する
        const currentProject = this.$store.state.project.current
        items.unshift({ text: currentProject.name })
      }
      return items
    },
  },
}
</script>
```

- `front/store/index.js`を編集<br>

```js:index.js
const homePath = 'projects'

// 変数
export const state = () => ({
  styles: {
    homeAppBarHeight: 56,
  },
  loggedIn: {
    homePath: {
      name: homePath,
    },
  },
  project: {
    current: null,
    list: [
      { id: 1, name: 'MyProject01', updatedAt: '2020-04-01T12:00:00+09:00' },
      // 編集
      {
        id: 2,
        name: 'MyProject02MyProject02MyProject02MyProject02',
        updatedAt: '2020-04-05T12:00:00+09:00',
      },
      { id: 3, name: 'MyProject03', updatedAt: '2020-04-03T12:00:00+09:00' },
      { id: 4, name: 'MyProject04', updatedAt: '2020-04-04T12:00:00+09:00' },
      { id: 5, name: 'MyProject05', updatedAt: '2020-04-01T12:00:00+09:00' },
    ],
  },
})

// 算出プロパティ
export const getters = {}

// stateの値を変更する場所
export const mutations = {
  setCurrentProject(state, payload) {
    state.project.current = payload
  },
}

// メソッド
export const actions = {
  // { state, getters, commit, dispatch, rootState, rootGetters } この6つの値が取得できる
  // rootState => ルート(store/index.js)のstateを取得(rootState = state)
  getCurrentProject({ state, commit }, params) {
    const id = Number(params.id)
    const currentProject =
      state.project.list.find((project) => project.id === id) || null
    commit('setCurrentProject', currentProject)
  },
}
```

- `front/components/LoggedIn/LoggedInAppBarBreadcrumbs.vue`を編集<br>

```vue:LoggedInAppBarBreadcrumbs.vue
<template>
  <v-breadcrumbs :items="items" class="d-block text-truncate">
    <template #item="{ item }">
      <v-breadcrumbs-item>
        <!-- 編集 -->
        <div class="text-truncate" :style="{ maxWidth: '120px' }">
          {{ item.text }}
        </div>
        <!-- ここまで -->
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
      // breakpoint.xs => 600未満の場合にtrueを返す
      if (currentRouteName.match(/project/) && !this.$vuetify.breakpoint.xs) {
        // プロジェクト名を表示する
        const currentProject = this.$store.state.project.current
        items.unshift({ text: currentProject.name })
      }
      return items
    },
  },
}
</script>
```
