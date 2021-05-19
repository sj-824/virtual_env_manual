# 環境構築手順書

## 目標
vagrantの仮想マシン上で、laravelフレームワークを用いてログインページを作成する。

## 前準備
  - virtualboxのインストール
  - vagrantのインストール
  - centOS7のvagrantboxのインストール

## インストールするソフトウェア

| package | version |
|---------|---------|
| CentOS  | 7       |
| PHP     | 7.3     |
| Laravel | 6.0     |
| MySQL   | 5.7     |
| Nginx   | 1.19    |

## 各ソフトウェアについて
  - CentOS[^1]
    - Red Hat Enterprise Linuxを元にして作られたLinuxディストリビューションの一つ
      - Linuxディストリビューション : Linuxをカーネルに持つOS  
  - PHP[^2]
    - 広く使用されているオープンソースの汎用スクリプト言語
    - Web開発に優れており、初学者に対しても非常に分かり易い
  - Laravel[^3]
    - PHPで記述されたframeworkの一つ
    - MVCモデルを採用しており、学習が低コスト
    - パッケージ管理はComposer
    - データ操作を容易にするEloquent ORMを装備
    - テンプレートエンジンはBlade
  - MySQL[^4]
    - リレーショナルデータベース(RDB)管理システムの一つ
      - RDB : 全てのデータを一つの場所に保存するのではなく、独立したテーブルに保存
  - Nginx[^5]
    - Webサーバーの一つ
    - Apacheとの比較
      - 日本語の文献情報が少ない
      - 大量のリクエストに対するレスポンスが早い

## 環境構築の手順

  ### 1. vagrant用ディレクトリの作成
  先ずは、作業用ディレクトリを作成します。  
  ディレクトリの作成場所は、以下のディレクトリ配下に作成してください。

  - 自分の作業用ディレクトリ
  - デスクトップ  
  
  ```
  $ mkdir new_vagrant && cd new_vagrant
  ```
  作成したフォルダの中で以下のコマンドを実行してください。

  ```
  # vagrant init box名 vagrantに使用するboxを指定
  $ vagrant init centos/7

  #実行後問題なければ以下が出力されます。
  A `Vagrantfile` has been placed in this directory. You are now
  ready to `vagrant up` your first virtual environment! Please read
  the comments in the Vagrantfile as well as documentation on
  `vagrantup.com` for more information on using Vagrant. 
  ```

  - boxとは？[^6]
    仮想マシンのテンプレートとなるファイルです。1からOSをインストールするのではなく、あらかじめ用意されたpackageをインストールすることでOS環境を作成することができます。OSのイメージファイルという認識で大丈夫です。

  ### 2. Vagrantfileの編集
  では、次にvagrant作業用ディレクトリ下のVagrantfileの編集を行います。Vagrantfileは仮想マシンの構築設定などを記述するファイルであり、このファイルを元に仮想環境が作られます。  
  今回の編集箇所は3箇所です。

  ```
  Vagrant.configure("2") do |config|

  ~省略~
  
  *変更点①
  # config.vm.network "forwarded_port", guest: 80, host: 8080  
  ↓  
  config.vm.network "forwarded_port", guest: 80, host: 8080  

  ~省略~

  *変更点②
  # config.vm.network "private_network", ip: "192.168.33.10"  
  ↓  
  config.vm.network "private_network", ip: "192.168.33.19"  
  
  ~省略~

  *変更点③  
  # config.vm.synced_folder "../data", "/vagrant_data"  
  ↓  
  config.vm.synced_folder "./", "/vagrant", type:"virtualbox"  
  ```

  変更点③はホストOSとゲストOS間で共有フォルダが作成されます。今回の設定では、ホストOSのVagrantfileの置かれたディレクトリとゲストOSの/vagrantディレクトリが共有されています。フォルダを共有する事で、仮想環境にログインしていない状態でもホストOS側から各種ファイルの記述を変更することができ、作業効率が上がります。
  - ポートとは？[^7]
    インターネットとコンピュータの間にあるドアのことです。ポートはサーバー側にもパソコン側にもあり、特にサーバー側のポートの番号は重要です。サーバー側では、ポート番号ごとに決まった機能が割り当てられています。例えば、インターネットのデータは80番ポートを必ず通ります。
  - ポートフォワーディング[^8]
    インターネットから特定のポート番号に届いたパケットを、予め設定しておいたLAN側の機器に転送する機能です。1つのグローバルIPアドレスでポート毎に複数のサーバーへ振り分けを行ったり、ポート変換を行うことができます。
    グローバルIPアドレスを1つに絞る理由は、セキュリティのためです。複数のローカルコンピュータが直接外部と接続することを防ぐことで、効率的にセキュリティ対策を行うことができます。
  - プライベートネットワーク
    プライベートIPアドレスで構築されたネットワークのことを示し、外部ネットワークとは接続することができません。
  
  ### 3. vagrantプラグインのインストール
  今回は以下3つのpluginをインストールします。

  ```
  #pluginのインストールは、vagrant plugin install {name}
  $ vagrant plugin install vagrant-vbguest --plugin-version 0.21  
  $ vagrant plugin install vagrant-global-status sahara

  #インストールできているか確かめる
  $ vagrant plugin list
  sahara (0.0.17, global)
  vagrant-global-status (0.1.4, global)
  vagrant-vbguest (0.21.0, global)
    - Version Constraint: 0.21
  ```

  - vagrant-vbguest[^9]
    Vagrant起動時およびリロード時にGuestAdditionsの更新があれば自動的にゲストOSが更新されます。今回はversion0.21をインストールすること[^10]。
    - GuestAdditions
    ゲストマシンに下記機能を追加することができます。
      - クリップボードの共有
      - フォルダの共有
      - 自動ログイン
      - ホスト麻疹との時刻同期
    

  - vagrant-global-status[^11]
    ローカルマシン上で稼働している全てのVMの状態を取得することができます。

  - sahara
  ゲストOSにソフトウェアをインストールしたり設定に変更を加えたりした内容を破棄し、元のゲストOSの状態に戻すことができるpluginです。使用方法は下記リンクを参照してください。  
  [Vagrant sahara で行う状態管理 (後編：使ってみよう)](https://weblabo.oscasierra.net/beginning-vagrant-sahara-2/)

  ### 4. vagrantを使用してゲストOSを起動
  次は、実際にホストOS上にゲストOSを起動したいと思います。  
  ゲストOSを起動するためには、Vagrantfileのあるディレクトリで下記コマンドを実行します。  
  ```
  #ゲストOSを起動する
  $ vagrant up

  #起動の確認
  $ vagrant status

  Current machine states:

  default                   running (virtualbox) # runningであれば起動状態

  ~省略~

  ```

  これで、ゲストOSを起動させることができました。

  ### 5. ゲストOSへのログイン
  次に、ゲストOSへログインしてみましょう。
  ゲストOSへログインするには、下記コマンドを実行します。

  ```
  $ vagrant ssh
  $ [vagrant@localhost ~]$  #ログインされると右のような表記になる
  ```

  以降は、ゲストOSへログインした状態で作業していきます。

  ### 6. 開発に必要なソフトウェアやコマンドをインストールする
  5章までで構築したゲストOSには、まだ開発に必要なソフトウェアがインストールされていません。ゲストOSにログインした状態で必要なパッケージをインストールしていきます。  
  パッケージの導入に当たって実行するコマンドは以下になります。  

  ```
  $ sudo yum -y install パッケージ名
  $ sudo yum -y groupinstall "導入する名称"
  ```
  - sudo
    rootユーザーの権限を一時的に借りる (**無闇に使用しないこと**)
  - yum[^12]
    - CentOSのパッケージ管理ツールコマンド
    - パッケージ提供元であるリポジトリを参照しインストール可能
    - リポジトリは追加することもできる
  - groupinstall
    - 複数のパッケージを一つのグループパッケージとみなし導入することが可能
    - パッケージごとインストールする必要がなくなり環境構築が楽になる

  > #### development toolsのインストール
  まずは、groupパッケージであるdevelopment toolsをインストールします。  

  ```
  $ sudo yum -y groupinstall "development tools"
  ```

  development toolsにはgitのパッケージなど開発に必要なパッケージがいくつか入っています。

  > #### phpのインストール
  次に、PHPをインストールしていきたいと思います。下記のコマンドを実行てください。今回導入するPHPのバージョンは7.2ですがyumでインストール可能なPHPのバージョンは5.4です。そのため、yum以外の場所からPHPをインストールする必要があります。  
  まずは実際に、yumでインストール可能なPHPのバージョンを確かめてみましょう。
  
  ```
  # yumでインストール可能なPHPのバージョンを確認 yum info package名
  $ yum info php

  ~ 省略 ~

  Name        : php
  Arch        : x86_64
  Version     : 5.4.16 ←
  Release     : 48.el7

  ~ 省略 ~
  ```

  インストール可能なバージョンが出力されたと思います。このようにデフォルトのリポジトリからはPHP7.2をインストールすることができません。  
  それでは、PHP7.2をインストールしていきたいと思います。PHP7.2はremiと呼ばれるリポジトリが提供しています[^13]。そのため、remiリポジトリをデフォルトリポジトリに追加する必要があります。
  
  ```
  # remiと依存関係のあるEPELリポジトリおよびwgetコマンドをインストール
  $ sudo yum -y install epel-release wget
  
  # wgetコマンドでremiリポジトリをインストール
  $ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm  
  $ sudo rpm -Uvh remi-release-7.rpm

  # php7.3をインストール
  $ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip

  # phpのversion確認
  $ php -v
  PHP 7.3.28 (cli) (built: Apr 27 2021 13:57:06) ( NTS )
  Copyright (c) 1997-2018 The PHP Group
  Zend Engine v3.3.28, Copyright (c) 1998-2018 Zend Technologies
  ```

  以上でPHPのインストールは終了です。  
  - wgetコマンド
    URLを指定してファイルをダウンロードするコマンド

  > #### Composerのインストール
  次にPHPのパッケージ管理ツールであるcomposerをインストールします[^14]。
  
  ```
  # セットアップ用PHPスクリプトのダウンロード
  $ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

  # PHPスクリプトの実行
  $ php composer-setup.php

  # PHPスクリプトの削除
  $ php -r "unlink('composer-setup.php');"

  # どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います
  $ sudo mv composer.phar /usr/local/bin/composer

  # composerのインストールを確認
  $ composer -v
  ```

  以上でcomposerのインストールは終了です。

  ### 7. laravelのinstallおよびprojectの作成
  次に、laravelのインストールとprojectの作成を行います。作業ディレクトリは/vagrant、project名はlaravel_testとします。

  ```
  $ cd /vagrant
  $ composer create-project laravel/laravel --prefer-dist laravel_test 6.0  

  $ cd /vagrant/laravel_test
  $ php artisan --version
  Laravel Framework 6.20.27 # 6.*ならなんでもOK
  ```

  ### 8. Nginxのインストール
  それでは、次にwebサーバーであるNginxをインストールしていきます。
  viエディタを使用して以下のファイルを作成します。

  ```
  $ sudo vi /etc/yum.repos.d/nginx.repo
  ```

  書き込む内容は以下になります。

  ```
  [nginx]
  name=nginx repo
  baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
  gpgcheck=0
  enabled=1
  ```

  書き終えたら保存し、以下のコマンドを実行してNginxをインストールします。

  ```
  $ sudo yum install -y nginx  

  #nginxのインストールを確認
  $ nginx -v
  ```

  それではNginxを起動してみましょう。

  ```
  $ sudo systemctl start nginx
  ```

  ブラウザにて、http://192.168.33.19(Vagrantfileのipアドレス)と入力し、Nginxのwelcomeページが表示されたら成功です。

  > #### firewallの起動
  それでは、次にfirewallを起動してみましょう。下記コマンドを実行してください。  
  
  ```
  $ sudo systemctl start firewalld.service
  ```

  実行後、再び http://192.168.33.19 へアクセスしてみてください。するとアクセスができなくなっていると思います。  
  firewallは外部ネットワークからのサイバー攻撃や不正アクセスを防ぐセキュリティ機能です。そのため、firewallが起動したことにより、外部アクセスであるhttp通信ができなくなっています。firewall自体はセキュリティにおいて欠かせない機能を提供するため、firewallを起動した状態でhttp通信を許可するよう設定しましょう。下記コマンドを実行してください。

  ```
  $ sudo firewall-cmd --add-service=http --zone=public --permanent
  # 新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行します
  $ sudo firewall-cmd --reload
  ```

  `firewall-cmd --add-service=http`は、httpからのアクセスを許可するコマンドになります[^15]。  
  では、実際に http://192.168.33.19 へアクセスしてください。nginxのwelcomページが表示されていれば問題ないです。

  > #### Laravelを動かす
  まずは、Nginxの設定ファイルを編集していきます。  
  下記コマンドを実行してください。

  ```
  $ sudo vi /etc/nginx/conf.d/default.conf
  ```

  では、下記のようにviエディタで編集していきます[^16]。

  ```
  server {
  listen       80;
  server_name  192.168.33.19; # Vagrantfileで記述したIPアドレス
  root /vagrant/laravel_test/public; # $document_rootを設定
  index  index.html index.htm index.php;

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  ~省略~

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;                                              # $fastcgi_script_nameを設定
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  ~省略~
  ```

  - location / ブロックの`try_files $uri $uri/ /index.php$is_args$args`
    try_filesはリクエストのURLにファイルやディレクトリがあるか確認し、あればそのファイルを返します。`$uri`はリクエストのURIを示し、左から順にファイルがあるか確認していきます。`$uri,$uri/`どちらもなければindex.phpファイルを返します。`$is_args`はクエリパラメータがあれば?を示し、`$args`はクエリパラメータを示します。
  - location ~ \.phpブロックの `fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;`
    リクエストのエントリポイントをphp-fpmに渡しています。laravelのエントリポイントである`public/index.php`ファイルを渡している記述になります。

  次にphp-fpmの設定ファイルを編集していきます。
  
  ```
  $ sudo vi /etc/php-fpm.d/www.conf
  ```

  viエディタで以下の通り編集してください。

  ```
  # userをnginxに変更
  user = apache
  ↓
  user = vagrant

  ~省略~

  #groupをnginxに変更
  group = apache
  ↓
  group = vagrant
  ```

  編集は以上になります。  
  では早速起動してみましょう (nginxは再起動します)

  - php-fpmとは
    - PHPのFastCGI実装の一つ
    - Webサーバー上でPHPを動作させるための仕組みであり、Nginx単体ではPHPを動作させることができません。

  ```
  $ sudo systemctl restart nginx
  $ sudo systemctl start php-fpm
  ```

  再度ブラウザにて、 http://192.168.33.19 へアクセスしてください。  
  すると403 forbiddenと表示されると思います。これはDocumentRootへのアクセスが拒否されている可能性があります。SELinuxを無効にすることで解決できます。

  > #### SELinuxの無効化
  SELinuxを無効化することでDocumentRootへのアクセスを許可します。 (ローカル環境ではこれでいいですが、本番環境では別の方法が必要です[^17])  
  まずは現在のSELinuxの状態を確認します。下記コマンドを実行してください。  
  
  ```
  $ getenforce
  Enforcing
  ```

  Enforcingと表示されている場合は、SELinuxは有効状態にあります。無効状態にするためにSELinuxの設定ファイルを編集します。

  ```
  $ sudo vi /etc/selinux/config
  
  ~省略~

  SELINUX=disabled ←enforcingから変更

  ~省略~
  ```

  編集した内容を反映するためserverの再起動を行います。

  ```
  $ exit
  $vagrant reload

  # 再度ログイン
  $vagrant ssh

  # nginxおよびphp-fpmを起動
  $sudo systemctl start nginx
  $sudo systemctl start php-fpm
  ```

  では、再びブラウザにて http://192.168.33.19 へアクセスしてください。  
  Laravelのページが表示されていれば問題ないです。

  ### 9. MySQLのインストール
  では、MySQLのインストールを行うため、下記コマンドを実行してください[^18]。

  ```
  $ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm  
  $ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm  
  $ sudo yum install -y mysql-community-server  
  $ mysql --version
  ```

  MySQLのversionが確認できたら次にMySQLを起動し接続を行います。  
  今回はデフォルトでrootにパスワード設定がされているため、パスワードを調べる必要があります。下記コマンドを実行してください。

  ```
  # MySQLを起動
  $ sudo systemctl start mysqld

  # MySQLのrootパスワードを確認
  $ sudo cat /var/log/mysqld.log | grep 'temporary password'
  2021-05-18T03:48:54.138769Z 1 [Note] A temporary password is generated for root@localhost: hogehoge ←hogehogeの部分がパスワード
  ```

  では、確認したパスワードを用いてMySQLにログインします。  
  ログインできましたら成功です。

  ```
  $ mysql -u root -p
  Enter password:
  ```

  次にMySQLのログインのパスワードを変更します。MySQLにログインしている場合はログアウトしたのち下記コマンドを実行してください。

  ```
  $ sudo vi /etc/my.cnf

  ~省略~

  [mysqld]

  ~省略~

  # read_rnd_buffer_size = 2M
  datadir=/var/lib/mysql
  socket=/var/lib/mysql/mysql.sock

  # 下記の一行を追加
  validate-password=OFF
  ```

  MySQLの設定ファイルへ`validate-password=OFF`を追加する事でシンプルなパスワードを設定することができます。今回は開発環境であるため、シンプルなパスワードを設定します。しかし、本番環境やステージング環境ではMySQL5.7のパスワードポリシーを遵守したパスワードを利用してください。

  パスワードを再設定するため、MySQLにログインする。

  ```
  # 設定ファイルの変更を反映させるため再起動
  $ sudo systemctl restart mysqld

  #パスワードの再設定
  $ mysql -u root -p
  Enter password:

  mysql > set password = "新たなpassword";
  ```

  パスワードの再設定を終えたらそのままデータベースを作成しましょう。

  ```
  mysql > create database laravel_test;
  mysql > show databases;
  ```

  データベースの作成を確認できたらいよいよログイン機能の作成です。
  
  ### 10. ログイン機能の作成
  まずは、MySQLのログインパスワードをlaravel_testディレクトリ下の.envファイルの内容を以下に変更します。

  ```
  $ cd /vagrant/laravel_test && vi .env
  APP_NAME=Laravel
  APP_ENV=local
  APP_KEY=base64:AL52MvnN6feoOS+dArQMdrpkFpY5lXO2nkqanuZvDUI=
  APP_DEBUG=true
  APP_URL=http://localhost

  LOG_CHANNEL=stack

  DB_CONNECTION=mysql
  DB_HOST=127.0.0.1
  DB_PORT=3306
  DB_DATABASE=laravel_test        # 編集
  DB_USERNAME=root
  DB_PASSWORD=登録したpassword      # 編集
  ```

  編集を終えたら、migrateしましょう。

  ```
  $ php artisan migrate
  ```

  では、実際にDatabaseにテーブルが作成されているか確認しましょう。

  ```
  $ mysql -u root -p
  Enter password :

  mysql > use laravel_test;
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A

  Database changed
  mysql > show tables;
  +------------------------+
  | Tables_in_laravel_test |
  +------------------------+
  | failed_jobs            |
  | migrations             |
  | password_resets        |
  | users                  |
  +------------------------+
  4 rows in set (0.00 sec)
  ```

  usersテーブルが作成されていることが確認できたら次はルート定義をします。

  > #### ルート定義
  認証ルートを定義するため以下のコマンドを実行してください[^19]。  
  ```
  $ composer require laravel/ui "^1.0" --dev
  $ php artisan ui vue --auth
  ```

  laravel/uiパッケージは認証に必要なルートとビューを自動的に用意してくれるコマンドを提供してくれています。上記コマンドを実行する事で、レイアウトビュー、登録ログインビューおよび認証エンドポイントのルートを定義してくれます。実際にルート定義ができているか確認してみましょう。下記コマンドを実行してください。

  ```
  php artisan route:list
  ```

  いかがでしょうか。認証に関するルートが定義されていると思います。
  では実際に、ログインしてみましょう。http://192.168.33.19 へアクセスしページ右上のResisterボタンを押してユーザーの新規作成ページへ飛んでみましょう。任意のユーザー名やアドレスを打ち込み、Resisterボタンを押し、homeページへ遷移できたら成功です！

  以上で環境構築は終了となります。

  ### 引用文献
  [^1]: [「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典](https://wa3.i-3-i.info/word15502.html)  
  [^2]: [PHP とはなんでしょう? - Manual - PHP](https://www.php.net/manual/ja/intro-whatis.php)  
  [^3]: [Laravelとは？Laravelの特徴や始め方をご紹介](https://vitalify.jp/app-lab/vietnam-offshore/20200609_laravel/)  
  [^4]: [MySQLとは？MySQLの基本から他のデータベースとの違いと優位性を解説！](https://style.potepan.com/articles/11311.html#i)
  [^5]: [【入門】Nginx（エンジンエックス）とは？Apacheとの違いと初期設定](https://www.kagoya.jp/howto/rentalserver/nginx/)
  [^6]: [Vagrantで仮想環境構築入門](https://webbibouroku.com/Blog/Article/vagrant)  
  [^7]: [用語集　「ポート(PORT)とは？」](https://www.cman.jp/network/term/port/)  
  [^8]: [用語集｜ポートフォワーディング](https://www.idcf.jp/words/port-forwarding.html)  
  [^9]: [プラグインvagrant-vbguestを使うときの注意点 - ぺんたん info](https://pentan.info/server/vagrant/vagrant_vbguest_note.html)
  [^10]: [Vagrantで共有フォルダのエラーがでるのでその対応](https://yk5656.hatenablog.com/entry/20201202/1609916685)
  [^11]: [稼動している全てのVMの状態を取得](https://tech.withsin.net/2015/08/06/vagrant-global-status/)  
  [^12]: [【初心者でもわかる】yumコマンドの使い方とリポジトリの追加方法](https://eng-entrance.com/linux-package-yum#yum)  
  [^13]: [CentOSでremiとEPELを使いphpのバージョンをアップ/ダウングレードする方法](https://www.geekfeed.co.jp/geekblog/centos-remi-epel-php)  
  [^14]: [Composer を CentOS にインストールする手順](https://weblabo.oscasierra.net/php-composer-centos-install/)  
  [^15]: [CentOS 7のファイアウォールで80番ポートを許可する](https://inaba.hatenablog.com/entry/2017/02/26/040218)
  [^16]: [nginxの設定ファイル nginx.conf の読み方 超入門](https://hack-le.com/nginx/)  
  [^17]: [Nginxで403 Forbiddenが表示された時のチェックポイント5選](https://engineers.weddingpark.co.jp/nginx-403-forbidden/)  
  [^18]: [【最新版】CentOSにMySQLをインストールする方法](https://pursue.fun/tech/how-to-mysql-install/)  
  [^19]: [Laravel 6.x 認証](https://readouble.com/laravel/6.x/ja/authentication.html)  

