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
