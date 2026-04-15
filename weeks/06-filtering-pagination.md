---
theme: academic
title: "Laravel Filtering, Pagination & Advanced Eloquent"
info: |
  Query Scopes • Collections • Aggregates • Observers • Architecture
paginate: true
---

# Advanced Eloquent & Filtering (Week 6)
## State Management, Database Concurrency, and Scopes

---

## 1. Query Scopes (Encapsulation of Filtering)
To implement robust **Filtering**, developers use Query Scopes. Scopes abstract complex SQL constraints into reusable, readable methods directly on the model, enforcing the "Fat Model, Skinny Controller" architectural pattern.

* **Local Scopes:** Methods prefixed with `scope` (e.g., `scopePublished($query)`) that allow you to chain custom constraints fluidly on-demand (e.g., `Post::published()->get()`).
* **Global Scopes:** Constraints applied automatically to *all* queries for a given model. 
* **Soft Deletes (A Native Global Scope):** Exploring Laravel's native `SoftDeletes` trait, which intercepts `DELETE` operations and instead updates a `deleted_at` timestamp, automatically filtering out these records from future queries.

---

### Local Scope Implementation (Filtering Data)

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    // Local scope defined by prefixing the method with 'scope'
    public function scopePublished(Builder $query): Builder
    {
        return $query->where('status', 'Published')
                     ->whereNotNull('published_at');
    }
}

// In the Controller:
// Executing the scope fluidly alongside other filters
$posts = Post::published()->orderBy('created_at', 'desc')->get();
```

---

## 2. Pagination (Data Splitting)
As datasets grow, returning all rows becomes unsustainable. Pagination limits the number of records returned per query and automatically generates UI links for navigating pages.

* **Performance:** Reduces memory consumption and network payload.
* **Implementation:** Instead of calling `get()`, developers utilize `paginate()`.
* **Frontend Integration:** Laravel automates the generation of HTML pagination links directly in Blade templates.

---

### Implementing Eloquent Pagination

**The Controller Logic:**
```php
public function index()
{
    // Retrieves 15 published posts per page. 
    // Laravel automatically parses the `?page=X` query string.
    $posts = Post::published()->paginate(15);
    
    return view('posts.index', ['posts' => $posts]);
}
```

**The Blade Template Integration:**
```blade
<!-- Iterating over the constrained subset of data -->
@foreach ($posts as $post)
    <p>{{ $post->title }}</p>
@endforeach

<!-- Automatically generates Tailwind/Bootstrap pagination links -->
{{ $posts->links() }}
```

---

## 3. Collection Architecture & Manipulation
Whenever Eloquent retrieves multiple records, it does not return a basic PHP array. It returns an instance of `Illuminate\Database\Eloquent\Collection`.

* **Fluent Interface:** Allows chaining methods together to manipulate datasets without writing imperative `foreach` loops.
* **Method Richness:** Includes out-of-the-box methods for map-reduce paradigms (`pluck()`, `map()`, `filter()`, `groupBy()`).

```php
// 1. Pluck: Extract an array of exactly one specific column
$emails = $users->pluck('email'); 

// 2. Filter & Map: Cleanse and transform data structures structurally
$grouped = $users->filter(function ($user) {
    return $user->isActive();
})->map(function ($user) {
    return ['name' => $user->fullName, 'dept' => $user->department];
});
```

---

### Advanced Collection Patterns

**1. Higher Order Messages:**
Allows performing actions directly on collection items via PHP magic methods, producing highly readable code.
```php
// Standard loop versus Higher Order Message
$users->each(function ($user) { $user->markAsRead(); });
$users->each->markAsRead(); // Highly optimized syntax
```

**2. Memory Management (Lazy Collections):**
For processing massive datasets (like a 10GB CSV), standard collections cause RAM to overflow. `LazyCollection` solves this using PHP Generators.
```php
// Yields chunks from the database one by one, keeping memory usage flat
User::cursor()->filter(function ($user) {
    return $user->id > 500;
})->each->updateStatus();
```

---

## 4. Advanced Database Aggregates & Structuring
Modern applications require highly optimized database interactions beyond standard relationships.

* **Aggregate Loading (`withCount`):** Retrieves the total count of a relationship *without* loading the massive related dataset into memory. 
* **JSON Querying:** Relational databases natively parse JSON. Eloquent abstracts this via `whereJsonContains()`, bringing NoSQL flexibility to SQL rows.
* **UUIDs / ULIDs:** Using universally unique identifiers instead of auto-incrementing integers (`id`) to prevent URL-guessing attacks (e.g., `/orders/1` becomes `/orders/9b1deb4d...`).

---

### Implementation of Advanced Queries

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;

class Order extends Model
{
    // Replaces integer IDs with UUIDs automatically on creation
    use HasUuids; 
}

// In the Controller:
// Highly optimized: Only executes 2 queries, avoids N+1, 
// and only loads an aggregate count instead of raw comment data.
$posts = Post::withCount('comments')
             ->whereJsonContains('metadata->tags', 'urgent')
             ->get();

foreach ($posts as $post) {
    // Access the dynamically generated aggregate property
    echo "Post {$post->title} has {$post->comments_count} comments.";
}
```

---

## 5. Model Lifecycle Events & Observers
Eloquent models dispatch events during their lifecycle, allowing developers to hook into the state changes of the Active Record without polluting the controller logic.

* **Lifecycle Hooks:** Intercepting actions such as `creating`, `created`, `updating`, `updated`, `deleting`, and `deleted`.
* **Observers:** Dedicated classes that encapsulate all event listeners for a specific model. This cleanly segregates business side-effects from the core model design. For example, a `UserObserver` might automatically dispatch an email when the `created` event fires, or clear a cache when a model is `updated`.

---

### Implementing a Model Observer

```php
namespace App\Observers;

use App\Models\User;
use Illuminate\Support\Facades\Log;

class UserObserver
{
    // Executes AFTER a user is successfully inserted into the database
    public function created(User $user): void
    {
        // Example: Dispatching a welcome email or generating a default profile
        Log::info("New user registered: {$user->email}");
    }

    // Executes BEFORE a user's updates are persisted to the database
    public function updating(User $user): void
    {
        if ($user->isDirty('email')) {
            Log::warning("User {$user->id} is changing their email address.");
        }
    }
}
```
*Note: Observers are registered centrally in a Service Provider (e.g., `AppServiceProvider`).*

---

## 6. Data Transformation: Accessors and Mutators
Accessors and Mutators handle the transformation of data strictly as it enters or exits the application state.

* **Accessors:** Formatting database attributes when they are retrieved (e.g., combining `first_name` and `last_name` columns into a dynamic `full_name` property).
* **Mutators:** Altering data before it is saved to the database (e.g., ensuring a `password` attribute is automatically hashed using bcrypt whenever it is assigned).
* **Modern Syntax:** In Laravel 13, these transformations are defined using a single method that returns an `Illuminate\Database\Eloquent\Casts\Attribute` instance.

---

### Attribute Definition (Modern Syntax)

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Support\Facades\Hash;

class User extends Model
{
    // Accessor: A dynamically generated attribute that does not exist in the DB
    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn (mixed $value, array $attributes) => 
                 "{$attributes['first_name']} {$attributes['last_name']}",
        );
    }

    // Mutator: Ensuring passwords are never stored in plaintext
    protected function password(): Attribute
    {
        return Attribute::make(
            set: fn (string $value) => Hash::make($value),
        );
    }
}
```

---

## 7. Concurrency and Database Transactions
For university-level computer science students, addressing data integrity during concurrent HTTP requests is critical.

* **Database Transactions:** Using `DB::transaction()` ensures that multiple related database operations either succeed completely or fail gracefully (rolling back), preventing orphaned data.
* **Pessimistic Locking:** Using Eloquent methods like `sharedLock()` or `lockForUpdate()` to prevent other database transactions from modifying or reading a row until the current operation is complete (essential for financial, booking, or inventory systems).

---

### Ensuring Architectural Integrity

```php
use Illuminate\Support\Facades\DB;
use App\Models\Account;
use App\Models\Transaction;

// Using a closure-based transaction. If an exception is thrown, 
// changes are rolled back automatically.
DB::transaction(function () {
    
    // Applying a pessimistic lock to the account row.
    // Concurrent requests attempting to read/write this row will wait.
    $account = Account::where('id', 1)->lockForUpdate()->first();
    
    $account->balance -= 100;
    $account->save();
    
    Transaction::create([
        'account_id' => 1,
        'amount' => -100,
        'type' => 'withdrawal',
    ]);
});
```
