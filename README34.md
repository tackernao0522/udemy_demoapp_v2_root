## 96 エラーページをカスタマイズする

- `root $ touch front/layouts/error.vue`を実行<br>

* `front/layouts/error.vue`を編集<br>

```vue:error.vue
<template>
  <v-container fill-height>
    <v-row>
      <v-col cols="12">
        <v-card-title class="justify-center">
          {{ error.statusCode }}
        </v-card-title>
        <v-card-text class="text-center">
          {{ errorMessage }}
        </v-card-text>
        <v-card-actions class="justify-center">
          <v-icon>
            mdi-emoticon-sick-outline
          </v-icon>
        </v-card-actions>
        <v-card-actions class="justify-center">
          <v-btn icon x-large color="appblue" @click="redirect">
            <v-icon>
              mdi-home
            </v-icon>
          </v-btn>
        </v-card-actions>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  props: {
    error: {
      type: Object,
      default: null,
    },
  },
  computed: {
    // ログイン前後でリダイレクトパスを切り替える
    redirectPath() {
      const loggedInHomePath = this.$store.state.loggedIn.homePath
      const beforeLoginHomePath = { name: 'index' }
      return this.$auth.loggedIn() ? loggedInHomePath : beforeLoginHomePath
    },
    // axiosエラーの場合はstatusTextを参照する
    responsedMessage() {
      return this.error.response && this.error.response.statusText
        ? this.error.response.statusText
        : this.error.message
    },
    // i18nに翻訳パスが存在する場合は日本語翻訳メッセージを返す
    errorMessage() {
      const translationPath = `error.${this.responsedMessage}`
      return this.$te(translationPath)
        ? this.$t(translationPath)
        : this.responsedMessage
    },
  },
  async created() {
    // 認証エラーの場合はVuexを初期化する(セッションはサーバで削除済み)
    if (this.error.statusCode === 401) {
      await this.$auth.resetVuex()
    }
  },
  methods: {
    // リダイレクトパスが現在のルートと一致している場合はリロードを行う
    redirect() {
      this.redirectPath.name === this.$route.name
        ? this.$router.go()
        : this.$router.push(this.redirectPath)
    },
  },
}
</script>
```

- `front/locales/js.json`を編集<br>

```json:ja.json
{
  "menus": {
    "about": "サイトについて",
    "products": "製品",
    "price": "価格",
    "contact": "お問い合わせ",
    "company": "会社情報",
    "payments": {
      "month": "月払い",
      "year": "年払い"
    }
  },
  "pages": {
    "signup": "会員登録",
    "login": "ログイン",
    "logout": "ログアウト",
    "account": {
      "settings": "アカウント設定",
      "password": "パスワード変更"
    },
    "project": {
      "id": {
        "dashboard": "ダッシュボード",
        "layouts": "レイアウト",
        "pages": "ページ",
        "components": "コンポーネント",
        "settings": "設定",
        "help": "ヘルプ"
      }
    }
  },
  // 追加
  "error": {
    "Unauthorized": "認証に失敗しました",
    "Forbidden": "アクセスが拒否されました",
    "This page could not be found": "ページが見つかりません",
    "Network Error": "ネットワークエラーが発生しました"
  }
  // ここまで
}
```

## 97 Heroku にデプロイして、本番環境でログインしてみる

1. api エラー処理メソッドの作成<br>

2. ログ出力の制限<br>

3. Heroku デプロイ<br>

- `front/plugins/my-inject.js`を編集<br>

```js:my-inject.js
class MyInject {
  constructor(ctx) {
    this.app = ctx.app
    // 追加
    this.error = ctx.error
  }

  pageTitle(routeName) {
    // jsonPath => 'account-settings'
    const jsonPath = `pages.${routeName.replace(/-/g, '.')}`
    const title = this.app.i18n.t(jsonPath)
    return title
  }

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

  // 追加
  // apiエラーハンドラー
  apiErrorHandler(response) {
    // ネットワークエラーの場合はresponseが存在しないので500を代入
    const statusCode = response ? response.status : 500
    const message = response ? response.statusText : 'Network Error'
    return this.error({ statusCode, message })
  }
}

// 編集
export default ({ app, error }, inject) => {
  inject('my', new MyInject({ app, error }))
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
      redirectPath: $store.state.loggedIn.rememberPath,
      loggedInHomePath: $store.state.loggedIn.homePath,
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
    authSuccessful(response) {
      this.$auth.login(response)
      this.$router.push(this.redirectPath)
      // 記憶ルートを初期値に戻す
      this.$store.dispatch('getRememberPath', this.loggedInHomePath)
    },
    authFailure({ response }) {
      if (response && response.status === 404) {
        const msg = 'ユーザーが見つかりません😢'
        return this.$store.dispatch('getToast', { msg })
      }
      <!-- 編集 -->
      return this.$my.apiErrorHandler(response)
    },
  },
}
</script>
```

- `front/plugins/axios.js`を編集<br>

```js:axios.js
// 編集 isDevを追加
export default ({ $axios, $auth, isDev }) => {
  // リクエストログ
  $axios.onRequest((config) => {
    config.headers.common['X-Requested-With'] = 'XMLHttpRequest'
    if ($auth.token) {
      config.headers.common.Authorization = `Bearer ${$auth.token}`
    }
    // 編集
    if (isDev) {
      // eslint-disable-next-line no-console
      console.log(config)
    }
  })
  // レスポンスログ
  $axios.onResponse((config) => {
    if (isDev) {
      // eslint-disable-next-line no-console
      console.log(config)
    }
    // ここまで
  })
  // エラーログ
  $axios.onError((e) => {
    // eslint-disable-next-line no-console
    console.log(e.response)
  })
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
      <!-- 編集 ''空にする -->
      params: { auth: { email: '', password: '' } },
      redirectPath: $store.state.loggedIn.rememberPath,
      loggedInHomePath: $store.state.loggedIn.homePath,
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
    authSuccessful(response) {
      this.$auth.login(response)
      this.$router.push(this.redirectPath)
      // 記憶ルートを初期値に戻す
      this.$store.dispatch('getRememberPath', this.loggedInHomePath)
    },
    authFailure({ response }) {
      if (response && response.status === 404) {
        const msg = 'ユーザーが見つかりません😢'
        return this.$store.dispatch('getToast', { msg })
      }
      return this.$my.apiErrorHandler(response)
    },
  },
}
</script>
```

- front ディレクトリをコミットしておく<br>

* front ディレクトリを heorku に push しておく `$ front $ git push heroku && heroku open`<br>

- `api $ heroku open`を実行<br>

* `api/Gemfile`を修正<br>

```:Gemfile
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails', branch: 'main'
gem 'rails', '6.1.3.1' # 修正
# Use postgresql as the database for Active Record
gem 'pg', '>= 0.18', '< 2.0'
# Use Puma as the app server
gem 'puma', '~> 4.1'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
# gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'

# コンソールの出力結果を見やすく表示する
gem 'hirb', '~> 0.7.3'

# Hirbの文字列補正を行う
gem 'hirb-unicode-steakknife', '~> 0.0.9'

# パスワードを暗号化する
gem 'bcrypt', '~> 3.1', '>= 3.1.16'

# jwt Doc: https://rubygems.org/gems/jwt
gem 'jwt', '~> 2.3'

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
end

group :development do
  gem 'listen', '~> 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

group :test do
  # テスト結果色付け Doc: https://github.com/kern/minitest-reporters
  gem 'minitest-reporters', '~> 1.1', '>= 1.1.11'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```
