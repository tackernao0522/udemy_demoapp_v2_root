## 94 ログイン前ユーザーがアクセスしたルートを記憶する

### 記憶ルートの実装手順

1. Vuex にログイン前 h ユーザーがアクセスしようとしたルートを記憶<br>

2. 記憶するタイミングはログイン前のリダイレクト処理・セッション切れ<br>

3. ログイン後に記憶ルートにリダイレクト

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
    // 追加
    rememberPath: {
      name: homePath,
      params: {},
    },
    // ログイン後アクセス不可ルート一覧
    redirectPaths: ['index', 'signup', 'login'],
    // ここまで
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
  toast: {
    msg: null,
    color: 'error',
    timeout: 4000,
  },
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
  setToast(state, payload) {
    state.toast = payload
  },
  // 追加
  setRememberPath(state, payload) {
    state.loggedIn.rememberPath = payload
  },
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
  getToast({ commit }, { msg, color, timeout }) {
    color = color || 'error'
    timeout = timeout || 4000
    commit('setToast', { msg, color, timeout })
  },
  // 追加
  // ログイン前ユーザーがアクセスしたルートを記憶する
  getRememberPath({ state, commit }, { name, params }) {
    // ログイン前パスが渡された場合はloggedIn.homepathに書き換える
    if (state.loggedIn.redirectPaths.includes(name)) {
      name = state.loggedIn.homePath.name
    }
    params = params || {}
    commit('setRememberPath', { name, params })
  },
  // ここまで
}
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
    // トースター出力
    store.dispatch('getToast', { msg, color })
    // TODO アクセスルート記憶
    // コメントアウトを解除
    store.dispatch('getRememberPath', route)
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
        // トースター出力
        store.dispatch('getToast', { msg })
        // アクセスルート記憶
        // コメントアウト解除
        store.dispatch('getRememberPath', route)
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
      // 編集
      redirectPath: $store.state.loggedIn.rememberPath,
      loggedInHomePath: $store.state.loggedIn.homePath,
      // ここまで
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
    },
    // 編集
    authSuccessful(response) {
      this.$auth.login(response)
      this.$router.push(this.redirectPath)
      // 記憶ルートを初期値に戻す
      this.$store.dispatch('getRememberPath', this.loggedInHomePath)
    },
    // ここまで
    authFailure({ response }) {
      if (response && response.status === 404) {
        const msg = 'ユーザーが見つかりません😢'
        return this.$store.dispatch('getToast', { msg })
      }
      // TODO エラー処理
    },
  },
}
</script>
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
        // 編集
        // Vuexの初期化(セッションはサーバで削除済み)
        $auth.resetVuex()
        if (route.name === 'logout') {
          return redirect('/')
        } else {
          const msg =
            'セッションの有効期限が切れました。' +
            'もう一度ログインしてください'
          // トースター出力
          store.dispatch('getToast', { msg })
          // アクセスルート記憶
          store.dispatch('getRememberPath', route)
          return redirect('/login')
        }
        // ここまで
      })
  }
}
```

## 95 ログイン済みユーザーのリダイレクト処理を行う

### ログイン済みユーザーがアクセスできないルート

1. ホーム / (index.vue)<br>

2. 会員登録ページ /signup (signup.vue)<br>

3. ログインページ /login (login.vue)<br>

### ハンズオン

- `root $ touch front/middleware/logged-in-redirect.js`を実行<br>

* `front/middleware/logged-in-redirect.js`を編集<br>

```js:logged-in-redirect.js
export default ({ $auth, store, route, redirect }) => {
  // ログイン済ユーザーをリダイレクトさせる
  const redirectPaths = store.state.loggedIn.redirectPaths
  if ($auth.loggedIn() && redirectPaths.includes(route.name)) {
    return redirect(store.state.loggedIn.homePath)
  }
}
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
  name: 'LayoutsBeforeLogin',
  components: { BeforeLoginAppBar, AppFooter, AppToaster },
  <!-- 追加 -->
  middleware: ['logged-in-redirect'],
}
</script>
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <home-app-bar :menus="menus" :img-height="imgHeight" />
    <v-img
      id="scroll-top"
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
          <v-col :id="menu.title" cols="12">
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
  <!-- 追加 -->
  middleware: ['logged-in-redirect'],
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
