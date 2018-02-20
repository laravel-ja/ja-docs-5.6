# リリースノート

- [バージョニング規約](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel 5.6](#laravel-5.6)

<a name="versioning-scheme"></a>
## バージョニング規約

Laravelのバージョニングは、「パラダイム.メジャー・マイナー」の規約を維持しています。メジャーフレームワークリリースは、１月と６月の半年ごとにリリースします。一方、マイナーリリースは毎週のように、頻繁にリリースされます。マイナーリリースは、ブレーキングチェンジを**絶対に**含めません。

アプリケーションやパッケージで、Laravelフレームワークやコンポーネントを利用する場合、常に`5.5.*`のようにバージョンを指定してください。理由は上記の通り、Laravelのメジャーリリースは、ブレーキングチェンジを含んでいるからです。新しいメジャーリリースへの更新は、一日かからない程度になるように努力しています。

パラダイムシフトリリースは数年空けています。これはフレームワークの構造と規約に重要な変更が起きたことを表します。現在、パラダイムシフトリリースは開発されていません。

<a name="support-policy"></a>
## サポートポリシー

Laravel5.5のようなLTSリリースでは、バグフィックスは２年間、セキュリティフィックスは３年間提供します。これらのリリースは長期間に渡るサポートとメンテナンスを提供します。 一般的なリリースでは、バグフィックスは６ヶ月、セキュリティフィックスは１年です。

<a name="laravel-5.6"></a>
## Laravel 5.6

Laravel5.6はLaravel5.5からの持続的な向上に付け加え、向上したログシステム、シングルサーバタスクスケジュール、向上したモデルのシリアライズ、動的レート制限、ブロードキャストチャンネルクラス、APIリソースコントローラ生成、Eloquent日付フォーマットの向上、Bladeコンポーネント別名、Argon2パスワードハッシュサポート、Collisionパッケージの同梱などを追加しました。更に、フロントエンドのスカフォールドは、Bootstrap4向けにアップグレードされました。

Laravelが背後で使用しているSymfonyコンポーネントは、Symfonyの`~4.0`リリースシリーズへアップグレードされました。

Laravel5.6のリリースは[Spark6.0](https://spark.laravel.com)と同時にリリースされました。Laravel Sparkがリリースされてから、初めてのメジャーアップグレードです。Spark6.0では、StripeとBraintreeに対するユーザー数に応じた価格や、ローカリゼーション、Bootstrap4、UIの向上、Stripe Elementsのサポートを導入しました。

> {tip} このドキュメントはフレームワークで注目してもらいたい機能向上についてまとめたものです。より全体的な変更ログは、いつでも[GitHub](https://github.com/laravel/framework/blob/5.6/CHANGELOG-5.6.md)で確認できます。

### ログの向上

Laravel5.5ではログシステムが大いに向上しています。ログの設定はすべて、新しい`config/logging.php`設定ファイルにあります。ログメッセージを複数のハンドラへ送る、ログ「スタック」が簡単に構築できるようになりました。たとえば、`debug`レベルのメッセージは全てシステムログへ送り、`error`レベルのメッセージでは、すぐに対応できるようにSlackへ送ることができます。

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],
    ],

さらに、新しいログシステムの"tap"機能を使えば、既存のログチャンネルを簡単にカスタマイズできるようになりました。詳細は、[ログのドキュメント全文](/docs/{{version}}/logging)をご覧ください。

### シングルサーバ・タスクスケジュール

> {note} この機能を使用するには、アプリケーションのデフォルトキャッシュドライバとして、`memcached`か`redis`キャッシュドライバを使用する必要があります。更に、全サーバが同じ単一のキャッシュサーバに接続している必要があります。

アプリケーションが複数のサーバで実行される場合、スケジュール済みのジョブを単一サーバ上のみで実行するよう制限できるようになりました。たとえば、毎週の金曜の夜に、新しいレポートを生成するタスクをスケジュールしていると仮定しましょう。タスクスケジューラが３つのワーカーサーバ上で実行されているなら、スケジュールされているタスクは３つ全部のサーバで実行され、３回レポートが生成されます。これではいけません。

タスクをサーバひとつだけで実行するように指示するには、スケジュールタスクを定義するときに`onOneServer`メソッドを使用します。このタスクを最初に取得したサーバが、同じタスクを同じCronサイクルで他のサーバで実行しないように、ジョブにアトミックなロックを確保します。

    $schedule->command('report:generate')
             ->fridays()
             ->at('17:00')
             ->onOneServer();

### 動的レート制限

以前のLaravelのリリースでは、ルートのグループへ[レート制限](/docs/{{version}}/routing#rate-limiting)を指定する場合、リクエストの最大回数をハードコードする必要がありました。

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

Laravel5.6では、認証された`User`モデルの属性にもとづいて、リクエストの最大回数が動的に指定されます。たとえば、`User`モデルが`rate_limit`属性を含んでいれば、属性の名前を`throttle`ミドルウェアに指定することで、最大リクエストカウントを計算するために使用されます。

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

### ブロードキャストチャンネルクラス

アプリケーションで多くのチャンネルを利用していると、`routes/channels.php`ファイルは膨大になってしまいます。認証チャンネルのクロージャを使用する代わりに、チャンネルクラスを使用するのが良いでしょう。チャンネルクラスを生成するには、`make:channel` Aritisanコマンドが使用できます。このコマンドは、新しいチャンネルクラスを`App/Broadcasting`ディレクトリへ生成します。

    php artisan make:channel OrderChannel

次に、チャンネルを`routes/channels.php`ファイルで登録します。

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

最後に、チャンネルの認証ロジックをチャンネルクラスの`join`へ記述します。典型的な場合ではチャンネル認証クロージャに設置するのと同じロジックをこの`join`メソッドに設置します。もちろん、チャンネルモデル結合の利点も利用できます。

    <?php

    namespace App\Broadcasting;

    use App\User;
    use App\Order;

    class OrderChannel
    {
        /**
         * 新しいチャンネルインスタンスの生成
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * ユーザーのチャンネルへアクセスを認証
         *
         * @param  \App\User  $user
         * @param  \App\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

### APIコントローラの生成

APIにより使用されるリソースルートを定義する場合、`create`や`edit`のようにHTMLテンプレートに存在するルートを通常除外します。これらのメソッドを含まないリソースコントローラを生成するには、`make:controller`実行時に`--api`スイッチを使用できるようになりました。

    php artisan make:controller API/PhotoController --api

### モデルのシリアライズの向上

以前のLaravelリリースでは、キューされたモデルがロードしていたリレーションは、そのままリストアされませんでした。Laravel5.6では、キューされたモデルがロードしていたリレーションは、キューによりジョブが処理される際に自動的にリロードされます。

### Eloquent日付キャスト

Eloquent日付キャストカラムのフォーマットを個別にカスタマイズできるようになりました。最初に、キャスト宣言の中で、希望する日付形式を指定します。一度指定すると、このフォーマットはモデルを配列やJSON西リアライズするとき、そのフォーマットが使用されます。

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];

### Bladeコンポーネント別名

Bladeコンポーネントがサブディレクトリへ保存している場合、アクセスを簡単にするために別名が付けられるようになりました。たとえば、Bladeコンポーネントを`resources/views/components/alert.blade.php`として保存しているとイメージしてください。`components.alert`を`alert`として、このコンポーネントとして別名を付けるには、`component`メソッドを使用します。

    Blade::component('components.alert', 'alert');

コンポーネントに別名を付けると、ディレクティブを使いレンダできます。

    @alert('alert', ['type' => 'danger'])
        You are not allowed to access this resource!
    @endalert

追加のスロットがなければ、コンポーネントパラメータを省略できます。

    @alert
        You are not allowed to access this resource!
    @endalert

### Argon2パスワードハッシュ

アプリケーションをバージョン7.2.0以降のPHP上で構築している場合、LaravelはArgon2アルゴリズムによるパスワードハッシュをサポートするようになりました。アプリケーションのデフォルトハッシュドライバーは、新しい`config/hashing.php`設定ファイルでコントロールします。

### UUIDメソッド

Laravel5.6で新しいUUID生成メソッドが導入されました。`Str::uuid`と`Str::orderedUuid`です。`orderedUuid`メソッドは、MySQLのようなデータベースにより、より簡単に、より効率的にインディックスされるタイムスタンプ先行のUUIDを生成します。これらのメソッドは、`Ramsey\Uuid\Uuid`オブジェクトを返します。

    use Illuminate\Support\Str;

    return (string) Str::uuid();

    return (string) Str::orderedUuid();

### Collision

デフォルトの`laravel/laravel`アプリケーションは、`dev` Composer依存パッケージとして、Nuno Maduro氏によりメンテナンスされている[Collision](https://github.com/nunomaduro/collision)パッケージを含むようになりました。このパッケージは、コマンドラインのLaravelアプリケーションを操作する際、美しいエラーレポートを提供してくれます。

<img src="https://raw.githubusercontent.com/nunomaduro/collision/stable/docs/example.png" width="600" height="388">

### Bootstrap 4

認証の定形コードやVueコンポーネント例のような、フロントエンドの全スカフォールドが、[Bootstrap4](https://blog.getbootstrap.com/2018/01/18/bootstrap-4/)へアップグレードされました。デフォルトとして生成するペジネーションリンクも、Bootstrap4になりました。
