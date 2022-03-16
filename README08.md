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
