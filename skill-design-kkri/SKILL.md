---
name: skill-design-kkri
description: >
  Generates full-stack web applications with the KKRI design DNA â a distinctive glass morphism UI
  built on PHP + Vanilla JS with zero frameworks. Use this skill whenever the user asks to create
  a new web application, tool, dashboard, management system, or internal app. Also trigger when
  the user mentions "KKRI style", "same design as MASE", "glass morphism app", or wants a PHP+JS
  application with a professional, modern look. This skill covers the complete stack: visual identity
  (CSS design system with glass morphism, Inter font, layered shadows), backend architecture
  (entity-driven generic CRUD with PHP/MySQL/PDO), frontend patterns (SPA with sidebar navigation,
  dynamic rendering via template literals, Lucide icons), login page, responsive design, and
  deployment on shared hosting.
---

# SKILL DESIGN KKRI

Build complete web applications with the KKRI visual identity and technical architecture.
This is a zero-framework stack (PHP + Vanilla JS + MySQL) designed for shared hosting (Hostinger),
with a glass morphism design system that feels modern, clean, and professional.

## Philosophy

The KKRI approach favors simplicity and directness:
- **No frameworks** â Pure PHP backend, vanilla JS frontend, raw CSS
- **Entity-driven** â The data model drives everything: declare entities, get CRUD for free
- **Glass morphism** â Semi-transparent panels over a background image, with blur effects
- **Single page** â One `app.html` with sidebar navigation and dynamic content area
- **Shared hosting friendly** â No Node.js, no build step, no Docker. Just upload PHP files

## Architecture Overview

```
my-app/
âââ index.html          # Login page (dark glass morphism theme)
âââ app.html            # Main SPA shell (sidebar + topbar + content)
âââ css/
â   âââ app.css         # Complete design system
âââ js/
â   âââ app.js          # Client-side logic (API calls, rendering, navigation)
âââ api/
â   âââ .htaccess       # Protect config files from direct access
â   âââ config.php      # DB connection, session, CSRF, helpers
â   âââ entities.php    # Declarative entity definitions (the "brain")
â   âââ crud.php        # Generic CRUD handler (GET/POST/PUT/DELETE)
â   âââ auth.php        # Authentication (login, register, check, logout)
â   âââ *.php           # Specialized endpoints (*_tree.php, *_detail.php)
âââ uploads/            # User-uploaded files
âââ img/
    âââ bg.jpg          # Background image (landscape, blue/teal tones work best)
```

## Quick Start â How to Create a New App

1. **Define your entities** in `entities.php` â this is the heart of the app
2. **Create the database tables** matching those entity definitions
3. **Copy the infrastructure** files (config.php, crud.php, auth.php, .htaccess)
4. **Build the UI** in app.html with sidebar nav items matching your entities
5. **Write rendering functions** in app.js for each entity panel

Read `references/css-design-system.md` for the complete CSS variables and component library.
Read `references/php-backend-patterns.md` for entity definitions, CRUD, and config templates.
Read `references/js-frontend-patterns.md` for the JS architecture, API wrapper, and rendering patterns.
Read `references/login-page.md` for the dark-themed login/register page.

## Design System Summary

### Typography
- **Font**: Inter (Google Fonts) â `'Inter', system-ui, -apple-system, sans-serif`
- **Base size**: 14px, line-height 1.5
- **Font smoothing**: `-webkit-font-smoothing: antialiased`

### Color Palette
```css
:root {
  /* Accent */
  --accent: #2563eb;          /* Primary blue */
  --accent-hover: #1d4ed8;
  --accent-dark: #1e40af;
  --accent-glow: rgba(37,99,235,.12);
  --accent-bg: rgba(37,99,235,.06);
  /* Status */
  --success: #16a34a;
  --warning: #d97706;
  --danger: #dc2626;
  --purple: #7c3aed;
  /* Backgrounds */
  --bg-primary: #f7f8fa;
  --bg-secondary: #ffffff;
  /* Text */
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;
  /* Borders */
  --border: #e2e8f0;
  --border-hover: #cbd5e1;
  /* Radii */
  --radius: 10px;
  --radius-sm: 8px;
  --radius-xs: 5px;
  /* Shadows (layered, subtle) */
  --shadow-xs: 0 1px 2px rgba(0,0,0,.04);
  --shadow: 0 1px 3px rgba(0,0,0,.06), 0 1px 2px rgba(0,0,0,.04);
  --shadow-md: 0 4px 6px -1px rgba(0,0,0,.07), 0 2px 4px -2px rgba(0,0,0,.05);
  --shadow-lg: 0 10px 25px -5px rgba(0,0,0,.08), 0 8px 10px -6px rgba(0,0,0,.04);
}
```

### Glass Morphism Effect
The signature look: a background image visible through semi-transparent UI panels.

```css
/* Background image layer */
body::before {
  content: '';
  position: fixed; inset: 0; z-index: 0;
  background: url('../img/bg.jpg') center center / cover no-repeat;
}
/* Frosted overlay */
body::after {
  content: '';
  position: fixed; inset: 0; z-index: 0;
  background: rgba(240,245,250,.62);
}
/* Glass panels */
.sidebar, .topbar {
  background: rgba(255,255,255,.78);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
}
```

### Key Components
- **Stat cards**: Icon + value + label, with hover shadow
- **Data tables**: Border-collapse separate, accent border on first cell on hover
- **Tags**: `.tag-green`, `.tag-orange`, `.tag-red`, `.tag-blue`, `.tag-purple` â pill-shaped
- **Modals**: Fixed overlay + animated card (translateY + scale spring animation)
- **Toasts**: Bottom-right, animated slide-in, auto-dismiss after 4s
- **Forms**: Grid layout with `auto-fill, minmax(280px, 1fr)`, focus glow on accent
- **Buttons**: `.btn-primary` (accent), `.btn-outline`, `.btn-danger`, `.btn-sm`
- **Catalogue accordion**: Nested groups with chevron toggle, cards with badges
- **Profile header**: Avatar circle + name + meta + action buttons
- **File upload zone**: Dashed border, drag & drop styled
- **Progress bars**: Colored fills (green/orange/red)
- **Loading spinner**: Border-top accent color, 0.7s rotation

### Responsive
- Mobile (max-width 768px): sidebar becomes fixed overlay with toggle
- Form grids collapse to single column
- Stats shrink, topbar actions hidden

## Entity Definition Pattern

Each entity is declared once in `entities.php`. This single declaration drives:
- Database operations (which table, which fields)
- Validation (required fields, type casting)
- Table display (which columns to show)
- FK resolution (display names from related tables)
- Form rendering (field types â input types)

```php
'my_entity' => [
    'table' => 'mase_my_entity',          // MySQL table name
    'title' => 'My Entity',               // Display title
    'fields' => ['org_id', 'name', ...],   // All editable fields
    'required' => ['org_id', 'name'],      // Required for create
    'types' => ['org_id'=>'int', ...],     // Type casting (int, decimal, date, json)
    'columns' => ['name', 'status'],       // Columns shown in table view
    'display' => 'name',                   // FK display field (or array for composite)
    'fk' => ['org_id' => 'organisation']   // Foreign key relationships
],
```

## Backend Pattern

`config.php` provides: `getDB()` (PDO singleton), `jsonResponse()`, `requireAuth()`,
`validateCSRF()`, `getInput()` (JSON body), `sanitize()` (XSS protection).

`crud.php` is fully generic â it reads the entity definition and handles:
- **GET** without ID â list all with FK resolution and file counts
- **GET** with ID â single record with attached files
- **POST** â create with type-safe input handling
- **PUT** â update with existence check
- **DELETE** â cascade delete children + associated files

All write operations require CSRF token via `X-CSRF-Token` header.

## Frontend Pattern

```javascript
// Global state
let csrfToken = '';
let currentEntity = null;
let entityData = {};   // Cache: { entityKey: [records] }
let entityMeta = {};   // Cache: { entityKey: { fields, types, required, fk } }

// API wrapper â auto-injects CSRF, redirects on 401
async function api(endpoint, opts = {}) {
  const headers = { 'Content-Type': 'application/json' };
  if (csrfToken) headers['X-CSRF-Token'] = csrfToken;
  const res = await fetch('api/' + endpoint, { credentials:'same-origin', headers, ...opts });
  if (res.status === 401) { window.location.href = 'index.html'; return null; }
  return res.json();
}

// Navigation â show a panel by entity key
function showPanel(entity) {
  currentEntity = entity;
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  document.querySelector(`[data-panel="${entity}"]`)?.classList.add('active');
  document.getElementById('panel-title').textContent = panelTitles[entity];
  // Render content based on entity...
}

// Toast notifications
function toast(msg, type = 'info') { /* see reference */ }

// Lucide icons â call after any DOM update
function refreshIcons() { if (typeof lucide !== 'undefined') lucide.createIcons(); }
```

HTML is rendered via template literals â no JSX, no template engine. Each entity gets a
render function that builds the HTML string and writes to `document.getElementById('content')`.

## CDN Dependencies

```html
<!-- Google Fonts -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
<!-- Lucide Icons -->
<script src="https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"></script>
```

No npm, no bundler, no build step. Just CDN links and raw files.

## Security

- Session-based auth with httponly, secure, strict-mode cookies
- CSRF token generated server-side, validated on all write operations
- Input sanitization via `htmlspecialchars()` with `ENT_QUOTES`
- Prepared statements (PDO) for all SQL queries
- `.htaccess` blocks direct access to PHP config files
- Security headers: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection
- CORS restricted to specific origin

## Deployment

Upload files via FTP/File Manager to shared hosting. No build step needed.
Create MySQL database and tables, update credentials in `config.php`.
Ensure `uploads/` directory exists and is writable.
Add `.htaccess` in `api/` to protect sensitive files.

For detailed code templates and patterns, read the reference files:
- `references/css-design-system.md` â Full CSS with all components
- `references/php-backend-patterns.md` â config.php, entities.php, crud.php templates
- `references/js-frontend-patterns.md` â app.js architecture and rendering
- `references/login-page.md` â Dark glass morphism login page
