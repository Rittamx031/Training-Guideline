# Tạo file .sun-ci.yml & database-ci.yml ngang hàng với Gemfile
## .sun-ci.yml:
check image ruby hiện có tại [đây](https://hub.docker.com/r/sunci/ruby/builds) sau đó thay thế image tương ứng với phiên bản ruby trong project<br>
**Ví dụ:** 
<br>`image: sunci/ruby:2.6.5` nếu dùng ruby version 2.6.5<br>
`image: sunci/ruby:2.7.0` nếu dùng ruby version 2.7.0<br>
`image: manhbnt/ruby:2.7.1` và `bundle _2.1.4_` nếu dùng ruby version 2.7.1<br>
`image: sunci/ruby:3.0.2` và `bundle _2.1.4_` nếu dùng ruby version 3.0.2<br>

hiện tại bản build cao nhất là ruby 3.0.2, trường hợp các bạn dùng phiên bản ruby cao hơn hoặc bản ruby chưa được build, vui lòng chuyển sang những phiên bản ruby có sẵn :D

```ruby
workspace: true

stages:
- build
- test

jobs:
- name: build:prepare
  stage: build
  image: sunci/ruby:2.7.0
  script:
  - cp database-ci.yml config/database.yml
  - bundle _2.1.2_ install --path vendor/bundle  # _2.1.4_ if ruby 2.7.1 or 3.0.2
  cache:
  - key: vendor_$CI_BRANCH
    paths:
    - vendor/bundle

- name: test:rspec
  stage: test
  image: sunci/ruby:2.7.0
  services:
  - image: mysql:5.7.22
    name: mysql_test
    environment:
      MYSQL_DATABASE: db_test
      MYSQL_USER: user_test
      MYSQL_PASSWORD: password_test
      MYSQL_ROOT_PASSWORD: password_test
  before_script:
  - bundle _2.1.2_ install --path vendor/bundle # _2.1.4_ if ruby 2.7.1 or 3.0.2
  - mkdir .sun-ci
  script:
  - bundle _2.1.2_ exec rails db:drop db:create db:migrate 
  - bundle _2.1.2_ exec rspec --format html --out .sun-ci/rspec.html spec/ # _2.1.4_ if ruby 2.7.1 or 3.0.2
  only:
    branches:
    - master
  artifacts:
    name: rspec_report
    paths:
    - .sun-ci
    expires_in: 3 days

- name: test:rubocop
  stage: test
  image: sunci/ruby:2.7.0
  before_script:
  - bundle _2.1.2_ install --path vendor/bundle # _2.1.4_ if ruby 2.7.1 or 3.0.2
  script:
  - bundle exec rubocop --require rubocop/formatter/checkstyle_formatter --format RuboCop::Formatter::CheckstyleFormatter --no-color app/ lib/
```

## database-ci.yml
nếu các bạn dùng mysql thì chỉ cần copy & paste đoạn code bên dưới :D
```ruby
default: &default
  adapter: mysql2
  encoding: utf8mb4
  collation: utf8mb4_unicode_ci
  host: mysql_test
  port: 3306
  pool: 5

test:
  <<: *default
  database: db_test
  username: user_test
  password: password_test
```
#### Trước khi tạo pull request các bạn báo cho trainer để enable CI project của mình nhé :D. Happy hacking !!!
