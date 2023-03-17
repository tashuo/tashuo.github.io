---
title: Laravel跨域处理
date: 2020-06-05 16:17:50
categories:
- php
- laravel
tags:
- 跨域
- CORS
---

#### 背景
在前后端分离的应用中，需要使用CORS完成跨域访问。在CORS中发送 `非简单请求`时，前端会发一个请求方式为OPTIONS的预请求，前端只有收到服务器对这个OPTIONS请求的正确响应，才会发送正常的请求，否则将抛出跨域相关的错误。

#### 跨域
##### 可实现跨域的方式
* JSONP
* CORS
* Flash
* 服务器中转

比较常用的是 `JSONP`和 `CORS`，而后者相对前者来说有更方便实用：
1. `JSONP`只能实现`GET`请求，而`CORS`支持所有类型的HTTP请求。
2. 使用`CORS`，开发者可以使用普通的`XMLHttpRequest`发起请求和获得数据，比起`JSONP`有更好的错误处理。

此文暂不介绍jsonp

##### CORS
> `CORS`是一种网络浏览器的技术规范，它为Web服务器定义了一种方式，允许网页从不同的域访问其资源。而这种访问是被同源策略所禁止的。`CORS`系统定义了一种浏览器和服务器交互的方式来确定是否允许跨域请求。 
> 使用 `CORS`的方式非常简单，但是需要同时对前端和服务器端做相应处理。

**客户端使用XmlHttpRequest发起Ajax请求，当前绝大部分浏览器已经支持CORS方式，且主流浏览器均提供了对跨域资源共享的支持。**

如上所述，接着只需**在服务端配置可允许跨域的header**即可:
```php
<?php # TestController.php
class TestController
{
    public function __construct()
    {
        // 跨域头
        header('Access-Control-Allow-Origin: *');
    }
    
    public function test()
    {
        echo 'test';
    }
}
    
```
<!-- more -->

之后测试发现 `GET` 网络请求正常，不会报跨域的错误，但是被调用方一直报调用方未登录的异常，定位到是调用方的cookie无法传输给被调用方，查到是需要手动在ajax中增加配置 `withCredentials: true`
> *XMLHttpRequest.withCredentials*
> 
> 跨域请求是否提供凭据信息(cookie、HTTP认证及客户端SSL证明等)
也可以简单的理解为，当前请求为跨域类型时是否在请求中协带cookie。

需要注意的是调用方配置了该参数后，被调用方的header中必须指定 `Access-Control-Allow-Origin` 的值，不可以用 `*`，不然会报错：
```javascript
Response to preflight request doesn't pass access control check: A wildcard '*' cannot be used in the 'Access-Control-Allow-Origin' header when the credentials flag is true. 
Origin 'null' is therefore not allowed access. 
The credentials mode of an XMLHttpRequest is controlled by the withCredentials attribute.
```

更新后的服务端代码：
```php 
<?php # TestController.php
class TestController
{
    public function __construct()
    {
        $origin = $request->header('ORIGIN');
        header('Access-Control-Allow-Origin: ' . $origin);
        // 定义支持哪些方法的跨域请求
        header('Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, DELETE');
        // 服务端开启Access-Control-Allow-Credentials的支持
        header('Access-Control-Allow-Credentials: true');
    }
    
    public function test()
    {
        echo 'test';
    }
}
```

至此一般的 `GET` 跨域携带cookie的请求可以正常完成，那么 `POST`、`PUT`、`DELETE`这些请求呢？

##### Preflighted Requests(预检请求)
> Preflighted Requests是CORS中一种透明服务器验证机制。预检请求首先需要向另外一个域名的资源发送一个 HTTP OPTIONS 请求头，其目的就是为了判断实际发送的请求是否是安全的。

###### 什么是OPTIONS请求
> [https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)

RFC2616标准（现行的HTTP/1.1）中定义了  `options`请求，主要用途有两个：
1. 获取服务器支持的HTTP请求方法
2. 用来检查服务器的性能

什么情况会触发预检请求呢？就是上面提到的`POST`、`PUT`、`DELETE`等请求，大概来讲就是
RFC2616标准中规定的一些非 `Safe Methods`。

回过头看，现在js代码加上发起 `POST`的ajax请求，打开控制台发现只有一条 `OPTIONS`请求，并无 `POST`，而且报了跟最开始 `GET`请求一样的跨域错误，什么情况？

##### Laravel路由逻辑
```php
<?php # routes/api.php
    Route::post('/test', 'TestController@test');
```

路由文件中并未定义‘/test’的 `OPTIONS`类型请求，laravel是怎么匹配且响应200的？分析也可发现这个 `OPTIONS`请求没有进到此api路由文件的生命周期内，因为没有加上允许跨域的头部，问题在哪儿？

```php 
<?php # vendor/laravel/framework/src/Illuminate/Routing/RouteCollection.php
    $routes = $this->get($request->getMethod());

    // First, we will see if we can find a matching route for this current request
    // method. If we can, great, we can just return it so that it can be called
    // by the consumer. Otherwise we will check for routes with another verb.
    $route = $this->matchAgainstRoutes($routes, $request);

    if (! is_null($route)) {
        return $route->bind($request);
    }

    // If no route was found we will now check if a matching route is specified by
    // another HTTP verb. If it is we will need to throw a MethodNotAllowed and
    // inform the user agent of which HTTP verb it should use for this route.
    $others = $this->checkForAlternateVerbs($request);

    if (count($others) > 0) {
        return $this->getRouteForMethods($request, $others);
    }

    throw new NotFoundHttpException;
```

定位到Laravel路由的逻辑后知道了其中的端倪：

1. 首先根据当前HTTP方法（GET/POST/PUT/...）查找是否有匹配的路由，如果有`（if(! is_null($route))`条件成立)，非常好，绑定后直接返回，继续此后的调用流程即可；

2. 否则，根据$request的路由找到可能匹配的HTTP方法（即URL匹配，但是HTTP请求方式为其它品种的），如果`count($others) > 0)`条件成立，则继续进入`$this->getRouteForMethods($request, $others)`方法；

3. 否则抛出NotFoundHttpException，即上述说到的`404 NOT FOUND`错误。

倘若走的是第2步，可看到函数逻辑为：
```php
<?php # vendor/laravel/framework/src/Illuminate/Routing/RouteCollection.php
    /**
     * Get a route (if necessary) that responds when other available methods are present.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  array  $methods
     * @return \Illuminate\Routing\Route
     *
     * @throws \Symfony\Component\HttpKernel\Exception\MethodNotAllowedHttpException
     */
    protected function getRouteForMethods($request, array $methods)
    {
        if ($request->method() == 'OPTIONS') {
            return (new Route('OPTIONS', $request->path(), function () use ($methods) {
                return new Response('', 200, ['Allow' => implode(',', $methods)]);
            }))->bind($request);
        }

        $this->methodNotAllowed($methods);
    }
```

判断如果请求方式是`OPTIONS`，则返回状态码为200的正确响应（但是没有添加任何header信息），否则返回一个`methodNotAllowed`状态码为405的错误（即请求方式不允许的情况）。

由此可见，Laravel针对`OPTIONS`方式的HTTP请求处理方式已经固定了，最笨的方法是对跨域请求的每一个GET或POST请求都撰写一个同名的`OPTIONS`类型的路由，添加允许跨域的header，也有其他方法进行处理。

##### Laravel options请求跨域处理
###### 中间件方案
在文件 `app/Http/Kernel.php`中，有两处可以定义中间件。
```php
<?php # app/Http/Kernel.php

// 总中间件
protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        \App\Http\Middleware\TrustProxies::class,
];

   // 群组中间件
   /**
    * The application's route middleware groups.
    *
    * @var array
    */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
        'api' => [
            'throttle:60,1',
            'bindings',
            \Illuminate\Session\Middleware\StartSession::class,
        ],
    ];
```
第一处是总中间件 `$middleware`，任何请求都会通过这里；

第二处是群组中间件 `middlewareGroups`，**只有路由匹配上对应群组模式的才会通过这部分**，之前的OPTIONS请求尚未通过此处中间件的handle函数，就会返回。

因此我们添加的中间件，需要添加到$middleware数组中，不能添加到api群组路由中间件中。

在`app/Http/Middleware`文件夹下新建`PreflightResponse.php`文件：
```php
<?php #PreflightResponse.php

namespace App\Http\Middleware;
use Closure;
class PreflightResponse
{
    /**
    * Handle an incoming request.
    *
    * @param  \Illuminate\Http\Request  $request
    * @param  \Closure  $next
    * @param  string|null  $guard
    * @return mixed
    */
    public function handle($request, Closure $next, $guard = null)
    {
        if($request->getMethod() === 'OPTIONS'){
            $origin = $request->header('ORIGIN', '*');
            header("Access-Control-Allow-Origin: $origin");
            header("Access-Control-Allow-Credentials: true");
            header('Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, DELETE');
            header('Access-Control-Allow-Headers: Origin, Access-Control-Request-Headers, SERVER_NAME, Access-Control-Allow-Headers, cache-control, token, X-Requested-With, Content-Type, Accept, Connection, User-Agent, Cookie, X-XSRF-TOKEN');
        }
        return $next($request);
    }
}
```
其中这里针对OPTIONS请求的处理内容是添加多个header内容，可根据实际需要修改相关处理逻辑。

###### 通配路由匹配方案
```php
Route::options('/{all}', function(Request $request) {
    $origin = $request->header('ORIGIN', '*');
    header("Access-Control-Allow-Origin: $origin");
    header("Access-Control-Allow-Credentials: true");
    header('Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, DELETE');
    header('Access-Control-Allow-Headers: Origin, Access-Control-Request-Headers, SERVER_NAME, Access-Control-Allow-Headers, cache-control, token, X-Requested-With, Content-Type, Accept, Connection, User-Agent, Cookie');
})->where(['all' => '([a-zA-Z0-9-]|/)+']);
```
这样所有的OPTIONS请求都能找到匹配的路由，在此处可统一处理所有OPTIONS请求，不需要额外进行处理。

#### 参考
> [https://www.cnblogs.com/virtual/p/3720750.html](https://www.cnblogs.com/virtual/p/3720750.html)
> [https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
> [https://zhuanlan.zhihu.com/p/33542992](https://zhuanlan.zhihu.com/p/33542992)
> [https://www.jianshu.com/p/552daaf2869c](https://www.jianshu.com/p/552daaf2869c)