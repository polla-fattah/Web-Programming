---
theme: academic
title: "Laravel Authentication"
info: |
  Identity Validation • Session Management • Auth Facade • Middleware
paginate: true
---

# Authentication & Identity (Week 7)
## Validating Users, State, and Security

---

## 1. The Psychology of Authentication (Theory)
Authentication (AuthN) answers a single question: **"Who is this user?"**

* **The Stateless Protocol:** The HTTP protocol is inherently stateless. It has no memory of previous requests. 
* **Stateful (Web) Authentication:** To "remember" a user is logged in, Laravel utilizes **Sessions** and **Cookies**. After verifying credentials, Laravel hands the browser an encrypted cookie (session ID) that must be sent with every subsequent request.
* **Stateless (API) Authentication:** Typical for mobile apps or REST APIs. No cookies are used. Instead, a cryptographic token (e.g., Bearer token or JWT) is sent in the HTTP Headers of every request.

---

## 2. The Built-in `Auth` Facade
Laravel provides a unified interface, the `Auth` facade, for interacting with authentication services regardless of how the user logged in.

**Retrieving the User State:**
```php
use Illuminate\Support\Facades\Auth;

// 1. Check if the current request is from a logged-in user
if (Auth::check()) {
    // 2. Retrieve the currently authenticated User model
    $user = Auth::user();

    // 3. Retrieve only the ID (highly optimized, avoids DB lookup)
    $id = Auth::id();
}
```
*Architectural Note:* Because `$user` is a native Eloquent model, you immediately have access to all relationships (e.g., `Auth::user()->posts`).

---

## 3. Guards and Providers
Laravel's authentication architecture is comprised of "Guards" and "Providers".

* **Guards:** Define *how* users are authenticated for each request.
  * `web`: The default guard. Uses Session storage and Cookies.
  * `api` / `sanctum`: Uses API tokens sent in HTTP headers.
* **Providers:** Define *how* users are retrieved from persistent storage.
  * `eloquent`: Uses the Eloquent ORM to retrieve the user (default).
  * `database`: Uses raw SQL queries if Eloquent is unavailable.

*(These components are explicitly configured in `config/auth.php`)*

---

## 4. Route Protection (Middleware)
Once authentication is established, we must secure the application's endpoints. Laravel achieves this via Route Middleware.

* **Middleware:** An onion-like layer that intercepts HTTP requests *before* they reach the Controller.
* **The `auth` Middleware:** Demands that the user is logged in. If they are not, it intentionally bounces them (returns `401 Unauthorized` or redirects to `/login`).

```php
use App\Http\Controllers\DashboardController;

// Grouping routes to demand authentication
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/settings', [SettingsController::class, 'edit']);
});
```

---

### The Inverse: Guest Middleware
Conversely, authenticated users should not be allowed to access screens meant only for unauthenticated traffic (e.g., the Login or Registration pages).

* **The `guest` Middleware:** Checks if the user is already logged in. If they are, it bypasses the route and redirects them to their dashboard.

```php
use App\Http\Controllers\Auth\LoginController;

// Applying the 'guest' middleware directly to a route
Route::get('/login', [LoginController::class, 'create'])
     ->middleware('guest')
     ->name('login');
```

---

## 5. Implementation Strategy: Manual Authentication
To understand the underlying architecture, it is essential to observe manual authentication logic.

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Http\Request;

public function authenticate(Request $request)
{
    $credentials = $request->validate([
        'email' => ['required', 'email'],
        'password' => ['required'],
    ]);

    // The attempt() method accepts the credentials and a "Remember Me" boolean.
    // It automatically hashes the plain password via Bcrypt (or Argon2) 
    // and securely compares it to the database string under the hood.
    if (Auth::attempt($credentials, $request->boolean('remember'))) {
        
        // Regenerating the session ID prevents session fixation attacks.
        $request->session()->regenerate();
 
        return redirect()->intended('dashboard');
    }
 
    return back()->withErrors(['email' => 'The provided credentials do not match.']);
}
```

---

## 6. The Logout Lifecycle (Security Finalization)
Authentication is incomplete without a secure mechanism to destroy the state. Calling `Auth::logout()` is insufficient for strict security; the session metadata itself must be obliterated.

* **Invalidate:** Destroys all physical HTTP session data.
* **Regenerate Token:** Issues a brand new CSRF token to natively prevent cross-site request forgery hijacking on subsequent unauthenticated requests.

```php
public function logout(Request $request)
{
    // 1. Clears the authentication state from the Guard
    Auth::logout();
 
    // 2. Destroys the physical session data
    $request->session()->invalidate();
 
    // 3. Regenerates the CSRF token to prevent hijacking
    $request->session()->regenerateToken();
 
    return redirect('/');
}
```

---

## 7. Authentication Events (Audit Logging)
For enterprise architecture, tracking system access is mandatory. Laravel dispatches native application events throughout the authentication lifecycle.

* `Illuminate\Auth\Events\Login`: Fired upon successful authentication.
* `Illuminate\Auth\Events\Failed`: Fired when a user supplies incorrect credentials.
* `Illuminate\Auth\Events\Logout`: Fired when the user session is destroyed.

*Architectural Use Case:* Developers attach **Event Listeners** to these hooks to build security Audit Logs, block IPs during brute-force attacks, or trigger "New login from unknown device" email alerts.

---

## 8. Implementation Strategy: Laravel Starter Kits
While understanding manual logic is critical for debugging, modern enterprise development relies on structural scaffolding.

* **Laravel Breeze:** A minimal and simple starting point. It scaffolds all authentication controllers, routes, and Blade views using modern Tailwind CSS.
* **Laravel Jetstream:** A highly robust, advanced scaffolding kit providing Two-Factor Authentication (2FA), API Token management, and Livewire/Inertia.js integration.

**The Development Workflow:**
```bash
# 1. Install the Breeze package
composer require laravel/breeze --dev

# 2. Scaffold the Blade/Alpine UI templates
php artisan breeze:install blade

# 3. Compile assets and migrate the new schemas
pnpm install && pnpm build
php artisan migrate
```
