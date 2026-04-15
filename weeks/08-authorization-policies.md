---
theme: academic
title: "Laravel Authorization & Policies"
info: |
  Role-Based Access Control • Gates • Model Policies • Blade UI 
paginate: true
---

# Authorization & Policies (Week 8)
## Role-Based Access Control and Resource Protection

---

## 1. Authentication vs. Authorization
A critical distinction in software engineering security.

* **Authentication (AuthN):** Validating *who* the user is (e.g., verifying an email/password combination). Returns HTTP `401 Unauthorized` if failed.
* **Authorization (AuthZ):** Validating *what* the authenticated user is allowed to do. Returns HTTP `403 Forbidden` if failed.

*Example:* A doctor and a receptionist are both **Authenticated** into the clinic portal. However, only the doctor is **Authorized** to prescribe medication.

---

## 2. System-Wide Permissions (Gates)
Laravel uses "Gates" to evaluate simple closure-based permissions. Gates are ideal for system-wide checks that are *not* tied to a specific database model.

* **Definition Location:** Typically defined in the `boot` method of the `App\Providers\AppServiceProvider`.
* **Use Cases:** "Is this user a Super Admin?" or "Can this user access the system settings panel?"

**Defining a Gate:**
```php
use Illuminate\Support\Facades\Gate;
use App\Models\User;

public function boot(): void
{
    // Evaluates if the user is assigned the 'admin' role
    Gate::define('access-dashboard', function (User $user) {
        return $user->role === 'admin';
    });
}
```

---

### The "Super Admin" Bypass (`Gate::before`)
A standard enterprise architectural pattern is granting a "Super Admin" overriding access to all system actions without writing `if ($user->is_super_admin)` in every single method.

* **Functionality:** `Gate::before` intercepts all authorization checks before they are evaluated. If the closure returns `true`, it overrides all subsequent Gates and Policies.

```php
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    // Super Admins automatically pass ALL authorization checks application-wide
    Gate::before(function (User $user, string $ability) {
        if ($user->hasRole('Super Admin')) {
            return true; 
        }
    });
}
```

---

## 3. Route-Level Authorization Middleware
While authorization is commonly handled inside the Controller, it is often cleaner to block unauthorized users before the application even boots the controller logic, directly in the router.

* **The `can` Middleware:** Maps directly to Gate or Policy methods natively in the route file.

```php
use App\Http\Controllers\PostController;
use App\Http\Controllers\DashboardController;

// 1. System Gate check ('access-dashboard')
Route::get('/dashboard', [DashboardController::class, 'index'])
     ->middleware('can:access-dashboard');

// 2. Model Policy check (Passes the route parameter '{post}' to the policy)
Route::put('/posts/{post}', [PostController::class, 'update'])
     ->middleware('can:update,post');
```

---

## 4. Model-Specific Permissions (Policies)
While Gates handle system actions, **Policies** are dedicated classes that handle complex authorization logic bound directly to a specific Eloquent Model.

* **The Generation:** `php artisan make:policy PostPolicy --model=Post`
* **Auto-Discovery (Convention over Configuration):** Laravel automatically discovers policies as long as the models reside in `App\Models` and the policy is named exactly `[ModelName]Policy` within `App\Policies`.
* **Method Mapping:** Policies intuitively map to controller CRUD actions (`view`, `create`, `update`, `delete`).

**Defining a Policy Method:**
```php
namespace App\Policies;
use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    // The user can only update the post if they strictly own it
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

---

### Executing Model Policies & Resource Controllers
Because Policies are strictly bound to Eloquent models, Laravel's Controller component provides elegant methods, bypassing the `Gate` facade entirely.

**Standard Policy Execution:**
```php
public function update(Request $request, Post $post)
{
    // Automatically locates PostPolicy and evaluates the 'update' method.
    $this->authorize('update', $post);
    // ...
}
```

**Resource Controller Authorization:**
For students building full CRUD architectures, Laravel can authorize standard routes instantly.
```php
class PostController extends Controller
{
    public function __construct()
    {
        // 1 line of code maps: index, show, create, store, edit, update, destroy
        // directly to their respective Policy methods automatically!
        $this->authorizeResource(Post::class, 'post');
    }
}
```

---

## 5. Policy Integration: Form Requests
Policies synergize perfectly with the Validation techniques (Week 4) by leveraging Form Requests, establishing the highest form of cleanly separated logic.

**In `App\Http\Requests\UpdatePostRequest`:**
```php
public function authorize(): bool
{
    // Retrieve the active post mapped via Route Model Binding
    $post = $this->route('post');

    // Evaluate the policy directly within the request boundary
    return $this->user()->can('update', $post);
}

public function rules(): array
{
    return [
        'title' => 'required|string|max:255',
        'body'  => 'required|string',
    ];
}
```

---

## 6. Security Integration: Blade UI
It is poor user experience (UX) to show users buttons or navigations for actions they are not authorized to perform. Blade provides native directives to conditionally render UI.

**Using `@can` and `@cannot`:**
```blade
<!-- Displayed to all users -->
<h1>{{ $post->title }}</h1>
<p>{{ $post->body }}</p>

<!-- The edit button is only rendered in the HTML if the policy evaluates to true -->
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}" class="btn">Edit Post</a>
@endcan
```

*Architectural Warning:* Hiding a button in Blade **does not** secure the route! A malicious user can still manually forge an HTTP request to the endpoint. Your Controllers/Form Requests **must** be securely enforcing the Policy on the backend.
