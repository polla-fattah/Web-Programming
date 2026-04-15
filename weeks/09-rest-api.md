---
theme: academic
title: "REST APIs & Sanctum"
info: |
  REST Theory • Verbs & Routing • Status Codes • Token Authentication
paginate: true
---

# RESTful API Architecture (Week 9)
## Stateless Communication and Backend Decoupling

---

## 1. The Theory of REST API
In previous weeks, we built **Stateful Web Applications** where PHP rendered HTML templates (Blade) and managed user session cookies. This architecture is tightly coupled.

A **REST (Representational State Transfer) API** shifts the paradigm to a decoupled, stateless architecture.

* **Decoupling:** The backend (Laravel) no longer cares *how* the UI is rendered. It strictly returns structured data (JSON). This single backend can now simultaneously serve a React Web SPA, an iOS App, and an Android App.
* **Statelessness:** The server does not remember the client between requests (no Sessions). Every single HTTP request must contain all the information necessary to authenticate and process it.
* **Representations:** Clients interact with "Resources" (like a User or a Post) by performing actions upon their JSON representations via standard HTTP Verbs.

---

## 2. Cross-Origin Resource Sharing (CORS)
When decoupling backends and frontends, developers universally encounter the **CORS** security hurdle.

* **The Problem:** If your upcoming Vue/React frontend runs on `http://localhost:3000` and attempts to fetch data from your Laravel API on `http://localhost:8000`, the browser will aggressively block the HTTP request, throwing a red `Blocked by CORS policy` console error.
* **The Concept:** For security, web browsers prohibit scripts from pinging domains outside of their own "Origin" unless the destination explicitly permits it.
* **The Solution:** Laravel natively handles this in the `config/cors.php` file (or routing middleware in Laravel 11+) by explicitly whitelisting the frontend URL in the `allowed_origins` array.

---

## 3. HTTP Verbs and Route Conventions
REST strictly maps standardized HTTP Methods (Verbs) to CRUD database operations.

| HTTP Verb | CRUD Action | URI Route Format | Expected Behavior |
| :--- | :--- | :--- | :--- |
| **GET** | Read | `/api/posts` | Retrieve a collection of resources. |
| **GET** | Read | `/api/posts/{id}`| Retrieve a specific single resource. |
| **POST** | Create | `/api/posts` | Create a new resource. |
| **PUT/PATCH**| Update | `/api/posts/{id}`| Update an existing resource. |
| **DELETE** | Delete | `/api/posts/{id}`| Destroy an existing resource. |

*(Note: APIs do not use `create` or `edit` routes because there are no HTML forms to render!)*

---

### Laravel API Routing
Laravel segregates API routes into the `routes/api.php` file.

* **Differences from `web.php`:** Routes defined here automatically receive the `/api` URI prefix (e.g., `https://domain.com/api/posts`), and they are automatically assigned the `api` middleware group, which *disables* HTML session state completely.

**Defining Routes & API Versioning:**
To future-proof your architecture against mobile app crashes, enterprise APIs are always "versioned" (e.g., `/api/v1/posts`). If the database schema fundamentally changes next year, you launch a `/v2` group, ensuring legacy apps don't break!

```php
use App\Http\Controllers\Api\PostController;

// Wrapping routes in a 'v1' prefix guarantees backwards compatibility
Route::prefix('v1')->group(function () {
    
    // 1. Defining a specific endpoint
    Route::get('/posts', [PostController::class, 'index']);

    // 2. Defining a strict API Resource (Automatically excludes 'create' and 'edit')
    Route::apiResource('posts', PostController::class);
    
});
```

---

## 3. Standard HTTP Status Codes
Software engineers must speak the language of HTTP strictly. An API should never return an HTTP `200 OK` status if an error occurred.

* **2xx (Success):** 
  * `200 OK`: Request succeeded (standard for GET/PUT).
  * `201 Created`: Resource was successfully created (standard for POST).
  * `204 No Content`: Successful request but nothing to return (standard for DELETE).
* **4xx (Client Error):** 
  * `400 Bad Request`: General client mistake.
  * `401 Unauthorized`: Missing or invalid authentication token.
  * `403 Forbidden`: Authenticated, but lacks permission (Policy violation).
  * `404 Not Found`: Resource does not exist.
  * `422 Unprocessable Entity`: Validation failed.
* **5xx (Server Error):** `500 Internal Server Error` (Backend crashed).

---

## 5. API Controllers and JSON Responses
Laravel fundamentally simplifies building APIs. If your Controller returns a native array, an Eloquent Model, or a Collection, Laravel automatically serializes it into `application/json` with a default `200 OK` status.

**Native Serialization (Implicit):**
```php
public function show(Post $post)
{
    // Automatically converted to JSON with a 200 status code
    return $post; 
}
```

**Explicit Responses (Controlling Status Codes):**
```php
public function store(StorePostRequest $request)
{
    $post = Post::create($request->validated());

    // Explicitly returning a 201 Created status code
    return response()->json([
        'message' => 'Post created successfully',
        'data' => $post
    ], 201);
}
```

---

### Validation Exception Handling in APIs
In Week 4, we learned that when a rigorous Form Request fails, Laravel automatically interrupts the flow and redirects the user back to the HTML form with session errors.

**What happens in a Stateless API without sessions?**
Laravel is intelligent. It detects that the incoming HTTP request expects a JSON output (via the `Accept: application/json` header or the `/api` route prefix). 

Instead of crashing or attempting a redirect, Laravel gracefully intercepts the `ValidationException` and converts it into a standardized HTTP `422 Unprocessable Entity` JSON response containing the full exact validation error bag! No extra controller logic is required.

---

## 6. API Authentication (Laravel Sanctum)
Because API routes are stateless, we cannot use Session Cookies to remember users. We must issue Cryptographic Tokens instead. **Laravel Sanctum** is the industry standard for lightweight API token authentication.

* **The Flow:**
  1. Mobile App sends credentials to a `/api/login` endpoint via POST.
  2. Laravel verifies the credentials.
  3. Laravel generates a secure plaintext string (the Token) and saves a SHA-256 hash formatting in the database.
  4. Mobile App stores this Token locally.
  5. Mobile App attaches this Token to the `Authorization: Bearer <token>` HTTP header of *every* subsequent request.

---

### Installing & Configuring Sanctum
While Sanctum is often pre-installed in modern Laravel deployments, securing the foundation is required.

**1. Installation:** 
```bash
composer require laravel/sanctum
php artisan install:api
```

**2. Model Configuration:**
The `User` model must utilize the `HasApiTokens` trait to physically enable token generation capabilities.
```php
namespace App\Models;

use Laravel\Sanctum\HasApiTokens; // Added trait
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasApiTokens; 
    
    // ...
}
```

---

### Implementing Sanctum Token Issuance

**The Login Controller Method:**
```php
use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

public function login(Request $request)
{
    $request->validate(['email' => 'required', 'password' => 'required']);
 
    $user = User::where('email', $request->email)->first();
 
    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['Incorrect credentials provided.'],
        ]);
    }
 
    // Issue a brand new plaintext token strictly bound to this user
    $token = $user->createToken('ios-device')->plainTextToken;
    
    return response()->json(['token' => $token], 200);
}
```

---

### Token Abilities (Permissions)
Sanctum allows developers to assign "abilities" (scopes) to tokens, granting strict API permissions to specific devices (e.g., generating read-only tokens).

```php
// Issuing a token restricting the client to read-only capabilities
$token = $user->createToken('guest-app', ['post:read'])->plainTextToken;

// Later, checking abilities during an HTTP request
if ($request->user()->tokenCan('post:update')) {
    // Proceed with destructive update
} else {
    abort(403, 'Your token lacks update abilities.');
}
```

---

### Revoking Tokens (API Logout)
Stateless authentication does not possess an HTTP "session" to automatically destroy. To log a user out of an API, the explicit token must be physically deleted from the backend server.

```php
// Inside LogoutController
public function logout(Request $request)
{
    // Revoke the exact token that was used to authenticate the current request.
    // The mobile app will now receive 401 Unauthorized on the next ping.
    $request->user()->currentAccessToken()->delete();

    // Alternatively, revoke ALL tokens (Log out from all devices globally)
    // $request->user()->tokens()->delete();

    return response()->json(['message' => 'Successfully logged out']);
}
```

---

### Protecting Routes via Sanctum Middleware
Once the client has the token, it must pass it via the `Authorization` header. Laravel intercepts this header using the native `auth:sanctum` middleware layer to securely guard the endpoint.

**In `routes/api.php`:**
```php
// Any request hitting this group without a valid Bearer Token in the Header 
// will instantly be rejected with a 401 Unauthorized response.
Route::middleware('auth:sanctum')->group(function () {
    
    // The authenticated user object resolves dynamically strictly from the token!
    Route::get('/user', function (Request $request) {
        return $request->user();
    });

    Route::apiResource('posts', PostController::class);
});
```

---

### Custom API Middleware (Request Filtering)
Beyond built-in authentication, APIs often require custom filters to validate incoming traffic (e.g., enforcing an `Accept: application/json` header, or validating a custom third-party `X-API-Key`).

**1. Generating the Middleware:**
```bash
php artisan make:middleware EnsureJsonHeader
```

**2. Implementing the Filter Logic:**
```php
namespace App\Http\Middleware;
use Closure;
use Illuminate\Http\Request;

class EnsureJsonHeader
{
    public function handle(Request $request, Closure $next)
    {
        // 1. Intercept the request BEFORE it reaches the Controller
        if ($request->header('Accept') !== 'application/json') {
            // Immediately bounce the request with a strict format error
            return response()->json(['error' => 'Not Acceptable: JSON strictly required.'], 406);
        }

        // 2. If the filter passes, push the request deeper into the application
        return $next($request);
    }
}
```

---

### Applying Middleware Filters to Route Groups
Once you have created your customized filters, you assign them to your API route groups, stacking them to create an impenetrable security pipeline.

**In `routes/api.php`:**
```php
use App\Http\Middleware\EnsureJsonHeader;

// Stacking multiple middlewares (Sanctum Auth + Your Custom JSON Filter)
Route::middleware(['auth:sanctum', EnsureJsonHeader::class])->group(function () {
    
    // All routes inside this closure are now heavily filtered
    Route::apiResource('posts', PostController::class);
    
    // You can group further to add URL prefixes cleanly!
    Route::prefix('v1/admin')->group(function() {
        Route::delete('/logs', [LogController::class, 'flush']);
    });

});
```

---

## 6. Developer Tooling: API Testing Clients
Because API endpoints do not return HTML or frontend interfaces, you cannot easily test `POST`, `PUT`, or `DELETE` requests natively in a web browser's address bar. Software engineers rely on specialized API Clients.

* **Postman / Insomnia:** The legacy industry standards. Excellent for testing endpoints, managing environment variables (like base URLs), and inspecting JSON responses.
* **Bruno (The Modern Standard):** A fast, open-source API client. Unlike Postman which syncs to the cloud, Bruno stores API collections directly in your project folder as plain text `.bru` files. This makes it infinitely superior for **Git version control** and team collaboration.

---

### Executing an Authenticated Request in Bruno

To test a protected API endpoint (e.g., creating a new Post), you must configure three core components in your Bruno interface:

1. **The Method & URL:** 
   * Set the primary dropdown to `POST`.
   * Set the URL to `http://localhost:8000/api/posts`.
2. **The Authentication (Sanctum Token):**
   * Navigate to the **Auth** tab in Bruno and select `Bearer Token`.
   * Paste the plaintext token generated by your `/api/login` endpoint.
   * *(Bruno automatically injects the `Authorization: Bearer <token>` HTTP header for you).*
3. **The Data Payload (Body):**
   * Navigate to the **Body** tab and select `JSON`.
   * Write the exact JSON structure your Form Request explicitly expects:
     ```json
     {
       "title": "My First API Record",
       "body": "Testing via Bruno client."
     }
     ```
   * Click **Send** and watch for a `201 Created` HTTP Status Code!
