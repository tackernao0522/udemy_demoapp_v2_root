## 35 モデル開発に必要な Gem をインストール

- 参考: https://rubygems.org/ <br>

* `hirb`を検索する<br>

- https://rubygems.org/gems/hirb <br>

* `api/Gemfile`を編集<br>

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
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'

# 追加
# コンソールの出力結果を見やすく表示する
gem 'hirb', '~> 0.7.3'

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

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- https://rubygems.org/ で`hirb-unicode`を検索する<br>

* https://rubygems.org/gems/hirb-unicode <br>

- `hirb-unicode-steakknife`をクリックする<br>

* `api/Gemfile`に追加<br>

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
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'

# コンソールの出力結果を見やすく表示する
gem 'hirb', '~> 0.7.3'

# 追加
# Hirbの文字列補正を行う
gem 'hirb-unicode-steakknife', '~> 0.0.9'

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

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- https://rubygems.org/ で`bcrypt`を検索する<br>

* `bcrypt`をクリック<br>

`api/Gemfile`に追加<br>

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

# 追加
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

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

- `root $ docker-compose build api`を実行<br>

* `root $ docker image prune -f`を実行<br>

* `root $ docker compose run --rm api bundle info hirb`を実行<br>

```:terminal
  * hirb (0.7.3)
	Summary: A mini view framework for console/irb that's easy to use, even while under its influence.
	Homepage: http://tagaholic.me/hirb/
	Path: /usr/local/bundle/gems/hirb-0.7.3
```

- `root $ docker compose run --rm api bundle info hirb-unicode-steakknife`を実行<br>

```:terminal
  * hirb-unicode-steakknife (0.0.9)
	Summary: Unicode support for hirb
	Homepage:
	Path: /usr/local/bundle/gems/hirb-unicode-steakknife-0.0.9
```

- `root $ docker compose run --rm api bundle info bcrypt`を実行<br>

```:terminal
  * bcrypt (3.1.16)
	Summary: OpenBSD's bcrypt() password hashing algorithm.
	Homepage: https://github.com/codahale/bcrypt-ruby
	Path: /usr/local/bundle/gems/bcrypt-3.1.16
```

- `root $ vi api/.irbrc`を実行<br>

```vi:.irbrc
# コンソール起動時
if defined? Rails::Console
  # Hirb.enableの有効化
  Hirb.enable if defined? Hirb
end
```

保存する<br>

- `root $ docker compose run --rm api rails c`を実行<br>

`irb(main):001:0> Hirb::View.enabled?`を実行<br>

```:terminal
irb(main):002:0> Hirb::View.enabled?
=> true
irb(main):003:0>
```

- `irb(main):003:0> puts Hirb::Helpers::Table.render [[1,2], [2,3]]`を実行<br>

```:terminal
irb(main):003:0> puts Hirb::Helpers::Table.render [[1,2], [2,3]]
+---+---+
| 0 | 1 |
+---+---+
| 1 | 2 |
| 2 | 3 |
+---+---+
2 rows in set
=> nil
irb(main):004:0>
```

- `irb(main):004:0> puts Hirb::Helpers::Table.render [{:age=>10, :weight=>100}, {:age=>80, :weight=>500}]`を実行<br>

```:terminal
irb(main):004:0> puts Hirb::Helpers::Table.render [{:age=>10, :weight=>100}, {:age=>80, :weight=>500}]
+-----+--------+
| age | weight |
+-----+--------+
| 10  | 100    |
| 80  | 500    |
+-----+--------+
2 rows in set
=> nil
irb(main):005:0>
```
