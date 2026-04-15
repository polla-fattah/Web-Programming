---
theme: academic
title: "File Uploads & System Logging"
info: |
  Storage Facade • Secure Uploads • Error Handling • Logging
paginate: true
---

# File Operations & System Health (Week 11)
## Binaries, Exception Handling, and Logging

---

## 1. The File Storage Architecture
Modern backend applications frequently interact with physical binary files (e.g., PDFs, User Avatars). Laravel provides an elegant abstracted layer called the **Storage Facade** (`Illuminate\Support\Facades\Storage`).

The filesystem architecture (`config/filesystems.php`) relies on "Disks".
* **`local` Disk:** Files strictly hidden from the public web. Used for sensitive data (e.g., generated system invoices, backups).
* **`public` Disk:** Files intended for public browser consumption (e.g., profile pictures, blog graphics).
* **`s3` Disk:** Cloud storage via Amazon Web Services.

*Architectural Benefit:* Because of abstraction, you can change your entire system from storing files locally to storing them on Amazon S3 without rewriting any of your controller code!

---

## 2. The Symlink Hurdle
By default, the `public` disk stores files in `storage/app/public`. However, web browsers (and therefore frontend applications) are strictly prohibited from reading files inside the `storage` directory for security reasons.

To make these files accessible, we must bridge the secure folder to the public web folder using a **Symbolic Link** (a shortcut).

**The Native Command:**
```bash
php artisan storage:link
```
* **What it does:** Creates a shortcut at `public/storage` that physically points to `storage/app/public`. A mobile application can now access `http://localhost:8000/storage/avatar.png` seamlessly!

---

## 3. Secure File Retrieval (Private Local Disk)
While symlinks are perfect for public graphics, sensitive files (e.g., medical records, billing invoices) **must never** be symlinked to the public web.

* **The Concept:** Sensitive files remain on the strict `local` disk. The only way a user can download it is requesting it through an authenticated Route/Controller gatekeeper.

```php
public function downloadInvoice($id)
{
    $invoice = Invoice::findOrFail($id);
    
    // Evaluate the Authorization Policy (Week 8)
    $this->authorize('view', $invoice); 

    // Returns a secure binary stream directly to the browser
    return Storage::download('private/invoices/' . $invoice->file_path);
}
```

---

## 4. Securing File Uploads (Validation)
You must **never** trust user-uploaded binaries. If an attacker uploads a malicious `.php` executable disguised as an image, it could compromise the entire server.

**Validating Binaries via Form Request:**
```php
public function rules(): array
{
    return [
        // 'file' ensures a successful HTTP upload occurred.
        // 'image' forces it to be a valid graphic (jpeg, png, bmp, gif, or svg).
        // 'mimes' explicitly restricts allowable formats.
        // 'max:2048' restricts the upload to 2 Megabytes (2048 KB).
        
        'avatar' => 'required|file|image|mimes:jpeg,png|max:2048',
    ];
}
```

**Environment Constraints (`php.ini`):**
Regardless of Laravel's `max` validation rule, your physical server configuration has the final say. If the `upload_max_filesize` or `post_max_size` directives in your server's `php.ini` file are limited to `2M`, uploading a 5MB file will instantly kill the connection before PHP even boots!

---

## 5. API Upload Logic (The Full File Lifecycle)
Actually storing the binary file to the server takes only a single line of code. However, managing the full lifecycle (Deletion) is critical to prevent "Storage Bloat" from orphaned files.

**In the Controller:**
```php
public function store(UploadAvatarRequest $request)
{
    $user = $request->user();

    // 1. Delete the old orphaned binary from the disk FIRST
    if ($user->avatar_path) {
        Storage::disk('public')->delete($user->avatar_path);
    }
    
    // 2. The `store` method generates a unique UUID filename and saves it
    $path = $request->file('avatar')->store('avatars', 'public');

    // 3. Save the new path to the database record
    $user->update(['avatar_path' => $path]);

    return response()->json(['url' => url(Storage::url($path))], 201);
}
```

---

## 6. Defensive Programming & Synergy
Interacting with the File System or DB is unpredictable. Amazon S3 could timeout mid-upload.

**Global Exception Handler vs. Local `try/catch`:**
* **Global Handling:** Do NOT wrap every controller in `try/catch`. Laravel natively forwards unhandled exceptions to a Global Handler (in `bootstrap/app.php`), automatically converting them into clean HTTP 500 JSON responses.
* **Local Context:** Only use `try/catch` when you must execute specific **recovery logic**.

**Synergy: Database Transactions & File Uploads:**
```php
try {
    DB::beginTransaction(); // Week 5: Start DB Transaction
    $path = $request->file('document')->store('docs', 's3');
    $model->update(['document' => $path]);
    DB::commit();
} catch (Exception $e) {
    DB::rollBack();
    
    // If the DB crashes, we must REVERT the physical upload!
    if (isset($path)) Storage::disk('s3')->delete($path);
    
    throw $e; // Forward logic up to the Global Handler
}
```

---

## 7. The `Log` Facade (System Health)
When exceptions do occur, their technical reason (e.g., "S3 Connection Timeout") must be tracked utilizing **Log Channels** (`config/logging.php`).

**Log Rotation (`daily`):**
Never use the default `single` log driver in production architecture; it will eventually consume 100% of the server's disk space. Instead, configure the `daily` driver to automatically rotate logs every night (e.g., `laravel-2026-04-16.log`) and auto-delete logs older than a threshold (e.g., 14 days).

**Filing Technical Telemetry:**
```php
use Illuminate\Support\Facades\Log;

try {
    // Risky code
} catch (Exception $e) {
    // Write the exact hidden technical stack trace to the local server log!
    Log::error('Upload Failed: ' . $e->getMessage(), ['user_id' => auth()->id()]);
}
```

---

### Beyond Local Logging (Enterprise Architecture)
While reading `storage/logs/laravel.log` via SSH is acceptable for local development, it does not scale to enterprise.

If you have 5 server instances running simultaneously, you cannot physically read 5 different local `.log` files. Modern engineering teams rely on **Third-Party Exception Trackers** like **Sentry**.

**1. Installation:**
```bash
composer require sentry/sentry-laravel
php artisan sentry:publish
```

**2. Configuration (`.env`):**
```env
# The DSN securely connects your application to your specific Sentry project dashboard
SENTRY_LARAVEL_DSN="https://examplePublicKey@o0.ingest.sentry.io/0"
```

**The Benefit:** Once installed, Laravel implicitly integrates with the tracker. Any time `Log::error()` is called or an application crashes, an HTTP request is automatically fired to Sentry's dashboard. The engineering team is instantly alerted via Slack/Email the second an API exception occurs in production!
