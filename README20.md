## 65 動的なルートに表示する個別プロジェクトページを作成する

### Vue.js で動的なルーティングを生成するには？

アンダーバーを付けた Vue ファイル、もしくはディレクトリを用意する<br>

`_id.vue` or `_id`<br>

名前は params のキーと一致させる<br>
`params: { id: 1 }<br>

### ハンズオン

- `root $ mkdir -p front/pages/project/_id && touch $_/{dashboard.vue,layouts.vue,pages.vue,components.vue,settings.vue,help.vue}`を実行<br>

- `root $ tree front/pages/project`を実行して確認<br>

```:terminal
front/pages/project
└── _id
    ├── components.vue
    ├── dashboard.vue
    ├── help.vue
    ├── layouts.vue
    ├── pages.vue
    └── settings.vue
```

- `root $ touch front/layouts/project.vue`を実行<br>

- `root $ touch front/pages/project.vue`を実行<br>

* `front/layouts/project.vue`を編集<br>

```vue:project.vue
<template>
  <v-app>
    <logged-in-app-bar />
    <v-main>
      project.vue
      <Nuxt />
    </v-main>
  </v-app>
</template>

<script>
import LoggedInAppBar from '../components/LoggedIn/LoggedInAppBar.vue'
export default {
  components: { LoggedInAppBar },
}
</script>
```

- `front/pages/project.vue`を編集<br>

```vue:project.vue
<template>
  <v-container>
    <nuxt-child />
  </v-container>
</template>

<script>
export default {
  latyout: 'project',
  validate({ route }) {
    return route.name !== 'project'
  },
}
</script>
```

- `front/pages/project/_id/components.vue`を編集<br>

```vue:components.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project_id/dashboard.vue`を編集<br>

```vue:dashboard.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/help.vue`を編集<br>

```vue:help.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/layouts.vue`を編集<br>

```vue:layuts.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/pages.vue`を編集<br>

```vue:pages.vue
<template>
  <div>
    {{ $route.fullPath }}
  </div>
</template>

<script>
export default {}
</script>
```

- `front/pages/project/_id/settings.vue`を編集<br>

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
