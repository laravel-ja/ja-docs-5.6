# ハッシュ

- [イントロダクション](#introduction)
- [Configuration](#configuration)
- [基本的な使用法](#basic-usage)

<a name="introduction"></a>
## イントロダクション

The Laravel `Hash` [facade](/docs/{{version}}/facades) provides secure Bcrypt and Argon2 hashing for storing user passwords. If you are using the built-in `LoginController` and `RegisterController` classes that are included with your Laravel application, they will use Bcrypt for registration and authentication by default.

> {tip} Bcryptは「ストレッチ回数」が調整できるのでパスワードのハッシュには良い選択肢です。つまりハードウェアのパワーを上げればハッシュの生成時間を早くすることができます。

<a name="configuration"></a>
## Configuration

The default hashing driver for your application is configured in the `config/hashing.php` configuration file. There are currently two supported drivers: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) and [Argon2](https://en.wikipedia.org/wiki/Argon2).

> {note} The Argon2 driver requires PHP 7.2.0 or greater.

<a name="basic-usage"></a>
## 基本的な使用法

`Hash`ファサードの`make`メソッドを呼び出し、パスワードをハッシュできます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * ユーザーパスワードを更新
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // 新しいパスワードの長さのバリデーション…

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### Adjusting The Bcrypt Work Factor

If you are using the Bcrypt algorithm, the `make` method allows you to manage the work factor of the algorithm using the `rounds` option; however, the default is acceptable for most applications:

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

#### Adjusting The Argon2 Work Factor

If you are using the Argon2 algorithm, the `make` method allows you to manage the work factor of the algorithm using the `memory`, `time`, and `threads` options; however, the defaults are acceptable for most applications:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} For more information on these options, check out the [official PHP documentation](http://php.net/manual/en/function.password-hash.php).

#### パスワードとハッシュ値の比較

`check`メソッドにより指定した平文文字列と指定されたハッシュ値を比較確認できます。しかし[Laravelに含まれている](/docs/{{version}}/authentication)`LoginController`を使っている場合は、これを直接使用することはないでしょう。このコントローラがこのメソッドを自動的に呼び出します。

    if (Hash::check('plain-text', $hashedPassword)) {
        // パスワード一致
    }

#### パスワードの再ハッシュが必要か確認

パスワードがハシュされてからハッシャーのストレッチ回数が変更されているかを調べるには、`needsRehash`メソッドを使います。

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
