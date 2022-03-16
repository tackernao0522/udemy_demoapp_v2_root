## 41 ユーザーテーブルに Seed データを投入する

- `root $ mkdir api/db/seeds && mkdir $_/{test,development,production}`を実行<br>

* `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> Rails.root`を実行<br>

```:terminal
irb(main):001:0> Rails.root
=> #<Pathname:/app>
```

- `irb(main):002:0> Rails.env`を実行<br>

```:terminal
irb(main):002:0> Rails.env
=> "development"
```

- `irb(main):003:0> table_name = "users"`を実行<br>

* `irb(main):005:0> path = Rails.root.join("db/seeds/#{Rails.env}/#{table_name}.rb")`を実行<br>

- `irb(main):006:0> path`を実行<br>

```:terminal
irb(main):006:0> path
=> #<Pathname:/app/db/seeds/development/users.rb>
```

- `api/db/seeds.rb`を編集<br>

```rb:seeds.rb
table_names = %w[users]

table_names.each do |table_name|
  path = Rails.root.join("db/seeds/#{Rails.env}/#{table_name}.rb")

  # ファイルが存在しない場合はdevelopmentディレクトリを読み込む
  path = path.sub(Rails.env, 'development') unless File.exist?(path)

  puts "#{table_name}..."
  require path
end
```

- `root $ touch api/db/seeds/development/users.rb`を実行<br>

* `api/db/seeds/development/users.rb`を編集<br>

```rb:users.rb
10.times do |n|
  name = "user#{n}"
  email = "#{name}@example.com" # オブジェクトが存在しない => User.new(email: email, activated: true)
  user = User.find_or_initialize_by(email: email, activated: true)

  if user.new_record?
    user.name = name
    user.password = 'password'
    user.save!
  end
end

puts "users = #{User.count}"
```

- `root $ docker compose run --rm api rails db:reset`を実行<br>

```:terminal
Dropped database 'app_development'
Dropped database 'app_test'
Created database 'app_development'
Created database 'app_test'
users...
users = 10
```

- `root $ docker compose run --rm api rails c`を実行<br>

* `irb(main):001:0> User.all`を実行<br>

```:terminal
  User Load (7.3ms)  SELECT "users".* FROM "users"
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
| id | name  | email             | password_digest                                              | activated | admin | created_at                | updated_at                |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
| 1  | user0 | user0@example.com | $2a$12$Chf4VJmO6Z8cqTixnWFAF.S/VD.EWI9jmtr9Ll54AyxgmakRVtY7q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-03-16 11:15:58 +0900 |
| 2  | user1 | user1@example.com | $2a$12$1oXVRoizciMz8PHqBe3pGOcmMm4B5EMKarznybSFXaseZuGSx/4/q | true      | false | 2022-03-16 11:15:58 +0900 | 2022-03-16 11:15:58 +0900 |
| 3  | user2 | user2@example.com | $2a$12$qWr3jkZ32XQRN.md47LgdeHIqN4kQUS49V22w177vm42X6xTWQDKi | true      | false | 2022-03-16 11:15:58 +0900 | 2022-03-16 11:15:58 +0900 |
| 4  | user3 | user3@example.com | $2a$12$SGahKd7XsR9EAHmEqs3GG.HSk215gX9J80fj8WIkwMYf6J5iyc0p. | true      | false | 2022-03-16 11:15:59 +0900 | 2022-03-16 11:15:59 +0900 |
| 5  | user4 | user4@example.com | $2a$12$JvZvuZjNA5k04iA2dT3jA.EiX.3En0lpVNDBKVF4RVtGXUIL6lkjG | true      | false | 2022-03-16 11:15:59 +0900 | 2022-03-16 11:15:59 +0900 |
| 6  | user5 | user5@example.com | $2a$12$m/J0Ba9vyIpqvJetQnWT1uIDp6DlFDkrVZEt4mPs6yOWMTqbT0HUC | true      | false | 2022-03-16 11:15:59 +0900 | 2022-03-16 11:15:59 +0900 |
| 7  | user6 | user6@example.com | $2a$12$tAy8BbMsNbEpS35.stpFE.izhi0dJYHpVlLxYaUpehqtdnu3SdFvC | true      | false | 2022-03-16 11:16:00 +0900 | 2022-03-16 11:16:00 +0900 |
| 8  | user7 | user7@example.com | $2a$12$upMxWhkM6UzT0.fzut9QQOyVZncw65IZcUT50clYnFSkKtF3JKEdq | true      | false | 2022-03-16 11:16:00 +0900 | 2022-03-16 11:16:00 +0900 |
| 9  | user8 | user8@example.com | $2a$12$V/9ylRZB3isFhUjUmI02Yu1buReh5iVQe7pJPGvtYoTFn1/eUDzkW | true      | false | 2022-03-16 11:16:00 +0900 | 2022-03-16 11:16:00 +0900 |
| 10 | user9 | user9@example.com | $2a$12$w/DCTEnN5URBgw2y6P6dQ.XCyDcXS0FmD9jNP/HbnyMJIFC750vg6 | true      | false | 2022-03-16 11:16:01 +0900 | 2022-03-16 11:16:01 +0900 |
+----+-------+-------------------+--------------------------------------------------------------+-----------+-------+---------------------------+---------------------------+
10 rows in set
```

## 42 バリデーションテストの事前準備

https://rubygems.org/gems/minitest-reporters/versions/0.5.0?locale=ja <br>

- `api/Gemfile`を編集<br>

```:Gemfile
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails', branch: 'main'
gem 'rails', '~> 6.0.4', '>= 6.0.4.6'
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

# 追加
group :test do
  # テスト結果色付け Doc: https://github.com/kern/minitest-reporters
  gem 'minitest-reporters', '~> 1.1', '>= 1.1.11'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- `root $ dokcer compose build api`を実行<br>

* `root $ root $ docker-compose run --rm api bundle info minitest-reporters`を実行<br>

```:terminal
  * minitest-reporters (1.5.0)
	Summary: Create customizable Minitest output formats
	Homepage: https://github.com/CapnKernul/minitest-reporters
	Path: /usr/local/bundle/gems/minitest-reporters-1.5.0
```

- `api/test/test_helper.rb`を編集<br>

```rb:test_helper.rb
ENV['RAILS_ENV'] ||= 'test'
require_relative '../config/environment'
require 'rails/test_help'

# 追加
# gem minitest-reporters setup
require 'minitest/reporters'
Minitest::Reporters.use!

class ActiveSupport::TestCase # Run tests in parallel with specified workers
  parallelize(workers: :number_of_processors)

  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all

  # Add more helper methods to be used by all tests here...
end # ここまで
```

- `root $ docker compose run --rm api rails t`を実行<br>

```:terminal
Started with run options --seed 64368

  0/0: [===================================================================================================================================================================] 100% Time: 00:00:00, Time: 00:00:00

Finished in 1.58011s
0 tests, 0 assertions, 0 failures, 0 errors, 0 skips
```

- コア数の確認 `root $ docker compose run --rm api sh`でコンテナ内に入る<br>

* `/app # nproc`を実行<br>

```:terminal
2
```

- `api/test/test_helper.rb`を編集<br>

```rb:test_helper.rb
ENV['RAILS_ENV'] ||= 'test'
require_relative '../config/environment'
require 'rails/test_help'

# gem minitest-reporters setup
require 'minitest/reporters'
Minitest::Reporters.use!

class ActiveSupport::TestCase # プロセスが分岐した直後に呼び出される # # 並列テスト ... 複数のプロセスを分岐させテスト時間の短縮を行う機能
  parallelize_setup do |worker|
    load # seedデータの読み込み
    "#{Rails.root}/db/seeds.rb"
  end

  # Run tests in parallel with specified workers
  # 並列テストの有効化・無効化
  # workers: プロセス数を渡す(2以上 => 有効、2未満 => 無効)
  # number_of_processors => 使用しているマシンのコア数(2)
  parallelize(workers: :number_of_processors) # parallelize(workers: 1) # 並列テストを無効にしたい場合

  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all

  # Add more helper methods to be used by all tests here...
end
```

- `root $ docker compose exec -u postgres db psql`を実行<br>

* `postgres=# \l`を実行<br>

```:terminal
      Name       |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------------+----------+----------+------------+------------+-----------------------
 app_development | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 app_test        | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 app_test-0      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 app_test-1      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres        | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0       | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                 |          |          |            |            | postgres=CTc/postgres
 template1       | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                 |          |          |            |            | postgres=CTc/postgres
(7 rows)

postgres=#
```

- `postgres=# \q`を実行して一度 postgresql から抜ける<br>

* `root $ docker compose run --rm api rails t`を実行<br>

```:terminal
Started with run options --seed 18302

  0/0: [===================================================================================================================================================================] 100% Time: 00:00:00, Time: 00:00:00
users...
users...
users = 10
users = 10

Finished in 1.49887s
0 tests, 0 assertions, 0 failures, 0 errors, 0 skips
```

- `root $ docker compose exec -u postgres db psql`を実行<br>

* `postgres=# \c app_test-0`を実行<br>

```:terminal
You are now connected to database "app_test-0" as user "postgres".
app_test-0=#
```

- `app_test-0=# \dt`を実行<br>

```:terminal
                List of relations
 Schema |         Name         | Type  |  Owner
--------+----------------------+-------+----------
 public | ar_internal_metadata | table | postgres
 public | schema_migrations    | table | postgres
 public | users                | table | postgres
(3 rows)

app_test-0=#
```

- `app_test-0=# select * from users;`を実行<br>

```:terminal
 id | name  |       email       |                       password_digest                        | activated | admin |         created_at         |         updated_at
----+-------+-------------------+--------------------------------------------------------------+-----------+-------+----------------------------+----------------------------
  1 | user0 | user0@example.com | $2a$04$vNEw2W.L71kbv7AqESyy7elCv0F7pbLw.qEsg.QpW3bmu.L0Q.64C | t         | f     | 2022-03-16 03:31:27.989397 | 2022-03-16 03:31:27.989397
  2 | user1 | user1@example.com | $2a$04$0QBPlYA2A6gsfYESYTbyW.mzzVAexm6wnZZaOtF8mVLrrlrnyStI2 | t         | f     | 2022-03-16 03:31:28.044135 | 2022-03-16 03:31:28.044135
  3 | user2 | user2@example.com | $2a$04$4Rgbhe69Rr7txT5hYQU/vOyba0QAb7f3vxauzzjSjjYxAhGmuHI6S | t         | f     | 2022-03-16 03:31:28.066606 | 2022-03-16 03:31:28.066606
  4 | user3 | user3@example.com | $2a$04$zedpoPkkivGSWnZj23Lb..nYuymFBR3vnTWGRcI/ZAPIXIsh0ooNm | t         | f     | 2022-03-16 03:31:28.095801 | 2022-03-16 03:31:28.095801
  5 | user4 | user4@example.com | $2a$04$.10qnO.UsBo5i.MBcEd0FeeiFsRnHU0iOQFGreY.fRu06oaxMEtPm | t         | f     | 2022-03-16 03:31:28.125517 | 2022-03-16 03:31:28.125517
  6 | user5 | user5@example.com | $2a$04$lPPJwGSCCr0AgfkrM.j.t.gWF06d/zsCAqAfZ3WIEezZObfqmwZzS | t         | f     | 2022-03-16 03:31:28.156844 | 2022-03-16 03:31:28.156844
  7 | user6 | user6@example.com | $2a$04$WbgzGL/gJLOENL6Za4/k7.QwnqzZIfjMyy2x1KgaXS2OhQBbkuVzq | t         | f     | 2022-03-16 03:31:28.200804 | 2022-03-16 03:31:28.200804
  8 | user7 | user7@example.com | $2a$04$6Q2FlRQRmFzdMXc69X6dUO8VzShVmLNow9OS12ed37BUTyB0ZBgOW | t         | f     | 2022-03-16 03:31:28.225538 | 2022-03-16 03:31:28.225538
  9 | user8 | user8@example.com | $2a$04$iIYiv7yJ9z4hs75erMqp4OHh4/rcT5uuwcbt0v7.iAuJZ8mr0G0wu | t         | f     | 2022-03-16 03:31:28.246018 | 2022-03-16 03:31:28.246018
 10 | user9 | user9@example.com | $2a$04$AUsEoVOgy8lypef1NhUPwOkw37WqjJxFs7JF64W3b.GjkRXS9rtqe | t         | f     | 2022-03-16 03:31:28.270957 | 2022-03-16 03:31:28.270957
(10 rows)

app_test-0=#
```

- `app_test-0=# \c app_test-1`を実行<br>

```:terminal
You are now connected to database "app_test-1" as user "postgres".
app_test-1=#
```

- `app_test-1=# select * from users;`を実行<br>

```:terminal
 id | name  |       email       |                       password_digest                        | activated | admin |         created_at         |         updated_at
----+-------+-------------------+--------------------------------------------------------------+-----------+-------+----------------------------+----------------------------
  1 | user0 | user0@example.com | $2a$04$S.RUrE27yzLy4DeYuzoSmODf/vufHSgqv29I7DwxgM5eWxMJ.wj86 | t         | f     | 2022-03-16 03:31:27.936886 | 2022-03-16 03:31:27.936886
  2 | user1 | user1@example.com | $2a$04$Y6echCuG2SzWhbAIl84up.9QS7Eic8sxF8mZBdmJwC6Pczyl1cYam | t         | f     | 2022-03-16 03:31:27.984796 | 2022-03-16 03:31:27.984796
  3 | user2 | user2@example.com | $2a$04$y8sp5qwIpRpgvlULgdjJq.KtgUhleo3pD4oC62BudMwL7RBTieAje | t         | f     | 2022-03-16 03:31:28.01627  | 2022-03-16 03:31:28.01627
  4 | user3 | user3@example.com | $2a$04$PKuWwRUQB1eFF1tSCMadMO0id8UBWq.WYDRnXM1PvEsH/99Eoseye | t         | f     | 2022-03-16 03:31:28.047367 | 2022-03-16 03:31:28.047367
  5 | user4 | user4@example.com | $2a$04$6JAaVtSQVgmaMxM.7lNOoeKEOdOHC8.egmt3pHmJOOfQHvugIrfKe | t         | f     | 2022-03-16 03:31:28.071277 | 2022-03-16 03:31:28.071277
  6 | user5 | user5@example.com | $2a$04$1xsLR2AYLLYZQcAu3qXmxuOLg1IN/NA1Xpul2pZ99QnrmSi/1dL2a | t         | f     | 2022-03-16 03:31:28.096324 | 2022-03-16 03:31:28.096324
  7 | user6 | user6@example.com | $2a$04$O.jETdCK4AA0N87MPXBs1ui1O9tjbtyhTv4KFNqQHZxCPamkDRmCa | t         | f     | 2022-03-16 03:31:28.138675 | 2022-03-16 03:31:28.138675
  8 | user7 | user7@example.com | $2a$04$9Gw1AAQRoPuW4uXXzk9FWOO/AUA9g5v8o.Nj/jEexkbnl78Azj9fi | t         | f     | 2022-03-16 03:31:28.193277 | 2022-03-16 03:31:28.193277
  9 | user8 | user8@example.com | $2a$04$ZuepOnCThDg6BevyizZRZ.sRmtiHoVbYrU2s4jB6BiC7LG1fQZ/4K | t         | f     | 2022-03-16 03:31:28.2391   | 2022-03-16 03:31:28.2391
 10 | user9 | user9@example.com | $2a$04$z7CPpxkP993qQBsj53D/LOY8CMkpXZvO0n4KenO5UkiTT9lRWTDEa | t         | f     | 2022-03-16 03:31:28.282953 | 2022-03-16 03:31:28.282953
(10 rows)

app_test-1=#
```

- `api/test/test_helper.rb`を編集<br>

```rb:test_helper.rb
ENV['RAILS_ENV'] ||= 'test'
require_relative '../config/environment'
require 'rails/test_help'

# gem minitest-reporters setup
require 'minitest/reporters'
Minitest::Reporters.use!

class ActiveSupport::TestCase # プロセスが分岐した直後に呼び出される # # 並列テスト ... 複数のプロセスを分岐させテスト時間の短縮を行う機能
  parallelize_setup do |worker|
    load # seedデータの読み込み
    "#{Rails.root}/db/seeds.rb"
  end

  # Run tests in parallel with specified workers
  # 並列テストの有効化・無効化
  # workers: プロセス数を渡す(2以上 => 有効、2未満 => 無効)
  # number_of_processors => 使用しているマシンのコア数(2)
  parallelize(workers: :number_of_processors) # 以下削除
end
```
