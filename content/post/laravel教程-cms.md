---
title: Laravel教程 - CMS
date: 2024-10-15T17:00:00+08:00
---
创建应用

### Composer 加速

```
# 查看命令
composer config -g -l repo.packagist # repositories.packagist.org.url 即为全局配置的镜像地址

# 全局配置（推荐）: 所有项目都会使用该镜像地址
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

### 创建 Laravel-Shop 项目

```
composer create-project laravel/laravel laravel-shop --prefer-dist "8.3.*"
```

```
APP_NAME="Laravel Shop"
.
.
.
APP_URL=http://shop.test
.
.
.
DB_DATABASE=laravel-shop
```

> 如果环境变量的值中包含空格，需要用双引号将值包含起来

### Git 代码版本控制

```
git init
```

*.gitignore*

```
/public/js
/public/css
```

```
git add -A
git commit -m "初始化应用"
```

## 基础布局

我们先创建主要布局文件：`resources/views/layouts/app.blade.php`：

```
$ mkdir -p resources/views/layouts/
$ touch resources/views/layouts/app.blade.php
```

*resources/views/layouts/app.blade.php*

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>@yield('title', 'Laravel CMS') - Laravel CMS</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div id="app">
        @include('layouts._header')
        <div class="container">
            @yield('content')
        </div>
        @include('layouts._footer')
    </div>
    <!-- JS 脚本 -->
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 1.创建控制器

```
$ php artisan make:controller PagesController
```

*app/Http/Controllers/PagesController.php*

```
    public function root()
    {
        return view('pages.root');
    }
```

### 2.视图

```
$ mkdir -p resources/views/pages/
$ touch resources/views/pages/root.blade.php
```

*resources/views/pages/root.blade.php*

```
@extends('layouts.app')
@section('title', '首页')

@section('content')
  <h1>这里是首页</h1>
@stop
```

### 3.绑定路由

*routes/web.php*

```
use App\Http\Controllers\PagesController;
...
Route::get('/', [PagesController::class, 'root'])->name('root');
```

```
$ git add -A
$ git commit -m "基础布局"
```

## 注册与登录

Laravel 自带了用户认证功能，我们将利用此功能来快速构建我们的用户中心。

```
composer require laravel/ui
php artisan ui bootstrap --auth
//php artisan ui vue --auth
//php artisan ui react --auth
//npm install && npm run dev
//npm run dev
```

会询问我们是否要覆盖 `app.blade.php`，因为我们在前面章节中已经自定义了『主要布局文件』—— `app.blade.php`，所以此处输入 `no`

使用 `git status` 可以查看文件更改的状态

*routes/web.php*

```
Route::get('/home', 'HomeController@index')->name('home');
```

我们已经有自己的主页了，不需要再次设置主页，直接删除这行

同时删除 `app/Http/Controllers/HomeController.php` 和 `resources/views/home.blade.php` 两个文件：

```
rm -f app/Http/Controllers/HomeController.php resources/views/home.blade.php
```

由于我们删除了 `/home` 这个路由，因此需要把引用了这个路由的地方都修改掉：

编辑器全局搜索，将 `redirect('/home')` 修改为 `redirect('/')`。

访问登录页面，看看效果

## 魔改注册

### 首先，让我创建一个邮件类：

```
php artisan make:mail UserRegistered
```

```
<?php

namespace App\Mail;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class UserRegistered extends Mailable
{
    use Queueable;
    use SerializesModels;

    public $user;
    public $password;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct(User $user, string $password)
    {
        $this->user = $user;
        $this->password = $password;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->subject('欢迎注册 ' . config('app.name'))
                    ->view('emails.user-registered');
    }
}
```

### 接下来创建邮件模板：

```
mkdir -p resources/views/emails
```

### 现在让我们创建并编辑邮件模板：

resources/views/emails/user-registered.blade.php

```
<!DOCTYPE html>
<html>

<head>
    <title>注册成功</title>
</head>

<body>
    <h1>{{ config('app.name') }} - 注册成功</h1>
    <p>您好，</p>
    <p>您已成功注册 {{ config('app.name') }}。以下是您的登录信息：</p>
    <p><strong>用户名：</strong> {{ $user->name }}</p>
    <p><strong>密码：</strong> {{ $password }}</p>
    <p>请使用上述信息登录系统。首次登录后，建议修改您的密码。</p>
    <p>谢谢！</p>
    <p>{{ config('app.name') }} 团队</p>
</body>

</html>
```

### 现在修改邮件类：

```
app/Mail/UserRegistered.php
```

```
<?php

namespace App\Mail;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class UserRegistered extends Mailable
{
    use Queueable;
    use SerializesModels;

    public $user;
    public $password;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct(User $user, string $password)
    {
        $this->user = $user;
        $this->password = $password;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->subject('欢迎注册 ' . config('app.name'))
                    ->view('emails.user-registered');
    }
}
```

现在修改注册控制器：

```
app/Http/Controllers/Auth/RegisterController.php

```

```
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Providers\RouteServiceProvider;
use App\Models\User;
use App\Mail\UserRegistered;
use Illuminate\Foundation\Auth\RegistersUsers;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Str;
use Illuminate\Http\Request;
use Illuminate\Auth\Events\Registered;

class RegisterController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Register Controller
    |--------------------------------------------------------------------------
    |
    | This controller handles the registration of new users as well as their
    | validation and creation. By default this controller uses a trait to
    | provide this functionality without requiring any additional code.
    |
    */

    use RegistersUsers;

    /**
     * Where to redirect users after registration.
     *
     * @var string
     */
    protected $redirectTo = RouteServiceProvider::HOME;

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest');
    }

    /**
     * Get a validator for an incoming registration request.
     *
     * @param  array  $data
     * @return \Illuminate\Contracts\Validation\Validator
     */
    protected function validator(array $data)
    {
        return Validator::make($data, [
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
        ]);
    }

    /**
     * Create a new user instance after a valid registration.
     *
     * @param  array  $data
     * @return \App\Models\User
     */
    protected function create(array $data)
    {
        // 生成随机用户名（基于邮箱）
        $name = explode('@', $data['email'])[0] . '_' . Str::random(5);

        // 生成随机密码
        $password = Str::random(10);

        $user = User::create([
            'name' => $name,
            'email' => $data['email'],
            'password' => Hash::make($password),
        ]);

        // 发送包含用户名和密码的邮件
        Mail::to($user->email)->send(new UserRegistered($user, $password));

        return $user;
    }

    /**
     * 重写注册方法
     */
    public function register(Request $request)
    {
        $this->validator($request->all())->validate();

        event(new Registered($user = $this->create($request->all())));

        return redirect()->route('login')
            ->with('status', '注册成功！请查收您的邮箱获取登录信息。');
    }
}
```

最后，修改注册视图：

resources/views/auth/register.blade.php

```
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">{{ __('Register') }}</div>

                <div class="card-body">
                    <form method="POST" action="{{ route('register') }}">
                        @csrf

                        <div class="row mb-3">
                            <label for="email" class="col-md-4 col-form-label text-md-end">{{ __('Email Address') }}</label>

                            <div class="col-md-6">
                                <input id="email" type="email" class="form-control @error('email') is-invalid @enderror" name="email" value="{{ old('email') }}" required autocomplete="email">

                                @error('email')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{{ $message }}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="row mb-3">
                            <div class="col-md-6 offset-md-4">
                                <p class="text-muted">
                                    注册后，我们将发送一封包含您的登录信息的邮件到您的邮箱。
                                </p>
                            </div>
                        </div>

                        <div class="row mb-0">
                            <div class="col-md-6 offset-md-4">
                                <button type="submit" class="btn btn-primary">
                                    {{ __('Register') }}
                                </button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

最后，让我们修改登录页面以显示注册成功的消息：

resources/views/auth/login.blade.php，添加代码

```
@if (session('status'))
    <div class="alert alert-success mb-3">
        {{ session('status') }}
    </div>
@endif
```

现在我们需要确保邮件配置正确。在本地开发环境中，你可以使用Mailhog来测试邮件发送。

.env

```
MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=noreply@example.com
MAIL_FROM_NAME="${APP_NAME}"
```

### 顶部导航

接下来我们需要把顶部导航的登录和注册按钮指向真实的地址：

*resources/views/layouts/_header.blade.php*

```
<nav class="navbar navbar-light bg-light static-top">
    <div class="container">
        <a class="navbar-brand" href="{{ url('/') }}">Laravel CMS</a>
        @guest
        <a class="btn btn-primary" href="{{ route('login') }}">登录</a>
        @else
        <div class="dropdown">
            <a href="#" class="dropdown-toggle" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
                <span class="user-avatar pull-left" style="margin-right:8px; margin-top:-5px;">
                    <img src="uploads/images/201709/20/1/PtDKbASVcz.png?imageView2/1/w/60/h/60" class="img-responsive img-circle" width="30px" height="30px">
                </span>
                {{ Auth::user()->name }} <span class="caret"></span>
            </a>
            <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1">
                <li>
                    <a href="{{ route('logout') }}"
                        onclick="event.preventDefault();
                                    document.getElementById('logout-form').submit();">
                        退出登录
                    </a>
                    <form id="logout-form" action="{{ route('logout') }}" method="POST" style="display: none;">
                        {{ csrf_field() }}
                    </form>
                </li>
            </ul>
        </div>
        @endguest
    </div>
</nav>
```

### 测试注册

先执行数据库迁移创建对应的数据库表结构：

```
php artisan migrate
```

```
git add -A
git commit -m "注册与登录"
```

## 验证邮箱（上）

```
php artisan make:migration users_add_email_verified --table=users
```

`--table=users` 参数是告诉 Laravel 我们这个迁移文件准备对 `users` 表进行变更，Laravel 就会帮我们生成好相关的代码。

编辑迁移文件 *database/migrations/< your_date >_users_add_email_verified.php*

```
 public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->boolean('email_verified')->default(false)->after('remember_token');
        });
    }

    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('email_verified');
        });
    }
```

然后执行迁移：

```
php artisan migrate
```

为了检查回滚代码有没有问题，我们执行一次回滚

```
php artisan migrate:rollback
php artisan migrate
```

*app/Models/User.php*

```
    protected $fillable = [
        'name', 'email', 'password', 'email_verified',
   ];
       protected $casts = [
        'email_verified' => 'boolean',
    ];
```

### 创建中间件

```
php artisan make:middleware CheckIfEmailVerified
```

*app/Http/Middleware/CheckIfEmailVerified.php*

```
    public function handle($request, Closure $next)
    {
        if (!$request->user()->email_verified) {
            // 如果是 AJAX 请求，则通过 JSON 返回
            if ($request->expectsJson()) {
                return response()->json(['msg' => '请先验证邮箱'], 400);
            }
            return redirect(route('email_verify_notice'));
        }
        return $next($request);
    }
```

### 控制器、路由与模板文件

*app/Http/Controllers/PagesController.php*

```
    public function emailVerifyNotice(Request $request)
    {
        return view('pages.email_verify_notice');
    }
```

新建 `resources/views/pages/email_verify_notice.blade.php` 文件：

```
touch resources/views/pages/email_verify_notice.blade.php
```

*resources/views/pages/email_verify_notice.blade.php*

```
@extends('layouts.app')
@section('title', '提示')

@section('content')
<div class="panel panel-default">
    <div class="panel-heading">提示</div>
    <div class="panel-body text-center">
        <h1>请先验证邮箱</h1>
        <a class="btn btn-primary" href="{{ route('root') }}">返回首页</a>
    </div>
</div>
@endsection
```

*routes/web.php*

```
Route::group(['middleware' => 'auth'], function() {
    Route::get('/email_verify_notice', [PagesController::class, 'emailVerifyNotice'])->name('email_verify_notice');
});
```

需要把这个路由放在 `auth` 这个中间件的路由组里面，因为只有已经登录的用户才能看到这个提示界面。

*app/Http/Kernel.php*

```
    .
    .
    .
    protected $routeMiddleware = [
        .
        .
        .
        'email_verified' => \App\Http\Middleware\CheckIfEmailVerified::class,
    ];
```

### 中间件测试

*routes/web.php*

```
.
.
.
Route::group(['middleware' => 'auth'], function() {
    Route::get('/email_verify_notice', 'PagesController@emailVerifyNotice')->name('email_verify_notice');
    // 开始
    Route::group(['middleware' => 'email_verified'], function() {
        Route::get('/test', function() {
            return 'Your email is verified';
        });
    });
    // 结束
});
```

然后访问 `http://domain/test`，可以看到浏览器自动跳转到了 `http://domain/email_verify_notice`

```
git add -A
git commit -m "邮箱验证中间件"
```

## laravel-admin后台

提示Your requirements could not be resolved to an installable set of packages.修改comoposer.json

```
    "require": {
        "encore/laravel-admin": "^1.8"
        ...
    }
```

```
composer update
composer require encore/laravel-admin
php artisan vendor:publish --provider="Encore\Admin\AdminServiceProvider"
php artisan admin:install
```

修改后台为中文语言，app.php文件locale = 'zh_CN'，然后修改resources\lang\zh-CN为zh_CN

## 在 Laravel 项目中实现多语言支持

在 Laravel 项目中实现多语言支持，可以通过以下步骤来完成：

1. **创建语言文件**：在 `resources/lang` 目录下，为每种语言创建一个子目录，例如 `en` 和 `zh`，并在这些目录中创建 `messages.php` 文件来存储翻译字符串。例如：

```php
// resources/lang/en/messages.php
return [
    'welcome' => 'Welcome to our application!',
];

// resources/lang/zh/messages.php
return [
    'welcome' => '欢迎使用我们的应用程序！',
];
```

  对于根据领土差异的语言，应根据 ISO 15897 来命名语言目录，例如使用 "en_GB" 用于英式英语而不是 "en-gb" 。

2. **配置应用程序**：在 `config/app.php` 文件中，设置 `locale` 为默认语言，`fallback_locale` 为备用语言（如果没有找到默认语言的翻译）。

```php
return [
    'locale' => 'en',
    'fallback_locale' => 'en',
];
```

3. **动态设置语言**：可以使用 `App::setLocale($lang)` 方法动态设置当前语言，通常在中间件或控制器的构造函数中根据用户的选择来设置语言。

```php
use Illuminate\Support\Facades\App;
App::setLocale($user->preferred_language);
```

4. **使用翻译字符串**：在路由和控制器中，使用 `__()` 函数来输出翻译后的字符串。

```php
echo __('messages.welcome');
```

5. **视图**：在视图中，使用 `{{ __('messages.welcome') }}` 语法来显示翻译文本。
6. **语言切换器**：在视图中，创建一个语言切换器，允许用户选择语言。这通常是一个下拉菜单或链接列表，当用户选择一种语言时，它会发送一个请求到服务器以更新用户的语言偏好。
7. **存储用户偏好**：你可以选择将用户的语言偏好存储在数据库、会话或 Cookie 中，以便在用户的后续访问中记住其选择。

此外，Laravel 还支持使用 JSON 文件来定义翻译字符串，这适用于具有大量翻译字符串的应用程序 。你可以通过 Composer 引入 `spatie/laravel-lang` 包来获取更多的语言支持 。这个包包含了 Laravel 核心及许多流行扩展包的所有本地化文件，支持超过 100 种语言，并且易于集成和使用。

## 在Git中，如果你想要将代码回滚到指定的版本

1. **使用`git checkout`**:
   如果你只是想查看某个特定版本的代码，而不打算修改当前分支，可以使用`git checkout`命令。例如，如果你知道想要回滚到的版本号是`abc123`，你可以执行：

  bash

```bash
git checkout abc123
```

  这会创建一个新分支并切换到该分支，其中包含指定版本的代码。

2. **使用`git reset`**:
   如果你想要将当前分支的HEAD指针移动到指定的版本，并更新工作目录和索引，可以使用`git reset`。这会将代码回滚到指定版本，但不会创建新分支

**硬回滚（hard）**: 移动HEAD指针，更新索引和工作目录，丢弃所有未提交的更改。

 ```bash
  git reset --hard abc123
  git clean -df
  ```

请注意，使用`--hard`选项会丢失所有未提交的更改，因此在使用之前请确保这是你想要的操作。

## git

在git项目里使用了git revert命令，结果git log无法显示全部版本历史了。试了好多办法也行。索性删除了项目，又从github上重新下载了项目，但是在本地git log还是不显示全部log，使用命令git reset --hard 1cb21dea3f到某个指定版本（不要revert的那个版本），然后git checkout 9b5625ab5到某个版本就好了。如果访问网站显示错误，记得配置一下laravel项目，比如composer intall 和composer update

## laravel 8安装多语言包

```
composer require overtrue/laravel-lang
php artisan lang:publish zh-CN #没有en
php artisan lang:publish ja
```

使用 trans('demo.user_not_exists');或  __('demo.user_not_exists');

## SiteMap
```
composer require spatie/laravel-sitemap
php artisan make:command GenerateSitemap
```

app/Console/Commands/GenerateSitemap.php

```
<?php

namespace App\Console\Commands;

use App\Models\Post;
use App\Models\Tool;
use App\Models\CustomPages;
use Illuminate\Console\Command;
use Spatie\Sitemap\Sitemap;
use Spatie\Sitemap\Tags\Url;

class GenerateSitemap extends Command
{
    protected $signature = 'sitemap:generate';
    protected $description = '生成网站地图';

    public function handle()
    {
        $this->info('开始生成网站地图...');
        $sitemap = Sitemap::create();

        // 添加首页
        $sitemap->add(
            Url::create('/')
                ->setChangeFrequency(Url::CHANGE_FREQUENCY_DAILY)
                ->setPriority(1.0)
        );

        // 从数据库中获取文章并添加到 Sitemap 中
        Post::all()->each(function (Post $post) use ($sitemap) {
            $sitemap->add(
                Url::create("/{$post->slug}")
                    ->setLastModificationDate($post->updated_at)
                    ->setChangeFrequency(Url::CHANGE_FREQUENCY_WEEKLY)
                    ->setPriority(0.8)
            );
        });

        // 添加工具页面
        Tool::all()->each(function (Tool $tool) use ($sitemap) {
            $sitemap->add(
                Url::create("/tools/{$tool->slug}")
                    ->setLastModificationDate($tool->updated_at)
                    ->setChangeFrequency(Url::CHANGE_FREQUENCY_WEEKLY)
                    ->setPriority(0.7)
            );
        });

        // 添加静态页面
        CustomPages::all()->each(function (CustomPages $page) use ($sitemap) {
            $sitemap->add(
                Url::create("/{$page->slug}")
                    ->setLastModificationDate($page->updated_at)
                    ->setChangeFrequency(Url::CHANGE_FREQUENCY_MONTHLY)
                    ->setPriority(0.6)
            );
        });

        $sitemap->writeToFile(public_path('sitemap.xml'));
        $this->info('网站地图生成成功！');
    }
}
```
app/Console/Kernel.php 

```
    protected $commands = [
        \App\Console\Commands\GenerateSitemap::class,
    ];
```
### 管理界面：
控制器：app/Admin/Controllers/SitemapController.php

```
<?php

namespace App\Admin\Controllers;

use App\Http\Controllers\Controller;
use Encore\Admin\Layout\Content;
use Encore\Admin\Widgets\Box;
use Encore\Admin\Widgets\Form;
use Encore\Admin\Widgets\Alert;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Artisan;

class SitemapController extends Controller
{
    /**
     * 网站地图管理页面
     *
     * @param Content $content
     * @return Content
     */
    public function index(Content $content)
    {
        return $content
            ->title(__('admin.sitemap.title'))
            ->description(__('admin.sitemap.description'))
            ->body($this->form())
            ->body($this->sitemapInfo());
    }

    /**
     * 创建表单
     *
     * @return Box
     */
    protected function form()
    {
        $form = new Form();
        $form->action(admin_url('sitemap/generate'));

        $form->display('说明', __('admin.sitemap.instruction'));
        $form->hidden('_token')->default(csrf_token());
        $form->submit(__('admin.sitemap.generate'));

        return (new Box(__('admin.sitemap.generate'), $form))->style('info');
    }

    /**
     * 显示网站地图信息
     *
     * @return Box
     */
    protected function sitemapInfo()
    {
        $sitemapPath = public_path('sitemap.xml');
        $content = '';

        if (file_exists($sitemapPath)) {
            $lastModified = date('Y-m-d H:i:s', filemtime($sitemapPath));
            $fileSize = round(filesize($sitemapPath) / 1024, 2);
            $sitemapUrl = url('sitemap.xml');

            $content = <<<HTML
            <div class="table-responsive">
                <table class="table table-striped">
                    <tbody>
                        <tr>
                            <th>{$this->trans('admin.sitemap.url')}</th>
                            <td><a href="{$sitemapUrl}" target="_blank">{$sitemapUrl}</a></td>
                        </tr>
                        <tr>
                            <th>{$this->trans('admin.sitemap.last_modified')}</th>
                            <td>{$lastModified}</td>
                        </tr>
                        <tr>
                            <th>{$this->trans('admin.sitemap.file_size')}</th>
                            <td>{$fileSize} KB</td>
                        </tr>
                    </tbody>
                </table>
            </div>
HTML;
        } else {
            $content = (new Alert(__('admin.sitemap.not_generated')))->style('warning');
        }

        return (new Box(__('admin.sitemap.info'), $content))->style('default');
    }

    /**
     * 生成网站地图
     *
     * @param Request $request
     * @return \Illuminate\Http\RedirectResponse
     */
    public function generate(Request $request)
    {
        try {
            Artisan::call('sitemap:generate');
            admin_toastr(__('admin.sitemap.generate_success'), 'success');
        } catch (\Exception $e) {
            admin_toastr(__('admin.sitemap.generate_failed') . $e->getMessage(), 'error');
        }

        return redirect()->back();
    }

    /**
     * 翻译辅助函数
     * 
     * @param string $key
     * @return string
     */
    protected function trans($key)
    {
        return __($key);
    }
}
```
文件存储：
网站地图文件保存在 public/sitemap.xml

已添加到 .gitignore 防止被提交到版本控制系统

定时任务：
 - 在 app/Console/Kernel.php 中注册了命令并设置每日凌晨 1 点自动执行
 - 输出日志保存到 storage/logs/sitemap.log
```
    protected function schedule(Schedule $schedule)
    {
        // $schedule->command('inspire')->hourly();
        $schedule->command('sitemap:generate')
                 ->daily()
                 ->at('01:00')
                 ->appendOutputTo(storage_path('logs/sitemap.log'));
    }
```

app\Admin\routes.php

```
    // 网站地图管理
    $router->get('sitemap', 'SitemapController@index')->name('sitemap.index');
    $router->post('sitemap/generate', 'SitemapController@generate')->name('sitemap.generate');
```

前端访问路由：
 - 在 routes/web.php 中定义了访问路由：/sitemap.xml
 - 如果文件不存在会自动调用命令生成

routes/web.php

```
// 网站地图路由
Route::get('/sitemap.xml', function() {
    // 如果文件不存在，则生成
    if (!file_exists(public_path('sitemap.xml'))) {
        \Artisan::call('sitemap:generate');
    }
    return response()->file(public_path('sitemap.xml'));
});
```
