
---
theme: default
title: "Week 2 – PHP OOP for Laravel"
info: |
  Backend Web Programming (PHP + Laravel)
  Week 2: Object-Oriented PHP Standards
paginate: true
---

# PHP OOP for Laravel

**Goal:** Master modern PHP 8.x OOP features required to understand Laravel's architecture.

---


## Agenda
- Why OOP matters for Laravel
- Classes, Properties, Methods
- **Type Hinting** & Return Types
- **Access Modifiers** (Visibility)
- **Constructor Promotion** (PHP 8)
- Inheritance & Abstract Classes
- **Interfaces** (Contracts)
- **Namespaces** & `use`
- **Traits** (Mixins)
- **Composer** & Autoloading (PSR-4)

---

## Why OOP?
Laravel is heavily object-oriented. To use it, you must understand:
- How **Classes** encapsulate logic (Controllers, Models).
- How **Interfaces** allow swapping implementations (Storage, Mail).
- How **Traits** add superpowers to classes (Auth, Notifications).
- How **Namespaces** organize thousands of files.

---

## 1. Classes & Objects

The blueprint and the instance.

```php
class User {
    public $name;

    public function introduce() {
        // $this refers to the current instance
        return "Hi, I am " . $this->name;
    }
}

$u = new User();
$u->name = "Ali";
echo $u->introduce(); // "Hi, I am Ali"
```

---

## 2. Type Hinting & Return Types

PHP is dynamically typed with optional type declarations. Strict behavior requires `declare(strict_types=1);`.

```php
<?php
declare(strict_types=1);
```

```php
class Calculator {
    // Parameter types and Return type
    public function add(int $a, int $b): int {
        return $a + $b;
    }

    // Nullable type (?string)
    public function find(?string $query): ?array {
        if (!$query) return null;
        return ["result" => $query];
    }
}
```

*Common types:* `int`, `string`, `bool`, `array`, `float`, `object`, `void`, `mixed`.

---

## 3. Access Modifiers (Visibility)

Control who can touch your data.

- **`public`**: Accessible from anywhere (default).
- **`protected`**: Accessible only within the class and child classes.
- **`private`**: Accessible ONLY within the class itself.

```php
class BankAccount {
    private float $balance = 0; // Hidden from outside

    public function deposit(float $amount): void {
        if ($amount > 0) $this->balance += $amount;
    }
}
```

---

## 4. Constructor Property Promotion (PHP 8.x)

*Crucial for Laravel Controllers and DTOs.*

**Old Way:**
```php
class User {
    public string $name;
    public function __construct(string $name) {
        $this->name = $name;
    }
}
```

**New Way (shorter):**
```php
class User {
    public function __construct(
        public string $name,
        public string $email,
        private int $age = 18, // Can have defaults too
    ) {}
}
```

---

## 5. Inheritance & Abstract Classes

Reusing logic. `Child` is a `Parent`.

```php
abstract class Animal {
    public function eat() { echo "Eating..."; }
    abstract public function sound(): string;
}

class Cat extends Animal {
    public function meow() { echo "Meow!"; }
    public function sound(): string { return "Meow!"; }
}

$c = new Cat();
$c->eat(); // Inherited
$c->meow();
echo $c->sound();
```

*Key:* Use `parent::__construct()` if you override the constructor.

---

## 6. Interfaces (Contracts)

Defines *what* a class must do, not *how*.
Laravel uses this everywhere (e.g., `ShouldQueue`, `Arrayable`).

```php
interface PaymentGateway {
    public function charge(float $amount): bool;
}

class Stripe implements PaymentGateway {
    public function charge(float $amount): bool {
        // Stripe specific logic
        return true;
    }
}
```

---

## 7. Namespaces

Solves the "Name Collision" problem. Like folders for code.

File: `src/Models/User.php`
```php
namespace App\Models;

class User { /* ... */ }
```

File: `index.php`
```php
use App\Models\User; // Import it

$u = new User();
```

*Without namespaces, you can't have two classes named `User` in your app.*

---

## 8. Traits

Horizontal code reuse. "Copy-paste" methods into classes.

```php
trait Loggable {
    public function log(string $msg) {
        echo "[LOG]: $msg";
    }
}

class User {
    use Loggable;
}

class Product {
    use Loggable;
}

(new User())->log("Created");
```
*Laravel Example:* `use HasFactory, Notifiable, HasApiTokens;`

---

## 9. Composer & Autoloading

We use **Composer** for dependency management and autoloading instead of manually requiring each class file.

1. **`composer init`**: Creates `composer.json`.
2. **PSR-4 Autoloading**: Maps namespaces to folders.

`composer.json` snippet:
```json
"autoload": {
    "psr-4": {
        "App\\": "src/"
    }
}
```
3. `composer dump-autoload`
4. In your code: `require "vendor/autoload.php"`.

---

## Practical Lab: "RPG Game System"

**Goal:** Build a small class hierarchy using namespaces and autoloading.

**Requirements:**
1. Folder structure: `src/Characters`, `src/Weapons`, `index.php`.
2. Namespace: `App\Characters`, `App\Weapons`.
3. Base Class `Character` (name, health).
4. Subclasses `Warrior` (high health), `Mage` (low health).
5. Interface `WeaponInterface` (method `attack()`).
6. Classes `Sword`, `Staff` implement `WeaponInterface`.
7. **Trait** `HasInventory` (array of items) with an `equip()` method.

---

## Lab Steps

1. `composer init` (accept defaults).
2. Edit `composer.json` to map `"App\\": "src/"`.
3. `composer dump-autoload`.
4. Create classes.
5. In `index.php`:
   ```php
   require __DIR__ . '/vendor/autoload.php';
   use App\Characters\Warrior;
   use App\Weapons\Sword;

   $aragorn = new Warrior("Aragorn");
   $aragorn->equip(new Sword());
   ```

---

## Summary

- **Classes/Objects**: The building blocks.
- **Typed Properties**: Safety first.
- **Namespaces**: Organization.
- **Composer**: Dependency & Autoloading manager.

**Next Week:** We start **Laravel**. Installing, Routing, and the "M-V-C" pattern.
