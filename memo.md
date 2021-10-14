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
