## 88 アクセストークンと使ってプロジェクト一覧を取得する

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
    // 編集
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
})

// 算出プロパティ
export const getters = {}

// stateの値を変更する場所
export const mutations = {
  // 追記
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
}

// メソッド
export const actions = {
  // 追加
  getProjectList({ commit }, projects) {
    projects = projects || []
    commit('setProjectList', projects)
  },
  // ここまで
  // { state, getters, commit, dispatch, rootState, rootGetters } この6つの値が取得できる
  // rootState => ルート(store/index.js)のstateを取得(rootState = state)
  getCurrentProject({ state, commit }, params) {
    const id = Number(params.id)
    const currentProject =
      state.project.list.find((project) => project.id === id) || null
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
}
```

- `root $ touch front/middleware/get-project-list.js`を実行<br>

* `front/middleware/get-project-list.js`を編集<br>

```js:get-project-list.js
export default async ({ store, $axios }) => {
  // プロジェクト一覧が存在しない場合
  if (!store.state.project.list.length) {
    await $axios
      .$get('/api/v1/projects')
      .then((projects) => store.dispatch('getProjectList', projects))
  }
}
```

- `front/pages/projects.vue`を編集<br>

```vue:projects.vue
<template>
  <div id="projects">
    <v-parallax>
      <v-img
        :src="homeImg"
        alt="homeImg"
        :aspect-ratio="16 / 9"
        gradient="to top right, rgba(100,115,201,.33), rgba(25,32,72,.7)"
      >
        <v-container fill-height>
          <v-row justify="center" align="center">
            <!-- プロジェクトの追加  -->
            <v-col cols="12" :sm="container.sm" :md="container.md">
              <v-card-title class="white--text">
                最近のプロジェクト
              </v-card-title>

              <v-divider dark class="mb-4" />

              <v-row align="center">
                <v-col cols="12" :sm="card.sm" :md="card.md">
                  <v-btn
                    block
                    :height="card.height"
                    :elevation="card.elevation"
                  >
                    <div>
                      <v-icon size="24" color="myblue" class="my-2">
                        mdi-plus
                      </v-icon>
                      <div class="caption myblue--text">
                        プロジェクトを追加
                      </div>
                    </div>
                  </v-btn>
                </v-col>

                <!-- 最近のプロジェクト -->
                <v-col
                  v-for="(project, i) in recentProjects.slice(0, 2)"
                  :key="`card-project-${i}`"
                  cols="12"
                  :sm="card.sm"
                  :md="card.md"
                >
                  <v-card
                    block
                    :height="card.height"
                    :elevation="card.elevation"
                    :to="$my.projectLinkTo(project.id)"
                    class="v-btn text-capitalize"
                  >
                    <v-card-title class="pb-1 d-block text-truncate">
                      {{ project.name }}
                    </v-card-title>
                    <v-card-text class="caption">
                      <v-icon size="14">
                        mdi-update
                      </v-icon>
                      {{ $my.dateFormat(project.updatedAt) }}
                    </v-card-text>
                  </v-card>
                </v-col>
              </v-row>
            </v-col>
          </v-row>
        </v-container>
      </v-img>
    </v-parallax>

    <v-container>
      <v-row justify="center">
        <v-col cols="12" :sm="container.sm" :md="container.md">
          <v-card-title>
            全てのプロジェクト
          </v-card-title>

          <v-divider class="mb-4" />

          <v-data-table
            :headers="tableHeaders"
            :items="recentProjects"
            item-key="id"
            hide-default-footer
          >
            <template #[`item.name`]="{ item }">
              <nuxt-link
                :to="$my.projectLinkTo(item.id)"
                class="text-decoration-none"
              >
                {{ item.name }}
              </nuxt-link>
            </template>
            <template #[`item.updatedAt`]="{ item }">
              {{ $my.dateFormat(item.updatedAt) }}
            </template>
          </v-data-table>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import homeImg from '@/assets/images/logged-in/home.png'

export default {
  layout: 'logged-in',
  <!-- 追記 -->
  middleware: ['get-project-list'],
  data() {
    return {
      homeImg,
      container: {
        sm: 10,
        md: 8,
      },
      card: {
        sm: 6,
        md: 4,
        height: 110,
        elevation: 4,
      },
      tableHeaders: [
        {
          text: '名前',
          value: 'name',
        },
        {
          text: '更新日',
          width: 150,
          value: 'updatedAt',
        },
      ],
    }
  },
  computed: {
    recentProjects() {
      const copyProjects = Array.from(this.$store.state.project.list)
      return copyProjects.sort((a, b) => {
        if (a.updatedAt > b.updatedAt) {
          return -1
        }
        if (a.updatedAt < b.updatedAt) {
          return 1
        }
        return 0
      })
    },
  },
}
</script>

<style lang="scss">
#projects {
  .v-parallax__content {
    padding: 0;
  }
}
</style>
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
  <!-- 編集(順番を間違えないように) -->
  middleware: ['get-project-list', 'get-project-current'],
  data() {
    return {
      drawer: null,
    }
  },
}
</script>
```

- localhost:8080/projects にアクセスすると 401 エラーになる<br>

* `front/plugins/axios.js`を編集<br>

```js:axios.js
// 編集
export default ({ $axios, $auth }) => {
  // リクエストログ
  $axios.onRequest((config) => {
    config.headers.common['X-Requested-With'] = 'XMLHttpRequest'
    // 追記
    if ($auth.token) {
      config.headers.common.Authorization = `Bearer ${$auth.token}`
    }
    // ここまで
    // eslint-disable-next-line no-console
    console.log(config)
  })
  // レスポンスログ
  $axios.onResponse((config) => {
    // eslint-disable-next-line no-console
    console.log(config)
  })
  // エラーログ
  $axios.onError((e) => {
    // eslint-disable-next-line no-console
    console.log(e.response)
  })
}
```

- localhost:8080 へアクセスしてからログインしてみる<br>

```
{url: '/api/v1/projects', method: 'get', headers: {…}, baseURL: 'http://localhost:3000', transformRequest: Array(1), …}
axios.js?dcce:14
{data: Array(10), status: 200, statusText: 'OK', headers: {…}, config: {…}, …}
config: {url: '/api/v1/projects', method: 'get', headers: {…}, baseURL: 'http://localhost:3000', transformRequest: Array(1), …}
data: Array(10)
0:
id: 1
name: "user0 project 01"
updatedAt: "2021-04-01T06:00:00.000+09:00"
__ob__: Observer {value: {…}, dep: Dep, vmCount: 0}
get id: ƒ reactiveGetter()
set id: ƒ reactiveSetter(newVal)
get name: ƒ reactiveGetter()
set name: ƒ reactiveSetter(newVal)
get updatedAt: ƒ reactiveGetter()
set updatedAt: ƒ reactiveSetter(newVal)
[[Prototype]]: Object
1: {__ob__: Observer}
2: {__ob__: Observer}
3: {__ob__: Observer}
4: {__ob__: Observer}
5: {__ob__: Observer}
6: {__ob__: Observer}
7: {__ob__: Observer}
```

## 89 ログアウト機能を実装する

### ログアウト業務の整理

#### 1. Vuex に保存した値の削除

1. 認証情報<br>
2. ユーザーオブジェクト<br>
3. プロジェクト一覧<br>
4. 選択中のプロジェクト<br>
   |<br>
   これらのものを全て
   ↓<br>
   \$auth.logout()<br>

### 2. リフレッシュトークンの削除

- `front/plugins/auth.js`を編集<br>

```js:auth.js
import jwtDecode from 'jwt-decode'

class Authentication {
  constructor(ctx) {
    this.store = ctx.store
    this.$axios = ctx.$axios
  }

  get token() {
    return this.store.state.auth.token
  }

  get expires() {
    return this.store.state.auth.expires
  }

  get payload() {
    return this.store.state.auth.payload
  }

  get user() {
    return this.store.state.user.current || {}
  }

  // 認証情報をVuexに保存する
  setAuth({ token, expires, user }) {
    const exp = expires * 1000
    const jwtPayload = token ? jwtDecode(token) : {}

    this.store.dispatch('getAuthToken', token)
    this.store.dispatch('getAuthExpires', exp)
    this.store.dispatch('getCurrentUser', user)
    this.store.dispatch('getAuthPayload', jwtPayload)
  }

  // ログイン業務
  login(response) {
    this.setAuth(response)
  }

  // 追加
  // Vuexの値を初期値に戻す
  resetVuex() {
    this.setAuth({ token: null, expires: 0, user: null })
    this.store.dispatch('getCurrentProject', null)
    this.store.dispatch('getProjectList', [])
  }

  // ログアウト業務
  async logout() {
    await this.$axios.$delete('/api/v1/auth_token')
    this.resetVuex()
  }
  // ここまで
}

export default ({ store, $axios }, inject) => {
  inject('auth', new Authentication({ store, $axios }))
}
```

- `front/pages/logout.vue`を編集<br>

```vue:logout.vue
<script>
export default {
  // ページをレンダリングする前に実行される
  // nuxtServerInitの後
  <!-- 編集 -->
  async middleware({ $auth, redirect }) {
    <!-- 編集 -->
    await $auth.logout()
    return redirect('/')
  },
  // asyncData () の前
}
</script>
```

- `font/store/index.js`を編集<br>

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
    // 編集
    let currentProject = null
    if (params && params.id) {
      const id = Number(params.id)
      currentProject =
        state.project.list.find((project) => project.id === id) || null
    }
    // ここまで
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
}
```

参考: https://github.com/axios/axios#request-config <br>

- `front/plugins/auth.js`を編集<br>

```js:auth.js
import jwtDecode from 'jwt-decode'

class Authentication {
  constructor(ctx) {
    this.store = ctx.store
    this.$axios = ctx.$axios
  }

  get token() {
    return this.store.state.auth.token
  }

  get expires() {
    return this.store.state.auth.expires
  }

  get payload() {
    return this.store.state.auth.payload
  }

  get user() {
    return this.store.state.user.current || {}
  }

  // 認証情報をVuexに保存する
  setAuth({ token, expires, user }) {
    const exp = expires * 1000
    const jwtPayload = token ? jwtDecode(token) : {}

    this.store.dispatch('getAuthToken', token)
    this.store.dispatch('getAuthExpires', exp)
    this.store.dispatch('getCurrentUser', user)
    this.store.dispatch('getAuthPayload', jwtPayload)
  }

  // ログイン業務
  login(response) {
    this.setAuth(response)
  }

  // Vuexの値を初期値に戻す
  resetVuex() {
    this.setAuth({ token: null, expires: 0, user: null })
    this.store.dispatch('getCurrentProject', null)
    this.store.dispatch('getProjectList', [])
  }

  // 追加
  // axiosのレスポンス401を許容する
  // Doc: https://github.com/axios/axios#request-config
  resolveUnauthorized(status) {
    return (status >= 200 && status < 300) || status === 401
  }

  // ログアウト業務
  async logout() {
    // 編集
    await this.$axios.$delete('/api/v1/auth_token', {
      validateStatus: (status) => this.resolveUnauthorized(status),
    })
    this.resetVuex()
  }
}

export default ({ store, $axios }, inject) => {
  inject('auth', new Authentication({ store, $axios }))
}
```

- `front/layouts/default.vue`を編集<br>

```vue:default.vue
<template>
  <v-app>
    <Nuxt />
  </v-app>
</template>

<script>
export default {
  <!-- 追記 -->
  name: 'LayoutsDefault',
}
</script>
```
