
# ねこカフェWebアプリ作成

## ゴール
- Udemyで講座を学んで、個人ブログ風のサイトを作成できるようになる

- ポートフォリオもここに掲載しても良いかも

### 今回はAWS LightSailにLaravelアプリを作成
- `52.196.56.39`

### インスタンスにSSH
- VSコードでSSH

### LAMP環境のバージョン確認
`php --version` `PHP 8.0.24`
`mysqld --version` `mysqld  Ver 10.6.10-MariaDB for Linux on x86_64`
`httpd --version` `running now`

### Laravelプロジェクトの作成
- 前提条件

- Laravelのプロジェクトを作成するためには「composer」が必要

- composer（コンポーザー）とはPHPのパッケージ管理システムのこと

- composerのインストール
`curl -sS https://getcomposer.org/installer | php`

- composerのバージョン確認
`php --version` `version 2.4.2`

- どのディレクトリからでもcomposerを利用できるようにする
`sudo mv composer.phar /usr/local/bin/composer`

### Larvelプロジェクト作成後
`composer create-project --prefer-dist laravel/laravel project`

### プロジェクト名の変更
- `mv project neko-cafe01` 

### Laravekプロジェクト作成後の環境構築

```
// デフォルトの.envファイルをコピー(環境設定ファイル)
cp .env.example .env

// 隠しフォルダの表示(.env)
ls -a

// アプリケーションの個別識別キーの発行
php artisan key:generate

// サーバーからのログを書き込む必要があるのでbootstrapフォルダに777のパーミッションを付与
// bootstrapフォルダはサーバーからのキャッシュ等の速度を上げるためのディレクトリ
sudo chmod -R 0777 bootstrap

// 同様にstorageフォルダに777のパーミッションを付与
// Laravelのログなどが入るディレクトリ
sudo chmod -R 0777 storage

```

### Webサーバーの設定
- 以下参考URL
  - https://docs.bitnami.com/aws/infrastructure/lamp/configuration/configure-custom-application/

- laravel用の設定書く

- ドキュメントルートの指定

- アクセスがあったときに、一番最初にどこを見に行くか

-  /opt/bitnami/apache2/conf/vhosts directory. を見に行く

```
// apacheの設定変更&laravel.confファイルの作成
sudo vi /opt/bitnami/apache2/conf/vhosts/laravel.conf

=== テンプレート ===
 <VirtualHost 127.0.0.1:80 _default_:80>
    ServerAlias *
    DocumentRoot /opt/bitnami/APPNAME
    <Directory "/opt/bitnami/APPNAME">
      Options -Indexes +FollowSymLinks -MultiViews
      AllowOverride All
      Require all granted
    </Directory>
  </VirtualHost>

=== ここまで ===

=== 変更箇所 ===

DocumentRoot /home/bitnami/neko-cafe01/public
<Directory "/home/bitnami/neko-cafe01/public">

=== ここまで ===

// apacheの設定ファイルを再起動
sudo /opt/bitnami/ctlscript.sh restart apache
```

- ApacheのドキュメントルートはLaravelアプリの「/home/bitnami/neko-cafe01/public」になった

- パブリックIPをプラウザで打ち込めばログイン画面が表示されるはず

- まだ、DBの設定はしていないのでDB関連はエラーになる

### MariaDBの設定
- 今回はデータベースを作成するにあたって、PhpMyAdmin(GUI操作ツール)を使用する

- 接続するためには幾つか設定が必要

### puttyの設定

```
以下のツールをインストール

・putty
・puttygen ※ インスタンスにアクセスするプライベートキーをputty用に変換し、.putyy鍵を生成

putty上でインスタンスに接続するために、以下を設定

・HOSTNAME 
IPを指定

・SSH/Auth/Gredenials
putty用に変換した.puttyの鍵を指定

・Connection
Auto-login_username
bitnami

PhpMyAdmin用の設定を追加する

・SSH/Auth/Tunnels
Source port 8888
localhost:80

ここまでできたら、ローカルのポートとLightSailのポートを接続する形に設定できた

```

- puttyでlocalhostからphpmyadminにポートフォワーディングする

- ローカルのDB(MariaDB)のパスワードを以下のコマンドで確認する

`cat bitnami_application_password`

- 下記情報を元にPhpMyAdminへログインする

```
▽ 情報一覧
IP:
52.196.56.39

UserName:
bitnami

SecretKey:
~/.ssh/LightsailDefaultKey-ap-northeast-1.pem

databaseUser(default)
root

databasePass:(default)
cD1qbAAZGM7u

```

- phpMyAdminにログイン(root)

- phpMyAdmin上でDB作成(ec-amazon) (utf8mb4_general_ciを選択)

```

=== phpMyAdmin上のSQLで実行 === 

// DBを作成したrootユーザー以外でDBを操作できるユーザーを作成し、操作権限を付与する
grant all privileges on neko-cafe01.* TO 'dbuser'@'localhost' identified by 'cD1qbAAZGM7u';

// 権限周りをリフレッシュする
flush privileges;

=== ここまで === 

```

- laravelフォルダに移動

- .envファイルの修正(DBとのconnectionを設定)
`vi .env`

```
APP_NAME=neko-cafe01 //作成したアプリ名(任意)
APP_ENV=production // local(ローカル開発環境) or production(本番環境)
APP_KEY=base64:158KxuaYWWw6RmG0uHfGoJJ6FUD5L3uPppBvLk63+gk= //php artisan key:generateで作成したキ
APP_DEBUG=false // デバッグする/しない
APP_URL=http://localhost //そのまま

LOG_CHANNEL=stack
LOG_LEVEL=debug

DB_CONNECTION=mysql // mysqlを使用(MariaDBを利用しているが互換性があるため問題ない)
DB_HOST=localhost // 同じサーバー内にDBがあるので、localhostを指定
DB_PORT=3306 //MySQLポート
DB_DATABASE=neko_cafe01 // 作成したDB名
DB_USERNAME=dbuser // 作成したDB操作ユーザー名
DB_PASSWORD=cD1qbAAZGM7u //設定したパスワード(今回はrootユーザーのパスワードと同じ)
```

- DBへの書き込みに対応する（設定しないとpermissionエラーになる）
- storageフォルダに777の権限を付与
`sudo chmod -R 0777 storage`

- bootstrapフォルダに777の権限を付与
`sudo chmod -R 0777 bootstrap`

- DBとの疎通確認を実施
`php artisan migrate`

- DBが作成されればOK

### Laravelアプリの実装

- 確認用HTML/pages.htmlに、画面一覧がある

- まずは、テンプレートとして既にHTML、CSSファイルがある

- サイトの画面部分はできているため、ここにLaravelでシステム化できそうなところを選定

- お知らせとブログ投稿記事がLaravelで実装できそう

### フロントページ
- トップページ
- ブログ一覧
- ブログ詳細
- お問い合わせ
- お問い合わせ完了
- ねこちゃんたち

### 管理画面 
- ユーザー登録
- ログイン
- ブログ一覧
- ブログ登録
- お問い合わせ一覧
- お問い合わせ詳細


### 今回用意されたテンプレートについて
- 確認用HTMLはあくまでHTMLで確認するために作られたもの
- なので、Laravelではbladeファイルが必要のため、これらのHTMLをbladeファイルに組み込む必要がある
- 今回は、組み込み用というフォルダにbladeファイルが既にあるため、こちらを活用する

### ローカルにあるテンプレートをGitHbにpush
- GitHubにpushする
- サーバー上でGitHubからlaravelプロジェクトにクローンする





