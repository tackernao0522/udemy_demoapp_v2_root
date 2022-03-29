## 67 Vuex を理解する

### Vuex とは何か?

状態管理を行うライブラリのこと<br>

vue ファイルに用意したデータは、その vue ファイルを離れるとリセットされる<br>

`index.vue`<br>

```vue:index.vue
data () { count: 0 => 5 ↓ count: 0 リセットされる }
```

↓(ページ遷移)<br>

`login.vue`<br>

↓<br>

Vuex を使用して変化したデータの状態をアプリケーション内で維持する<br>

### Vuex の基本ルール（ファイル）

- store ディレクトリ内の JS ファイル内で扱う<br>

`store/index.js` -> デフォルトの呼び出し<br>

- 他にも任意の名前で呼び出すことができる<br>

`store/user.js`や`store/project.js`など -> 関数呼び出し時にファイル名を指定する<br>

### Vuex の基本ルール（構文）

- 4 つの export 文<br>

|     |           |                                                  |
| :-: | :-------: | :----------------------------------------------: |
|  1  |   state   |               データを用意する場所               |
|  2  |  getters  |     計算した値を返す<br>算出プロパティを宣言     |
|  3  | mutations |             state の値を変更する場所             |
|  4  |  actions  | state の値を変更するためのメソッドを管理する場所 |

state の値は mutations でしか変更できない<br>

データ加工や非同期処理を行いその結果を mutations に渡す<br>

### Vuex の基本ルール（呼び出し方）

- 各プロパティの呼び出し方法<br>

|     |           |                                       |
| :-: | :-------: | :-----------------------------------: |
|  1  |   state   |         store.state.データ名          |
|  2  |  getters  |      store.getters.プロパティ名       |
|  3  | mutations | store.commit('mutation 名', 新しい値) |
|  4  |  actions  |   store.dispatch('action 名', 引数)   |

### state の値を変更する 2 つの方法

`vueファイル`<br>

1. `store.dispatch`データ加工処理や非同期処理がなくてもこっちで統一<br>
   ↓<br>
   `Vuex` actions => データ加工・非同期処理<br>
   ↓<br>
   -(commit)-> mutations => state の値を変更<br>
   ↓<br>
2. store.commit -> mutatins => state の値を変更<br>

### ハンズオン

- `front/components/Project/ProjectNavigationDrawer.vue`を一部修正<br>

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
      <!-- 修正 -->
      <template v-if="isBreakpointLessThan">
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
    <!-- 修正 -->
    isBreakpointLessThan() {
      const windowWidth = this.$vuetify.breakpoint.width
      return this.mobileBreakpoint > windowWidth
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
      { id: 2, name: 'MyProject02', updatedAt: '2020-04-05T12:00:00+09:00' },
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

### 68 middleware を理解する

### middleware とは？

ページをレンダリング（描画）する前に実行する関数を指定する場所<br>

- 認証処理<br>

* リダイレクト処理<br>

- 描画データの取得 etc...<br>
  ↓<br>
  ユーザーが見る前の準備を行う場所<br>

### 2 つの書き方

1. 関数で処理を書く<br>

`page, layout`<br>

```
export default {
  middleware (context) {
    // 実行したい処理
  }
}
```

2. middleware ファイルで処理を書く<br>

`middleware/auth.js`<br>

```js:auth.js
export default (context) => {
  // 実行したい処理
}
```

`page, layout`<br>

```
export default {
  middleware: 'auth'
}
```

※ コンポーネント vue ファイルからは middleware は呼び出すことができない<br>

### 指定方法と実行順序

|                   使用目的                   |         指定場所         | 実行順序 |
| :------------------------------------------: | :----------------------: | :------: |
|               アプリ全体で実行               |  nuxt.config.js で指定   |    1     |
| レイアウトを使用している<br>ページ全体で実行 | レイアウトファイルで指定 |    2     |
|              特定のページで実行              |   ページファイルで指定   |    3     |

`nuxt.config.js`の場合<br>

```js:nuxt.config.js
router: {
  middleware: 'auth'
}
// NG => middleware ()
```

- 複数で middleware を呼び出したい場合<br>

```
export default {
  middleware: ['auth', 'get-project-list', 'get-project-current'], // 左から順番に呼び出される
}
```

### 洗濯中のプロジェクトを取得する

`layouts/project.vue`今回は layout で currentProject を取得する<br>
↑<br>
どちらかで取得
↓<br>
`layouts/pages/project.vue`<br>
`layouts/pages/project/\_id/dashboard.vue<br> の順番で呼ばれる->ここまでに`currentProject を取得する<br>

### ハンズオン

- `root $ touch front/middleware/get-project-current.js`を実行<br>

* `front/middleware/get-project-current.js`を編集<br>

```js:get-project-current.js
export default async ({ store, params }) => {
  return await store.dispatch('getCurrentProject', params)
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
  <!-- 追加 -->
  middleware: 'get-project-current',
  data() {
    return {
      drawer: null,
    }
  },
}
</script>
```

- `front/pages/project.vue`を編集<br>

```vue:project.vue
<template>
  <v-container>
    <!-- 追加 -->
    {{ $store.state.project.current }}
    <nuxt-child />
  </v-container>
</template>

<script>
export default {
  layout: 'project',
  <!-- 編集 -->
  validate ({ store, route }) {
    return !!store.state.project.current && route.name !== 'project'
  },
  <!-- ここまで -->
}
</script>
```

- `http://localhost:8080/project/2/dashboard`にアクセスすると<br>

```:browser
{ "id": 4, "name": "MyProject04", "updatedAt": "2020-04-04T12:00:00+09:00" }
/project/4/dashboard
```

- `front/pages/account.vue`を編集<br>

```vue:account.vue
<template>
  <v-container>
    <!-- 追加 -->
    <v-btn
      v-if="!!currentProject"
      small
      plain
      color="primary"
      :to="$my.projectLinkTo(currentProject.id, dashboardPath)"
    >
      <v-icon left>
        mdi-chevron-double-left
      </v-icon>
      {{ $my.pageTitle(dashboardPath) }}に戻る
    </v-btn>
    <!-- ここまで -->
    <nuxt-child />
  </v-container>
</template>

<script>
export default {
  layout: 'logged-in',
  // falseを返すページのアクセスを制限する
  validate({ route }) {
    // account
    return route.name !== 'account'
  },
  <!-- 追加 -->
  data() {
    return {
      dashboardPath: 'project-id-dashboard',
    }
  },
  computed: {
    currentProject() {
      return this.$store.state.project.current
    },
  },
  <!-- ここまで -->
}
</script>
```
