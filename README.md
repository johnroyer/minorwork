# MinorWork Framework

[![Build Status](https://travis-ci.org/CQD/minorwork.svg?branch=master)](https://travis-ci.org/CQD/minorwork)

A minimal PHP framework that is serious on being minimal.

Features:

- A container.
- Named routing with redirection support
- Less than 300 lines.
- Only external dependency is `nikic/fast-route` for routing.

## Install

You can install MinorWork with composer.

```
composer require minorwork/minorwork
```

PHP 5.4 or newer required.

## Usage

Using MinorWork framework looks like this:

```php
<?php
// This is index.php
// All traffic shoud bed handled by this file.
// You may need edit your .htaccess for this.

include __DIR__ . '/../vendor/autoload.php';

use MinorWork\App;

$app = new App();
$app->setRouting([
    'root'         => ['/', '\Controller\Index:home'],
    'loginForm'    => ['GET', '/login', '\Controller\Login:form'],
    'loginAction'  => ['POST', '/login', '\Controller\Login:login'],
    'article'      => ['/articles/{id}', '\Controller\Article:show'],
    'users'        => ['\Controller\User:list'], // matches /users
    'userProfile'  => ['/user/{id}', '\Controller\User:profile'],
]);

$app->run();
```

Define routing, the run the app. MinorWork will take care the rest.

Another example is in [example/](example/) folder. You can see it in action using PHP dev server.

```shell
cd example
php -S localhost:8765
```

### Routing

You can config routing by passing an array of routes to App object;

Key of each route is the name that route. And a route itself looks like this:

```php
'login' => [
    'GET',                    // Method
    '/login',                 // url pattern
    '\Controller\User:list',  // Handler
]
```

#### HTTP Method 
First element is allowed http method. Can either be a string (only one method allowed, ex: `GET`)
or a array of strings (multiple methods allowed, ex: `['GET', 'POST']`)

Method can be omitted. If so, `['GET', 'POST']` will be used.

```php
// This
'login' => ['/login', '\Controller\User:login']

// equals to this
'login' => [['GET', 'POST'], '/login', '\Controller\User:login']
```

#### URL pattern
Second element is the URL pattern to match. URL pattern uses [fast-route's syntax](https://github.com/nikic/FastRoute#defining-routes):

- `/user` Matches only /user
- `/user/{id:\d+}` Matches /user/42 and /user/53, but not /user and /user/xyz
- `/user/{name}` Matches /user/foobar, but not /user/foo/bar
- `/user/{name:.+}` Matches /user/foo/bar as well
- `/user/{id:\d+}[/{name}]` Matches /user/42 AND /user/42/xyz
- `/user[/{id:\d+}[/{name}]]` Option parts can be nested

URL pattern can also be omitted, if so, route name will be used.

```php
// This
'login' => ['\Controller\User:login']

// equals to this
'login' => ['/login', '\Controller\User:login']

// also equals to this
'login' => [['GET', 'POST'], '/login', '\Controller\User:login']
```

#### Request Handler

The third element is the request handler. Request handler can one of those:

- A controller/method pair (recommended).
  - `\Controller\User:login` means `$obj = new \Controller\User(); $obj->login($app, $params)`
- A [callable](http://php.net/manual/en/language.types.callable.php), like name of a function, or a closure, or other callable type. `login_handler` means calling `login_handler($app, $params)`

Either case two arguments will be pass to request hander
- `$app`, the container object.
- `$param`, matched parameters in URL pattern.

You can access GET, POST, and SERVER parameters from the container.

```php
$get = $app->get('_GET');
$post = $app->get('_POST');
$server = $app->get('_SERVER');
```

You can not omit request handler in a route (why define a route that no one cares?)

#### Default handler

There is a predefined default handler. If no route matches, the default handler will be used. So it is most likely your 404 not found handler.

The predefined one is pretty basic. You can override default request handler with route named `default`:

```php
'default' => ['\Controller\Error:FourOFour']
```

### Container

TBD

### Helper

`$app->routePath($routeName, $params = [], $query = [])` returns path to route of given name. `$param` will be used to generate that path. if `$query` is provided, it will be used as query string parameter. Throws exception if failed to generate that path.

`$app->redirectTo($routeName, $params = [], $query = [])` redirect you to the same path `routePath()` gives you, and terminate current request.




