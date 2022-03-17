## 45 Nuxt.js からユーザーテーブルを取得する

- `root $ docker compose run --rm api rails destroy controller Api::V1::Hello`を実行<br>

- `root $ docker compose run --rm api rails g controller Api::V1::Users`を実行<br>

* `api/app/controllers/api/v1/users_controller.rb`を編集<br>

```rb:users_controller.rb
class Api::V1::UsersController < ApplicationController
  def index
    users = User.all # as_json => ハッシュの形でSONデータを返す { "id" => 1, "name" => "test", ... }
    render json: users.as_json(only: %i[id name email created_at])
  end
end
```

- `api/config/routes.rb`を編集<br>

```rb:routes.rb
class Api::V1::UsersController < ApplicationController
  def index
    users = User.all # as_json => ハッシュの形でSONデータを返す { "id" => 1, "name" => "test", ... }
    render json: users.as_json(only: %i[id name email created_at])
  end
end
```

- http://localhost:3000/api/v1/users にアクセスしてみる<br>

```
[
    {
        "id": 1,
        "name": "user0",
        "email": "user0@example.com",
        "created_at": "2022-03-16T11:15:58.112+09:00"
    },
    {
        "id": 2,
        "name": "user1",
        "email": "user1@example.com",
        "created_at": "2022-03-16T11:15:58.486+09:00"
    },
    {
        "id": 3,
        "name": "user2",
        "email": "user2@example.com",
        "created_at": "2022-03-16T11:15:58.860+09:00"
    },
    {
        "id": 4,
        "name": "user3",
        "email": "user3@example.com",
        "created_at": "2022-03-16T11:15:59.199+09:00"
    },
    {
        "id": 5,
        "name": "user4",
        "email": "user4@example.com",
        "created_at": "2022-03-16T11:15:59.541+09:00"
    },
    {
        "id": 6,
        "name": "user5",
        "email": "user5@example.com",
        "created_at": "2022-03-16T11:15:59.880+09:00"
    },
    {
        "id": 7,
        "name": "user6",
        "email": "user6@example.com",
        "created_at": "2022-03-16T11:16:00.219+09:00"
    },
    {
        "id": 8,
        "name": "user7",
        "email": "user7@example.com",
        "created_at": "2022-03-16T11:16:00.554+09:00"
    },
    {
        "id": 9,
        "name": "user8",
        "email": "user8@example.com",
        "created_at": "2022-03-16T11:16:00.898+09:00"
    },
    {
        "id": 10,
        "name": "user9",
        "email": "user9@example.com",
        "created_at": "2022-03-16T11:16:01.241+09:00"
    }
]
```

- `front/pages/index.vue`を編集<br>

```vue:index.vue
<template>
  <div>
    <h2>Usersテーブルの取得</h2>
    <table v-if="users.length">
      <thead>
        <tr>
          <th>id</th>
          <th>name</th>
          <th>email</th>
          <th>created_at</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(user, i) in users" :key="`user-${i}`">
          <td>{{ user.id }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>{{ dateFormat(user.created_at) }}</td>
        </tr>
      </tbody>
    </table>

    <div v-else>
      ユーザーが取得できませんでした
    </div>
  </div>
</template>

<script>
export default {
  // asyncData => コンポーネントのデータを表示する前に実行されるメソッド
  // async => promiseを返す(promise => 非同期処理の結果を表示するオブジェクトのこと)
  // await => promiseを返すまでJavaScriptを待機させる
  async asyncData({ $axios }) {
    let users = []
    await $axios.$get('/api/v1/users').then((res) => (users = res))
    return { users }
  },
  // 算出プロパティ => 計算したデータを返す関数のこと
  computed: {
    // "2022-03-16T11:15:58.112+09:00"
    // return '2021/03/01'
    dateFormat() {
      return (date) => {
        const dateTimeFormat = new Intl.DateTimeFormat('ja', {
          dateStyle: 'medium',
          timeStyle: 'short',
        })
        return dateTimeFormat.format(new Date(date))
      }
    },
  },
}
</script>
```

- http://localhost:8080/ にアクセスしてみる<br>

* `front $ git push heroku`を実行<br>

- `api $ git push heroku`を実行<br>

* `api $ heroku run db:migrate`を実行<br>

- `api $ hroku run db:seed`を実行<br>
