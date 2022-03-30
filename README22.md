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

## 70 エンターキーでフォームを送信する

- `front/pages/login.vue`を編集<br>

```vue:login.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <!-- @submitを追加 -->
      <v-form ref="form" v-model="isValid" @submit.prevent="login">
        <user-form-email :email.sync="params.user.email" />
        <user-form-password :password.sync="params.user.password" />
        <v-card-actions>
          <nuxt-link to="#" class="body-2 text-decoration-none">
            パスワードを忘れた？
          </nuxt-link>
        </v-card-actions>
        <v-card-text class="px-0">
          <!-- disabled=true => ボタンクリックを無効にする -->
          <v-btn
            type="submit" // 記述して@clickイベントを削除
            :disabled="!isValid || loading"
            :loading="loading"
            block
            color="appblue"
            class="white--text"
          >
            ログインする
          </v-btn>
        </v-card-text>
      </v-form>
    </template>
  </user-form-card>
</template>

<script>
import UserFormCard from '../components/User/UserFormCard.vue'
import UserFormEmail from '../components/User/UserFormEmail.vue'
import UserFormPassword from '../components/User/UserFormPassword.vue'

export default {
  components: { UserFormCard, UserFormEmail, UserFormPassword },
  layout: 'before-login',
  data({ $store }) {
    return {
      name: '',
      isValid: false,
      loading: false,
      params: { user: { email: '', password: '' } },
      redirectPath: $store.state.loggedIn.homePath,
    }
  },
  methods: {
    login() {
      this.loading = true
      this.$router.push(this.redirectPath)
    },
  },
}
</script>
```

- `front/pages/signup.vue`を編集<br>

```vue:signup.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <!-- @submit.prevent="signup"を追加 -->
      <v-form ref="form" v-model="isValid" @submit.prevent="signup">
        <user-form-name :name.sync="params.user.name" />
        <user-form-email :email.sync="params.user.email" placeholder />
        <user-form-password
          :password.sync="params.user.password"
          set-validation
        />
        <!-- disabled=true => ボタンクリックを無効にする -->
        <v-btn
          type="submit" // 追加して@clickイベントを削除
          :disabled="!isValid || loading"
          :loading="loading"
          block
          color="appblue"
          class="white--text"
        >
          登録する
        </v-btn>
      </v-form>
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
  data() {
    return {
      name: '',
      isValid: false,
      loading: false,
      params: { user: { name: '', email: '', password: '' } },
    }
  },
  methods: {
    signup() {
      this.loading = true
      setTimeout(() => {
        this.formReset()
        this.loading = false
      }, 1500)
    },
    formReset() {
      this.$refs.form.reset()
      for (const key in this.params.user) {
        this.params.user[key] = ''
      }
    },
  },
}
</script>
```
