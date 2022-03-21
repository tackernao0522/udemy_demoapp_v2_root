## 52 [ウェルカムページ 3/4] スクロールイベントでツールバーの色を変化させる

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <!-- 編集 -->
    <home-app-bar :menus="menus" :img-height="imgHeight" />
    <v-img
      dark
      src="https://picsum.photos/id/20/1920/1080?blur=5"
      gradient="to top right, rgba(19,84,122,.6), rgba(128,208,199,.9)"
      :height="imgHeight"
    >
      <v-row
        align="center"
        justify="center"
        :style="{ height: `${imgHeight}px` }"
      >
        <v-col cols="12" class="text-center">
          <h1 class="display-1 mb-4">
            未来を作ろう。ワクワクしよう。
          </h1>
          <h4 class="subheading" :style="{ letterSpacing: '5px' }">
            中小企業に特化した事業計画策定ツール
          </h4>
        </v-col>
      </v-row>
    </v-img>
    <v-sheet>
      <v-container fluid :style="{ maxWidth: '1280px' }">
        <v-row v-for="(menu, i) in menus" :key="`menu-${i}`">
          <v-col cols="12">
            <v-card flat>
              <v-card-title class="justify-center display-1">
                {{ $t(`menus.${menu.title}`) }}
              </v-card-title>
              <v-card-text class="text-center">
                {{ menu.subtitle }}
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12">
            <!-- home-about, home-company ... -->
            <div :is="`Home-${menu.title}`" />
          </v-col>
        </v-row>
      </v-container>
    </v-sheet>
    <app-footer />
  </v-app>
</template>

<script>
// isを使ったときはimport文が必要になる
import AppFooter from '../components/App/AppFooter.vue'
import HomeAbout from '~/components/Home/HomeAbout'
import HomeProducts from '~/components/Home/HomeProducts'
import HomePrice from '~/components/Home/HomePrice'
import HomeContact from '~/components/Home/HomeContact'
import HomeCompany from '~/components/Home/HomeCompany'

export default {
  components: {
    HomeAbout,
    HomeProducts,
    HomePrice,
    HomeContact,
    HomeCompany,
    AppFooter,
  },
  data() {
    return {
      imgHeight: 500,
      menus: [
        {
          title: 'about',
          subtitle:
            'このサイトはブログ"独学プログラマ"で公開されているチュートリアルのデモアプリケーションです',
        },
        { title: 'products', subtitle: '他にはない優れた機能の数々' },
        { title: 'price', subtitle: '会社の成長に合わせた3つのプラン' },
        { title: 'contact', subtitle: 'お気軽にご連絡を' },
        { title: 'company', subtitle: '私たちの会社' },
      ],
    }
  },
}
</script>
```

- `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar app dark>
    <app-logo />
    <v-toolbar-title />
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn v-for="(menu, i) in menus" :key="`menu-btn-${i}`" text>
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
    <!-- 追加 -->
    imgHeight: {
      type: Number,
      default: 0,
    },
    <!-- ここまで -->
  },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName } }) {
    return {
      appName,
    }
  },
}
</script>
```

- `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar app dark>
    <app-logo />
    <v-toolbar-title />
    <!-- 追加 -->
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn v-for="(menu, i) in menus" :key="`menu-btn-${i}`" text>
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
    <!-- ここまで -->
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
  },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName } }) {
    return {
      appName,
    }
  },
}
</script>
```

- `front/components/Home/HomeAppBar.vue`を再編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar app dark>
    <app-logo />
    <v-toolbar-title />
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn v-for="(menu, i) in menus" :key="`menu-btn-${i}`" text>
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
    <!-- 追加(確認した後削除) -->
    {{ scrollY }}
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
    imgHeight: {
      type: Number,
      default: 0,
    },
  },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName } }) {
    return {
      appName,
      <!-- 追加 -->
      scrollY: 0,
    }
  },
  <!-- 追加 -->
  // Vue.new() => Vueインスタンス
  // マウント => Vueの実行準備が完全に整った後
  mounted() {
    window.addEventListener('scroll', this.onScroll)
  },
  // Vueインスタンスが破壊される前に実行される
  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
  },
  methods: {
    onScroll() {
      this.scrollY = window.scrollY
    },
  },
  <!-- ここまで -->
}
</script>
```

- `root $ touch front/store/index.js`を実行<br>

* `front/store/index.js`を編集<br>

```js:index.js
// 変数
export const state = () => ({
  styles: {
    homeAppBarHeight: 56,
  },
})

// 算出プロパティ
export const getters = {}

// stateの値を変更する場所
export const mutations = {}

// メソッド
export const actions = {}
```

- `front/components/Home/HomeAppBar.vue`を編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar app dark>
    <app-logo />
    <v-toolbar-title />
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn v-for="(menu, i) in menus" :key="`menu-btn-${i}`" text>
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
    <!-- 追加(確認後削除) -->
    {{ isScrollPoint }}
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
    imgHeight: {
      type: Number,
      default: 0,
    },
  },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName }, $store }) {
    return {
      appName,
      scrollY: 0,
      homeAppBarHeight: $store.state.styles.homeAppBarHeight,
    }
  },
  <!-- 追加 -->
  computed: {
    isScrollPoint() {
      // 500 -56 = 444px超の場合、trueを返す
      return this.scrollY > this.imgHeight - this.homeAppBarHeight
    },
  },
  <!-- ここまで -->
  // Vue.new() => Vueインスタンス
  // マウント => Vueの実行準備が完全に整った後
  mounted() {
    window.addEventListener('scroll', this.onScroll)
  },
  // Vueインスタンスが破壊される前に実行される
  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
  },
  methods: {
    onScroll() {
      this.scrollY = window.scrollY
    },
  },
}
</script>
```

- `front/components/Home/HomeAppBar.vue`を再編集<br>

```vue:HomeAppBar.vue
<template>
  <!-- 編集 -->
  <v-app-bar
    app
    :dark="!isScrollPoint"
    :height="homeAppBarHeight"
    :color="toolbarStyle.color"
    :elevation="toolbarStyle.elevation"
  >
    <!-- ここまで -->
    <app-logo />
    <v-toolbar-title />
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn v-for="(menu, i) in menus" :key="`menu-btn-${i}`" text>
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
    imgHeight: {
      type: Number,
      default: 0,
    },
  },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName }, $store }) {
    return {
      appName,
      scrollY: 0,
      homeAppBarHeight: $store.state.styles.homeAppBarHeight,
    }
  },
  computed: {
    isScrollPoint() {
      // 500 - 56 = 444px超の場合、trueを返す
      return this.scrollY > this.imgHeight - this.homeAppBarHeight
    },
    <!-- 追加 -->
    toolbarStyle() {
      const color = this.isScrollPoint ? 'white' : 'transparent'
      const elevation = this.isScrollPoint ? 4 : 0
      return { color, elevation }
    },
    <!-- ここまで -->
  },
  // Vue.new() => Vueインスタンス
  // マウント => Vueの実行準備が完全に整った後
  mounted() {
    window.addEventListener('scroll', this.onScroll)
  },
  // Vueインスタンスが破壊される前に実行される
  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
  },
  methods: {
    onScroll() {
      this.scrollY = window.scrollY
    },
  },
}
</script>
```

- `front/components/Home/HomeAppBar.vue`を再編集<br>

```vue:HomeAppBar.vue
<template>
  <v-app-bar
    app
    :dark="!isScrollPoint"
    :height="homeAppBarHeight"
    :color="toolbarStyle.color"
    :elevation="toolbarStyle.elevation"
  >
    <!-- 編集 -->
    <app-logo @click.native="$vuetify.goTo('#scroll-top')" />
    <v-toolbar-title />
    <v-spacer />

    <v-toolbar-items class="ml-2">
      <v-btn
        v-for="(menu, i) in menus"
        :key="`menu-btn-${i}`"
        text
        <!--
        編集
        --
      >
        @click="$vuetify.goTo(`#${menu.title}`)" >
        {{ $t(`menus.${menu.title}`) }}
      </v-btn>
    </v-toolbar-items>
  </v-app-bar>
</template>

<script>
import AppLogo from '../App/AppLogo.vue'
export default {
  components: { AppLogo },
  props: {
    menus: {
      type: Array,
      default: () => [],
    },
    imgHeight: {
      type: Number,
      default: 0,
    },
  },
  // data (context: { $config: { appNeme: "Value(BizPlanner)"} })
  data({ $config: { appName }, $store }) {
    return {
      appName,
      scrollY: 0,
      homeAppBarHeight: $store.state.styles.homeAppBarHeight,
    }
  },
  computed: {
    isScrollPoint() {
      // 500 - 56 = 444px超の場合、trueを返す
      return this.scrollY > this.imgHeight - this.homeAppBarHeight
    },
    toolbarStyle() {
      const color = this.isScrollPoint ? 'white' : 'transparent'
      const elevation = this.isScrollPoint ? 4 : 0
      return { color, elevation }
    },
  },
  // Vue.new() => Vueインスタンス
  // マウント => Vueの実行準備が完全に整った後
  mounted() {
    window.addEventListener('scroll', this.onScroll)
  },
  // Vueインスタンスが破壊される前に実行される
  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
  },
  methods: {
    onScroll() {
      this.scrollY = window.scrollY
    },
  },
}
</script>
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <v-app>
    <home-app-bar :menus="menus" :img-height="imgHeight" />
    <v-img <!-- 追加 -->
      id="scroll-top" dark src="https://picsum.photos/id/20/1920/1080?blur=5"
      gradient="to top right, rgba(19,84,122,.6), rgba(128,208,199,.9)"
      :height="imgHeight" >
      <v-row
        align="center"
        justify="center"
        :style="{ height: `${imgHeight}px` }"
      >
        <v-col cols="12" class="text-center">
          <h1 class="display-1 mb-4">
            未来を作ろう。ワクワクしよう。
          </h1>
          <h4 class="subheading" :style="{ letterSpacing: '5px' }">
            中小企業に特化した事業計画策定ツール
          </h4>
        </v-col>
      </v-row>
    </v-img>
    <v-sheet>
      <v-container fluid :style="{ maxWidth: '1280px' }">
        <v-row v-for="(menu, i) in menus" :key="`menu-${i}`">
          <!-- 編集 -->
          <v-col :id="menu.title" cols="12">
            <!-- ここまで -->
            <v-card flat>
              <v-card-title class="justify-center display-1">
                {{ $t(`menus.${menu.title}`) }}
              </v-card-title>
              <v-card-text class="text-center">
                {{ menu.subtitle }}
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12">
            <!-- home-about, home-company ... -->
            <div :is="`Home-${menu.title}`" />
          </v-col>
        </v-row>
      </v-container>
    </v-sheet>
    <app-footer />
  </v-app>
</template>

<script>
// isを使ったときはimport文が必要になる
import AppFooter from '../components/App/AppFooter.vue'
import HomeAbout from '~/components/Home/HomeAbout'
import HomeProducts from '~/components/Home/HomeProducts'
import HomePrice from '~/components/Home/HomePrice'
import HomeContact from '~/components/Home/HomeContact'
import HomeCompany from '~/components/Home/HomeCompany'

export default {
  components: {
    HomeAbout,
    HomeProducts,
    HomePrice,
    HomeContact,
    HomeCompany,
    AppFooter,
  },
  data() {
    return {
      imgHeight: 500,
      menus: [
        {
          title: 'about',
          subtitle:
            'このサイトはブログ"独学プログラマ"で公開されているチュートリアルのデモアプリケーションです',
        },
        { title: 'products', subtitle: '他にはない優れた機能の数々' },
        { title: 'price', subtitle: '会社の成長に合わせた3つのプラン' },
        { title: 'contact', subtitle: 'お気軽にご連絡を' },
        { title: 'company', subtitle: '私たちの会社' },
      ],
    }
  },
}
</script>
```
