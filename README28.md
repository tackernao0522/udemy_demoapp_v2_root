## 84 認証モジュールのテストをする

- `root $ docker compose run --rm api rails g controller Api::V1::Projects`を実行<br>

* `api/app/controllers/api/v1/projects_controller.rb`を編集<br>

```rb:projects_controller.rb
class Api::V1::ProjectsController < ApplicationController
  before_action :authenticate_active_user

  def index
    projects = []
    date = Date.new(2021, 4, 1)
    10.times do |n|
      id = n + 1
      name = "#{current_user.name} project #{id.to_s.rjust(2, '0')}"
      updated_at = date + (id * 6).hours
      projects << { id: id, name: name, updatedAt: updated_at }
    end # 本来はcurrent_user.projects

    render json: projects
  end
end
```

- `api/config/routes.rb`を編集<br>

```rb:routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      ## aauth_token
      resources :auth_token, only: %i[create] do
        post :refresh, on: :collection
        delete :destroy, on: :collection
      end

      # 追加
      # projects
      resources :projects, only: %i[index]
    end
  end
end
```

- `root $ docker compose run --rm api rails g integration_test Authentication`を実行<br>

* `api/test/integration/authentication_test.rb`を編集<br>

```rb:authentication_test.rb
require 'test_helper'

class AuthenticationTest < ActionDispatch::IntegrationTest
  def setup
    @user = active_user
    @params = { auth: { email: @user.email, password: 'password' } }
    @access_lifetime = UserAuth.access_token_lifetime
    @session_key = UserAuth.session_key.to_s
    @access_token_key = 'token'
  end

  # プロジェクトapi
  def projects_api(token)
    get api('/projects'), xhr: true, headers: auth(token)
  end

  # 認証メソッドテスト
  test 'authenticate_user_method' do
    login(@params)
    access_token = res_body[@access_token_key]

    # 有効なtokenでアクセスできているか
    projects_api(access_token)
    assert_response 200
    assert response.body.present?

    # 有効期限切れは401が返っているか
    travel_to (@access_lifetime.from_now) do
      # アクセス前のcookieは存在するか
      assert_not cookies[@session_key].blank?

      # レスポンスは想定通りか
      projects_api(access_token)
      assert_response 401
      assert_not response.body.present?

      # cookieは削除されているか
      assert cookies[@session_key].blank?
    end

    # 不正なtokenが投げられた場合
    invalid_token = 'a.' + access_token
    projects_api(invalid_token)

    assert_response 401
    assert_not response.body.present?
  end
end
```

- `$ docker compose run --rm api rails t test/integration`を実行<br>

```
users...---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=---=] 0% Time: 00:00:00,  ETA: ??:??:??
users...
users = 10
users = 10
  9/9: [==============================================================================================================================================================================] 100% Time: 00:00:04, Time: 00:00:04

Finished in 4.10767s
9 tests, 57 assertions, 0 failures, 0 errors, 0 skips
```
