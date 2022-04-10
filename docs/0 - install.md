# Installation 

Installation is done through [Composer](https://getcomposer.org). The example assumes you have it installed globally. 
If you have it installed as a phar, or othewise you will need to adjust the way you call composer itself.

```
> composer require lonnieezell/codeigniter-shield
```

This requires the [CodeIgniter Settings](https://github.com/codeigniter4/settings) package, which uses a database
table to store configuration options. As such, you should run the migrations. 

```
> php spark migrate --all
```

## Intial Setup

There are a few setup items to do before you can start using Shield in 
your project. 

1. Copy the `Auth.php` and  `AuthGroups.php` from `vendor/lonnieezell/codeigniter-shield/src/Config/` into your project's config folder and update the namespace to `Config`. These files contain all of the settings, group, and permission information for your application and
will need to be modified to meet the needs of your site.

2. **Helper Setup** The `auth` and `setting` helpers need to be included in almost every page. The simplest way to do this is to add them to the `BaseController::initController` method: 

```php
public function initController(RequestInterface $request, ResponseInterface $response, LoggerInterface $logger)
{
    $this->helpers = array_merge($this->helpers, ['auth', 'setting']);

    // Do Not Edit This Line
    parent::initController($request, $response, $logger);
}
```

This requires that all of your controllers extend the `BaseController`, but that's a good practice anyway.

3. **Routes Setup** The default auth routes can be setup with a single call in `app/Config/Routes.php`: 

```php
auth()->routes($routes);
```

## Further Customization

### Route Configuration

If you need to customize how any of the auth features are handled, you will likely need to update the routes to point to the correct controllers. You can still use the `auth()->routes()` helper, but you need to pass a list of routes to an `except` option: 

```php
auth()->routes($routes, ['except' => ['login', 'register']]);
```

### Extending the Controllers

Shield has the following controllers that can be extended to handle 
various parts of the authentication process: 

- **ActionController** handles the after login and after-registration actions that can be ran, like Two Factor Authentication and Email Verification. 

- **LoginController** handles the login process. Contains several methods that can be overriden to customize where a user is redirected
to after login or logout. 

- **RegisterController** handles the registration process. Overriding this class allows you to customize the User Provider, the User Entity, 
the validation rules, and where a user is redirected after registration.

- **MagicLinkController** handles the "lost password" process that allows a user to login with a link sent to their email. Allows you to
override the message that is displayed to a user to describe what is happening, if you'd like to provide more information than simply swapping out the view used.

It is not recommended to copy the entire controller into app and change it's namespace. Instead, you should create a new controller that extends
the existing controller and then only override the methods needed. This allows the other methods to always stay up to date with any security 
updates that might happen in the controllers.

```php
<?php

namespace App\Controllers;

use Sparks\Shield\Controllers\LoginController as ShieldLogin;

class LoginController extends ShieldLogin
{
    public function getLoginRedirect($user)
    {
        // new functionality 
    }
}
```
