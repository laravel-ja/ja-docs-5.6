# コントローラ

- [イントロダクション](#introduction)
- [基本のコントローラ](#basic-controllers)
    - [コントローラの定義](#defining-controllers)
    - [コントローラと名前空間](#controllers-and-namespaces)
    - [シングルアクションコントローラ](#single-action-controllers)
- [コントローラミドルウェア](#controller-middleware)
- [リソースコントローラ](#resource-controllers)
    - [部分的なリソースルート](#restful-partial-resource-routes)
    - [リソースルートの命名](#restful-naming-resource-routes)
    - [リソースルートパラメータの命名](#restful-naming-resource-route-parameters)
    - [リソースURIのローカライズ](#restful-localizing-resource-uris)
    - [リソースコントローラへのルート追加](#restful-supplementing-resource-controllers)
- [依存注入とコントローラ](#dependency-injection-and-controllers)
- [ルートキャッシュ](#route-caching)

<a name="introduction"></a>
## イントロダクション

全リクエストの処理をルートファイルのクロージャで定義するよりも、コントローラクラスにより組織立てたいと、皆さんも考えるでしょう。関連のあるHTTPリクエストの処理ロジックを一つのクラスへまとめ、グループ分けができます。コントローラは`app/Http/Controllers`ディレクトリ下に設置します。

<a name="basic-controllers"></a>
## 基本のコントローラ

<a name="defining-controllers"></a>
### コントローラの定義

これは基本的なコントローラの一例です。全てのLaravelコントローラはLaravelに含まれている基本コントローラクラスを拡張します。コントローラアクションにミドルウェアを追加するために使う`middleware`メソッドのように、便利なメソッドをベースクラスは提供しています。

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 指定ユーザーのプロフィール表示
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

コントローラアクションへルート付けるには、次のようにします。

    Route::get('user/{id}', 'UserController@show');

これで指定したルートのURIにリクエストが一致すれば、`UserController`の`show`メソッドが実行されます。もちろん、ルートパラメーターはメソッドに渡されます。

> {tip} コントローラはベースクラスの拡張を**要求**してはいません。しかし、`middleware`、`validate`、`dispatch`のような便利な機能へアクセスできなくなります。

<a name="controllers-and-namespaces"></a>
### コントローラと名前空間

とても重要な注目ポイントはコントローラルートの定義時に、コントローラの完全な名前空間を指定する必要がないことです。`RouteServiceProvider`が、コントローラの名前空間を指定したルートグループの中で、ルートファイルをロードしていますので、「先頭」の`App\Http\Controllers`名前空間に続くクラス名の部分だけを指定するだけで済みます。

`App\Http\Controllers`ディレクトリより深く、コントローラのPHP名前空間をネストしたり、組織立てたりする場合でも、先頭の`App\Http\Controllers`名前空間からの相対クラス名を指定します。ですから、コントローラの完全なクラス名が`App\Http\Controllers\Photos\AdminController`ならば、次のようにルートを登録します。

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### シングルアクションコントローラ

アクションを一つだけ含むコントローラを定義したい場合は、そのコントローラに`__invoke`メソッドを設置してください。

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * 指定ユーザーのプロフィール表示
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

シングルアクションコントローラへのルートを定義するとき、メソッドを指定する必要はありません。

    Route::get('user/{id}', 'ShowProfile');

`make:controller` Artisanコマンドに、`--invokable`オプションを指定すると、`__invoke`メソッドを含んだコントローラを生成できます。

    php artisan make:controller ShowProfile --invokable

<a name="controller-middleware"></a>
## コントローラミドルウェア

[ミドルウェア](/docs/{{version}}/middleware)はルートファイルの中で、コントローラのルートに対して指定します。

    Route::get('profile', 'UserController@show')->middleware('auth');

もしくは、コントローラのコンストラクタの中でミドルウェアを指定するほうが、より便利でしょう。コントローラのコンストラクタで、`middleware`メソッドを使い、コントローラのアクションに対するミドルウェアを簡単に指定できます。コントローラクラスの特定のメソッドに対してのみ、ミドルウェアの適用を制限することもできます。

    class UserController extends Controller
    {
        /**
         * 新しいUserControllerインスタンスの生成
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

コントローラではクロージャを使い、ミドルウェアを登録することもできます。これはミドルウェア全体を定義せずに、一つのコントローラのために、一つのミドルウェアを定義する便利な方法です。

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} コントローラアクションの一部へミドルウェアを適用することはできますが、しかしながら、これはコントローラが大きくなりすぎたことを示しています。代わりに、コントローラを複数の小さなコントローラへ分割することを考えてください。

<a name="resource-controllers"></a>
## リソースコントローラ

Laravelリソースルートは一行のコードで、典型的な「CRUD」ルートをコントローラへ割り付けます。たとえば、アプリケーションへ保存されている「写真(photo)」に対する全HTTPリクエストを処理するコントローラを作成したいとしましょう。`make:controller` Artisanコマンドを使えば、このようなコントローラは素早く生成できます。

    php artisan make:controller PhotoController --resource

このArtisanコマンドは`app/Http/Controllers/PhotoController.php`としてコントローラファイルを生成します。コントローラは使用可能な各リソース操作に対するメソッドを含んでいます。

次に、コントローラへのリソースフルルートを登録します。

    Route::resource('photos', 'PhotoController');

リソースに対する様々なアクションを処理する、複数のルートがこの１定義により生成されます。これらのアクションをHTTP動詞と処理するURIの情報を注記と一緒に含むスタブメソッドとして、生成されたコントローラはすでに含んでいます。

一度に多くのリソースコントローラを登録するには、`resources`メソッドへ配列で渡します。

    Route::resources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

#### リソースコントローラにより処理されるアクション

動詞      | URI                  | アクション       | ルート名
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### リソースモデルの指定

ルートモデル結合を使用しているが、リソースコントローラのメソッドでタイプヒントされるモデルインスタンスを指定したい場合は、コントローラの生成時に`--model`オプションを使用します。

    php artisan make:controller PhotoController --resource --model=Photo

#### 擬似フォームメソッド

HTMLフォームは`PUT`、`PATCH`、`DELETE`リクエストを作成できませんので、HTTP動詞を偽装するために、`_method`隠しフィールドを追加する必要が起きるでしょう。`@method` Bladeディレクティブでこのフィールドを生成できます。

    <form action="/foo/bar" method="POST">
        @method('PUT')
    </form>

<a name="restful-partial-resource-routes"></a>
### 部分的なリソースルート

リソースルートの宣言時に、デフォルトアクション全部を指定する代わりに、ルートで処理するアクションの一部を指定可能です。

    Route::resource('photos', 'PhotoController')->only([
        'index', 'show'
    ]);

    Route::resource('photos', 'PhotoController')->except([
        'create', 'store', 'update', 'destroy'
    ]);

#### APIリソースルート

APIに使用するリソースルートを宣言する場合、`create`や`edit`のようなHTMLテンプレートを提供するルートを除外したいことがよく起こります。そのため、これらの２ルートを自動的に除外する、`apiResource`メソッドが使用できます。

    Route::apiResource('photos', 'PhotoController');

`apiResources`メソッドに配列として渡すことで、一度に複数のAPIリソースコントローラを登録できます。

    Route::apiResources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

`create`や`edit`メソッドを含まないAPIリソースコントローラを素早く生成するには、`make:controller`コマンドを実行する際、`--api`スイッチを使用してください。

    php artisan make:controller API/PhotoController --api

<a name="restful-naming-resource-routes"></a>
### リソースルートの命名

全てのリソースコントローラアクションは、デフォルトのルート名が決められています。しかし、オプションに`names`配列を渡せば、こうした名前をオーバーライドできます。

    Route::resource('photos', 'PhotoController')->names([
        'create' => 'photos.build'
    ]);

<a name="restful-naming-resource-route-parameters"></a>
### リソースルートパラメータの命名

By default, `Route::resource` will create the route parameters for your resource routes based on the "singularized" version of the resource name. You can easily override this on a per resource basis by using the `parameters` method. The array passed into the `parameters` method should be an associative array of resource names and parameter names:

    Route::resource('users', 'AdminUserController')->parameters([
        'users' => 'admin_user'
    ]);

上記のサンプルコードは、リソースの`show`ルートで次のURIを生成します。

    /users/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### リソースURIのローカライズ

`Route::resource`はデフォルトで、リソースURIに英語の動詞を使います。`create`と`edit`アクションの動詞をローカライズする場合は、`Route::resourceVerbs`メソッドを使います。このメソッドは、`AppServiceProvider`の`boot`メソッド中で呼び出します。

    use Illuminate\Support\Facades\Route;

    /**
     * 全アプリケーションサービスの初期起動処理
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }

動詞をカスタマイズすると、`Route::resource('fotos', 'PhotoController')`のようなリソースルートの登録により、以下のようなURIが生成されるようになります。

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### リソースコントローラへのルート追加

デフォルトのリソースルート以外のルートをリソースコントローラへ追加する場合は、`Route::resource`の呼び出しより前に定義する必要があります。そうしないと、`resource`メソッドにより定義されるルートが、追加のルートより意図に反して優先されます。

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} コントローラの責務を限定することを思い出してください。典型的なリソースアクションから外れたメソッドが繰り返して必要になっているようであれば、コントローラを２つに分け、小さなコントローラにすることを考えましょう。

<a name="dependency-injection-and-controllers"></a>
## 依存注入とコントローラ

#### コンストラクターインジェクション

全コントローラの依存を解決するために、Laravelの[サービスコンテナ](/docs/{{version}}/container)が使用されます。これにより、コントローラが必要な依存をコンストラクターにタイプヒントで指定できるのです。依存クラスは自動的に解決され、コントローラへインスタンスが注入されます。

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * ユーザーリポジトリインスタンス
         */
        protected $users;

        /**
         * 新しいコントローラインスタンスの生成
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

もちろん、[Laravelの契約](/docs/{{version}}/contracts)もタイプヒントに指定できます。コンテナが解決できるのであれば、タイプヒントで指定できます。 アプリケーションによっては、依存をコントローラへ注入すれば、より良いテスタビリティが得られるでしょう。

#### メソッドインジェクション

コンストラクターによる注入に加え、コントローラのメソッドでもタイプヒントにより依存を指定することもできます。メソッドインジェクションの典型的なユースケースは、コントローラメソッドへ`Illuminate\Http\Request`インスタンスを注入する場合です。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 新ユーザーの保存
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

コントローラメソッドへルートパラメーターによる入力値が渡される場合も、依存定義の後に続けてルート引数を指定します。たとえば以下のようにルートが定義されていれば：

    Route::put('user/{id}', 'UserController@update');

下記のように`Illuminate\Http\Request`をタイプヒントで指定しつつ、コントローラメソッドで定義している`id`パラメータにアクセスできます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 指定ユーザーの更新
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## ルートキャッシュ

> {note} ルートキャッシュはクロージャベースのルートには動作しません。ルートキャッシュを使用するには、全クロージャルートをコントローラクラスを使用するように変更する必要があります。

アプリケーションがコントローラベースのルート定義だけを使用しているなら、Laravelのルートキャッシュを利用できる利点があります。ルートキャッシュを使用すれば、アプリケーションの全ルートを登録するのに必要な時間を劇的に減らすことができます。ある場合には、ルート登録が１００倍も早くなります。ルートキャッシュを登録するには、`route:cache` Arisanコマンドを実行するだけです。

    php artisan route:cache

このコマンドを実行後、キャッシュ済みルートファイルが、リクエストのたびに読み込まれます。新しいルートを追加する場合は、新しいルートキャッシュを生成する必要があることを覚えておきましょう。ですからプロジェクトの開発期間の最後に、一度だけ`route:cache`を実行するほうが良いでしょう。

キャッシュルートのファイルを削除するには、`route:clear`コマンドを使います。

    php artisan route:clear
