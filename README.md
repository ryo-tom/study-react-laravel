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

## Importing and Exporting Components

ドキュメント参考箇所:

- [Importing and Exporting Components – React](https://react.dev/learn/importing-and-exporting-components#exporting-and-importing-a-component)

### root component fileとは

ドキュメントを参考に、`App.js`というファイルに`User`関数を定義して、それを`UserList`関数内で呼び出してみた。この`App.js`をレンダリングする際、このファイルはroot component fileとして扱われる。（Reactの公式ドキュメントには、ブラウザ上でコードを編集して結果を確認できるサンドボックスが提供されているためそれを利用している。）

```jsx
// App.js
function User() {
  return <p>Name: Ryosuke</p>
}

export default function UserList() {
  return (
    <section>
      <h1>User</h1>
      <User />
      <User />
      <User />
    </section>
  );
}
```

### componentのexportとimport

`App.js`のように一つのファイル内でcomponentを定義し使用する場合、再利用性を高めるためにファイルを分けたくなる。その場合の手順は:

1. componentを入れたいJSファイルを新規作成
2. function componentをexportする
3. componentを使いたいファイルでimportする

まず、JavaScriptのexportについて知識がなかったので、MDNで基礎を確認した。

- [export - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)

exportには2種類ある。各モジュールの中で持てるexportは、

- _named export_ : 複数
- _default export_ : 1つのみ

```js
// named export
export { myFunction2, myVariable2 };

// default export
export default myFunction;
export default function () { /* … */ }
```

前回のBreezeパッケージで生成された`resources/js`下の`.jsx`ファイルを一通り確認したところ、全て**default export**が使われていた。

Reactドキュメントに戻り、サンプルコードを参考に`App.js`に定義した関数を`UserList.js`に切り出してimportする形に変更。

```jsx
// UserList.js
function User() {
  return <p>Name: Ryosuke</p>;
}

export default function UserList() {
  return (
    <section>
      <h1>User</h1>
      <User />
      <User />
      <User />
    </section>
  );
}
```

```jsx
// App.js
import UserList from "./UserList";

export default function App()
{
  return (
    <UserList />
  );
}
```

### 同一ファイルから複数のcomponentをexportとimport

`UserList`ではなく`User`だけを表示したい場合はどうしたらいいか？という観点でコードを書き換えていく。

先に学んだ*named export*を使う。`User`をexportとする。

```jsx
// UserList.js
export function User() {
  return <p>Name: Ryosuke</p>;
}

export default function UserList() {
  // 省略
}
```

`User`を`{}`で囲ってimportする。そして`<User />`をreturnする。

```jsx
// App.js
import { User } from "./UserList";

export default function App()
{
  return <User />
}
```

使用上のTipsも書かれていた。defaultとnamedで混乱を招かないように以下のようなスタイルをとるのがよさそう。

- defaultかnamedかどちらか一つに統一する
- 単一ファイル内ではdefaultとnamedを混在させない

## Writing Markup with JSX

ドキュメント参考箇所:

- [Writing Markup with JSX – React](https://react.dev/learn/writing-markup-with-jsx)

ポイント:

- **JSX**はJavaScriptの拡張構文で、JavaScriptファイル内でHTML風にマークアップできる
- **JSX**は拡張構文で、**React**はJavaScriptのライブラリである
- **JSX**はHTMLに似ているが、より厳格なルールがある

### The Rules of JSX

#### ルート要素は一つだけ

先の`User`の`p`タグを増やしてみた。同階層に複数のタグがあるとエラーになるため、親要素に`<div>`を使って単一のルート要素にした。

```jsx
export function User() {
  return (
    <div>
      <p>Name: Ryosuke</p>
      <p>Learn: React</p>
      <p>Like: Coffee</p>
    </div>
    );
}
```

JSX構文に対応するために`<div>`を使ったが、本当はHTMLの構造に`<div>`タグを増やしたくない、という時は**Fragment**を使う。

**Fragment**は、`<> ... </>`のように空タグで要素を囲う。

#### タグは閉じる

HTMLでは閉じタグがなくても記述できるタグであっても、

```html
<!-- HTML -->
<input type="text" value="test">
<ul>
  <li>List1
  <li>List2
</ul>
```

JSXでは必ずタグを閉じること

```jsx
// JSX
<input type="text" value="test" />
<ul>
  <li>List1</li>
  <li>List2</li>
</ul>
```

#### ほとんどcamelCase

- JSX で記述された属性がJavaScript オブジェクトのキーになる
- 属性はcamelCaseにする
- `class`などの予約語は使えない（`class`属性は代わりに`className`にする）

## JavaScript in JSX with Curly Braces

ドキュメント参考箇所:

- [JavaScript in JSX with Curly Braces – React](https://react.dev/learn/javascript-in-jsx-with-curly-braces)

文字列を渡す: `const`で変数を定義して、`{ }`内で使用できる。

```jsx
export function User() {
  const name = "Ryosuke";
  return (
      <input type="text" value={name} />
    );
}
```

`{ }`内では、JavaScriptのメソッドを書くこともできる。例えば、

```jsx
const today = new Date();

function formatDate(date) {
  const year  = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day   = String(date.getDate()).padStart(2, '0');

  return `${year}-${month}-${day}`;
}

return <h1>Today is {formatDate(today)}</h1>; // Today is 2023-10-08
```

JavaScriptのオブジェクトとしての使い方

```jsx
export function User() {
  const UserInfo = {
    name: "Ryosuke",
    like: "coffee",
  };

  return (
    <>
      <p>Name: {UserInfo.name}</p>
      <p>Like: {UserInfo.like}</p>
    </>
  );
}
```

## Passing Props to a Component

ドキュメント参考箇所:

- [Passing Props to a Component – React](https://react.dev/learn/passing-props-to-a-component)

- props = 小道具
- 親componentは**props**を使って子componentに情報を渡す
- **props**はオブジェクト、配列、関数などあらゆるJavaScriptを渡せる

componentとpropsを学んで、昨日見た`Inertia::render`メソッドの引数の意味が分かってきた。

```php
// Inertia\Inertia::render
public static function render($component, $props = []) { }
```

### propsの渡し方

1. 子component（`User`）に**props**を渡す（ここでは`userInfo`オブジェクト）
2. 子component（`User`）内で**props**を読み込む

```jsx
function User({userInfo}) {
  return (
    <>
      <p>Name: {userInfo.name}</p>
      <p>Like: {userInfo.like}</p>
    </>
  );
}

export default function UserList() {
  return (
    <section>
      <h1>UserList</h1>
      <User 
        userInfo={{name: 'Ryosuke', like: 'coffee'}}
      />
    </section>
  );
}
```

ネストする方法

```jsx
// App.js
import User from './User.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function UserList() {
  return (
    <Card>
      <User 
        userInfo={{
          name: 'Ryosuke', like: 'coffee'
        }}
      />
    </Card>
  );
}
```

```jsx
// User.js

export default function User({ userInfo }) {
  return (
    <>
      <p>Name: {userInfo.name}</p>
      <p>Like: {userInfo.like}</p>
    </>
  );
}
```

以下のパートは後から出てくる**state**に関わるようで軽く読むだけにした。

- [Passing Props to a Component – React](https://react.dev/learn/passing-props-to-a-component#how-props-change-over-time)

## 条件付きレンダリング

ドキュメント参照箇所:

- [Conditional Rendering – React](https://react.dev/learn/conditional-rendering)

JavaScriptの構文と同じように`if`や`&&`、`? :`演算子が使える。

以下はTodoリストの例。`isDone`の値で条件分岐させている。ただしDRYのため次で書き換える。

```jsx
// App.js
function Task({ task, isDone }) {
  if (isDone) {
    return <li className="task">{task} ✅</li>;
  }
  return <li className="task">{task}</li>;
}

export default function ToDoList() {
  return (
    <section>
      <h1>Today's Tasks</h1>
      <ul>
        <Task 
          isDone={false} 
          task="Study React" 
        />
        <Task 
          isDone={false} 
          task="Write an article" 
        />
        <Task 
          isDone={true} 
          task="Buy some coffee" 
        />
      </ul>
    </section>
  );
}
```

三項演算子（`? :`）によってDRYを解消。

```jsx
function Task({ task, isDone }) {
    return (
      <li className="task">
        { isDone ? task + ' ✅' : task }
      </li>
    );
}
```

条件式の中でさらにHTMLタグを使う例。JSX内のJavaScript式は`{}`で、複数行のJSXは`()`で囲む。

```jsx
function Task({ task, isDone }) {
  return (
    <li className="task">
      { isDone ? (<del>{task + ' ✅'}</del>) : task }
    </li>
  );
}
```

論理積（`&&`）演算子を使う例。左から全て`true`で通過した場合は最後のオペランドが返される。`false`があった時点で評価されず終了。文法はMDNを参照: [Logical AND (&&) - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND)

```jsx
function Task({ task, isDone }) {
  return (
    <li className="task">
      {task} { isDone && '✅' }
    </li>
  );
}
```

## Rendering Lists

ドキュメント参考箇所:

- [Rendering Lists – React](https://react.dev/learn/rendering-lists)

**Describing the UI**という最初のチャプターの最後のパート。

例えば「従業員 : 部署」というリストを表示したいとする。

```html
<ul>
  <li>青木: 営業部</li>
  <li>井上: 商品部</li>
  <li>上田: 総務部</li>
  <li>江原: 人事部</li>
  <li>小田: 営業部</li>
</ul>
```

JavaScriptの`map`を使って

- [Array.prototype.map() - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

```jsx
const employees = [
  '青木 : 営業部',
  '井上 : 商品部',
  '上田 : 総務部',
  '江原 : 人事部',
  '小田 : 営業部',
];

export default function List() {
  const listItems = employees.map(employee =>
    <li>{employee}</li>
  );
  return <ul>{listItems}</ul>;
}
```

`map`メソッドによって`listItems`は以下のような配列になるはず。それを`<ul>`タグの中でレンダリングさせている。

```jsx
// listItemsの中身
[
  <li>青木 : 営業部</li>,
  <li>井上 : 商品部</li>,
  <li>上田 : 総務部</li>,
  <li>江原 : 人事部</li>,
  <li>小田 : 営業部</li>
]
```

表示はされるがコンソールに次のようなエラーが出る。

```bash
Warning: Each child in a list should have a unique "key" prop.
```

これは、配列の各要素に`key`が必要だから（ユニークキー）

```jsx
<li key={employee.id}>...</li>
```

次のように各リストにユニークキーを持たせるように書き換える。

```jsx
const employees = [
  { id: 1, name: '青木', department: '営業部' },
  { id: 2, name: '井上', department: '商品部' },
  { id: 3, name: '上田', department: '総務部' },
  { id: 4, name: '江原', department: '人事部' },
  { id: 5, name: '小田', department: '営業部' }
];

export default function List() {
  const listItems = employees.map(employee =>
    <li key={employee.id}>
      {employee.name} : {employee.department}
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

`filter`メソッドを使って「営業部」だけに絞り込んだ例を書いてみる。

- [Array.prototype.filter() - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

```jsx
const employees = [
  { id: 1, name: '青木', department: '営業部' },
  { id: 2, name: '井上', department: '商品部' },
  { id: 3, name: '上田', department: '総務部' },
  { id: 4, name: '江原', department: '人事部' },
  { id: 5, name: '小田', department: '営業部' }
];

export default function List() {
  const sales = employees.filter(employee => 
    employee.department === '営業部'
  );
  
  const listItems = sales.map(employee => 
    <li key={employee.id}>
      {employee.name} : {employee.department}
    </li>
  );
  
  return <ul>{listItems}</ul>;
}

```
