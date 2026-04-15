---
theme: academic
title: "Laravel API Resources & Rate Limiting"
info: |
  Transformers • Serializers • Pagination • Throttling
paginate: true
---

# API Resources & Rate Limiting (Week 10)
## Resource Transformers and Backend Protection

---

## 1. The Serialization Problem
When an API directly returns `return User::all();`, it executes default JSON serialization. For an enterprise application, this is a **severe architectural flaw**.

* **Information Exposure:** It blindly exposes internal database structures, such as sensitive columns (`password`, `remember_token`, `email_verified_at`) unless manually guarded.
* **Lack of Transformation:** Data is not formatted cleanly for the frontend (e.g., raw ISO timestamps instead of formatted strings, integer flags instead of booleans).
* **Breaking Consumer Contracts:** If you rename a database column from `first_name` to `f_name`, it immediately breaks the mobile app relying on that API endpoint structure.

---

## 2. Eloquent API Resources (The Transformer Pattern)
Laravel solves this using an intermediate layer called **API Resources**. They act as a data transformer, mapping your physical database structure to your desired API output structure.

* **Generation:** `php artisan make:resource UserResource`
* **Consumer Contracts:** Resources guarantee that no matter how the database changes, the API JSON structure remains perfectly consistent.

**The Transformer Architecture:**
```php
namespace App\Http\Resources;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    // Transforms the Eloquent model into a structured JSON array
    public function toArray(Request $request): array
    {
        return [
            'identifier' => $this->id, // Shielding internal column names
            'full_name'  => $this->name,
            'contact'    => $this->email,
            'joined_at'  => $this->created_at->format('Y-m-d'), // Formatting
        ];
    }
}
```

---

### Implementing Resources in Controllers
Once defined, a Resource is instantiated by passing standard Eloquent data into it.

**Transforming a Single Record:**
```php
public function show(User $user)
{
    // The model data is processed through the transformer and safely returned
    return new UserResource($user);
}
```

**JSON Output:**
```json
{
  "data": {
    "identifier": 1,
    "full_name": "Dr. Polla Fattah",
    "contact": "polla@example.com",
    "joined_at": "2026-04-15"
  }
}
```
*(Notice how Laravel automatically wraps the response in a `data` object to comply with proper JSON:API formatting standards.)*

---

### Removing the Data Wrapper (Customization)
While the `"data"` wrap is an industry standard, many frontend developers prefer "flat", unnested JSON structures to simplify their Axios or Fetch calls. 

* **The Solution:** You can globally strip this wrapper off all API Resources by explicitly calling `withoutWrapping()` inside the `boot` method of your `App\Providers\AppServiceProvider`.

```php
use Illuminate\Http\Resources\Json\JsonResource;

public function boot(): void
{
    // Strips the 'data' wrapper off all API JSON responses globally
    JsonResource::withoutWrapping();
}
```

---

## 3. Resource Collections & Pagination
Returning multiple records requires passing an Eloquent Collection (or Paginated Query) through the `collection` method of the Resource class.

**In the Controller:**
```php
public function index()
{
    $users = User::paginate(10); // Standard Eloquent pagination
    
    // Transforms every user in the paginated set through UserResource
    return UserResource::collection($users);
}
```

**Architectural Benefit:**
When you pass a standard paginator into a Resource Collection, Laravel automatically appends `links` and `meta` nodes to the API JSON response containing standard cursor and page-counting data for the frontend to render navigation!

---

## 4. Conditional JSON Attributes & Nested Relationships
APIs must be highly optimized and strict regarding data privacy.

**1. Field-Level Security (`when`)**
You can leverage the `$this->when()` method to conditionally include a sensitive attribute based on strict logical checks (e.g., securely hiding contact details unless the requesting user holds an Admin role).

```php
public function toArray(Request $request): array
{
    return [
        'identifier' => $this->id,
        'full_name'  => $this->name,
        
        // The 'contact' key is ENTIRELY omitted from the JSON unless the logic passes
        'contact'    => $this->when($request->user()->isAdmin(), $this->email),
    ];
}
```

---

**2. Relationship Eager Loading (`whenLoaded`)**
You should not return a user's `posts` if the mobile app didn't explicitly request them, as that triggers massive N+1 database queries.

By using `$this->whenLoaded()`, the Resource will only include the relationship in the JSON if the controller explicitly Eager Loaded it!

```php
public function toArray(Request $request): array
{
    return [
        'full_name' => $this->name,
        
        // This key will be entirely omitted from the JSON unless 
        // User::with('posts')->get() was called in the controller!
        'recent_posts' => PostResource::collection($this->whenLoaded('posts')),
    ];
}
```

---

## 5. Protecting the Backend: Rate Limiting
Because APIs are completely stateless, they are prime targets for automated abuse, such as DDoS bursts, web scraping, and password brute-forcing.

* **Rate Limiting (Throttling):** The act of strictly capping the number of HTTP requests a specific IP address or User ID can make within a specified timeframe.
* **The `throttle` Middleware:** Laravel natively handles this using caching layers like Redis or the database.

**In `routes/api.php`:**
```php
// Caps traffic to exactly 60 requests per 1 minute.
// Subsequent requests receive a '429 Too Many Requests' error.
Route::middleware(['auth:sanctum', 'throttle:60,1'])->group(function () {
    
    Route::apiResource('posts', PostController::class);
});
```

*Architectural Detail:* When a system throttles a user, Laravel automatically injects `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `Retry-After` headers into the HTTP response so the client machine knows exactly when it can safely ping the server again.

---

### Dynamic Rate Limiting (Enterprise Scaling)
The static `throttle:60,1` syntax is great for basics, but modern SaaS applications require programmable, dynamic limits (e.g., offering a free tier vs. a premium tier).

**1. Defining the Logic (`AppServiceProvider`):** 
You register programmatic limiters using the `RateLimiter` facade.
```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;

public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        // Premium users get 1000 requests per minute. Guests get restricted to 60 based on IP.
        return $request->user()?->isPremium()
                    ? Limit::perMinute(1000)
                    : Limit::perMinute(60)->by($request->ip());
    });
}
```

**2. Applying the Dynamic Limit:**
```php
// Apply the limit by its registered name ('api') instead of raw integers
Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('posts', PostController::class);
});
```
