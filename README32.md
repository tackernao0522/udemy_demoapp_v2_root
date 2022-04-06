## 92 ログイン前のユーザーのリダイレクト処理を行う

### リダイレクト処理のタイミング

|    user    | expires | access token | refresh token | logged in |     redirect     |
| :--------: | :-----: | :----------: | :-----------: | :-------: | :--------------: |
| 存在しない |    -    |      -       |       -       |   FALSE   | 「ログインして」 |

### ハンズオン

- `root $ touch front/middleware/authentication.js`を実行<br>

```js:authentication.js
export default async ({ $auth, store, route, redirect }) => {
  // リダイレクトを必要としないパス
  const notRedirectPaths = ['account', 'project']
  if (notRedirectPaths.includes(route.name)) {
    return false
  }

  // ログイン前ユーザー処理
  if (!$auth.loggedIn()) {
    // ユーザー以外の値が存在する可能性があるので全てを削除する
    await $auth.logout()

    const msg = 'まずはログインしてください'
    const color = 'info'
    // TODO test
    // eslint-disable-next-line no-console
    console.log(msg, color)
    // TODO トースター出力
    // store.dispatch('getToast', { msg, color })
    // TODO アクセスルート記憶
    // store.dispatch('getRememberPath', route)
    return redirect('/login')
  }
}
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
  <!-- 編集 authenticationは先頭に記述 -->
  middleware: ['authentication', 'get-project-list', 'get-project-current'],
  data() {
    return {
      drawer: null,
    }
  },
}
</script>
```

- `fornt/layouts/logged-in.vue`を編集<br>

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
export default {
  <!-- 追記 -->
  middleware: ['authentication'],
}
</script>
```

## 93 グローバルイベントのトースターを作成する

### トースターの実装手順

1. Vuex に State, Action, Mutation を追加<br>

2. トースターコンポーネントを作成<br>

- State の値をフラグとして表示<br>

* どこからでも呼び出せるトースターに<br>

### トースターを呼び出す３つの場所

1. `middleware/authentication.js` ログインしてください<br>
2. `middleware/silent-refresh-token.js` セッション切れ<br>
3. `pages/login.vue` ログインに失敗<br>

### ハンズオン

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
    list: [],
  },
  user: {
    current: null,
  },
  auth: {
    token: null,
    expires: 0,
    payload: {},
  },
  // 追加
  toast: {
    msg: null,
    color: 'error',
    timeout: 4000,
  },
  // ここまで
})

// 算出プロパティ
export const getters = {}

// stateの値を変更する場所
export const mutations = {
  setProjectList(state, payload) {
    state.project.list = payload
  },
  setCurrentProject(state, payload) {
    state.project.current = payload
  },
  setCurrentUser(state, payload) {
    state.user.current = payload
  },
  setAuthToken(state, payload) {
    state.auth.token = payload
  },
  setAuthExpires(state, payload) {
    state.auth.expires = payload
  },
  setAuthPayload(state, payload) {
    state.auth.payload = payload
  },
  // 追加
  setToast(state, payload) {
    state.toast = payload
  },
  // ここまで
}

// メソッド
export const actions = {
  getProjectList({ commit }, projects) {
    projects = projects || []
    commit('setProjectList', projects)
  },
  // { state, getters, commit, dispatch, rootState, rootGetters } この6つの値が取得できる
  // rootState => ルート(store/index.js)のstateを取得(rootState = state)
  getCurrentProject({ state, commit }, params) {
    let currentProject = null
    if (params && params.id) {
      const id = Number(params.id)
      currentProject =
        state.project.list.find((project) => project.id === id) || null
    }
    commit('setCurrentProject', currentProject)
  },
  getCurrentUser({ commit }, user) {
    commit('setCurrentUser', user)
  },
  getAuthToken({ commit }, token) {
    commit('setAuthToken', token)
  },
  getAuthExpires({ commit }, expires) {
    expires = expires || 0
    commit('setAuthExpires', expires)
  },
  getAuthPayload({ commit }, jwtPayload) {
    jwtPayload = jwtPayload || {}
    commit('setAuthPayload', jwtPayload)
  },
  // 追加
  getToast({ commit }, { msg, color, timeout }) {
    color = color || 'error'
    timeout = timeout || 4000
    commit('setToast', { msg, color, timeout })
  },
  // ここまで
}
```

- `root $ touch front/components/App/AppToaster.vue`を実行<br>

* `front/components/App/AppToaster.vue`を編集<br>

```vue:AppToaster.vue
<template>
  <!--
    2022.02.20 追記
    動画上とアプリの挙動が変わりました。
    <v-snackbar>にはappプロパティを追加してください。
    appは領域をVuetifyコンポーネントに正しく伝えるためのプロパティです。
    v-app-bar, v-navigation-drawer, v-footerなどの他のアプリコンポーネントと重ならないようにするために使用します。
    Doc: https://vuetifyjs.com/en/api/v-snackbar/
    バグ詳細: https://www.udemy.com/course/jwt-login-authentication-with-railsapi-nuxtjs/learn/#questions/16948682/
  -->
  <!--
    v-model="setSnackbar"
    get()がtrueを返す時、トースターが開く
    set()でfalseを返す時、トースターが閉じる
  -->
  <v-snackbar
    v-model="setSnackbar"
    app
    top
    text
    :timeout="toast.timeout"
    :color="toast.color"
  >
    {{ toast.msg }}
    <template #action="{ attrs }">
      <v-btn v-bind="attrs" text :color="toast.color" @click="resetToast">
        Close
      </v-btn>
    </template>
  </v-snackbar>
</template>

<script>
export default {
  computed: {
    // Vuexのtoastオブジェクトを呼び出し & 監視
    toast() {
      return this.$store.state.toast
    },
    setSnackbar: {
      // Vuexのtoastオブジェクトのmsgがあった場合にtrueを返す
      get() {
        return !!this.toast.msg
      },
      // (val)にはfalseが返ってくる（Vuetifyのv-snackbarの仕様）
      // set()内で return false を行うと、トースターが閉じる
      // return falseの前に、Vuexのtoast.msgをnullにリセットしている
      // set () => timeout: -1の場合は通過しない
      set(val) {
        return this.resetToast() && val
      },
    },
  },
  beforeDestroy() {
    // Vueインスタンスが破棄される直前にVuexのtoast.msgを削除する(無期限toastに対応)
    this.resetToast()
  },
  methods: {
    // Vuexのtoast.msgの値を変更する
    resetToast() {
      return this.$store.dispatch('getToast', { msg: null })
    },
  },
}
</script>
```

- `front/layouts/before-login.vue`を編集<br>

```vue:before-login.vue
<template>
  <v-app>
    <before-login-app-bar />
    <v-main>
      <!-- 追記 -->
      <app-toaster />
      <Nuxt />
    </v-main>
    <app-footer />
  </v-app>
</template>

<script>
import AppFooter from '../components/App/AppFooter.vue'
<!-- 追記 -->
import AppToaster from '../components/App/AppToaster.vue'
import BeforeLoginAppBar from '../components/BeforeLogin/BeforeLoginAppBar.vue'
export default {
  <!-- 編集 -->
  components: { BeforeLoginAppBar, AppFooter, AppToaster },
}
</script>
```

- `front/middleware/authentication.js`を編集<br>

```js:authentication.js
export default async ({ $auth, store, route, redirect }) => {
  // リダイレクトを必要としないパス
  const notRedirectPaths = ['account', 'project']
  if (notRedirectPaths.includes(route.name)) {
    return false
  }

  // ログイン前ユーザー処理
  if (!$auth.loggedIn()) {
    // ユーザー以外の値が存在する可能性があるので全てを削除する
    await $auth.logout()

    const msg = 'まずはログインしてください'
    const color = 'info'
    // 編集 コメントアウト解除
    // トースター出力
    store.dispatch('getToast', { msg, color })
    // TODO アクセスルート記憶
    // store.dispatch('getRememberPath', route)
    return redirect('/login')
  }
}
```

- `front/middleware/silent-refresh-token.js`を編集<br>

```js:silent-refresh-token.js
export default async ({ $auth, $axios, store, route, redirect, isDev }) => {
  if ($auth.isExistUserAndExpired()) {
    if (isDev) {
      // eslint-disable-next-line no-console
      console.log('Execute silent refresh!!')
    }
    await $axios
      .$post('/api/v1/auth_token/refresh')
      .then((response) => $auth.login(response))
      .catch(() => {
        const msg =
          'セッションの有効期限が切れました。' + 'もう一度ログインしてください'
        // 編集 コメントアウト解除
        // トースター出力
        store.dispatch('getToast', { msg })
        // TODO アクセスルート記憶
        // store.dispatch('getRememberPath', route)
        // Vuexの初期化(セッションはサーバで削除済み)
        $auth.resetVuex()
        return redirect('/login')
      })
  }
}
```

- `front/pages/login.vue`を編集<br>

```vue:login.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <v-form ref="form" v-model="isValid" @submit.prevent="login">
        <user-form-email :email.sync="params.auth.email" />
        <user-form-password :password.sync="params.auth.password" />
        <v-card-actions>
          <nuxt-link to="#" class="body-2 text-decoration-none">
            パスワードを忘れた？
          </nuxt-link>
        </v-card-actions>
        <v-card-text class="px-0">
          <!-- disabled=true => ボタンクリックを無効にする -->
          <v-btn
            type="submit"
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
      // TODO 削除する
      params: { auth: { email: 'user0@example.com', password: 'password' } },
      redirectPath: $store.state.loggedIn.homePath,
    }
  },
  methods: {
    async login() {
      this.loading = true
      if (this.isValid) {
        await this.$axios
          .$post('/api/v1/auth_token', this.params)
          .then((response) => this.authSuccessful(response))
          .catch((error) => this.authFailure(error))
      }
      this.loading = false
      <!-- ここの行の記述削除 -->
    },
    authSuccessful(response) {
      // eslint-disable-next-line no-console
      console.log('authSuccessful', response)
      this.$auth.login(response)
      // TODO test
      // eslint-disable-next-line no-console
      console.log('token', this.$auth.token)
      // eslint-disable-next-line no-console
      console.log('expires', this.$auth.expires)
      // eslint-disable-next-line no-console
      console.log('payload', this.$auth.payload)
      // eslint-disable-next-line no-console
      console.log('user', this.$auth.user)
      // TODO 記憶ルートリダイレクト
      this.$router.push(this.redirectPath)
    },
    authFailure({ response }) {
      if (response && response.status === 404) {
        <!-- 追加 -->
        const msg = 'ユーザーが見つかりません😢'
        return this.$store.dispatch('getToast', { msg })
      }
      // TODO エラー処理
    },
  },
}
</script>
```

- `front/layouts/before-login.vue`を編集<br>

```vue:before-login.vue
<template>
  <v-app>
    <before-login-app-bar />
    <v-main>
      <app-toaster />
      <Nuxt />
    </v-main>
    <app-footer />
  </v-app>
</template>

<script>
import AppFooter from '../components/App/AppFooter.vue'
import AppToaster from '../components/App/AppToaster.vue'
import BeforeLoginAppBar from '../components/BeforeLogin/BeforeLoginAppBar.vue'
export default {
  <!-- 追加 -->
  name: 'LayoutsBeforeLogin',
  components: { BeforeLoginAppBar, AppFooter, AppToaster },
}
</script>
```
