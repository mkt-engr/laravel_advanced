# Laravel Advanced

## 初期設定

- Laravel インストール
  - `--prefer-dist`で圧縮ファイルをダウンロードしてくるのでちょっと速い。

```
composer create-project laravel/laravel umarche "8.*" --prefer-dist
```

デバックバーのインストール（ブラウザの下の方になんか出る。）

```
composer require barryvdh/laravel-debugbar
```

- .env
  本番では false に変える。

```
APP_DEBUG=true
```

.env が読み込まれていないときは次のコマンドを打つ。

```
php artisan config:clear
php artisan cache:clear
```

# セクション 2

## Laravel Breeze(認証ライブラリ)

いろんな認証ライブラリがある。

- Laravel/UI (Laravel6.x~)
- Laravel Breeze (Laravel8.x~)★ これ使う
- Laravel Fortify(Laravel8.x~)
- Laravel Jetstream (Laravel8.x~)

* laravel のバージョン確認方法

```sh
php artisan --version
# output:Laravel Framework 8.64.0
```

Laravel Breeze インストール手順：https://readouble.com/laravel/8.x/ja/starter-kits.html

```sh
composer require laravel/breeze "1.*" --dev

php artisan breeze:install
npm install && npm run dev

php artisan serve
# http://127.0.0.1:8000にサーバが立ち上がる
```

### アクセスの流れ

まず`public/index.php`にアクセスされる。

その後ミドルウェアで認証しているか確認する。

その後ルーティング

その後 MVC でごにょごにょ

サービスプロバイダ`config/app.php`

ルーティング：`routes/web.php`

ルーティングの書き方が Laravel6 系から変わった。

```php
Route::get("/register",
[RegisteredUserController::class,"create"])//6系では`RegisteredUserController@create`だった。
->middleware("guest")//ゲストユーザーなら
->name("register");//名前付きルート
```

- コマンドでルーティングを確認する

```sh
php artisan route:list
php artisan route:list > route.txt # テキストに吐き出す
```

- 認証系：auth.php

## バリデーションエラーメッセージを日本語化する

２箇所

メニューの一番下の「言語ファイル」から。
validation.php:https://readouble.com/laravel/8.x/ja/validation-php.html

- Laravel-Lang(GitHub のリポジトリ)：https://github.com/Laravel-Lang/lang
  `locales/ja`の下に日本語化するためのファイルが有る。

これを`umarche/resources/lang/`下にコピーする

これでもまだ微妙に日本語化できていないところがある。（ユーザー登録画面でパスワードと確認パスワードをあえて違うものにするとわかる。）

`resources/lang/ja/validation.php`に以下の配列を追加する。

```php
"attributes" => [
  "name" => "名前",
  "email" => "メールパスワード",
  "password" => "パスワード"
]
```

続いて"Whoops! Something went wrong."を日本語化するために`ja.json`を編集し`lang`直下に置く。

```json
 "Whoops! Something went wrong.": "エラーの内容を確認してください。",
```

## tailwind CSS

TailWind 自体がおもすぎるので使わない CSS はビルドに含まないことができる。その設定が`tailwind.config.js`に書いてある。

- tailwind.config.js

purge で書かれてあるところ以外の CSS は削除する

```js
module.exports = {
    purge: [
        "./vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php",
        "./storage/framework/views/*.php",
        "./resources/views/**/*.blade.php",
    ],
```
