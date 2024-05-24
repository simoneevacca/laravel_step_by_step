# Laravel Auth

Install Laravel

```bash
# command you use to install laravel
composer create-project laravel/laravel:^10.0 example-app
```

Install Breeze

```bash

composer require laravel/breeze --dev
php artisan breeze:install
```

 ┌ Which Breeze stack would you like to install? ───────────────┐
 │ Blade with Alpine                                            │
 └──────────────────────────────────────────────────────────────┘

 ┌ Would you like dark mode support? ───────────────────────────┐
 │ No                                                           │
 └──────────────────────────────────────────────────────────────┘

 ┌ Which testing framework do you prefer? ──────────────────────┐
 │ Pest
 └──────────────────────────────────────────────────────────────┘

## Install laravel preset package

```bash

 
 
composer require pacificdev/laravel_9_preset
# Esegui comando preset
php artisan preset:ui bootstrap --auth
npm i

```

Update the vite config file

```bash
mv vite.config.js vite.config.cjs

php artisan serve
npm run dev
```

⚡ Ricorda di committare e pushare regolarmente!

## Refactoring Admin dashboard

create a controller to handle the dashboard requests

```bash
php artisan make:controller Admin/DashboardController

```

update the route and use the above controller

```php
Route::get('/dashboard', [DashboardController::class, 'index'])->middleware(['auth', 'verified'])->name('dashboard');
```

retun the view in the index method in the dashboard controller

```php
class DashboardController extends Controller
{
    function index()
    {
        return view('dashboard');
    }
}

```

## Add a route group

First create a new route group and add the following methods:

- middleware
- name
- prefix
- group

```php

// web.php

Route::middleware(['auth', 'verified'])
    ->name('admin.')
    ->prefix('admin')
    ->group(function () {
        // All routes need to share a common name and prefix and the middleware
       // 👇
        // Put here all routes that needs to be protected by our authenticatio system


    });

```

Move the dashboard route insde the group create above

```php


Route::middleware(['auth', 'verified'])
    ->name('admin.')
    ->prefix('admin')
    ->group(function () {
        // All routes need to share a common name and prefix and the middleware
       // 👇
        // Put here all routes that needs to be protected by our authenticatio system
        Route::get('/', [DashboardController::class, 'index'])->name('dashboard'); //admin


    });
```

Note: the dashbord route now uses only a `/` as the URI because it inherits the prefix from the group.

Update the constant at line 20 used in RouteServiceProvider.php from:

```php
//RouteServiceProvider.php
public const HOME = '/dashboard';

```

to:

```php
//RouteServiceProvider.php
    public const HOME = '/admin';

```

## Add a new layout file for the admin

Copy the original app.blade.php layout file and create a new file called admin.blade.php that we will use on all admin pages including our dashboard.

```bash

cd resources/views/layouts
cp app.blade.php admin.blade.php
```

extend the new admin layout inside the dashboard.blade.php

```php
@extends('layouts.admin')

@section('content')
<div class="container">
    <h2 class="fs-4 text-secondary my-4">
        {{ __('Dashboard') }}
    </h2>
    <div class="row justify-content-center">
        <div class="col">
            <div class="card">
                <div class="card-header">{{ __('User Dashboard') }}</div>

                <div class="card-body">
                    @if (session('status'))
                    <div class="alert alert-success" role="alert">
                        {{ session('status') }}
                    </div>
                    @endif

                    {{ __('You are logged in!') }}
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

Update the dropdown link in the header navigation menu for the admin layout and app layout (around line 73~) as follow

```php
<div class="dropdown-menu dropdown-menu-right" aria-labelledby="navbarDropdown">
  <a class="dropdown-item" href="{{ url('admin') }}">{{__('Dashboard')}}</a> // 👈 Update this link
  <a class="dropdown-item" href="{{ url('profile') }}">{{__('Profile')}}</a>
  <a class="dropdown-item" href="{{ route('logout') }}" onclick="event.preventDefault();
                       document.getElementById('logout-form').submit();">
      {{ __('Logout') }}
  </a>

    <form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
        @csrf
    </form>
</div>

```

## CRUD for posts

Now you can start working to your crud operations for the given resource

create:

- model
- migration
- controller (resource)
- seeder
- two form requests

```php
php artisan make:model Post -mscrR
```

### Implement all operations

Do as usual...