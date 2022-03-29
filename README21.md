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
