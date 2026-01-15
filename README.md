# Senior Laravel Coding Test - Artificially

Welcome to our technical assessment at **Artificially**. This test evaluates your proficiency in **Laravel** architecture, async processing, advanced validation, and testing practices.

Our website: [artificially.io](https://artificially.io)

---

## Objective

Build a **File Processing API** that handles file uploads with async processing, metadata extraction, and webhook notifications.

---

## Requirements

### 1. API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/files` | Upload a file for processing |
| `GET` | `/files` | List all files with filtering & pagination |
| `GET` | `/files/{id}` | Retrieve file details and processing status |
| `DELETE` | `/files/{id}` | Soft delete a file |

---

### 2. File Upload & Processing Flow

1. **Upload (`POST /files`)**:
   - Accept file + metadata (name, description, webhook_url)
   - Validate and store file immediately
   - Return `202 Accepted` with file ID and `status: pending`
   - Dispatch a job for async processing

2. **Processing Job**:
   - Extract file metadata (size, mime type, hash)
   - Simulate processing delay (2-3 seconds)
   - Update file status to `completed` or `failed`
   - Send POST request to `webhook_url` with results (if provided)

---

### 3. Advanced Validation

Implement the following validation rules on `POST /files`:

| Field | Rules |
|-------|-------|
| `file` | Required, max 10MB, allowed types: pdf, png, jpg, docx |
| `name` | Required, max 255 chars, alphanumeric + spaces only |
| `description` | Optional, max 1000 chars |
| `webhook_url` | Optional, must be valid HTTPS URL, must respond to HEAD request within 2 seconds |

**Custom Validation Rules Required:**
- Create a custom rule that validates `webhook_url` is reachable
- Create a custom rule that checks file content matches its extension (not just mime spoofing)

---

### 4. Architecture Requirements
```
app/
├── Http/
│   ├── Controllers/FileController.php
│   ├── Requests/StoreFileRequest.php
│   ├── Middleware/TokenAuthMiddleware.php
│   └── Resources/FileResource.php
├── Services/
│   ├── FileService.php
│   └── WebhookService.php
├── Jobs/
│   └── ProcessFileJob.php
├── Rules/
│   ├── ReachableUrl.php
│   └── ValidFileContent.php
└── Models/
    └── File.php
```

---

### 5. Database Schema

`files` table should include:
- `id`, `uuid`, `name`, `description`, `original_filename`, `path`, `mime_type`, `size`, `hash`, `status` (pending/processing/completed/failed), `webhook_url`, `processed_at`, `deleted_at`, `timestamps`

---

### 6. Middleware

- Protect `POST` and `DELETE` routes with token authentication
- Header: `Authorization: Bearer artificially-token`
- Return proper `401` response with JSON error message

---

### 7. Artisan Command

Create `php artisan files:retry-failed`
- Finds all files with `status: failed`
- Re-dispatches processing jobs for each
- Outputs count of retried files

---

### 8. Testing Requirements

Write **Feature** and **Unit** tests covering:

| Test Type | Coverage |
|-----------|----------|
| Feature | All endpoints with auth success/failure |
| Feature | Validation rejection scenarios |
| Feature | File upload triggers job dispatch |
| Unit | Custom validation rules |
| Unit | FileService methods |
| Unit | ProcessFileJob execution |

Use `Queue::fake()` and `Http::fake()` where appropriate.

---

## Setup Instructions
```bash
git clone git@github.com:ArtificiallyLTD/Laravel-Coding-Test.git
cd Laravel-Coding-Test

composer install
cp .env.example .env
php artisan key:generate

touch database/database.sqlite
php artisan migrate

# Queue setup (use sync for testing, database/redis for demo)
php artisan queue:table
php artisan migrate
```

---

## Submission

1. Push to a **private** GitHub repository
2. Invite `contact@artificially.io` as collaborator
3. Include updated README with any setup notes

---

## Evaluation Criteria

| Criteria | Weight |
|----------|--------|
| Architecture & code organization | 25% |
| Queue/job implementation | 20% |
| Validation (including custom rules) | 20% |
| Test coverage & quality | 20% |
| Error handling & edge cases | 15% |

---

## Time Expectation

This task is designed for **2-3 hours**. Focus on clean implementation over extra features.

Good luck!
