---
theme: academic
title: "Laravel Eloquent ORM & Relationships"
info: |
  Active Record Pattern • Relational Mapping • N+1 Optimization • Strict Mode
paginate: true
---

# Laravel Eloquent ORM (Week 5)
## Active Record Architecture, Data Mapping, and Performance

---

## 1. Understanding Object-Relational Mapping (ORM)
Before diving into Eloquent specifically, it is crucial to understand the purpose of an ORM. An ORM acts as a bridge between the **Relational Database** (tables, rows, foreign keys) and your **Application Code** (classes, objects, properties).

* **The Problem with Raw SQL:** Writing raw SQL inside PHP controllers mixes data querying with business logic, creating brittle code that is vulnerable to SQL injection and difficult to refactor.
* **The ORM Solution:** Developers interact with PHP objects instead of writing SQL strings. The ORM translates these object interactions into highly optimized SQL queries dynamically.
* **Database Agnosticism:** Because the ORM generates the SQL dialect, you can switch from MySQL to PostgreSQL simply by changing a configuration file, without rewriting application logic.

---

### Comparison: Raw SQL vs. ORM Syntax

**Raw PDO SQL Query:**
```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE status = :status ORDER BY created_at DESC');
$stmt->execute(['status' => 'active']);
$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

// Requires manual array traversal and mapping
foreach ($users as $userArray) {
    echo $userArray['name'];
}
```

**Eloquent ORM Equivalent:**
```php
// Clean, chainable, and inherently secure against SQL injection (PDO binding)
$users = User::where('status', 'active')->orderByDesc('created_at')->get();

foreach ($users as $userObject) {
    // $userObject is an instance of the User class, encapsulating behavior
    echo $userObject->name;
}
```

---

## 2. The Active Record Architecture
Laravel’s Eloquent implements the **Active Record** architectural pattern. 

* **Object-Table Mapping:** Each database table has a corresponding "Model" class used to interact with that table. Instances of the model represent individual rows.
* **Abstraction of SQL:** Eloquent abstracts complex SQL queries into a fluent, chainable object-oriented API, improving code readability.
* **Encapsulation of State:** Models encapsulate both the data (state) and the database access logic (behavior), allowing developers to define business rules directly on the data entities.

---

### Eloquent Mapping Example

By convention, Eloquent automatically maps the `Post` model to the `posts` table. 

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    // Custom business logic encapsulated within the model
    public function isPublished(): bool
    {
        return $this->status === 'Published';
    }
}
```

---

## 3. Core Model Configuration & Properties
While Eloquent utilizes "convention over configuration" (e.g., assuming a `Post` model maps to a `posts` table), developers must often explicitly configure the model's architectural properties.

* **Database Overrides:** 
  * `$table`: Explicitly define the table name if it deviates from convention.
  * `$primaryKey`: Override the default `id` column name.
  * `$keyType` & `$incrementing`: Essential when utilizing non-standard keys, such as UUIDs or ULIDs.

---

### Security and Mass Assignment
Mass Assignment vulnerabilities occur when a user passes an unexpected parameter (e.g., `is_admin => true`), and the application inadvertently persists it. Eloquent models mitigate this via explicit properties:

* **`$fillable` (An Allow-list):** An array of attributes explicitly permitted to be assigned en masse (e.g., via `Post::create($request->all())`). This is the standard, secure approach.
* **`$guarded` (A Block-list):** An array of attributes that *cannot* be mass-assigned. Setting `protected $guarded = [];` opens the model entirely, shifting the security burden strictly to the Form Request validation layer.

---

### Data Serialization and Casting
Models often act as the API boundary layer, requiring strict control over how data is exposed and typed.

* **Visibility (`$hidden` / `$visible`):** Prevents sensitive attributes (e.g., `password`, `remember_token`) from being exposed when the model is automatically serialized to JSON for API responses.
* **Type Casting (`$casts` property or `casts()` method):** Automatically transforms raw database values into robust PHP data types upon retrieval. For instance, instructing Eloquent to securely cast a `json` database column into a native PHP `array`, or mapping an integer to a strictly-typed PHP `Enum`.

---

## 4. Fundamental Relationship Typologies
Relational databases rely on interconnected tables. Eloquent defines these connections as descriptive methods on the model class.

* **One-to-One (`hasOne` / `belongsTo`):** A direct, singular relationship. Example: A `User` has one `Profile`.
* **One-to-Many (`hasMany` / `belongsTo`):** The most common relational pattern. Example: A `Blog` has many `Posts`. The `Post` model holds the foreign key (e.g., `blog_id`).
* **Many-to-Many (`belongsToMany`):** Requires an intermediate "pivot" table. Example: A `User` can have many `Roles`, and a `Role` can belong to many `Users`. The pivot table (e.g., `role_user`) connects them.

---

### Relationship Definition Examples

Defining a **One-to-Many** relationship establishes the bi-directional link between entities.

```php
// In the Blog Model
public function posts()
{
    // A Blog has many Posts
    return $this->hasMany(Post::class);
}

// In the Post Model
public function blog()
{
    // The Post holds the blog_id foreign key
    return $this->belongsTo(Blog::class);
}
```

---

## 5. Advanced Relational Mapping
For complex enterprise applications, standard relationships are often insufficient. Eloquent provides advanced mapping patterns:

* **Has-Many-Through:** Defines a distant relationship via an intermediate model. Example: A `Country` has many `Posts` *through* its `Users`.
* **Polymorphic Relationships:** Allows a model to belong to more than one type of model on a single association. 
* **Polymorphic Example:** A `Comment` model could belong to either a `Post` model or a `Video` model without requiring separate `post_id` and `video_id` columns. It uses `commentable_id` and `commentable_type` instead.

---

## Structural Comparison: Standard vs. Polymorphic

| Feature | Standard (One-to-Many) | Polymorphic (One-to-Many) |
| :--- | :--- | :--- |
| **Foreign Keys** | Dedicated column (e.g., `post_id`) | Dual columns (`_id` and `_type`) |
| **Reusability** | Bound to a single parent model | Can attach to infinite parent models |
| **Database Integrity** | Supports strict foreign key constraints | Cannot use database-level foreign keys |

---

### Polymorphic Implementation Example

```php
// Implementation of the Polymorphic 'commentable' relationship
class Comment extends Model
{
    public function commentable()
    {
        return $this->morphTo();
    }
}

class Video extends Model
{
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

---

# Performance Optimization in Eloquent
## Diagnosing and Resolving the N+1 Query Problem

---

## 6. The Anti-Pattern: Lazy Loading (The N+1 Problem)
By default, Eloquent utilizes "Lazy Loading." Related models are not retrieved from the database until their property is explicitly accessed in the code.

While convenient, iterating over a collection of models and accessing a relationship triggers a new database query for *every single iteration*.

**The Vulnerable Code:**
```php
// Executes 1 query: SELECT * FROM posts;
$posts = Post::all();

foreach ($posts as $post) {
    // Executes N queries (one for each post): 
    // SELECT * FROM users WHERE id = ?
    echo $post->author->name; 
}
```
*Architectural Impact:* Retrieving 50 posts results in 51 total database queries (1 to fetch posts + 50 to fetch authors). This causes severe latency and database strain.

---

## 7. The Solution: Eager Loading
To resolve this, developers must implement "Eager Loading" using the `with()` method. This instructs the ORM to fetch the parent models and all specified related models in a highly optimized manner before the iteration begins.

**The Optimized Code:**
```php
// Executes exactly 2 queries regardless of record count:
// 1. SELECT * FROM posts;
// 2. SELECT * FROM users WHERE id IN (1, 2, 3, 4...);
$posts = Post::with('author')->get();

foreach ($posts as $post) {
    // No database hit occurs here. The data is already in memory.
    echo $post->author->name; 
}
```
*Architectural Impact:* Database executions are reduced from *N+1* down to exactly *2*, drastically minimizing network overhead and improving execution time.

---

## 8. Nested and Conditional Eager Loading
Enterprise applications often require deeper relational extraction or filtered loading.

**Loading Multiple/Nested Relationships:**
```php
// Loads the author, the post's comments, and the author of each comment
$posts = Post::with(['author', 'comments.author'])->get();
```

**Constraining Eager Loads:**
```php
// Only eagerly loads comments that are approved
$posts = Post::with(['comments' => function ($query) {
    $query->where('is_approved', true);
}])->get();
```

---

## 9. Architectural Prevention: Strict Mode
In modern Laravel architectures, developers can completely disable Lazy Loading in local environments to catch N+1 vulnerabilities before they reach production.

**Implementation (Typically in `AppServiceProvider`):**
```php
use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    // Throws an exception if lazy loading is attempted, 
    // forcing the developer to use eager loading.
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

---

## 10. Eloquent Caveats: Eager Loading vs. SQL JOINs
A critical misconception is that Eager Loading (`with()`) executes an SQL `JOIN`. It does not. It relies on executing consecutive queries and mapping them in memory.

* **The Execution:** Calling `Post::with('author')->get()` executes:
  1. `SELECT * FROM posts;`
  2. `SELECT * FROM users WHERE id IN (1, 2, 3...);`
* **The Caveat (Filtering & Sorting):** Because there is no raw `JOIN`, you **cannot** use an eager-loaded relationship to filter or sort the DB query directly. Attempting `$query->orderBy('users.name')` will throw an SQL syntax error.
* **The Solution:** If you must filter or order results based on a related table's columns (e.g., ordering posts by the author's name), you *must* write an explicit `join()` statement.

---

### Handling the Eager Loading Limitation

**Failed Implementation (Does NOT work):**
```php
// Will crash: The 'users' table is not actually joined during this query!
$posts = Post::with('author')->orderBy('users.last_name')->get();
```

**Correct Implementation (Using a true JOIN):**
```php
// Explicitly joining the tables allows cross-table sorting
$posts = Post::select('posts.*')
    ->join('users', 'posts.author_id', '=', 'users.id')
    ->orderBy('users.last_name')
    ->with('author') // Still eager-load the relation so $post->author works
    ->get(); 
```

---

## 11. Essential Developer Tooling for Eloquent
Because Eloquent relies heavily on PHP "magic methods" (`__get`, `__call`) and abstracts raw SQL into the background, developers rely on essential ecosystem packages to maintain visibility and strict typing.

### 1. Laravel Debugbar (`barryvdh/laravel-debugbar`)
The industry standard for catching database performance bottlenecks specifically during the development phase.
* **Functionality:** Injects a toolbar into the browser displaying the raw SQL strings and execution times for every query in the current HTTP lifecycle.
* **Academic Value:** It visually provides students instant feedback on the N+1 problem, showing 50 duplicate database hits turn into exactly 2 hits after implementing Eager Loading.

### 2. Laravel IDE Helper (`barryvdh/laravel-ide-helper`)
Because Eloquent model attributes are inferred dynamically from the physical database schema, code editors (VS Code / PhpStorm) often fail to provide proper autocomplete capabilities.
* **Functionality:** Connects to the database and generates a meta-file containing PHP DocBlocks for all models.
* **Academic Value:** Grants students instant, type-aware autocomplete for dynamic database columns, relationships, and Query Scopes directly in the editor, vastly reducing typos.

### 3. Laravel Telescope
The official, first-party, comprehensive debug assistant for Laravel.
* **Functionality:** Provides a beautifully designed dedicated GUI dashboard (e.g., `/telescope`) that deeply tracks executed database queries, dispatched Model Events, hit Observers, and overall application state.
