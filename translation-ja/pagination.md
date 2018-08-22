# データベース：ペジネーション

- [イントロダクション](#introduction)
- [基本的な使用法](#basic-usage)
    - [クエリビルダの結果](#paginating-query-builder-results)
    - [Eloquentの結果](#paginating-eloquent-results)
    - [独自ペジネータ作成](#manually-creating-a-paginator)
- [ペジネーション結果の表示](#displaying-pagination-results)
    - [結果のJSON変換](#converting-results-to-json)
- [ペジネーションビューのカスタマイズ](#customizing-the-pagination-view)
- [ペジネータインスタンスメソッド](#paginator-instance-methods)

<a name="introduction"></a>
## イントロダクション

他のフレームワークのペジネーションは苦痛に満ちています。Laravelのペジネータは[クエリビルダ](/docs/{{version}}/queries)と[Eloquent ORM](/docs/{{version}}/eloquent)に統合されており、データベースの結果を簡単、お手軽にペジネーションできます。ペジネータが生成するHTMLは、[Bootstrap CSSフレームワーク](https://getbootstrap.com/)コンパチブルです。

<a name="basic-usage"></a>
## 基本的な使用法

<a name="paginating-query-builder-results"></a>
### クエリビルダの結果

アイテムをペジネーションするには多くの方法があります。一番簡単な方法は、[クエリビルダ](/docs/{{version}}/queries)と[Eloquent query](/docs/{{version}}/eloquent)へ`paginate`メソッドを使う方法です。`paginate`メソッドは、ユーザーが表示している現在のページに基づき、正しいアイテム数とオフセットを指定する面倒を見ます。デフォルトではHTTPリクエストの`page`クエリ文字列引数の値により現在ページが決められます。もちろんこの値はLaravelが自動的に探し、さらにペジネーターが挿入するリンクを自動的に生成します。

以下の例では、`paginate`に一つだけ引数を渡しており、「ページごと」に表示したいアイテム数です。この例ではページごとに`15`アイテムを表示するように指定しています。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * アプリケーションの全ユーザー表示
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} 現在`groupBy`文を使用したペジネーション操作は、Laravelで効率よく実行できません。`groupBy`を使用したペジネーションを使用する必要がある場合はデータベースクエリを実行し、その結果を元にペジネーターを自前で作成してください。

#### シンプル・ペジネーション

「次」と「前」のリンクだけのシンプルなペジネーションビューを表示したい場合は`simplePaginate`メソッドを使用し、より効率的にクエリすべきでしょう。これはビューに正確なページ番号を表示する必要がない、巨大なデータセットを扱う場合に便利です。

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### Eloquentの結果

さらに[Eloquent](/docs/{{version}}/eloquent)モデルもペジネーションできます。例として`User`モデルの`15`アイテムをページ付け表示してみましょう。ご覧の通り、クエリビルダ結果のペジネーションを行う記法はきれいでわかりやすいものです。

    $users = App\User::paginate(15);

もちろん`where`節のような制約をクエリに指定した後に`paginate`を呼び出すこともできます。

    $users = User::where('votes', '>', 100)->paginate(15);

Elqouentモデルをページづけするときにも、`simplePaginate`メソッドを使用できます。

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 独自ペジネータ作成

渡された配列を元にして、ペジネーションインスンタンスを作成したいこともあります。必要に応じて`Illuminate\Pagination\Paginator`か、`Illuminate\Pagination\LengthAwarePaginator`インスタンスを生成することで実現できます。

`Paginator`クラスは結果にセットされているアイテムの総数を知る必要はありません。そのためクラスは最終ページのインデックスを取得するメソッドを持っていません。`LengthAwarePaginator`は`Paginator`とほとんど同じ引数を取りますが、結果にセットされているアイテム総数も指定する必要がある点が異なっています。

言い換えれば、`Paginator`はクエリビルダとEloquentに対する`simplePaginate`メソッドに対応し、一方の`LengthAwarePaginator`は`paginate`に対応しています。

> {note} 自前でペジネーターインスタンスを生成する場合、ペジネーターに渡す結果の配列を自分で"slice"する必要があります。その方法を思いつかなければ、[array_slice](https://secure.php.net/manual/en/function.array-slice.php) PHP関数を調べてください。

<a name="displaying-pagination-results"></a>
## ペジネーション結果の表示

`paginate`メソッドを呼び出す場合、`Illuminate\Pagination\LengthAwarePaginator`インスタンスを受け取ります。`simplePaginate`メソッドを呼び出すときは、`Illuminate\Pagination\Paginator`インスタンスを受け取ります。これらのオブジェクトは結果を表すたくさんのメソッドを提供しています。こうしたヘルパメソッドに加え、ペジネーターインスタンスはイテレータでもあり、配列としてループ処理できます。つまり結果を取得したら、その結果とページリンクを[Blade](/docs/{{version}}/blade)を使い表示できます。

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

`links`メソッドは結果の残りのページヘのリンクをレンダーします。それらの各リンクには`page`クエリ文字列変数が含まれています。`links`メソッドが生成するHTMLは[Bootstrap CSSフレームワーク](https://getbootstrap.com)と互換性があることを覚えておいてください。

#### ペジネーターURIのカスタマイズ

`withPath`メソッドにより、ペジネーターがリンクを生成するときに使用するURIをカスタマイズできます。たとえばペジネーターで`http://example.com/custom/url?page=N`のようなリンクを生成したい場合、`withPath`メソッドに`custom/url`を渡してください。

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

#### ペジネーションリンクの追加

ペジネーションリンクにクエリ文字列を付け加えたいときは、`appends`メソッドを使います。たとえば`sort=votes`を各ペジネーションリンクに追加する場合には、以下のように`appends`を呼び出します。

    {{ $users->appends(['sort' => 'votes'])->links() }}

ペジネーションのURLに「ハッシュフラグメント」を追加したい場合は、`fragment`メソッドが使用できます。例えば各ペジネーションリンクの最後に`#foo`を追加したい場合は、以下のように`fragment`メソッドを呼び出します。

    {{ $users->fragment('foo')->links() }}

<a name="converting-results-to-json"></a>
### 結果のJSON変換

Laravelのペジネーター結果クラスは`Illuminate\Contracts\Support\Jsonable`インターフェイス契約を実装しており、`toJson`メソッドを提示しています。ですからペジネーション結果をJSONにとても簡単に変換できます。またルートやコントローラアクションからペジネーターインスタンスを返せば、JSONへ変換されます。

    Route::get('users', function () {
        return App\User::paginate();
    });

ペジネーターのJSON形式は`total`、`current_page`、`last_page`などのメタ情報を含んでいます。実際の結果オブジェクトはJSON配列の`data`キーにより利用できます。ルートから返されたペジネーターインスタンスにより生成されるJSONの一例を見てください。

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // 結果のオブジェクト
            },
            {
                // 結果のオブジェクト
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## ペジネーションビューのカスタマイズ

デフォルトで、ペジネーションリンクを表示するためのビューはBootstrap CSSフレームワークを用いてレンダーされます。しかし、Bootstrapを使っていない場合でも、そうしたリンクをレンダーする独自のビューを自由に定義できます。ペジネータインスタンスの`links`メソッドを呼び出す際に、ビュー名をメソッドの最初の引数として渡してください。

    {{ $paginator->links('view.name') }}

    // ビューへデータを渡す
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

しかし、`vendor:publish`コマンドを使用し、`resources/views/vendor`ディレクトリへペジネーションビューを作成し、カスタマイズする方法が一番簡単でしょう。

    php artisan vendor:publish --tag=laravel-pagination

このコマンドは、`resources/views/vendor/pagination`ディレクトリへビューを設置します。このディレクトリの`bootstrap-4.blade.php`ファイルが、デフォルトのペジネーションビューに当ります。ペジネーションHTMLを変更するために、このファイルを編集できます。

デフォルトのペジネーションビューとして、他のファイルを指定したい場合は、`AppServiceProvider`の中で、ペジネータの`defaultView`と`defaultSimpleView`メソッドを使用します。

    use Illuminate\Pagination\Paginator;

    public function boot()
    {
        Paginator::defaultView('pagination::view');

        Paginator::defaultSimpleView('pagination::view');
    }

<a name="paginator-instance-methods"></a>
## ペジネータインスタンスメソッド

ペジネータインスタンスは以下の追加ペジネーション情報を提供しています。

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (simplePaginateでは使用不可)`
- `$results->nextPageUrl()`
- `$results->onFirstPage()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (simplePaginateでは使用不可)`
- `$results->url($page)`
