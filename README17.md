## 58 Vuetify を使った会員登録フォームのバリデーション設定

- `front/components/User/UserFormName.vue`を編集<br>

```vue:UserFormName.vue
<template>
  <!-- v-bind:value="name" => 描写 -->
  <!-- v-on:input => インプットがあった時に発火する -->
  <!-- $emit('キー(自由に決められる)', $event) => 子から親へデータを送信 -->
  <!-- syncを使った場合、$emit('update:name', $event)とする -->
  <!-- 編集 -->
  <v-text-field
    v-model="setName"
    :rules="rules"
    :counter="max"
    label="ユーザー名を入力"
    placeholder="あなたの表示名"
    outlined
  />
</template>

<script>
export default {
  props: {
    name: {
      type: String,
      default: '',
    },
  },
  <!-- 追加 -->
  data() {
    const max = 30
    return {
      max,
      rules: [
        // 入力必須
        (v) => !!v || '',
        // 30文字制限
        (v) => (!!v && max >= v.length) || `${max}文字以内で入力してください`,
      ],
    }
  },
  <!-- ここまで -->
  computed: {
    setName: {
      get() {
        return this.name
      },
      set(newValue) {
        return this.$emit('update:name', newValue)
      },
    },
  },
}
</script>
```

- `front/components/User/UserFormEmail.vue`を編集<br>

```vue:UserFormEmail.vue
<template>
  <!-- 編集 -->
  <v-text-field
    v-model="setEmail"
    :rules="rules"
    label="メールアドレスを入力"
    :placeholder="placeholder ? 'your@email.com' : undefined"
    outlined
  />
</template>

<script>
export default {
  props: {
    email: {
      type: String,
      default: '',
    },
    <!-- 追加 -->
    placeholder: {
      type: Boolean,
      default: false,
    },
  },
  <!-- 追加 -->
  data() {
    return {
      rules: [
        // 入力必須
        (v) => !!v || '',
        // 書式チェック
        (v) => /.+@.+\..+/.test(v) || '',
      ],
    }
  },
  computed: {
    setEmail: {
      get() {
        return this.email
      },
      set(newValue) {
        return this.$emit('update:email', newValue)
      },
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
      <v-form v-model="isValid">
        <user-form-name :name.sync="params.user.name" />
        <!-- 編集 -->
        <user-form-email :email.sync="params.user.email" placeholder />
        <user-form-password :password.sync="params.user.password" />
        <!-- disabled=true => ボタンクリックを無効にする -->
        <v-btn :disabled="!isValid" block color="appblue" class="white--text">
          登録する
        </v-btn>
      </v-form>
      {{ params }}
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
      params: { user: { name: '', email: '', password: '' } },
    }
  },
}
</script>
```

- `front/components/User/UserFormPassword.vue`を編集<br>

```vue:UserFormPassword.vue
<template>
  <!-- 編集 -->
  <v-text-field
    v-model="setPassword"
    :rules="form.rules"
    :hint="form.hint"
    label="パスワードを入力"
    :placeholder="form.placeholder"
    :hide-details="!setValidation"
    :counter="setValidation"
    :append-icon="toggle.icon"
    :type="toggle.type"
    outlined
    autocomplete="on"
    @click:append="show = !show"
  />
</template>

<script>
export default {
  props: {
    password: {
      type: String,
      default: '',
    },
    setValidation: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      show: false,
    }
  },
  computed: {
    setPassword: {
      get() {
        return this.password
      },
      set(newValue) {
        return this.$emit('update:password', newValue)
      },
    },
    form() {
      const min = '8文字以上'
      const msg = `${min}。半角英数字・ハイフン・アンダーバーが使えます`
      // ログインページ = 入力必須
      // 会員登録ページ = 入力必須, 8文字以上、72文字以下、書式チェック
      const required = (v) => !!v || ''
      const format = (v) => /^[\w-]{8,72}$/.test(v) || msg

      const rules = this.setValidation ? [format] : [required]
      const hint = this.setValidation ? msg : undefined
      const placeholder = this.setValidation ? min : undefined

      return { rules, hint, placeholder }
    },
    <!-- 追加 -->
    toggle() {
      const icon = this.show ? 'mdi-eye' : 'mdi-eye-off'
      const type = this.show ? 'text' : 'password'
      return { icon, type }
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
      <v-form v-model="isValid">
        <user-form-name :name.sync="params.user.name" />
        <user-form-email :email.sync="params.user.email" placeholder />
        <!-- 編集 -->
        <user-form-password
          :password.sync="params.user.password"
          set-validation
        />
        <!-- disabled=true => ボタンクリックを無効にする -->
        <v-btn :disabled="!isValid" block color="appblue" class="white--text">
          登録する
        </v-btn>
      </v-form>
      {{ params }}
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
      params: { user: { name: '', email: '', password: '' } },
    }
  },
}
</script>
```

## 59 ログインページを作成

https://vuetifyjs.com/ja/components/forms/ <br>

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
  data() {
    return {
      name: '',
      isValid: false,
      loading: false,
      params: { user: { email: '', password: '' } },
    }
  },
  methods: {
    login() {
      this.loading = true
      setTimeout(() => (this.loading = false), 1500)
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
      <!-- 編集 -->
      <v-form ref="form" v-model="isValid">
        <user-form-name :name.sync="params.user.name" />
        <user-form-email :email.sync="params.user.email" placeholder />
        <user-form-password
          :password.sync="params.user.password"
          set-validation
        />
        <!-- disabled=true => ボタンクリックを無効にする -->
        <!-- 編集 -->
        <v-btn
          :disabled="!isValid || loading"
          :loading="loading"
          block
          color="appblue"
          class="white--text"
          @click="signup"
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
  <!-- 編集 -->
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
