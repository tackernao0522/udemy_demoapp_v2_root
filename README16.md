## 56 親子コンポーネント間の双方向バインディングを理解する

親のデータを、子コンポーネントで編集し、更新する<br>

`signup.vue`<br>

```vue:signup.vue
{ name: '', email: '', password: '' }
```

↓
`UserFormName.vue`<br>

```vue:UserFormName.vue
aaa
```

`UserFormEmail.vue`<br>

```vue:UserFormEmail.vue
aaa
```

`UserFormPassword.vue`<br>

```vue:UserFormPassword.vue
aaa
```

`signup.vue`<br>

```vue:signup.vue
{ name: 'aaa', email: 'aaa', password: 'aaa' }
```

### 双方向バインディングとは何か？

`sample.html`<br>

```html:sample.html
aaa
```

1. データを更新<br>

`js:sample.js`<br>

```js:sample.js
{
  name: 'bbb'
}
```

2. 更新データを描写<br>

`sample.html`<br>

```html:sample.html
bbb
```

フォームの入力値を、即座にデータと同期させる、仕組みのこと<br>

### 双方向バインディングの方法

`JavaScript`<br>

```
data () {
  return {
    name: ''
  }
}
```

`HTML`<br>

```
<template>
  <input
    v-model="name"
  />
  <p>{{ name }}</p>
</template>
```

ただし、親のデータは、子コンポーネントで更新できない。<br>

### 親子コンポーネント間の双方向バインディングの方法<br>

`親コンポーネント`<br>
(1. v-bind で子コンポーネントにデータ送信)<br>
↓<br>
`子コンポーネント`<br>
(2. props でデータ受け取り)<br>
(3. データ描写)<br>
(4. \$emit()で更新データを親コンポーネントに送信)<br>
↓<br>
`親コンポーネント`<br>
(5. v-on で更新データ受け取り)<br>
(6. 更新データを代入)<br>

### ハンズオン

- `front/pages/signup.vue`を編集<br>

```vue:signup.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <v-form v-model="isValid">
        <!-- 編集 -->
        <user-form-name :name="name" @input="name = $event" />
        <!-- 確認後削除 -->
        name => {{ name }}
        <!-- ここまで -->
        <user-form-email />
        <user-form-password />
        <!-- disabled=true => ボタンクリックを無効にする -->
        <v-btn :disabled="!isValid" block color="appblue" class="white--text">
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
  data () {
    return {
      <!-- 追加 -->
      name: '',
      isValid: false
    }
  }
}
</script>
```

- `front/components/User/UserFormName.vue`を編集<br>

```vue:UserFormName.vue
<template>
  <!-- v-bind:value="name" => 描写 -->
  <!-- v-on:input => インプットがあった時に発火する -->
  <!-- $emit('キー(自由に決められる)', $event) => 子から親へデータを送信 -->
  <v-text-field
    :value="name"
    label="ユーザー名を入力"
    placeholder="あなたの表示名"
    outlined
    @input="$emit('input', $event)"
  />
</template>

<script>
export default {
  <!-- 追加 -->
  props: {
    name: {
      type: String,
      default: ''
    }
  }
  <!-- ここまで -->
}
</script>
```

- `front/pages/signup.vue`上記コードを省略形で書く<br>

```vue:signup.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <v-form v-model="isValid">
        <!-- 編集 -->
        <user-form-name :name.sync="name" />
        name => {{ name }}
        <user-form-email />
        <user-form-password />
        <!-- disabled=true => ボタンクリックを無効にする -->
        <v-btn :disabled="!isValid" block color="appblue" class="white--text">
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
    }
  },
}
</script>
```

- `front/components/User/UserFormName.vue`上記を省略形で書いた場合は編集<br>

```vue:UserFormName.vue
<template>
  <!-- v-bind:value="name" => 描写 -->
  <!-- v-on:input => インプットがあった時に発火する -->
  <!-- $emit('キー(自由に決められる)', $event) => 子から親へデータを送信 -->
  <!-- syncを使った場合、$emit('update:name', $event)とする -->
  <v-text-field
    :value="name"
    label="ユーザー名を入力"
    placeholder="あなたの表示名"
    outlined
    <!-- 編集 -->
    @input="$emit('update:name', $event)"
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
}
</script>
```

- `front/components/User/UserFormName.vue`を更に別の書き方<br>

```vue:UserFormName.vue
<template>
  <!-- v-bind:value="name" => 描写 -->
  <!-- v-on:input => インプットがあった時に発火する -->
  <!-- $emit('キー(自由に決められる)', $event) => 子から親へデータを送信 -->
  <!-- syncを使った場合、$emit('update:name', $event)とする -->
  <!-- 編集 -->
  <v-text-field
    v-model="setName"
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
  <!-- ここまで -->
}
</script>
```

### まとめ

1. 双方向バインディングとは、フォームの入力値・選択値を、即座にデータと同期させる仕組み<br>
2. 親コンポーネントのデータを、子コンポーネントで更新することはできない<br>
3. データの受け渡し方法<br>
   &nbsp;&nbsp;&nbsp;&nbsp;+ 親) v-bind:キー -> 子) props: { キー }<br>
   &nbsp;&nbsp;&nbsp;&nbsp;+ 子) $emit('イベント名', $event) => 親) v-on:イベント名, \$event<br>
4. 親に sync 修飾子を使えば、データの送受信を一度に行える<br>
5. 親からの sync 修飾子には、$emit('update:バインドキー', $event)で送信する<br>
6. 子で v-model を使用するには、computed の get()と set()を使用する<br>

## 57 会員登録フォームの双方向バインディングを実装

- `front/pages/signup.vue`を編集<br>

```vue:signup.vue
<template>
  <user-form-card>
    <template #user-form-card-content>
      <v-form v-model="isValid">
        <!-- 編集 -->
        <user-form-name :name.sync="params.user.name" />
        <user-form-email :email.sync="params.user.email" />
        <user-form-password :password.sync="params.user.password" />
        <!-- ここまで -->
        <!-- disabled=true => ボタンクリックを無効にする -->
        <v-btn :disabled="!isValid" block color="appblue" class="white--text">
          登録する
        </v-btn>
      </v-form>
      <!-- 確認後削除 -->
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
      <!-- 追加 -->
      params: { user: { name: '', email: '', password: '' } },
    }
  },
}
</script>
```

- `front/components/User/UserFormEmail.vue`を編集<br>

```vue:UserFormEmail.vue
<template>
  <v-text-field
    v-model="setEmail"
    label="メールアドレスを入力"
    placeholder="your@email.com"
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

- `front/components/User/UserFormPassword.vue`を編集<br>

```vue:UserFormPassword.vue
<template>
  <v-text-field
    v-model="setPassword"
    label="パスワードを入力"
    placeholder="8文字以上"
    outlined
  />
</template>

<script>
export default {
  props: {
    password: {
      type: String,
      default: '',
    },
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
  },
}
</script>
```
