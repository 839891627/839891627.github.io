---
title: Laravel 5.3 下添加自定义 Facade 的步骤
date: 2017-06-27 15:03:05
categories: 编程
tags: 
    - laravel
    - php
---

> 原文链接 [Laravel 5.3 下添加自定义 Facade 的步骤](https://laravel-china.org/topics/3265/laravel-53-add-custom-facade-steps)

##### 1、目录结构

我在 /app 下建立了一个自定义目录，结构如下：

```
/app
|- Custom
    |- Classes 这里放置自定义 Facade 所需的类
    |- Facades 这里放置自定义 Facade
```

<!-- more -->

##### 2、创建自定义 Facade 所需的类

文件位置：`app/Custom/Classes/MLS.php`
```php
<?php
namespace App\Custom\Classes;

class MLS
{
    public function test()
    {
        return 'MLS Test';
    }
}
```
上面这个 class 我只想实现一个功能，就是在调用 test() 方法时返回一个字符串。

##### 3、创建自定义 Facade

文件位置：`app/Custom/Facades/MLS.php`

```php
<?php
namespace App\Custom\Facades;

use Illuminate\Support\Facades\Facade; 

class MLS extends Facade
{
    protected static function getFacadeAccessor() {
        return 'mls';
    }
}
```
这个 class 没有什么可说的，就算不理解也请先当作一个规矩来照做。唯一会让初学者迷惑的是这个 return 'mls'; 有人可能会问“mls”是什么，为什么只是一个字符串。这个问题如果想透彻的解释，那就必须从理解 Laravel 中的“容器”说开来，但可惜我对这个概念也是一知半解，不能误人子弟。如果大家对此感兴趣，可以去找一些讲解“容器”和 Facade 原理的文章，这里先不提，继续往下看。

##### 4、将自定义的 Facade 和自定义的 Class 绑定起来。

该步骤又分为2部分：

> 部分1：创建与自定义 Facade 对应的 ServerProvider。

在命令行下运行：

`$ php artisan make:provider MLSServiceProvider`
成功后会创建一个新文件：app/Providers/MLSServiceProvider.php，然后我们在 register() 部分添点东西，如下：
```php
<?php
namespace App\Providers;

use App\Custom\Classes\MLS; //注意这里，引入了我们的自定义 Class
use Illuminate\Support\ServiceProvider;

class MLSServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }

    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
        //注意这里的 bind 写法，这是 Laravel 5.3 的正确写法；网上搜索到的很多资料都是不知何年何月的老版本的写法，如果你照抄就只会很郁闷。
        $this->app->bind('mls', function() {
            return new MLS();
        });
    }
}
```
看到 bind() 部分了吗？对应写的也是“mls”，和自定义 Facade 中返回的一样。其实前后联系起来看就是，这个“mls”只是 Laravel 容器中的一个“代号”，前面的 Facade 返回“mls”是告诉”中介“，也就是 我们自定义的 ServiceProvider：“如果有人虚要调用我时，请使用我的代号是 mls”，然后 ServiceProvider 在 register() 这里则告诉框架：“我介绍一个叫 MLS 的类入伙了，它的代号是 mls！“。这样，框架在你调用 Facade 时就会在自己的”容器“里找到代号为”mls“的自定义类给你，你才能使用这个自定义类里的方法。

> 部分2：将新添加的 ServiceProvider 写进框架的配置中

编辑文件 `config/app.php`，里面有2个数组：providers 和 aliases。首先，在 providers 中添加：
```php
'providers' => [

        略
        App\Providers\RouteServiceProvider::class,

        App\Providers\MLSServiceProvider::class, //我们新建的 Provider

    ],
```
然后，在 aliases 中添加：
```php
'aliases' => [

        'App' => Illuminate\Support\Facades\App::class,
        略

        'MLS' => App\Custom\Facades\MLS::class, //这里写上我们的自定义 Facade
    ],
```
注意，aliases，也就是”别名“部分的”MLS“可以是任意我们喜欢的名字，它只是用来指代我们自定义的 Facade。

##### 5、结束了，可以使用我们的自定义 Facade 了。

好，现在假设我们已经设置好了路由规则，并准备好了一个 controller ，比如：`app/Http/Controllers/MLSController.php`

```php
<?php
namespace App\Http\Controllers;

use MLS; //这里就是在 aliases 添加的别名了，如果别名换了，也请记得更新这里。
use Illuminate\Http\Request;

class MLSController extends Controller
{
    public function foo()
    {
        echo MLS::test(); //现在，我们就可以像使用静态类一样使用我们的自定义 Facade 了。
    }
}
```
到此，一个简单的自定义 Facade 就添加完成了。

其实，关于 Facade 的争论很大，反对它的人认为它不过是一个针对 Class 的语法糖和快捷方式，甚至认为它最大的意义不过是让程序员不用敲一堆代码才能引用一个 Class，也就是如下的区别：
```php
echo App\Custom\Classes\MLS::test(); //方式1

echo MLS::test(); //方式2
```

