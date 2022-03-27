## 62 NuxtChild コンポーネントを理解する

### NuxtChild コンポーネントとは？

ネストされたルート内で子コンポーネント表示する<br>

## 63 NuxtChild を使ってアカウントページを作成する

https://nuxtjs.org/ja/docs/features/nuxt-components/#nuxtchild-%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88 <br>

- `root $ mkdir front/pages/account && touch $_/{password.vue,settings.vue}`を実行<br>

* `root $ touch front/pages/account.vue`を実行<br>

- `root $ tree front/page`を実行して階層の確認<br>

```:terminal
front/pages
├── account
│   ├── password.vue
│   └── settings.vue
├── account.vue
├── index.vue
├── login.vue
├── projects.vue
└── signup.vue
```

- `front/pages/account.vue`(親)を編集<br>

```vue:account.vue
<template>
  <v-container>
    <nuxt-child />
  </v-container>
</template>

<script>
export default {
  layout: 'logged-in',
}
</script>
```

- `front/pages/account/password.vue`(子)を編集<br>

```vue:password.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/account/settings.vue`(子)を編集<br>

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

- http://localhost:8080/account/password にアクセスして確認<br>

* http://localhost:8080/account/settings にアクセスして確認<br>

- `front/pages/account.vue`を編集<br>

```vue:account.vue
<template>
  <v-container>
    <nuxt-child />
  </v-container>
</template>

<script>
export default {
  layout: 'logged-in',
  // 追加
  // falseを返すページのアクセスを制限する
  validate({ route }) {
    // account
    return route.name !== 'account'
  },
}
</script>
```

- `root $ touch front/pages/logout.vue`を実行<br>

- `front/pages/logout.vue`を編集<br>

```vue:logout.vue
<script>
export default {
  // ページをレンダリングする前に実行される
  // nuxtServerInitの後
  middleware({ redirect }) {
    return redirect('/')
  },
  // asyncData () の前
}
</script>
```

- `front/pages/login.vue`を編集<br>

```vue:login.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <v-form ref="form" v-model="isValid">
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
            :disabled="!isValid || loading"
            :loading="loading"
            block
            color="appblue"
            class="white--text"
            @click="login"
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
  <!-- 編集 -->
  data({ $store }) {
    return {
      name: '',
      isValid: false,
      loading: false,
      params: { user: { email: '', password: '' } },
      // 追加
      redirectPath: $store.state.loggedIn.redirectPath,
    }
  },
  methods: {
    login() {
      this.loading = true
      // 編集
      this.$router.push(this.redirectPath)
    },
  },
}
</script>
```

## 64 ログイン後のトップページにプロジェクト一覧を表示する

- `front/store/index.js`を編集<br>

```js:index.js
// 編集
const homePath = 'projects'

// 変数
export const state = () => ({
  styles: {
    homeAppBarHeight: 56,
  },
  loggedIn: {
    // 編集
    homePath: {
      name: homePath,
    },
    // ここまで
  },
})

// 算出プロパティ
export const getters = {}

// stateの値を変更する場所
export const mutations = {}

// メソッド
export const actions = {}
```

- `front/components/LoggedIn/LoggedInAppBar.vue`を編集<br>

```vue:LoggedInAppBar.vue
<template>
  <v-app-bar app dense elevation="1" color="white">
    <!-- 編集 -->
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
      // 編集
      homePath: $store.state.loggedIn.homePath,
    }
  },
}
</script>
```

- `front/pages/login.vue`を編集<br>

```vue:login.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <v-form ref="form" v-model="isValid">
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
            :disabled="!isValid || loading"
            :loading="loading"
            block
            color="appblue"
            class="white--text"
            @click="login"
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
      <!-- 編集 -->
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
  // 追加
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
  // ここまで
})

// 算出プロパティ
export const getters = {}

// stateの値を変更する場所
export const mutations = {}

// メソッド
export const actions = {}
```

- `front/pages/projects.vue`を編集<br>

```vue:projects.vue
<template>
  <div>
    <div v-for="(project, i) in recentProjects" :key="`project-${i}`">
      {{ project.name }}
      {{ project.updatedAt }}
    </div>
  </div>
</template>

<script>
export default {
  layout: 'logged-in',
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
```

- `root $ mkdir front/assets/images/logged-in`を実行<br>

- `front/assets/images/logged-in`ディレクトリに`home.png`を配置<br>

* `front/pages/projects.vue`を編集<br>

```vue:projects.vue
<template>
  <!-- 編集 -->
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
              </v-row>
            </v-col>
          </v-row>
        </v-container>
      </v-img>
    </v-parallax>
  </div>
  <!-- ここまで -->
</template>

<script>
import homeImg from '@/assets/images/logged-in/home.png'

export default {
  layout: 'logged-in',
  <!-- 追加 -->
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
        height: 120,
        elevation: 4,
      },
    }
  },
  <!-- ここまで -->
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

<!-- 追加 -->
<style lang="scss">
#projects {
  .v-parallax__content {
    padding: 0;
  }
}
</style>
```

- `front/plugins/my-inject.js`を編集<br>

```js:my-inject.js
class MyInject {
  constructor(ctx) {
    this.app = ctx.app
  }

  pageTitle(routeName) {
    // jsonPath => 'account-settings'
    const jsonPath = `pages.${routeName.replace(/-/g, '.')}`
    const title = this.app.i18n.t(jsonPath)
    return title
  }

  // 追加
  // 日付のフォーマット変換
  dateFormat(dateStr) {
    const dateTimeFormat = new Intl.DateTimeFormat('ja', {
      dateStyle: 'medium',
      timeStyle: 'short',
    })
    return dateTimeFormat.format(new Date(dateStr))
  }

  // プロジェクトリンク
  projectLinkTo(id, name = 'project-id-dashboard') {
    return { name, params: { id } }
  }
  // ここまで
}

export default ({ app }, inject) => {
  inject('my', new MyInject({ app }))
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

                <!-- 追加 -->
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
            <!-- ここまで -->
          </v-row>
        </v-container>
      </v-img>
    </v-parallax>
  </div>
</template>

<script>
import homeImg from '@/assets/images/logged-in/home.png'

export default {
  layout: 'logged-in',
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
        height: 120,
        elevation: 4,
      },
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

    <!-- 追加 -->
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
    <!-- ここまで -->
  </div>
</template>

<script>
import homeImg from '@/assets/images/logged-in/home.png'

export default {
  layout: 'logged-in',
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
        // 編集
        height: 110,
        elevation: 4,
      },
      <!-- 追加 -->
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
      <!-- ここまで -->
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
