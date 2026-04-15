---
theme: academic
title: "Automated Testing & TDD"
info: |
  Pest PHP • Feature vs Unit • AAA Pattern • Assertions
paginate: true
---

# Automated Testing (Week 12)
## Guaranteeing Reliability and Preventing Regressions

---

## 1. The Philosophy of Testing
As your application grows, the risk of a **Regression** increases exponentially. A regression occurs when writing new code inadvertently breaks previously functioning code elsewhere in the system.

* **Manual Testing:** Clicking through the UI or firing Bruno requests manually. Extremely slow, highly prone to human error, and impossible to scale.
* **Automated Testing:** Writing dedicated PHP code strictly to test your production PHP code. The tests run through the entire system in milliseconds, acting as a living safety net.

---

## 2. Test-Driven Development (TDD)
TDD is the enterprise methodology of writing the failing test *before* you write the production code. It dictates a strict three-step loop:

1. **🔴 Red:** Write a test for a feature that doesn't exist yet. The test will immediately fail.
2. **🟢 Green:** Write the absolute minimum, ugliest production code required to satisfy the test and make it pass.
3. **✨ Refactor:** Clean up, optimize, and organize the production code. You can refactor fearlessly because your passing test will instantly warn you if you break the underlying logic!

---

## 3. The Testing Framework (Pest PHP)
Historically, PHP developers utilized the **PHPUnit** framework, which relied on verbose, heavy Object-Oriented class structures. Modern Laravel defaults to **Pest PHP**.

* **What is Pest?** A highly elegant testing framework focusing on beautiful, human-readable closures. 
* **The Command:** Automatically execute your entire test suite via `php artisan test`.

**Example Pest Syntax:**
```php
// Tests are written in a way that reads exactly like an English sentence!
test('it calculates the shipping cost accurately', function () {
    $cost = calculateShipping(100);

    // The Expectation API natively reads like human language
    expect($cost)->toBe(10.50);
});
```

---

## 4. The `AAA` Pattern
To maintain readable and consistent testing architectures, engineers universally follow the **Arrange, Act, Assert (AAA)** pattern mathematically.

1. **Arrange:** Set up the "world". Create necessary users, set database flags, or prepare specific payloads.
2. **Act:** Execute the specific snippet of code, or fire the HTTP request against the endpoint.
3. **Assert:** Mathematically verify that the resulting outcome explicitly changed the state exactly as expected.

*Architectural Rule:* Tests should ideally contain exactly one "Act" step. If you are acting multiple times, you should split your code into multiple separate tests!

---

## 5. The Test Environment & State Mutability
**You must NEVER run automated tests against your production database.** Tests actively destroy and recreate data!

Laravel natively prevents this catastrophic error seamlessly:
* **The Config:** When `php artisan test` is executed, Laravel automatically ignores your standard `.env` and uses the `.env.testing` file (often pointing to a temporary, in-memory SQLite database).

**The State Mutability Problem:**
If "Test A" creates a user with `user@mail.com`, and "Test B" tries to register the same email, it throws a duplicate database error. 

**The Solution (`RefreshDatabase`):**
Laravel natively wraps every single test in a Database Transaction. The instant a specific test finishes, Laravel rolls back the transaction. The database is restored to perfect pristine condition before the next test begins!

---

## 6. Feature Tests vs. Unit Tests
Enterprise applications rely on a mixture of rigorous broad coverage and hyper-fast specific coverage.

* **Unit Tests (`tests/Unit/`)**
  * **Scope:** Tests one specific method of one specific class in absolute isolation.
  * **Constraints:** *Banned* from hitting the database, filesystem, or making network requests.
  * **Speed:** Executes in microseconds.

* **Feature Tests (`tests/Feature/`)**
  * **Scope:** Tests a massive "slice" of the entire application top-to-bottom.
  * **Execution:** Simulates an HTTP Request passing through the Router → Middleware → Controller → Database → and returning the JSON result.
  * **Value:** Highly confident tests that prove the entire machine works correctly.

---

## 7. Model Factories: Demystifying the Magic
To effectively test the application, you need data. Manually typing out 50 lines of fake user parameters for every single test is exhausting.

**The Solution:** Laravel Model Factories (`database/factories/UserFactory.php`).
Factories utilize the internal `Faker` library to instantly generate randomized, highly realistic dummy data (names, secure passwords, valid emails) dynamically.

```php
// Instantly generates a perfect User record and writes it to the Test DB
$author = User::factory()->create();

// Generates an array of raw dummy data mimicking a legitimate Form Request payload
$payload = Post::factory()->raw(); 
```

---

## 8. Feature Testing: The Happy Path
Let's test the endpoint we built in Week 9: Creating a Post via `.postJson()`. The "Happy Path" verifies what happens when the user does everything correctly.

```php
use App\Models\User;
use function Pest\Laravel\postJson;
use function Pest\Laravel\actingAs;

test('a verified user can publish a new post', function () {
    // 1. Arrange: Boot a fake user via the Factory
    $user = User::factory()->create();

    // 2. Act: Authenticate the user and blindly fire the HTTP request
    $response = actingAs($user)
        ->postJson('/api/posts', [
            'title' => 'My First Automated Test',
            'body'  => 'Pest makes testing incredibly simple.',
        ]);

    // 3. Assert: Verify the endpoint gracefully returned HTTP 201 Created
    $response->assertStatus(201);
});
```

---

## 9. Feature Testing: The Sad Path (Negative Testing)
In enterprise software, 80% of testing is ensuring the system **fails gracefully**. We must test our Validation rules!

```php
test('the api rejects post creations without a title validation', function () {
    $user = User::factory()->create();

    // ACT: Intentionally send a payload that is completely missing the required 'title'
    $response = actingAs($user)
        ->postJson('/api/posts', [
            'body' => 'Missing title payload',
        ]);

    // ASSERT: Verify the API blocked the route and returned a strict 422 Error
    $response->assertStatus(422)
             ->assertJsonValidationErrors(['title']); 
});
```

---

## 10. Database Assertions
Checking the HTTP `201` status is not enough. We must strictly verify that the Controller explicitly persisted the data correctly into the SQL database schema. 

Laravel's Testing suite provides powerful native database assertions natively out of the box.

```php
test('a verified user can publish a new post', function () {
    // ... [Arrange and Act Steps] ...

    // Assert the HTTP layer successfully processed the request
    $response->assertStatus(201);
    
    // Assert the Database layer explicitly holds the exact payload
    $this->assertDatabaseHas('posts', [
        'user_id' => $user->id,
        'title'   => 'My First Automated Test',
    ]);
});
```

---

## 11. Introduction to Mocking
What happens if your Controller charges a customer's credit card by communicating with the `Stripe` API? **You cannot fire a real credit card charge during an automated test!**

* **Mocking:** The architectural practice of swapping out a real service with a fake "dummy" service specifically during the test run.

```php
use Illuminate\Support\Facades\Http;

test('it processes the refund using the third-party api', function () {
    // Instruct Laravel to intercept any outgoing HTTP requests and Fake a 200 OK
    Http::fake([
        'api.stripe.com/*' => Http::response(['status' => 'refunded'], 200)
    ]);

    // Act: Your controller will "talk" to the fake API and succeed instantly!
    $response = postJson('/api/refunds', ['order_id' => 123]);

    $response->assertStatus(200);
});
```

---

## 12. The Enterprise End-Game: Continuous Integration (CI)
Why do we spend 40% of our coding time writing tests? To power **DevOps**.

If you are working on a team of 10 backend engineers, humans will inevitably make mistakes. Enterprise architectures solve this using **Continuous Integration (CI) Pipelines** (e.g., GitHub Actions, GitLab CI).

**The Golden Rule of CI:**
* When a developer submits a Pull Request, the code *cannot* be merged into the `main` branch. 
* First, a remote cloud server automatically boots up, installs Laravel, and runs `php artisan test`.
* If even a single test fails, the code is firmly rejected. The pipeline guarantees that a broken feature can theoretically *never* reach the production server!
