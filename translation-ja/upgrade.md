# アップグレードガイド

- [Upgrading To 5.6.0 From 5.5](#upgrade-5.6.0)

<a name="upgrade-5.6.0"></a>
## 5.5から5.6.0へのアップグレード

#### 見積もり時間：１０〜３０分

> {note} 私達は、互換性を失う可能性がある変更を全部ドキュメントにしようとしています。しかし、変更点のいくつかは、フレームワークの明確ではない部分で行われているため、一部の変更が実際にアプリケーションに影響を与えてしまう可能性があります。

### PHP

Laravel5.6の動作には、PHPバージョン7.1.3以上が必要です。

### 依存パッケージのアップデート

`composer.json`ファイルの`laravel/framework`依存指定を`5.6.*`に、`fideloper/proxy`依存指定を`~4.0`へアップデートしてください。

さらに、以下のファーストパーティLaravelパッケージを使用している場合は、最新のリリースへアップグレードしてください。

<div class="content-list" markdown="1">
- Dusk (`~3.0`へアップグレード)
- Passport (`~5.0`へアップグレード)
- Scout (`~4.0`へアップグレード)
</div>

もちろん、アプリケーションで使用しているサードパーティパッケージについても調査し、Laravel5.6をサポートしているバージョンを使用していることを確認してください。

#### Symfony4

Laravelを動作させるために使用している、すべてのSymfonyコンポーネントは、Symfony `~4.0`シリーズへアップグレードされました。アプリケーションで直接Symfonyコンポーネントを取り扱っている場合は、[Symfonyの変更ログ](https://github.com/symfony/symfony/blob/master/UPGRADE-4.0.md)をレビューすべきでしょう。

#### PHPUnit

アプリケーションの`phpunit/phpunit`依存指定は、`~7.0`にアップグレードしてください。

### 配列

#### `Arr::wrap`メソッド

`Arr::wrap`メソッドへ`null`を渡すと、空の配列が返ってくるようになりました。

### Artisan

#### `optimize`コマンド

以前から非推奨になっていた、`optimize` Artisanコマンドは削除されました。PHP自身がOPcacheを含んだ、最近の向上により、`optimize`コマンドは妥当なパフォーマンスの向上を提供できなくなりました。そのため、`composer.json`ファイルの`scripts`項目から、`php artisan optimize`を取り除いてください。

### Blade

#### HTMLエンティティエンコーディング

以前のバージョンのLaravelでは、Blade（および`e`ヘルパ）は、HTMLエンティティをダブルエンコードしませんでした。これは、裏で使用している`htmlspecialchars`関数のデフォルト動作とは異なり、コンテンツのレンダリングやインラインJSONコンテンツをJavaScriptフレームワークへ渡す際に、予想外の動作を引き起こしました。

Laravel5.6では、Bladeと`e`ヘルパはデフォルトとして、特別な文字をダブルエンコードします。これにより、裏で動作しているPHPの`htmlspecialchars`関数のデフォルトと、この機能は動作が統一されました。ダブルエンコーディングを行わない、以前の振る舞いを継続したい場合は、`Blade::withoutDoubleEncoding`メソッドを使用します。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションの全サービスの初期処理
         *
         * @return void
         */
        public function boot()
        {
            Blade::withoutDoubleEncoding();
        }
    }

### キャッシュ

#### レートリミッターの`tooManyAttempts`メソッド

このメソッドの引数から、使用されていなかった`$decayMinutes`引数が削除されました。このメソッドを自分の実装でオーバーライドしている場合、メソッドからこの引数を削除してください。

### データベース

#### morphsカラムのインデックス順

`morphs`マイグレーションメソッドにより作られるカラムのインデックスが、効率を改善するために改善されました。マイグレーションで`morphs`メソッドを使用している場合は、マイグレーションの`down`メッソッドを実行しようと試みると、エラーが発生します。アプリケーションが開発段階の場合は、`migrate:fresh`コマンドを使い、初めからデータベースを構築し直してください。アプリケーションが実働している場合は、明確に`morphs`メソッドにインデックス名を渡す必要があります。

#### `MigrationRepositoryInterface`メソッド追加

`MigrationRepositoryInterface`へ、新しく`getMigrationsBatches`メソッドが追加されました。このクラスを自分で実装するというのは、まずあり得ませんが、実装している場合は実装にこのメソッドを追加してください。フレームワークのデフォルト実装を例として参照してください。

### Eloquent

#### `getDateFormat`メソッド

`getDateFormat`メソッドが、`protected`の代わりに`public`になりました。

### ハッシュ

#### 新しい設定ファイル

ハッシュの全設定は、`config/hashing.php`設定ファイルへ保存されるようになりました。[デフォルトの設定ファイル](https://github.com/laravel/laravel/blob/develop/config/hashing.php)をアプリケーションへコピーしてください。ほとんどの場合、デフォルトドライバーとして`bcrypt`を継続する必要があります。しかしながら、`argon`もサポートされました。

### ヘルパ

#### `e`ヘルパ

以前のバージョンのLaravelでは、Blade（および`e`ヘルパ）は、HTMLエンティティをダブルエンコードしませんでした。これは、裏で使用している`htmlspecialchars`関数のデフォルト動作とは異なり、コンテンツのレンダリングやインラインJSONコンテンツをJavaScriptフレームワークへ渡す際に、予想外の動作を引き起こしました。

Laravel5.6では、Bladeと`e`ヘルパはデフォルトとして、特別な文字をダブルエンコードします。これにより、裏で動作しているPHPの`htmlspecialchars`関数のデフォルトと、この機能は動作が統一されました。ダブルエンコーディングを行わない、以前の振る舞いを継続したい場合は、`e`ヘルパの第２引数に、`false`を渡してください。

    <?php echo e($string, false); ?>

### ログ

#### 新しい設定ファイル

新しいログ設定は、`config/logging.php`設定ファイルで全て保存されるようになりました。[デフォルト設定ファイル](https://github.com/laravel/laravel/blob/develop/config/logging.php)をアプリケーションへコピーし、必要に応じて設定を調整してください。

`config/app.php`設定ファイルから、`log`と`log_level`設定オプションは削除されました。

#### `configureMonologUsing`メソッド

アプリケーションのために、`configureMonologUsing`メソッドを使用し、Monologインスタンスをカスタマイズしている場合は、`custom`ログチャンネルを作成する必要があります。カスタムチャンネル作成の詳細は、[ログのドキュメント](/docs/5.6/logging#creating-custom-channels)で確認してください。

#### ログ`Writer`クラス

`Illuminate\Log\Writer`クラスは、`Illuminate\Log\Logger`に名前が変更されました。アプリケーションのクラスで、このクラスをタイプヒントで明確に指定している場合は、新しい名前を参照するように変更してください。もしくは代わりに、標準化された`Psr\Log\LoggerInterface`インターフェイスをタイプヒントで使うように、よく考慮してください。

#### `Illuminate\Contracts\Logging\Log`インターフェイス

`Psr\Log\LoggerInterface`インターフェイスと完全にダブってしまうため、このインターフェイスは削除されました。代わりに、`Psr\Log\LoggerInterface`をタイプヒントに使用してください。

### メール

#### `withSwiftMessage`コールバック

Laravelの以前のリリースでは、`withSwiftMessage`を使用し登録した、Swiftメッセージのカスタマイズコールバックは、コンテンツが既にエンコードされ、メッセージに追加された**後**に呼び出されていました。これらのコールバックが、コンテンツが追加される**前**に呼び出されるようになり、必要に応じてエンコードや他のメッセージオプションをカスタマイズできるようになりました。

### ペジネーション

#### Bootstrap4

ペジネータにより生成されるペジネーションリンクが、デフォルトでBootstrap4向けになりました。Bootstrap3向けに生成するように、ペジネータへ指示するには、`AppServiceProvider`の`boot`メソッドから、`Paginator::useBootstrapThree`メソッドを呼び出してください。

    <?php

    namespace App\Providers;

    use Illuminate\Pagination\Paginator;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションの全サービスの起動処理
         *
         * @return void
         */
        public function boot()
        {
            Paginator::useBootstrapThree();
        }
    }

### リソース

#### `original`プロパティ

[リソースレスポンス](/docs/5.6/eloquent-resources)の`original`プロパティへ、JSON文字列／配列の代わりにオリジナルのモデルがセットされるようになりました。これにより、テスト時にレスポンスのモデルの検査が簡単になりました。

### ルート

#### 新しく生成されたモデルの返却

ルートから直接、新たに生成されたEloquentモデルをリターンする場合、レスポンス状態は`200`の代わりに、`201`が自動的にセットされます。アプリケーションのテストで明確に、`200`レスポンスを期待している場合は、`201`を期待するように変更する必要があります。

### 信頼するプロキシ

Laravelが使用しているSymfony HttpFoundationの、信頼するプロキシの機能変更により、アプリケーションの`App\Http\Middleware\TrustProxies`ミドルウェアに、僅かな変更がおきました。

`$headers`プロパティは以前配列でしたが、様々な異なった値を受け付けるようになりました。たとえば、フォワーディングヘッダを全て信頼するには、`$headers`プロパティを次のように変更してください。

    use Illuminate\Http\Request;

    /**
     * プロキシを検出するために使用するヘッダ
     *
     * @var string
     */
    protected $headers = Request::HEADER_X_FORWARDED_ALL;

`$headers`に指定可能な値についての詳細は、[trusting proxies](/docs/5.6/requests#configuring-trusted-proxies)のドキュメントで確認してください。

### バリデーション

#### `ValidatesWhenResolved`インターフェイス

`ValidatesWhenResolved`インターフェイス／トレイトの`validate`メソッドは、`$request->validate()`によるコンフリクトを避けるために、`validateResolved`に変更されました。

### その他

`laravel/laravel` [GitHubリポジトリ](https://github.com/laravel/laravel)で、変更を確認することも推奨します。変更の多くが必要なくても、それらのファイルを皆さんのアプリケーションで、最新版に合わせておきたいでしょう。変更のいくつかは、このアップグレードガイドで取り扱われていますが、設定ファイルやコメントの変更は取り扱っていません。変更は、[GitHub比較ツール](https://github.com/laravel/laravel/compare/5.5...master)で簡単に確認できるので、どの変更が自分にとって重要か選んでください。
