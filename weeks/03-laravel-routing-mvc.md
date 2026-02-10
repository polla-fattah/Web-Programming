---
theme: default
title: "Laravel Fundamentals"
info: |
  Project structure • Routing • Controllers • Middleware • Validation • Blade • Artisan • Debugbar
paginate: true
---

# Laravel Fundamentals
## Structure • Routing • Middleware • Validation • Blade • Artisan • Debugbar

---

## Today’s Agenda
- Laravel project structure
- Routing + controllers
- Request lifecycle + middleware
- Validation
- Blade (minimal)
- Artisan CLI (intro)
- Laravel Debugbar (profiling + inspection)
- Mini build plan (in-class)

---

## Learning Outcomes
By the end of this lecture, you can:
- Navigate a Laravel project confidently
- Create routes + controllers for real endpoints
- Explain the request lifecycle (where middleware fits)
- Validate input (and handle errors properly)
- Render minimal views with Blade
- Use Artisan to scaffold + run core commands
- Use Debugbar to inspect queries, timing, requests

---

# Part 1 — Project Structure

---

## Laravel Project “Map”
Key folders/files you’ll touch most:
- `app/` → application code
- `routes/` → route definitions
- `resources/views/` → Blade templates
- `database/` → migrations, seeders, factories
- `config/` → app configuration
- `.env` → environment values (local only)
- `public/` → web entry (public files)
- `storage/` → logs, caches, uploads

---

## app/ (Where your backend lives)
Important subfolders:
- `app/Http/Controllers/` → controllers
- `app/Http/Middleware/` → middleware
- `app/Http/Requests/` → FormRequest validation classes
- `app/Models/` → Eloquent models

---

## Where to configure things
- `.env` → *your machine settings* (DB, app key, debug)
- `config/*.php` → framework config (reads `.env`)

Rule:
- Don’t hardcode secrets in code
- Use `.env` for environment differences

---

# Part 2 — Routing + Controllers

---

## Routes: What they do
A route maps:
- HTTP method + URL
→ to a closure or controller action
→ returns a response (HTML/JSON/redirect)

---

## Route files
- `routes/web.php` → browser flows (sessions, CSRF)
- `routes/api.php` → stateless API (often token-based)

---

## Basic Route
```php
// routes/web.php
use Illuminate\Support\Facades\Route;

Route::get('/hello', function () {
    return 'Hello Laravel';
});
````

---

## Route returning a view

```php
Route::get('/welcome', function () {
    return view('welcome');
});
```

---

## Route parameters

```php
Route::get('/users/{id}', function (string $id) {
    return "User: {$id}";
});
```

---

## Named routes (important later)

```php
Route::get('/dashboard', fn () => view('dashboard'))
    ->name('dashboard');
```

Usage:

```php
route('dashboard')
```

---

## Route groups (cleaner structure)

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', fn () => 'Admin users');
});
```

---

## Controllers: Why?

Use controllers to:

* organize related endpoints
* keep route files readable
* prepare for validation, auth, services

---

## Create a controller (Artisan)

```bash
php artisan make:controller TaskController
```

---

## Controller example

```php
// app/Http/Controllers/TaskController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function index() {
        return view('tasks.index');
    }
}
```

---

## Route to controller

```php
use App\Http\Controllers\TaskController;

Route::get('/tasks', [TaskController::class, 'index']);
```

---

## Resource routes (CRUD shortcut)

```bash
php artisan make:controller TaskController --resource
```

```php
Route::resource('tasks', TaskController::class);
```

---

# Part 3 — Request Lifecycle + Middleware

---

## The Request Lifecycle (mental model)

Request comes in → Laravel boots → middleware pipeline → router matches → controller runs → response returned

---

## Lifecycle (high-level steps)

1. `public/index.php` receives request
2. framework bootstraps (config, providers)
3. HTTP Kernel runs middleware stack
4. Router matches route
5. Controller/action executes
6. Response returned to client

---

## Middleware: What is it?

Think “layers” around your route:

* can allow the request through
* can block/redirect
* can modify request/response

Examples:

* auth checks
* rate limiting
* logging
* locale setting

---

## Middleware in routes

```php
Route::get('/admin', fn () => 'Admin area')
    ->middleware('auth');
```

---

## Middleware groups (concept)

Common groups you’ll see:

* `web` → sessions, CSRF, cookies
* `api` → stateless defaults

(We’ll inspect them later in the project config)

---

## Create custom middleware

```bash
php artisan make:middleware EnsureIsAdmin
```

---

## Middleware skeleton

```php
// app/Http/Middleware/EnsureIsAdmin.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class EnsureIsAdmin
{
    public function handle(Request $request, Closure $next)
    {
        // check condition...
        return $next($request);
    }
}
```

---

# Part 4 — Validation

---

## Why validation matters

* protects your DB integrity
* prevents bad inputs
* improves UX (clear errors)
* prevents “silent bugs”

---

## Quick validation in controller

```php
public function store(Request $request)
{
    $data = $request->validate([
        'title' => ['required', 'string', 'min:3'],
        'due_date' => ['nullable', 'date'],
    ]);

    // store using $data...
}
```

---

## What happens on validation failure?

* Web request: redirects back with errors + old input
* API request: returns JSON error response

(We’ll see it live in the demo)

---

## Form Request (cleaner validation)

```bash
php artisan make:request StoreTaskRequest
```

---

## Form Request example

```php
// app/Http/Requests/StoreTaskRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreTaskRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'min:3'],
            'due_date' => ['nullable', 'date'],
        ];
    }
}
```

---

## Using Form Request in controller

```php
use App\Http\Requests\StoreTaskRequest;

public function store(StoreTaskRequest $request)
{
    $data = $request->validated();
    // store...
}
```

---

# Part 5 — Blade (Minimal)

---

## Blade: What you need today

* where views live: `resources/views`
* pass data to views
* show errors
* basic loops/conditions

---

## Return a view with data

```php
return view('tasks.index', [
    'tasks' => $tasks,
]);
```

---

## Blade basics

```blade
<h1>Tasks</h1>

<ul>
  @foreach ($tasks as $task)
    <li>{{ $task->title }}</li>
  @endforeach
</ul>
```

---

## Blade: form + error display

```blade
<form method="POST" action="{{ route('tasks.store') }}">
  @csrf

  <input name="title" value="{{ old('title') }}" />

  @error('title')
    <div>{{ $message }}</div>
  @enderror

  <button type="submit">Save</button>
</form>
```

---

# Part 6 — Artisan CLI (Intro)

---

## Artisan: Examples

* **`php artisan list`**: Displays a complete directory of all available Artisan commands and their descriptions.
* **`php artisan serve`**: Starts a local development server so you can access your Laravel application via a web browser.
* **`php artisan route:list`**: Generates a table showing all registered routes, including their URI, names, and associated controller actions.
* **`php artisan migrate`**: Executes all pending migrations to synchronize your database schema with your application code.
* **`php artisan tinker`**: Opens an interactive REPL (Read-Eval-Print Loop) to interact with your database and application logic using PHP code.


---

## Scaffolding shortcuts you’ll use a lot

```bash
php artisan make:model Task -mcr
# -m migration
# -c controller
# -r resource controller
```

---

## Typical build flow (repeatable)

1. `make:model -mcr`
2. edit migration
3. migrate
4. add routes
5. implement controller
6. build Blade views
7. validate inputs

---

# Part 7 — Laravel Debugbar

---

## Debugbar: Why we use it

It helps you see:

* executed SQL queries
* request/response details
* route/controller used
* render time + memory usage
* exceptions and logs (in dev)

---

## Install Debugbar (dev only)

```bash
composer require fruitcake/laravel-debugbar --dev
```

Rules:

* dev only
* never expose it on public production

---

## Debugbar: what to check first

In browser toolbar:

* **Queries** (N+1 issues)
* **Timeline** (slow steps)
* **Request** (inputs, headers)
* **Route** (what matched)
* **Views** (what rendered)

---

# In-class Mini Build (guided)

---

## Goal (Small but real)

Build a minimal **Tasks** flow:

* GET `/tasks` → list page
* GET `/tasks/create` → create form
* POST `/tasks` → validate + store
* Debugbar: inspect queries + timing

---

## Step 1 — Scaffold

```bash
php artisan make:model Task -mcr
php artisan migrate
```

---

## Step 2 — Add routes

```php
use App\Http\Controllers\TaskController;

Route::resource('tasks', TaskController::class);
```

---

## Step 3 — Add validation in store()

* `title` required, min 3
* optional `due_date` is a date

---

## Step 4 — Create Blade views

Create:

* `resources/views/tasks/index.blade.php`
* `resources/views/tasks/create.blade.php`

Minimal UI:

* list tasks
* form submit + error display

---

## Step 5 — Debugbar checks (5 minutes)

* Open `/tasks`
* Verify:

  * which route/controller executed
  * how many queries executed
  * total time

---

## End of Lecture Checklist

You should be able to:

* explain the route → controller path
* describe where middleware fits
* validate a store request
* render a Blade form and show errors
* scaffold with Artisan
* use Debugbar to inspect queries/time
 