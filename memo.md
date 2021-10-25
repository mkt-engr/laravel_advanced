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

## Blade コンポーネント

コントローラーが View を返したりモデルを扱ったりとコントローラーが肥大化しがちだった。

Laravel6 系では
`@yield,@extend,@section,include`などが使われていた。

Laravel7 系以降では
`<x-fefefe>`みたいなタグを使う。

- login.blade.php

```php
<x-guest-layout>
  <x-auth-card>
//<x-＜ファイル名＞>となる。
```

のように使う。

- `resources/views/components`の下に`auth-card.blade.php`がある。
- `app/View/Components/`の下に`guest-layout.php`がある。

- login.php が呼び出されたときの処理

  1. `<x-guest-layout>`が最初にあるので GuestLayout.php (コンポーネントクラス)が呼び出される
  2. render 関数が呼び出される

  ```php
   public function render()
  {
      return view('layouts.guest');
  }
  ```

  3. layouts/guest.blade.php が呼び出される
  4. guest.blade.php の`{{$slot}}`で中身(`<x-auth-card>`)が埋め込まれる。(`auth-card.blade.php`が埋め込まれる。)
     - `<x-auth-card>`は`components/auth-card.blade.php`と対応している。

### Blade コンポーネントクラスを作ってみる

`routes/web.php`でルーティングする。

- web.php

```php
//コントローラ追加
use App\Http\Controllers\ComponentTestController;

//ルーティング追加
Route::get('/component-test1', [ComponentTestController::class, 'showComponent1']);
Route::get('/component-test2', [ComponentTestController::class, 'showComponent2']);
//ROute::get("＜URL＞",[受け取りたいコントローラー名::class,"＜コントローラーのメソッド＞"]);
```

- コントローラーのファイルを作成する。

```sh
php artisan make:controller ComponentTestController
# app/Http/Controllers/ComponentTestController.phpが作成される。
```

- ComponentTestController.php

```php
class ComponentTestController extends Controller
{
    public function showComponent1()
    {
        return view("tests.component-test1");
        //return view("＜フォルダ名.ファイル名＞")のように書く
    }
    public function showComponent2()
    {
        return view("tests.component-test2");
    }
}

```

### $slot に関して

- コンポーネントパターン

  1. コンポーネントを Blade ファイルに埋め込む。Card コンポーネントを別ファイルで作成し、Body コンポーネントに埋め込む的な
  2. スロットを使う。大枠を作っておいて中身は使う側が決める。

  - コンポーネントフォルダの作成場所：`resources/views/components`
  - 使い方:`<x-コンポーネント名></x-コンポーネント名>`
    - フォルダ分けしたい場合(`resources/views/components/tests`フォルダの場合)：`<x-tests.コンポーネント名></x-tests.コンポーネント名>`

- Slot の作成
  guest.blade.php の中に`$slot`がある。

まず`resources/views/components/tests/app.blade.php`を作成する。(`guest.blade.php`を丸コピする)

`app.blade.php`を`component-test1.blade.php`で使ってみる。

- component-test1.php

```php
<x-tests.app>
  コンポーネントテスト1
</x-tests.app>
```

こう書くことで`tests/app.blade.php`にある`$slot`に`コンポーネントテスト1`という文章が差し込まれる(slot は差込口という意味)

### 名前付き Slot

app.blade.php は`{{ $slot }}`が 1 つしかないからこれでよかったけど Slot を複数作りたいときは困る。そんなときに名前付き Slot を使う。

```php
# Component側
{{$header}}

# Blade側
<x-slot name="header">
ここの文章が差し込まれる。
</x-slot>
```

- layouts/app.blade.php(コンポーネント側)
  ここに名前付きスロットがある。

```php
<header class="bg-white shadow">
  <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
    {{ $header }}
  </div>
</header>
```

- dashboard.blade.php(Blade 側)

slot の name="header"が app.blade.php の`{{$header}}`に埋め込まれる。

```php
<x-app-layout>
  #######　ここから
　<x-slot name="header">
      <h2 class="font-semibold text-xl text-gray-800 leading-tight">
          {{ __('Dashboard') }}
      </h2>
  </x-slot>
  ####### ここまでdashboard.blade.phpの{{$header}}に埋め込まれる

  #######　ここから
  <div class="py-12#######　ここから">
          <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
              <div class="p-6 bg-white border-b border-gray-200">
                  You are logged in!
              </div>
          </div>
      </div>
  </div>
  ####### ここまでdashboard.blade.phpの{{$slot}}に埋め込まれる。
</x-app-layout>
```

実際に作ってみる。

- `tests/app.blade.php`

```php
<body>
    <header>
        {{ $header }}
    </header>
    <div class="font-sans text-gray-900 antialiased">
        {{ $slot }}
    </div>
</body>
```

- component-test1.blade.php

```php
<x-tests.app>
    <x-slot name="header">ヘッダー1</x-slot>
    コンポーネントテスト1
</x-tests.app>

```

### View ファイルに値を渡す方法その１（props）

|                         | Blade ファイル(渡す側)                      | Blade コンポーネント(受け取る側)                        |
| ----------------------- | ------------------------------------------- | ------------------------------------------------------- |
| $slot                   | `<x-app>ここに文字</x-app>`                 | `{{$slot}}`                                             |
| 名前付き slot           | `<x-slot name="header">ここに文字</x-slot>` | `{{$header}}`                                           |
| 属性(props)             | `<x-card message="メッセージ"></x-card>`    | `{{$message}}`                                          |
| 初期値`@props`連想配列  | 設定しない場合初期値が表示される            | `@props(['message'=>'初期値です。'])`                   |
| クラスの設定 属性バック | `<div {{$attribute}}>`                      | `<div {{$message->merge(['class'=>'text-sm'])}}></div>` |

**実際に使う**

- `component/tests/card.blade.php`（受け取る側）

```php
<div class="border-2 shadowmd w-1/4 p-2">
    <div>{{ $title }}</div>
    <div>画像</div>
    <div>{{ $content }}</div>
</div>
```

- `tests/component-test1.blade.php`

値を渡すときは`$`をつけない

```php
<x-tests.app>
    <x-tests.card title="タイトル" content="本文" />
</x-tests.app>
```

### View ファイルに値を渡す方法その２（変数）

コントローラーなどから変数を渡す。

- コントローラー側（渡す側）
  ```php
  $message = 'メッセージ'
  return view("Viewファイル",compact('message'));
  ```

* Blade 側(受け取る側)
  ```php
  <x-card :message='$message'>
  ```

**実際に書く**

- ComponentTestController.php（値渡す側）

```php
public function showComponent1()
{
    $message = 'メッセージ';
    return view("tests.component-test1", compact("message"));
}
```

- component-test1.blade.php（コントローラーから受け取る）

```php
<x-tests.app>
    // 左辺：”:”をつけて属性ではなく変数であることを明記する
    // 右辺："$"をつけてコントローラーから受け取ったことを明記する。
    <x-tests.card title="タイトル" content="本文" :message="$message" />
</x-tests.app>
```

- card.blade.php（Blade コンポーネントから受け取る）

```php
<div class="border-2 shadow-md w-1/4 p-2">
    <div>{{ $title }}</div>
    <div>画像</div>
    <div>{{ $content }}</div>
    <div>{{ $message }}</div>//追加
</div>
```

#### props に初期値を設定する方法

card.blade.php では 3 つの変数(`title`,`content`,`message`)を定義している。値を渡す側で

```php
<x-tests.card title="タイトル" />
```

としていると`content`を定義してないよってエラーが出る。

エラーを回避するために初期値を設定したい。

受け取る側に初期値を書く。

- card.blade.php

`@props`を用いる場合は、表示されているすべての変数に関して初期値を書く必要がある。

```php
@props([
    'title' => 'タイトル初期値',
    'message' => '本文初期値です。',
    'content' => '本文初期値です。',
])
<div class="border-2 shadow-md w-1/4 p-2">
    <div>{{ $title }}</div>
    <div>{{ $content }}</div>
    <div>{{ $message }}</div>
</div>
```

### クラスの設定、属性バッグ

CSS のクラスを渡す際に用いられる機能

単に以下のようにするだけだと class 名が反映されない。

```php
<x-tests.card title="CSSを変更したい" class="bg-red-300"/>
```

そこで`$attributes`を使う。

- component-test1.blade.php(値を渡す側)

```php
<x-tests.app>
    <x-tests.card title="CSSを変更したい" class="bg-red-300" />
</x-tests.app>
```

- card.blade.php（値を受け取る側）

```php
<div {{ $attributes }} class="border-2 shadow-md w-1/4 p-2">
    <div>{{ $title }}</div>
</div>
```

**しかしこうすると受け取る側のクラス`border-2`などがすべて値を渡す側の`bg-red-300`でウワが枯れてしまう。**

受け取る側と渡す側のクラス名を両方使いたい場合は以下のように書く。(受け取り側のみ変更)

- card.blade.php

```php
<div {{ $attributes->merge(['class' => 'border-2 shadow-md w-1/4 p-2']) }}>
//{{ $attributes->merge(['＜属性(だいたいclassになりそう＞)' => '＜もともとの属性値＞']) }}
</div>
```

## クラスベースのコンポーネント

以下のコマンドで

- `View/Component`
- `resources/views/components`
  にファイルが作られる。

```sh
php artisan make:component FEFEFE
```

なおオプションで`--inline`をつけると`View/Component`似のみファイルが作られる。

`App/View/Components`内のクラスを指定する。
クラス名：TestClassBase(パスカルケース)
Blade 内：x-test-class-base(ケバブケース)

コンポーネントクラス内で

```php
public function render(){
  return view("bladeコンポーネント内");
}
```

- component-test2.blade.php(クラスベースコンポーネントの呼び出し側)

```php
<x-tests.app>
    <x-test-class-base />
    // View/Components/TestClassBase.phpのケバブケース
</x-tests.app>
```

- TestClassBase.php(Blade ファイルを呼び出す)

```php
public function render()
{
    return view('components.tests.test-class-base-component');
    // components/testsの中のtest-class-base-component.blade.phpを呼び出す。
}
```

## クラスベースコンポーネントで属性(props)と`@props`の初期値を設定する

### 属性

- test-class-base-component.blade.php(最終的な受け取り口)

```php
<div>
    クラスベースのコンポーネントです。
    <div class="">{{ $classBaseMessage }}</div>
</div>
```

- component-test2.blade.php(出発地点となる Blade ファイル)

```php
<x-tests.app>
    <x-slot name="header">ヘッダー2</x-slot>
    コンポーネントテスト2
    <x-test-class-base classBaseMessage="メッセージです" />
</x-tests.app>
```

このままだと`test-class-base-component.blade.php`で`classBaseMessage`が Undefined のエラーが出る。

`<x-test-class-base classBaseMessage="メッセージです" />`のクラスコンポーネントである`TestClassBase.blade.php`で`classBaseMessage`を定義する。

`classBaseMessage`は以下のようにコンストラクタの中に書く。

- TestClassBase.blade.php(`component-test2.blade.php`->**`TestClassBase.blade.php`**->`test-class-base-component.blade.php`)

return するときに compact 関数の中にメンバ変数を入れる必要はない。

```php
class TestClassBase extends Component
{
    //メンバ変数として$classBaseMessageを定義する
    public $classBaseMessage;
    /**
     * Create a new component instance.
     *
     * @return void
     */
    public function __construct($classBaseMessage)
    {
      //コンストラクタ内で初期化
        $this->classBaseMessage = $classBaseMessage;
    }
```

test-class-base-component.blade.php にもう 1 つ変数($defaultMessage)を追加してみる。

- test-class-base-component.blade.php

```php
<div>
    クラスベースのコンポーネントです。
    <div class="">{{ $classBaseMessage }}</div>
    <div class="">{{ $defaultMessage }}</div>
</div>
```

- component-test2.blade.php
  このままだと 1 つ目`x-test-class-base`の`defaultMessage`属性がないほうがエラーになる。

```php
<x-tests.app>
    <x-test-class-base classBaseMessage="メッセージです" />
    <x-test-class-base classBaseMessage="メッセージです" defaultMessage="初期値から変更している。" />
</x-tests.app>
```

このエラーを回避するために`TestClassBase.php`で初期値を設定する。

- TestClassBase.php

コンストラクタの引数の中で初期値を定義する。

```php
class TestClassBase extends Component
{

    public $classBaseMessage;
    public $defaultMessage;
    /**
     * Create a new component instance.
     *
     * @return void
     */
    public function __construct($classBaseMessage, $defaultMessage = "初期値です。")
    {
        $this->classBaseMessage = $classBaseMessage;
        $this->defaultMessage = $defaultMessage;
    }s
```

### デフォルトで作られている Blade ファイルを除く

- resources/views/components/auth-validation-errors.blade.php

`__`のようにアンスコ 2 つで多言語ファイルに設定している情報から日本語に変換する

```php
~~~
{{ __('Whoops! Something went wrong.') }}
~~~
```

## Alpine.js について(あるぱいん)

`dropdown.blade.php`の L24 あたりの`x-data`が Alpine.js の部分

tailwind の JavaScript 版と公式では言ってる。

```php
<div x-show="isOpen()"></div>
```

```php
<div @click="open = ! open"></div>
<div x-on:click="open = ! open"></div>
//この2つは同じ。Vueやん。
```

`open`という変数が`true`なら div タグを表示する。Vue の`v-show`と同じやん

```php
<div x-show="open">
~~~
</div>
```
