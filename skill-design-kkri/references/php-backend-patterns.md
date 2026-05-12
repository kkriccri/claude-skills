# PHP Backend Patterns â KKRI Style

Templates for the server-side architecture: config, entity definitions, generic CRUD, auth.

## Table of Contents
1. [config.php â Database & Helpers](#config)
2. [entities.php â Entity Definitions](#entities)
3. [crud.php â Generic CRUD Handler](#crud)
4. [auth.php â Authentication](#auth)
5. [Specialized Endpoints](#specialized)
6. [.htaccess â Security](#htaccess)
7. [SQL Table Creation](#sql)

---

## config.php

The foundation: database connection, session management, CSRF protection, and helper functions.

```php
<?php
// ============ DATABASE CONFIG ============
define('DB_HOST', 'localhost');
define('DB_NAME', 'your_database');
define('DB_USER', 'your_user');
define('DB_PASS', 'your_password');
define('DB_CHARSET', 'utf8mb4');

// ============ APP CONFIG ============
define('APP_NAME', 'My App');
define('APP_VERSION', '1.0');
define('SESSION_LIFETIME', 3600 * 8); // 8 hours
define('UPLOAD_DIR', __DIR__ . '/../uploads/');
define('UPLOAD_MAX_SIZE', 10 * 1024 * 1024); // 10 MB
define('UPLOAD_ALLOWED_TYPES', [
    'application/pdf', 'image/jpeg', 'image/png', 'image/gif',
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    'application/vnd.ms-excel', 'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
]);

// ============ SESSION ============
if (session_status() === PHP_SESSION_NONE) {
    ini_set('session.cookie_httponly', 1);
    ini_set('session.cookie_secure', 1);
    ini_set('session.use_strict_mode', 1);
    ini_set('session.gc_maxlifetime', SESSION_LIFETIME);
    session_set_cookie_params([
        'lifetime' => SESSION_LIFETIME,
        'path' => '/your-app/',
        'secure' => true,
        'httponly' => true,
        'samesite' => 'Strict'
    ]);
    session_start();
}

// ============ DATABASE CONNECTION (PDO Singleton) ============
function getDB() {
    static $pdo = null;
    if ($pdo === null) {
        try {
            $dsn = 'mysql:host=' . DB_HOST . ';dbname=' . DB_NAME . ';charset=' . DB_CHARSET;
            $pdo = new PDO($dsn, DB_USER, DB_PASS, [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                PDO::ATTR_EMULATE_PREPARES => false
            ]);
        } catch (PDOException $e) {
            http_response_code(500);
            echo json_encode(['error' => 'Database connection error']);
            exit;
        }
    }
    return $pdo;
}

// ============ HELPERS ============
function jsonResponse($data, $code = 200) {
    http_response_code($code);
    header('Content-Type: application/json; charset=utf-8');
    echo json_encode($data, JSON_UNESCAPED_UNICODE);
    exit;
}

function requireAuth() {
    if (empty($_SESSION['user_id'])) {
        jsonResponse(['error' => 'Not authenticated'], 401);
    }
}

function requireAdmin() {
    requireAuth();
    if ($_SESSION['user_role'] !== 'admin') {
        jsonResponse(['error' => 'Access denied'], 403);
    }
}

function getInput() {
    $raw = file_get_contents('php://input');
    return json_decode($raw, true) ?: [];
}

function sanitize($value) {
    return is_string($value)
        ? htmlspecialchars(trim($value), ENT_QUOTES, 'UTF-8')
        : $value;
}

function generateCSRFToken() {
    if (empty($_SESSION['csrf_token'])) {
        $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    }
    return $_SESSION['csrf_token'];
}

function validateCSRF() {
    $token = $_SERVER['HTTP_X_CSRF_TOKEN'] ?? '';
    if (empty($token) || $token !== ($_SESSION['csrf_token'] ?? '')) {
        jsonResponse(['error' => 'Invalid CSRF token'], 403);
    }
}

// ============ SECURITY HEADERS ============
header('X-Content-Type-Options: nosniff');
header('X-Frame-Options: DENY');
header('X-XSS-Protection: 1; mode=block');
header('Referrer-Policy: strict-origin-when-cross-origin');

// CORS (same-origin only)
if (isset($_SERVER['HTTP_ORIGIN'])) {
    $allowed = 'https://www.your-domain.com';
    if ($_SERVER['HTTP_ORIGIN'] === $allowed) {
        header("Access-Control-Allow-Origin: $allowed");
        header('Access-Control-Allow-Credentials: true');
    }
}
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');
    header('Access-Control-Allow-Headers: Content-Type, X-CSRF-Token');
    header('Access-Control-Max-Age: 86400');
    exit;
}
```

## entities.php

The "brain" of the application. Each entity is declared once, and this declaration drives
all CRUD operations, validation, display, and form rendering.

```php
<?php
function getEntityDefinitions() {
    return [
        // Example entity
        'project' => [
            'table' => 'app_project',          // MySQL table name (prefix all tables)
            'title' => 'Project',               // Human-readable title
            'fields' => [                       // All editable fields (not 'id')
                'organisation_id', 'name', 'description',
                'start_date', 'end_date', 'budget', 'status'
            ],
            'required' => ['organisation_id', 'name'],  // Must be non-empty for create
            'types' => [                        // Type casting for safe handling
                'organisation_id' => 'int',
                'start_date' => 'date',
                'end_date' => 'date',
                'budget' => 'decimal'
            ],
            'columns' => ['name', 'start_date', 'status', 'budget'],  // Table view columns
            'display' => 'name',                // What to show when this entity is referenced as FK
            'fk' => [                           // Foreign key relationships
                'organisation_id' => 'organisation'
            ]
        ],

        // Entity with composite display name
        'person' => [
            'table' => 'app_person',
            'title' => 'Person',
            'fields' => ['first_name', 'last_name', 'email', 'role'],
            'required' => ['first_name', 'last_name'],
            'types' => [],
            'columns' => ['first_name', 'last_name', 'email', 'role'],
            'display' => ['first_name', 'last_name'],  // Array = composite display
        ],

        // Hierarchical entity (group > items)
        'category' => [
            'table' => 'app_category',
            'title' => 'Category',
            'fields' => ['parent_id', 'name', 'description', 'ordre'],
            'required' => ['name'],
            'types' => ['parent_id' => 'int', 'ordre' => 'int'],
            'columns' => ['name', 'description'],
            'display' => 'name',
            'fk' => ['parent_id' => 'category']  // Self-referencing FK
        ],

        // Junction table (many-to-many)
        'project_person' => [
            'table' => 'app_project_person',
            'title' => 'Project Assignment',
            'fields' => ['project_id', 'person_id', 'role', 'date_assigned', 'status'],
            'required' => ['project_id', 'person_id', 'date_assigned'],
            'types' => ['project_id'=>'int', 'person_id'=>'int', 'date_assigned'=>'date'],
            'columns' => ['project_id', 'person_id', 'date_assigned', 'status'],
            'display' => 'role',
            'fk' => ['project_id'=>'project', 'person_id'=>'person']
        ],
    ];
}

function getEntityDef($entityKey) {
    $defs = getEntityDefinitions();
    return $defs[$entityKey] ?? null;
}

function getEntityList() {
    return array_keys(getEntityDefinitions());
}
```

**Supported types:**
- `'string'` (default) â sanitized with htmlspecialchars
- `'int'` â cast to integer
- `'decimal'` â cast to float
- `'date'` â kept as-is (YYYY-MM-DD)
- `'json'` â stored as JSON string

## crud.php

The generic CRUD handler. Reads entity definitions and handles all operations.

```php
<?php
require_once __DIR__ . '/config.php';
require_once __DIR__ . '/entities.php';

requireAuth();

$method = $_SERVER['REQUEST_METHOD'];
$entity = $_GET['entity'] ?? '';
$id = isset($_GET['id']) ? intval($_GET['id']) : null;

$def = getEntityDef($entity);
if (!$def) jsonResponse(['error' => "Unknown entity '$entity'"], 400);

switch ($method) {
    case 'GET':
        $id ? getOne($def, $id) : getAll($def, $entity);
        break;
    case 'POST':
        validateCSRF();
        create($def, $entity);
        break;
    case 'PUT':
        validateCSRF();
        if (!$id) jsonResponse(['error' => 'ID required'], 400);
        update($def, $entity, $id);
        break;
    case 'DELETE':
        validateCSRF();
        if (!$id) jsonResponse(['error' => 'ID required'], 400);
        delete($def, $id);
        break;
    default:
        jsonResponse(['error' => 'Method not supported'], 405);
}
```

**Key features of the CRUD functions:**

- **getAll()**: SELECT *, resolve FK display names (cached), inject file counts, return with meta
- **getOne()**: SELECT by ID, include attached files from files table
- **create()**: Validate required fields, type-cast inputs, INSERT
- **update()**: Check existence, type-cast, UPDATE
- **delete()**: Cascade delete children (configurable map), delete associated files, DELETE

**Cascade delete pattern:**
```php
$cascadeMap = [
    'app_category' => [
        ['table' => 'app_item', 'fk' => 'category_id'],
        ['table' => 'app_category', 'fk' => 'parent_id']  // Self-referencing
    ],
];
```

**Response format:**
```json
// GET list
{ "data": [...], "count": 42, "meta": { "fields": [...], "types": {...}, "required": [...], "fk": {...} } }

// GET single
{ "data": { "id": 1, "name": "...", "_files": [...] } }

// POST create
{ "success": true, "id": 5, "message": "Created successfully" }

// PUT update
{ "success": true, "message": "Updated successfully" }

// DELETE
{ "success": true, "message": "Deleted" }
```

## auth.php

```php
<?php
require_once __DIR__ . '/config.php';

$action = $_GET['action'] ?? '';

switch ($action) {
    case 'check':
        // Return auth status + CSRF token
        jsonResponse([
            'authenticated' => !empty($_SESSION['user_id']),
            'csrf_token' => generateCSRFToken(),
            'user' => $_SESSION['user_name'] ?? null,
            'role' => $_SESSION['user_role'] ?? null,
            'needs_setup' => checkNeedsSetup()
        ]);
        break;
    case 'login':
        $input = getInput();
        // Verify credentials with password_verify()
        // Set session variables: user_id, user_name, user_role
        break;
    case 'register':
        // First-time admin setup only
        // Hash password with password_hash(PASSWORD_DEFAULT)
        break;
    case 'logout':
        session_destroy();
        jsonResponse(['success' => true]);
        break;
}
```

## Specialized Endpoints

For complex data that goes beyond simple CRUD:

**Tree endpoint** (`*_tree.php`) â hierarchical data with computed stats:
```php
// Returns nested structure: groups > subgroups > items
// With computed counts (nb_items, nb_active, nb_expired, etc.)
// Used for catalogue accordion views
```

**Detail endpoint** (`*_detail.php`) â single record with JOINed data:
```php
// Returns the record + related data from junction tables
// Example: formation_detail.php returns the formation + all user attributions
// With breadcrumb (group > subgroup path)
```

## .htaccess

Protect sensitive PHP files:
```apache
# In api/.htaccess
<FilesMatch "^(config|entities)\.php$">
    Require all denied
</FilesMatch>
```

## SQL Table Creation

All tables follow this pattern:

```sql
CREATE TABLE IF NOT EXISTS app_my_entity (
    id INT AUTO_INCREMENT PRIMARY KEY,
    organisation_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_org (organisation_id),
    FOREIGN KEY (organisation_id) REFERENCES app_organisation(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Files table** (for attachments on any entity):
```sql
CREATE TABLE IF NOT EXISTS app_files (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_type VARCHAR(100) NOT NULL,
    entity_id INT NOT NULL,
    original_name VARCHAR(255) NOT NULL,
    stored_name VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100),
    file_size INT,
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_entity (entity_type, entity_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Users table** (for auth):
```sql
CREATE TABLE IF NOT EXISTS app_user (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    prenom VARCHAR(100),
    nom VARCHAR(100),
    role VARCHAR(50) DEFAULT 'user',
    actif TINYINT(1) DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```
