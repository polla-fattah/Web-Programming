
---
theme: default
title: "Week 1 – PHP Refresher & Backend Foundations"
info: |
  Backend Web Programming (PHP + Laravel)
  Week 1: Session 1 + Session 2
paginate: true
---

## PHP Foundations

**Goal:** Build the backend mindset + quick PHP readiness for Laravel

---

## Agenda
- Course framing + expectations
- Assessment + milestones
- Backend mental model (HTTP, sessions)
- Toolchain setup plan
- PHP quick taste
- Mini demo + in-class task
- Ensure everyone is set up
- PHP functions + input handling
- Backend pattern: Request → Validate → Service → Response
- OOP basics that matter for Laravel

---

## Course Framing
- How the course runs:
  - **Lecture** (concepts)
  - **Lab** (build)
  - **Project** (incremental)
- What “backend” means here:
  - Server logic
  - Database
  - Auth/roles
  - APIs
  - Deployment


---

## Backend Mental Model
**Client ↔ Server ↔ Database**
- Browser / Postman sends request
- Server runs code + talks to DB
- Server returns response (HTML/JSON)

---

## Request → Response

```mermaid
sequenceDiagram
  participant C as Client
  participant S as Server (PHP/Laravel)
  participant D as Database
  C->>S: HTTP Request (GET/POST/...)
  S->>D: Query / Transaction
  D-->>S: Result
  S-->>C: HTTP Response (HTML/JSON)
````

---

## HTTP Basics (What to Know)

Methods:

* **GET** (read)
* **POST** (create)
* **PUT/PATCH** (update)
* **DELETE** (remove)

Status codes (common):

* 200 OK, 201 Created
* 400 Bad Request
* 401 Unauthorized, 403 Forbidden
* 404 Not Found
* 422 Validation Error
* 500 Server Error

---

## Cookies vs Sessions (High-level)

* **Cookie:** stored in browser
* **Session:** stored on server
* Typically: cookie holds **session id**
* Laravel uses sessions heavily for web auth

---

## Toolchain (This Week)

Install:

* PHP 8.x
* Composer
* MySQL or PostgreSQL
* Git
* VS Code
* Postman (or Insomnia)

---

## Verify Commands

```bash
php -v
composer -V
git --version
```

DB running:

* MySQL: service running / can connect
* PostgreSQL: service running / can connect

---

## PHP Quick Taste

See PHP “style” (no deep dive yet):

* Variables + constants
* Arrays
* Functions + type hints
* Tiny OOP snippet

---

## Variables

```php
$name = "Ali";
$age  = 21;
$isActive = true;
$note = null;
```

Tip:

* `$` is mandatory for variables

---

## Constants

```php
const APP_ENV = "local";
define("APP_NAME", "BackendCourse");
```

Class constant:

```php
class User {
  public const ROLE_ADMIN = "admin";
}
```

---

## Arrays (Very Common in PHP)

Indexed:

```php
$colors = ["red", "green"];
```

Associative:

```php
$user = ["name" => "Sara", "age" => 22];
echo $user["name"];
```

---

## Function + Types (PHP 8+)

```php
function add(int $a, int $b): int {
  return $a + $b;
}
```

Null helpers:

```php
$email = $input["email"] ?? null;
```

---

## Tiny OOP Example

```php
class Student {
  public function __construct(
    public string $name,
    public int $age
  ) {}
}

$s = new Student("Aso", 20);
```

---

## Mini Demo (Backend-style Output)

Idea:

* Input array
* Validate
* Return JSON response

Response contract:

* `{ ok: true, data: ... }`
* `{ ok: false, errors: [...] }`

---

## Demo Snippet

```php
$input = ["name" => "Sara", "age" => 17];
$errors = [];

if (!isset($input["name"]) || trim($input["name"]) === "") $errors[] = "name required";
if (!isset($input["age"]) || $input["age"] < 18)          $errors[] = "age must be >= 18";

if ($errors) {
  echo json_encode(["ok" => false, "errors" => $errors]);
} else {
  echo json_encode(["ok" => true, "data" => $input]);
}
```


---

## Functions + Input Handling

Key tools:

* `isset()`, `empty()`, `??`
* `trim()`, `strlen()`
* `foreach`
* `array_map`, `array_filter`

---

## Null / Default Patterns

```php
$name  = $input["name"]  ?? "";
$email = $input["email"] ?? null;

if (trim($name) === "") { /* error */ }
```

---

## Array Helpers (Quick Taste)

```php
$numbers = [1,2,3];

$squares = array_map(fn($x) => $x*$x, $numbers);
$even    = array_filter($numbers, fn($x) => $x % 2 === 0);
```

---

## Core Backend Pattern

We will reuse this all semester:

**Request → Validate → Service → Response**

* Request: get input
* Validate: rules + errors
* Service: business logic
* Response: JSON / view

---

## Pattern Sketch (Plain PHP)

```txt
index.php
  ├─ Request (input array)
  ├─ Validator (errors)
  ├─ Service (store/register)
  └─ Response (json)
```

---

## PHP OOP

* class + constructor
* properties + methods
* visibility: public/protected/private
* namespaces (idea)
* “Service class” style

---

## Service Class Example

```php
class StudentService {
  private array $students = [];

  public function register(array $data): array {
    $this->students[] = $data;
    return $data;
  }
}
```

---

## In-class Lab

**Student Registration Validator + Storage (in memory)**

Input fields:

* name (required)
* email (required, contains `@`)
* age (required, >= 18)
* department (required)

Output:

* list of stored students
* errors (if invalid)

---

## Suggested Data Shape

```php
$student = [
  "name" => "Sara",
  "email" => "sara@example.com",
  "age" => 21,
  "department" => "CS"
];
```
