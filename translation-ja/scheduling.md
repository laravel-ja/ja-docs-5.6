# タスクスケジュール

- [イントロダクション](#introduction)
- [スケジュール定義](#defining-schedules)
    - [Artisanコマンドのスケジュール](#scheduling-artisan-commands)
    - [キュー投入するジョブのスケジュール](#scheduling-queued-jobs)
    - [シェルコマンドのスケジュール](#scheduling-shell-commands)
    - [繰り返しのスケジュールオプション](#schedule-frequency-options)
    - [タイムゾーン](#timezones)
    - [タスク多重起動の停止](#preventing-task-overlaps)
    - [単一サーバ上でのタスク実行](#running-tasks-on-one-server)
    - [メンテナンスモード](#maintenance-mode)
- [タスク出力](#task-output)
- [タスクフック](#task-hooks)

<a name="introduction"></a>
## イントロダクション

今までは実行をスケジュールする必要のあるタスクごとにサーバで、Cronエントリを制作していたかと思います。しかし、これはすぐに面倒になります。

Laravelのコマンドスケジューラは、Laravel自身の中でコマンドの実行スケジュールをスラスラと記述的に定義できるようにしてくれます。スケジューラを使用すると、サーバにはCronエントリがたった一つだけで済みます。タスクスケジュールは、`app/Console/Kernel.php`ファイルの`schedule`メソッドで定義します。使いはじめる手助けになるように、サンプルがこのメソッド中で定義してあります。

### スケジューラを使いはじめる

スケジューラを使用するには、サーバに以下のCronエントリを追加するだけです。サーバにどうやってCronエントリを追加するかわからない場合は、Cronエントリを管理できる[Laravel Forge](https://forge.laravel.com)のようなサービスを使用することを考慮してください。

    * * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1

このCronはLaravelコマンドスケジューラを毎分呼び出します。`schedule:run`コマンドが実行されると、Laravelはスケジュールされているジョブを評価し、実行する必要のあるジョブを起動します。

<a name="defining-schedules"></a>
## スケジュール定義

スケジュールタスクは全部`App\Console\Kernel`クラスの`schedule`メソッドの中に定義します。手始めに、スケジュールタスクの例を見てください。この例は毎日深夜１２時に「クロージャ」をスケジュールしています。「クロージャ」の中でテーブルをクリアするデータベースクエリを実行しています。

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * アプリケーションで提供するArtisanコマンド
         *
         * @var array
         */
        protected $commands = [
            //
        ];

        /**
         * アプリケーションのコマンド実行スケジュール定義
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

<a name="scheduling-artisan-commands"></a>
### Artisanコマンドのスケジュール

「クロージャ」の呼び出しをスケジュールするほかに、[Artisanコマンド](/docs/{{version}}/artisan)やオペレーティングシステムコマンドをスケジュールできます。たとえば、コマンド名かそのクラスのどちらかを用いて、Artisanコマンドをスケジュールする`command`メソッドを使ってみましょう。

    $schedule->command('emails:send --force')->daily();

    $schedule->command(EmailsCommand::class, ['--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### キュー投入するジョブのスケジュール

[キュー投入するジョブ](/docs/{{version}}/queues)をスケジュールするには、`job`メソッドを使います。このメソッドを使うと、ジョブをキューに入れるためのクロージャを自前で作成する`call`メソッドを使わずとも、ジョブをスケジュール実行することができます。

    $schedule->job(new Heartbeat)->everyFiveMinutes();

    // ジョブを"heartbeats"キューへ投入する
    $schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### シェルコマンドのスケジュール

オペレーティングシステムでコマンドを実行するためには`exec`メソッドを使います。

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### 繰り返しのスケジュールオプション

もちろん、タスクは様々なスケジュールを割り付けることができます。

メソッド  | 説明
------------- | -------------
`->cron('* * * * *');`  |  Cron指定に基づきタスク実行
`->everyMinute();`  |  毎分タスク実行
`->everyFiveMinutes();`  |  ５分毎にタスク実行
`->everyTenMinutes();`  |  １０分毎にタスク実行
`->everyFifteenMinutes();`  |  １５分毎にタスク実行
`->everyThirtyMinutes();`  |  ３０分毎にタスク実行
`->hourly();`  |  毎時タスク実行
`->hourlyAt(17);`  |  一時間ごと、毎時１７分にタスク実行
`->daily();`  |  毎日深夜１２時に実行
`->dailyAt('13:00');`  |  毎日13:00に実行
`->twiceDaily(1, 13);`  |  毎日1:00と13:00時に実行
`->weekly();`  |  毎週実行
`->weeklyOn(1, '8:00');`  |  毎週火曜日の8:00時に実行
`->monthly();`  |  毎月実行
`->monthlyOn(4, '15:00');`  |  毎月4日の15:00に実行
`->quarterly();` |  四半期ごとに実行
`->yearly();`  |  毎年実行
`->timezone('America/New_York');` | タイムゾーン設定

これらのメソッドは週の特定の曜日だけに実行させるために、追加の制約と組み合わせ細かく調整できます。

    // 週に一回、月曜の13:00に実行
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // ウィークエンドの8時から17時まで一時間ごとに実行
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

以下が追加のスケジュール制約のリストです。

メソッド  | 説明
------------- | -------------
`->weekdays();`  |  ウイークデーのみに限定
`->sundays();`  |  日曜だけに限定
`->mondays();`  |  月曜だけに限定
`->tuesdays();`  |  火曜だけに限定
`->wednesdays();`  |  水曜だけに限定
`->thursdays();`  |  木曜だけに限定
`->fridays();`  |  金曜だけに限定
`->saturdays();`  |  土曜だけに限定
`->between($start, $end);`  |  開始と終了時間間にタスク実行を制限
`->when(Closure);`  |  クロージャの戻り値が`true`の時のみに限定

#### 時間制限

`between`メソッドは一日の時間に基づき、実行時間を制限するために使用します。

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

同じように、`unlessBetween`メソッドは、その時間にタスクの実行を除外するために使用します。

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### 真値テスト制約

`when`メソッドは指定した真値テストの結果に基づき制限を実行します。言い換えれば指定した「クロージャ」が`true`を返し、他の制約が実行を停止しない限りタスクを実行します。

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

`skip`メソッドは`when`をひっくり返したものです。`skip`メソッドへ渡したクロージャが`true`を返した時、スケジュールタスクは実行されます。

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

`when`メソッドをいくつかチェーンした場合は、全部の`when`条件が`true`を返すときのみスケジュールされたコマンドが実行されます。

<a name="timezones"></a>
### タイムゾーン

`timezone`メソッドを使い、タスクのスケジュールをどこのタイムゾーンとみなすか指定できます。

    $schedule->command('report:generate')
             ->timezone('America/New_York')
             ->at('02:00')

> {note} タイムゾーンの中には夏時間を取り入れているものがあることを忘れないでください。夏時間の切り替えにより、スケジュールされたタスクが2回実行されたり、全くされないことがあります。そのため、可能であればタイムゾーンによるスケジュールは使用しないことをおすすめします。

<a name="preventing-task-overlaps"></a>
### タスク多重起動の防止

デフォルトでは以前の同じジョブが起動中であっても、スケジュールされたジョブは実行されます。これを防ぐには、`withoutOverlapping`メソッドを使用してください。

    $schedule->command('emails:send')->withoutOverlapping();

この例の場合、`emails:send` [Artisanコマンド](/docs/{{version}}/artisan)は実行中でない限り毎分実行されます。`withoutOverlapping`メソッドは指定したタスクの実行時間の変動が非常に大きく、予想がつかない場合に特に便利です。

必要であれば、「重起動の防止(without overlapping)」ロックを期限切れにするまでに、何分間経過させるかを指定できます。時間切れまでデフォルトは、２４時間です。

    $schedule->command('emails:send')->withoutOverlapping(10);

<a name="running-tasks-on-one-server"></a>
### 単一サーバ上でのタスク実行

> {note} この機能を使用するには、アプリケーションのデフォルトキャッシュドライバとして、`memcached`か`redis`キャッシュドライバを使用する必要があります。更に、全サーバが同じ単一のキャッシュサーバと通信している必要があります。

アプリケーションが複数のサーバで実行される場合、スケジュール済みのジョブを単一サーバ上のみで実行するよう制限できるようになりました。たとえば、毎週の金曜の夜に、新しいレポートを生成するタスクをスケジュールしていると仮定しましょう。タスクスケジューラが３つのワーカーサーバ上で実行されているなら、スケジュールされているタスクは３つ全部のサーバで実行され、３回レポートが生成されます。これではいけません。

タスクをサーバひとつだけで実行するように指示するには、スケジュールタスクを定義するときに`onOneServer`メソッドを使用します。このタスクを最初に取得したサーバが、同じタスクを同じCronサイクルで他のサーバで実行しないように、ジョブにアトミックなロックを確保します。

    $schedule->command('report:generate')
                    ->fridays()
                    ->at('17:00')
                    ->onOneServer();

<a name="maintenance-mode"></a>
### メンテナンスモード

Laravelのスケジュールタスクは、Laravelが[メンテナンスモード](/docs/{{version}}/configuration#maintenance-mode)の間は実行されません。メンテナンスが完了していないサーバ上で、タスクが実行されてほしくないからです。しかし、メンテナンスモードでも実行するように強制したい場合は、`evenInMaintenanceMode`メソッドを使用します。

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="task-output"></a>
## タスク出力

Laravelスケジューラはスケジュールしたタスクが生成する出力を取り扱う便利なメソッドをたくさん用意しています。最初に`sendOutputTo`メソッドを使い、後ほど内容を調べられるようにファイルへ出力してみましょう。

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

出力を指定したファイルに追加したい場合は、`appendOutputTo`メソッドを使います。

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

`emailOutputTo`メソッドを使えば、選択したメールアドレスへ出力をメールで送ることができます。タスク出力をメールで送信する前に、[メール送信サービス](/docs/{{version}}/mail)の設定を済ませておく必要があります。

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> {note} The `emailOutputTo`, `sendOutputTo` and `appendOutputTo` methods are exclusive to the `command` and `exec` methods.

<a name="task-hooks"></a>
## タスクフック

`before`と`after`メソッドを使えば、スケジュールされたタスクの実行前後に指定したコードを実行することができます。

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // タスク開始時…
             })
             ->after(function () {
                 // タスク終了時…
             });

#### URLへのPing

`pingBefore`と`thenPing`メソッドを使用し、タスク実行前、タスク完了後に指定したURLへ自動的にPingすることができます。これは[Laravel Envoyer](https://envoyer.io)のような外部サービスへスケジュールされたタスクが始まる、または完了したことを知らせるのに便利です。

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

`pingBefore($url)`か`thenPing($url)`のどちらを使用するにも、Guzzle HTTPライブラリーが必要です。GuzzleはComposerパッケージマネージャを利用し追加できます。

    composer require guzzlehttp/guzzle
