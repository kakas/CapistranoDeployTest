# 前言

今天來介紹使用 [DigitalOcean](https://m.do.co/c/4b15078c6d51) Deploy Rails App（資訊揭露：這是我個人的推薦連結，你拿 $10 我也拿 $10）

或是用 [Linode](https://www.linode.com/?r=fb83fb07e858858a66d517319d687e0f05ce4843) 也可以，差在開機器而已（資訊揭露：這是 Jimmy 的推薦連結）

環境
- Ubuntu 14.04 x64
- 一個準備好要被 Deploy 的 Rails APP
  - 本文使用 Rails 5.0.0 及 Ruby 2.3.0
  - Capistrano 3.6.0

**PS：本篇文章以 $ 開頭代表在本機輸入命令，#$ 開頭代表在遠端的 Ubuntu 機器上輸入命令**

**PS2：請隨時注意安裝每個東西後 Ubuntu 出現的訊息，不是打完指定就一定會安裝成功（因 為 你 可 能 會 手 殘）**

**PS3：建議指令都用複製貼上就好了，避免手殘**

**PS4：要 deploy 自己的專案只要把 `KaohsiungRubbishTruck` 改成自己的 appname 就可以了**




# 開機器

1. 註冊好並登入後，按 `Create Droplet`
1. 選 `Ubuntu 14.04 x64`
1. 選 `$10/mo` 的機器
1. 地區選`新加坡`（Singapore）
1. 把你本機的公鑰加到 DigitalOcean（不一定要做，但建議）
   - `$ cat ~/.ssh/id_rsa.pub` 即可查到本機的公鑰
1. 選 1 個 Droplet
1. 設定一個好記的名字
1. 等機器開一下
1. 選擇你剛剛的機器，進去後左邊選單 Access => Reset Root Password
1. 大概一兩分鐘後就可以在你註冊的信箱收到密碼了
1. 完成




# 安裝環境

接下來這章節下載安裝軟體都要花時間，請耐心等待。

1. Copy DigitalOcean 左上角的 IP
1. `$ ssh root@xxx.xxx.xxx.xxx`
1. `#$ sudo apt-get update`
1. `#$ sudo apt-get upgrade`
1. `#$ sudo apt-get autoremove`
1. 設定時區：`#$ sudo dpkg-reconfigure tzdata`
    - 進入 `Asia`
    - 按 `T` 找一下 `Taipei` 按 Enter
1. 安裝 utf-8 語系：`#$ sudo locale-gen zh_TW zh_TW.UTF-8 zh_CN.UTF-8 en_US.UTF-8`
1. 安裝 MySQL：
    - `#$ sudo apt-get install mysql-common mysql-client libmysqlclient-dev mysql-server`
    - 安裝時會提示你要建立密碼，但是可以直接按 Enter 忽略掉，我們之後再把 MySQL 的 Port 關起來就好了
    - Jimmy 之後會教怎麼關！
1. 測試 MySQL：
    - `#$ mysql -u root -p`
    - 若出現 `mysql >` 就表示安裝成功了
    - 這時候我們可以先來建立等等專案要用的 database，記得加分號（;）
        - `mysql > create database KaohsiungRubbishTruck;`
        - `mysql > show databases;`
    - 輸入 exit 離開
1. 安裝需要的套件：`#$ sudo apt-get install build-essential git-core curl libssl-dev libreadline5 libreadline-gplv2-dev zlib1g zlib1g-dev libmysqlclient-dev libcurl4-openssl-dev libxslt-dev libxml2-dev libffi-dev git`
    - （不要問我，我也不知道安裝了什麼...）
1. 安裝 rvm：`#$ \curl -sSL https://get.rvm.io | bash`
    - `-s` tells curl to download the file in 'silent mode'
    - `-S` tells curl to show an error message if it fails
    - `-L` tells curl to follow all HTTP redirects while retrieving the installation script
1. 離開 Ubuntu：`#$ exit`
    - 這邊要重新進入 rvm 指令才會生效
1. 重新進入 Ubuntu：`$ ssh root@xxx.xxx.xxx.xxx`
1. 安裝 ruby 2.3.0：`#$ rvm install 2.3.0`
    - 可以先去洗個澡、睡個覺再回來看一下安裝好了沒...
1. 確認 ruby 2.3.0 是否安裝成功：`#$ rvm list`、`#$ ruby -v`
1. 安裝 ImageMagick：`#$ sudo apt-get install imagemagick`
1. 從這邊開始跟 Growth School 不一樣
1. 安裝 `Nginx：#$ sudo apt-get install curl git-core nginx -y`
    - 先設定 Nginx 的 config 軟連結到你的專案
    - `#$ sudo rm /etc/nginx/sites-enabled/default`
    - `#$ sudo ln -nfs "/home/deploy/KaohsiungRubbishTruck/current/config/nginx.conf" "/etc/nginx/sites-enabled/KaohsiungRubbishTruck"`
1. 安裝 Rails：`#$ gem install rails -v '5.0.0' -V --no-ri --no-rdoc`
1. 安裝 Bundler：`#$ gem install bundler -V --no-ri --no-rdoc`
    - `-V` (Verbose Output): Prints detailed information about Gem installation
    - `--no-ri` (Skips Ri Documentation): Doesn't install Ri Docs, saving space and making installation fast
    - `--no-rdoc` (Skips RDocs): Doesn't install RDocs, saving space and speeding up installation



# 增加 Deploy 用的 user

1. `#$ sudo adduser deploy` （記得用 root 使用者建立，建立使用者及密碼後，他會問有的沒的可以按 Enter 略過）
1. 登出：`#$ exit`
1. 使用 deploy 這個使用者登入：`$ ssh deploy@xxx.xxx.xxx.xxx`
1. 幫 deploy 這個 user 加 SSH Key，這樣之後登入就不用密碼了
    - `$ cat ~/.ssh/id_rsa.pub` 即可查到本機的公鑰，把他複製起來
    - `#$ mkdir ~/.ssh`
    - `#$ vi ~/.ssh/authorized_keys`
    - 按 `i` 再按 `cmd+v` 貼上剛剛複製的公鑰，按 `esc` 再按 `:wq` 存檔並離開
    - 重新用 deploy 這個角色登入試試看，若不用密碼表示成功了
1. 因為之後我們會需要在 Ubuntu 上拉 github 或是 bitbucket 的程式碼，所以我們要先在 ubuntu 上建立 SSH key （記得是要建在 deploy 這個角色）
    - `#$ ssh-keygen -t rsa`
    - 會問你一些問題，按 enter 略過即可
    - `#$ cat ~/.ssh/id_rsa.pub` 即可查到 Ubuntu 上的公鑰，把他複製起來
1. 綁定公鑰到 github 上，如果是你自己的 Project 可以綁定到你的帳號
    - [SSH and GPG keys](https://github.com/settings/keys) 進入後按 `New SSH key`
    - Title 可以取跟你機器名稱一樣，方便記
    - 貼上 Ubuntu 機器的公鑰
    - 測試一下：`#$ ssh -T git@github.com` 成功的話會出現歡迎訊息




# 設定自動化部屬（請在你專案上的 master branch 上做）

## 在 `.gitignore` 加上

```ruby
/config/database.yml
/config/secrets.yml
```

這兩個檔案各複製一份出來
- config/database.yml -> config/database.yml.sample
- config/secrets.yml -> config/secrets.yml.sample
- 把 config/database.yml、config/secrets.yml 刪掉

## 修改 Gemfile 加上下列程式碼，並把其他 App Server 的 Gem 註解掉 （像是 Passenger、Unicorn 之類的）

```ruby
- gem 'sqlite3'
+ gem 'sqlite3', group: :development

- # gem 'therubyracer', platforms: :ruby
+ gem 'therubyracer', platforms: :ruby

+ group :development do
+     gem 'capistrano',         require: false
+     gem 'capistrano-rvm',     require: false
+     gem 'capistrano-rails',   require: false
+     gem 'capistrano-bundler', require: false
+     gem 'capistrano3-puma',   require: false
+ end

+ gem 'puma'

+ group :production do
+   gem "mysql2"
+ end
```

## `$ bundle install`

## `$ cap install`

- 他會問你要不要用 Harrow 這個功能，選 `no`

## 修改 Capfile

```ruby
# Load DSL and set up stages
require "capistrano/setup"

# Include default deployment tasks
require "capistrano/deploy"

require 'capistrano/rails'
require 'capistrano/rvm'
require 'capistrano/bundler'
require 'capistrano/rails/assets'
require 'capistrano/rails/migrations'
require 'capistrano/puma'

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }
```

說明：
This Capfile loads some pre-defined tasks in to your Capistrano configuration files to make your deployments hassle-free, such as automatically:

- Selecting the correct Ruby
- Pre-compiling Assets
- Cloning your Git repository to the correct location
- Installing new dependencies when your Gemfile has changed

## 覆寫 config/deploy.rb

把原本的內容砍掉，換成下面的。

```ruby
# config valid only for current version of Capistrano

set :repo_url,        'git@github.com:kakas/KaohsiungRubbishTruck.git' # 改成你的
set :application,     'KaohsiungRubbishTruck' # 改成你的 appname
set :user,            'deploy' # 這個對應到我們剛剛增加的 user: deploy
set :puma_threads,    [4, 16]
set :puma_workers,    0

# Don't change these unless you know what you're doing
set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.error.log"
set :puma_error_log,  "#{release_path}/log/puma.access.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w(~/.ssh/id_rsa.pub) }
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true  # Change to false when not using ActiveRecord

## Defaults:
# set :scm,           :git
# set :branch,        :master
# set :format,        :pretty
# set :log_level,     :debug
# set :keep_releases, 5

## Linked Files & Directories (Default None):
set :linked_files, %w{config/database.yml config/secrets.yml}
set :linked_dirs,  %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

namespace :puma do
  desc 'Create Directories for Puma Pids and Socket'
  task :make_dirs do
    on roles(:app) do
      execute "mkdir #{shared_path}/tmp/sockets -p"
      execute "mkdir #{shared_path}/tmp/pids -p"
    end
  end

  before :start, :make_dirs
end

namespace :deploy do
  desc "Make sure local git is in sync with remote."
  task :check_revision do
    on roles(:app) do
      unless `git rev-parse HEAD` == `git rev-parse origin/master`
        puts "WARNING: HEAD is not the same as origin/master"
        puts "Run `git push` to sync changes."
        exit
      end
    end
  end

  desc 'Initial Deploy'
  task :initial do
    on roles(:app) do
      before 'deploy:restart', 'puma:start'
      invoke 'deploy'
    end
  end

  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      invoke 'puma:restart'
    end
  end

  desc 'Upload to shared/config'
  task :upload do
    on roles (:app) do
      upload! "config/database.yml", "#{shared_path}/config/database.yml"
      upload! "config/secrets.yml",  "#{shared_path}/config/secrets.yml"
    end
  end

  before :starting,  :check_revision
  after  :finishing, :compile_assets
  after  :finishing, :cleanup
  after  :finishing, :restart
end

# ps aux | grep puma    # Get puma pid
# kill -s SIGUSR2 pid   # Restart puma
# kill -s SIGTERM pid   # Stop puma
```

## 修改 config/deploy/production.rb

IP 請自行修改

```ruby
set :stage, :production
set :branch, :master

role :app, %w(deploy@xxx.xxx.xxx.xxx)
role :web, %w(deploy@xxx.xxx.xxx.xxx)
role :db, %w(deploy@xxx.xxx.xxx.xxx)

set :rails_env, "production"
set :puma_env, "production"
set :puma_config_file, "#{shared_path}/config/puma.rb"
set :puma_conf, "#{shared_path}/config/puma.rb"
```

## 修改 config/nginx.conf

還記得剛剛安裝 nginx 的時候有設定軟連結嗎？
就是要把他連到我們專案的設定檔。
如果想檢查軟連結可以
1. 先用 root 登入
1. `#$ cd /etc/nginx/sites-enabled/`
1. `#$ ls -la`

扯遠了，回到 deploy，在專案新增 `config/nginx.conf`

```conf
upstream puma {
  server unix:///home/deploy/KaohsiungRubbishTruck/shared/tmp/sockets/KaohsiungRubbishTruck-puma.sock;
}

server {
  listen 80 default_server deferred;
  # server_name example.com;

  root /home/deploy/KaohsiungRubbishTruck/current/public;
  access_log /home/deploy/KaohsiungRubbishTruck/current/log/nginx.access.log;
  error_log /home/deploy/KaohsiungRubbishTruck/current/log/nginx.error.log info;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @puma;
  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}
```

## 將剛剛做的修改 commit 起來，push 到遠端

- `$ git add .`
- `$ git commit -m "ready to deploy"`
- `$ git push origin master`

## 準備 deploy

這兩個檔案各複製一份出來
- config/database.yml.sample -> config/database.yml
- config/secrets.yml.sample -> config/secrets.yml


修改 config/database.yml

```ruby
 production:
-  <<: *default
-  database: db/production.sqlite3
+  adapter: mysql2
+  encoding: utf8
+  database: KaohsiungRubbishTruck
+  pool: 5
+  username: root
+  password:
+  socket: /var/run/mysqld/mysqld.sock
```

修改 config/secrets.yml

```ruby
-  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
+  secret_key_base: e4a98dd1063a6765xxxxxxx # 這一串從上面的 test 或是 development 複製過來即可
```

#### 第一次 deploy
- 把 database.yml 跟 secrets.yml 丟到 Ubuntu（之後如果有更改也可以這樣丟）
  - `$ cap production deploy:upload`
- `$ cap production deploy:initial`
- `ssh 用 root 登入，重開 nginx`
- `#$ sudo service nginx restart`

#### 之後如果要 deploy，只要 commit 然後 push 到 master，然後下令 `$ cap production deploy` 就可以了。



# 參考資料：

- [(Mini Course) Deploy Rails Project to Linux Server | Growth School](http://courses.growthschool.com/courses/deploy-rails-project-to-linux-server)
- [Deploying a Rails App on Ubuntu 14.04 with Capistrano, Nginx, and Puma | DigitalOcean](https://www.digitalocean.com/community/tutorials/deploying-a-rails-app-on-ubuntu-14-04-with-capistrano-nginx-and-puma)
- Jimmy
