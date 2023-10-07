# study-react-laravel

Running Laravel and React using Vite and Inertia on Docker.

初めてのReact.

## 環境

- [docker-laravel](https://github.com/rk-techs/docker-laravel)をクローン

## React, Laravelの環境構築

docker環境でLaravelプロジェクトをインストールした状態から始めます。

### LaravelにReactをインストール

Laravel Breeze パッケージのインストールをします。まずはcomposerで依存関係の追加。

```bash
composer require laravel/breeze --dev
```

`artisan`コマンドでbreezeのインストール実行。

```bash
php artisan breeze:install
```

プロンプトで選択肢を聞かれるので、Reactを選択。他のオプションはそのまま進む。

```bash
 ┌ Which Breeze stack would you like to install? ───────────────┐
 │   ○ Blade with Alpine                                        │
 │   ○ Livewire with Alpine                                     │
 │ › ● React with Inertia                                       │
 │   ○ Vue with Inertia                                         │
 │   ○ API only                                                 │
 └──────────────────────────────────────────────────────────────┘
```

### vite.config.jsの設定

docker環境でViteを使うため以下のコードを追加してホットリロードを有効にする。

```js
server: {
    host: true,
    hmr: {
        host: 'localhost',
    },
},
```

追加後の`vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.jsx',
            refresh: true,
        }),
        react(),
    ],
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
    },
});

```

## インストール後の構成確認

breezeインストールによってファイルが大量に追加された。まずは`web.php`を確認。ルートページでは`resources/js/Pages/Welcome.jsx`が表示されているみたい。

```php
Route::get('/', function () {
    return Inertia::render('Welcome', [
        'canLogin' => Route::has('login'),
        'canRegister' => Route::has('register'),
        'laravelVersion' => Application::VERSION,
        'phpVersion' => PHP_VERSION,
    ]);
});
```

Inertiaは初めて使うため、`Inertia`の`render`メソッドのDocコメントで文法を確認。

```php
Inertia\Inertia::render

<?php
public static function render($component, $props = []) { }
@param string $component

@param array|\Illuminate\Contracts\Support\Arrayable $props

@return \Inertia\Response
```

次に、ログインページを確認。URLは`/login`で、`routes/auth.php`に定義されている。

```php
Route::get('login', [AuthenticatedSessionController::class, 'create'])
            ->name('login');
```

`AuthenticatedSessionController`の`create`メソッドを確認すると、やはり`Inertia`の`render`メソッドでResponseを返している。

```php
public function create(): Response
{
    return Inertia::render('Auth/Login', [
        'canResetPassword' => Route::has('password.request'),
        'status' => session('status'),
    ]);
}
```

`resources/js/Pages/Auth/Login.jsx`を表示しているみたい。`render`メソッドの第二引数に表示したい`jsx`ファイルのパスを渡せばよさそう。（`Pages`ディレクトリ起点？）

## Reactの基礎を学ぶ

React公式ドキュメントの「LEARN REACT > Your First Component」から確認していく。

- [Your First Component – React](https://react.dev/learn/your-first-component)

ポイント:

- ComponentはReactのコアであり、UIを作る基礎部分になる

### コンポーネントのExport

`export default`によって、ファイル内のmainの関数を他のファイルでインポートできるようにしているみたい。（React特有の構文ではなくてJavaScritpの一般的な構文）

### 関数の定義

`function 関数名() { }` はJavaScriptの関数と同じだが、**大文字**で始めること！

```jsx
// Login.jsxの例
export default function Login({ status, canResetPassword }) {
  // ...
}
```

### マークアップ

コンポーネントはHTMLのようタグの記述を返すが、内部的にはJavaScriptらしい。この構文を**JSX**と呼ぶみたいだ。

- `return`は一行で記述する
- 一行に収まらない時は、カッコ`()`で囲うこと

```js
// Login.jsxのLoginメソッドの例

return (
    <GuestLayout>
        <Head title="Log in" />

        {status && <div className="mb-4 font-medium text-sm text-green-600">{status}</div>}

        <form onSubmit={submit}>
          // ...省略
        </form>
    </GuestLayout>
);
```

### コンポーネントを使う

上記`Login.jsx`の記述を参考に

- 小文字で始まるタグは通常のHTMLタグとして認識される（`<div>`など）
- 大文字で始まるものはReactのコンポーネントとして認識（`<GuestLayout>`や`<Head title="Log in" />`など）

コンポーネントのネストと整理について: （[Nesting and organizing components](https://react.dev/learn/your-first-component#nesting-and-organizing-components) ）

- コンポーネントは同じファイルに複数書ける
- ファイルが大きくなったら単一のファイルに分けたらいい
- コンポーネントはネストしない！（コンポーネントは他のコンポーネントをレンダリングできるけど、コンポーネントの中に他のコンポーネントの定義は書かないこと）