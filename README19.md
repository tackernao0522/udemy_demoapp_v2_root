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
