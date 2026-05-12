# CSS Design System â KKRI Style

Complete CSS reference for building apps with the KKRI glass morphism identity.
Copy the full CSS below into `css/app.css` and adapt entity-specific sections to your app.

## Table of Contents
1. [CSS Variables](#css-variables)
2. [Base Reset & Body](#base-reset)
3. [Glass Morphism Layers](#glass-morphism)
4. [Layout (Sidebar + Main)](#layout)
5. [Navigation](#navigation)
6. [Buttons](#buttons)
7. [Forms](#forms)
8. [Tables](#tables)
9. [Tags](#tags)
10. [Stat Cards](#stat-cards)
11. [Modals](#modals)
12. [Toasts](#toasts)
13. [Loading & Empty States](#loading)
14. [File Upload](#file-upload)
15. [Tab Bar](#tab-bar)
16. [Catalogue Accordion](#catalogue-accordion)
17. [Profile Header](#profile-header)
18. [Progress Bars](#progress-bars)
19. [Responsive](#responsive)

---

## CSS Variables

```css
:root {
  /* Backgrounds */
  --bg-primary: #f7f8fa;
  --bg-secondary: #ffffff;
  --bg-card: #ffffff;
  --bg-card-hover: #f1f5f9;
  /* Borders */
  --border: #e2e8f0;
  --border-hover: #cbd5e1;
  --border-strong: #94a3b8;
  /* Text */
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;
  /* Accent */
  --accent: #2563eb;
  --accent-hover: #1d4ed8;
  --accent-dark: #1e40af;
  --accent-glow: rgba(37,99,235,.12);
  --accent-bg: rgba(37,99,235,.06);
  /* Status */
  --success: #16a34a;
  --warning: #d97706;
  --danger: #dc2626;
  --purple: #7c3aed;
  /* Layout */
  --sidebar-width: 256px;
  --topbar-height: 56px;
  /* Radii */
  --radius: 10px;
  --radius-sm: 8px;
  --radius-xs: 5px;
  /* Shadows */
  --shadow-xs: 0 1px 2px rgba(0,0,0,.04);
  --shadow: 0 1px 3px rgba(0,0,0,.06), 0 1px 2px rgba(0,0,0,.04);
  --shadow-md: 0 4px 6px -1px rgba(0,0,0,.07), 0 2px 4px -2px rgba(0,0,0,.05);
  --shadow-lg: 0 10px 25px -5px rgba(0,0,0,.08), 0 8px 10px -6px rgba(0,0,0,.04);
}
```

## Base Reset

```css
* { margin:0; padding:0; box-sizing:border-box; }
body {
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  background: var(--bg-primary);
  color: var(--text-primary);
  min-height: 100vh;
  font-size: 14px;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  position: relative;
}

/* Thin scrollbar */
* { scrollbar-width: thin; scrollbar-color: #cbd5e1 transparent; }
*::-webkit-scrollbar { width: 6px; height: 6px; }
*::-webkit-scrollbar-track { background: transparent; }
*::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
*::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
```

## Glass Morphism

The signature effect â background image visible through frosted UI panels.

```css
/* Layer 1: Background image */
body::before {
  content: '';
  position: fixed; inset: 0; z-index: 0;
  background: url('../img/bg.jpg') center center / cover no-repeat;
}
/* Layer 2: Frosted overlay (adjust opacity 0.5-0.8 for more/less image) */
body::after {
  content: '';
  position: fixed; inset: 0; z-index: 0;
  background: rgba(240,245,250,.62);
}
```

**Tuning the effect:**
- Lower overlay opacity (e.g. `.50`) â more image visible, more "glassy" feel
- Higher overlay opacity (e.g. `.80`) â cleaner, more "solid" feel
- Panel backgrounds use `rgba(255,255,255,.78)` with `backdrop-filter: blur(14px)`

## Layout

```css
.app { display: flex; height: 100vh; position: relative; z-index: 1; }

.sidebar {
  width: var(--sidebar-width);
  background: rgba(255,255,255,.78);
  backdrop-filter: blur(14px); -webkit-backdrop-filter: blur(14px);
  border-right: 1px solid var(--border);
  overflow-y: auto; flex-shrink: 0;
  transition: transform .3s cubic-bezier(.4,0,.2,1);
  z-index: 50; display: flex; flex-direction: column;
}

.sidebar-header {
  padding: 16px 18px; display: flex; align-items: center; gap: 12px;
  border-bottom: 1px solid var(--border); position: sticky; top: 0;
  background: rgba(255,255,255,.82);
  backdrop-filter: blur(14px); -webkit-backdrop-filter: blur(14px); z-index: 1;
}

.sidebar-logo {
  width: 54px; height: 54px; border-radius: 12px;
  background: var(--accent); display: flex; align-items: center; justify-content: center;
  flex-shrink: 0; padding: 8px;
}
.sidebar-logo img { width: 100%; height: 100%; object-fit: contain; }

.main { flex: 1; overflow-y: auto; display: flex; flex-direction: column; background: transparent; }

.topbar {
  padding: 0 24px; border-bottom: 1px solid var(--border);
  display: flex; align-items: center; gap: 16px;
  background: rgba(255,255,255,.78);
  backdrop-filter: blur(14px); -webkit-backdrop-filter: blur(14px);
  position: sticky; top: 0; z-index: 40;
  min-height: var(--topbar-height);
}
.topbar h1 { font-size: 16px; font-weight: 600; flex: 1; letter-spacing: -.2px; }

.content-area { flex: 1; padding: 28px; max-width: 1400px; }

.sidebar-footer {
  padding: 12px 18px; text-align: center; font-size: 10px;
  color: var(--text-muted); border-top: 1px solid var(--border);
  margin-top: auto; flex-shrink: 0;
}
```

## Navigation

```css
.nav { flex: 1; overflow-y: auto; padding: 8px 0; }
.nav-group-title {
  padding: 10px 18px 6px; font-size: 11px; text-transform: uppercase;
  letter-spacing: .8px; color: var(--text-muted); font-weight: 600;
  display: flex; align-items: center; gap: 6px;
}
.nav-group-title::before {
  content: ''; width: 6px; height: 6px; border-radius: 50%;
  flex-shrink: 0; background: var(--text-muted);
}

.nav-item {
  padding: 7px 18px; cursor: pointer; font-size: 13px;
  display: flex; align-items: center; gap: 10px;
  transition: all .15s ease; border-left: 2px solid transparent;
  color: var(--text-secondary); margin: 1px 8px 1px 0;
  border-radius: 0 6px 6px 0;
}
.nav-item:hover { background: #f1f5f9; color: var(--text-primary); }
.nav-item.active {
  background: var(--accent-bg); color: var(--accent);
  border-left-color: var(--accent); font-weight: 600;
}

/* Item count badge */
.count {
  margin-left: auto; background: #f1f5f9; padding: 2px 8px;
  border-radius: 10px; font-size: 10px; color: var(--text-muted);
  font-weight: 600; min-width: 20px; text-align: center;
}
.nav-item.active .count { background: var(--accent-glow); color: var(--accent); }

/* Lucide nav icons */
.nav-item svg, .nav-item i { width: 16px; height: 16px; flex-shrink: 0; stroke-width: 1.75; opacity: .7; }
.nav-item.active svg { opacity: 1; }
```

## Buttons

```css
.btn {
  padding: 8px 16px; border-radius: var(--radius-xs); border: none; cursor: pointer;
  font-size: 13px; font-weight: 500; transition: all .15s ease;
  font-family: inherit; display: inline-flex; align-items: center; gap: 6px;
}
.btn-primary { background: var(--accent); color: #fff; }
.btn-primary:hover { background: var(--accent-hover); }
.btn-success { background: var(--success); color: #fff; }
.btn-danger { background: var(--danger); color: #fff; }
.btn-danger-outline { background: transparent; border: 1px solid var(--danger); color: var(--danger); }
.btn-danger-outline:hover { background: var(--danger); color: #fff; }
.btn-outline { background: transparent; border: 1px solid var(--border); color: var(--text-secondary); }
.btn-outline:hover { border-color: var(--border-hover); background: #f8fafc; color: var(--text-primary); }
.btn-sm { padding: 5px 10px; font-size: 12px; }
.btn svg, .btn i[data-lucide] { width: 14px; height: 14px; flex-shrink: 0; }
```

## Forms

```css
.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px; margin-bottom: 20px;
}
.form-group { display: flex; flex-direction: column; gap: 5px; }
.form-group label { font-size: 12px; color: var(--text-secondary); font-weight: 500; }
.form-group label .req { color: var(--danger); }

input, select, textarea {
  padding: 9px 12px; background: #fff; border: 1px solid var(--border);
  border-radius: var(--radius-xs); color: var(--text-primary);
  font-size: 13px; font-family: inherit; transition: all .15s ease; outline: none;
}
input:focus, select:focus, textarea:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px var(--accent-glow);
}
textarea { min-height: 70px; resize: vertical; }

.form-actions {
  display: flex; gap: 8px; margin-top: 20px;
  padding-top: 16px; border-top: 1px solid var(--border);
}
```

## Tables

```css
table {
  width: 100%; border-collapse: separate; border-spacing: 0;
  margin-top: 16px; font-size: 13px; border-radius: var(--radius-sm);
  overflow: hidden; border: 1px solid var(--border); background: #fff;
}
th {
  text-align: left; padding: 10px 14px; background: #f8fafc;
  color: var(--text-secondary); font-size: 12px; font-weight: 600;
  border-bottom: 1px solid var(--border);
}
td { padding: 10px 14px; border-bottom: 1px solid #f1f5f9; }
tr:last-child td { border-bottom: none; }
tr:hover td { background: #f8fafc; }
tr:hover td:first-child { box-shadow: inset 3px 0 0 var(--accent); }
```

## Tags

```css
.tag {
  display: inline-block; padding: 3px 10px; border-radius: 20px;
  font-size: 11px; font-weight: 500;
}
.tag-green { background: #f0fdf4; color: var(--success); }
.tag-orange { background: #fffbeb; color: var(--warning); }
.tag-red { background: #fef2f2; color: var(--danger); }
.tag-blue { background: #eff6ff; color: var(--accent); }
.tag-purple { background: #f5f3ff; color: var(--purple); }
```

## Stat Cards

```css
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(170px, 1fr));
  gap: 14px; margin-bottom: 24px;
}
.stat-card {
  background: #fff; border: 1px solid var(--border); border-radius: var(--radius);
  padding: 20px; min-width: 150px; transition: all .2s ease; cursor: pointer;
}
.stat-card:hover { border-color: var(--border-hover); box-shadow: var(--shadow-md); }
.stat-card .stat-icon {
  font-size: 18px; width: 40px; height: 40px;
  display: flex; align-items: center; justify-content: center;
  background: #f1f5f9; border-radius: var(--radius-sm);
  margin-bottom: 12px; color: var(--accent);
}
.stat-card .val { font-size: 28px; font-weight: 700; letter-spacing: -.5px; }
.stat-card .lbl { font-size: 12px; color: var(--text-muted); margin-top: 2px; font-weight: 500; }
```

## Modals

```css
.modal-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,.4); z-index: 200;
  display: flex; align-items: center; justify-content: center; padding: 20px;
}
.modal-card {
  background: #fff; border: 1px solid var(--border);
  border-radius: var(--radius); width: 100%; max-width: 520px;
  box-shadow: var(--shadow-lg);
  animation: cardAppear .25s cubic-bezier(.16,1,.3,1);
}
.modal-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 18px 22px; border-bottom: 1px solid var(--border);
}
.modal-header h3 { font-size: 16px; font-weight: 600; }
.modal-close {
  background: none; border: none; color: var(--text-muted);
  font-size: 20px; cursor: pointer; padding: 4px 8px; border-radius: 6px;
}
.modal-close:hover { background: #f1f5f9; color: var(--text-primary); }
.modal-body { padding: 22px; }
.modal-footer {
  display: flex; gap: 10px; justify-content: flex-end;
  padding: 14px 22px; border-top: 1px solid var(--border);
}

@keyframes cardAppear {
  from { opacity: 0; transform: translateY(10px) scale(.98); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}
```

## Toasts

```css
.toast-container {
  position: fixed; bottom: 24px; right: 24px; z-index: 9999;
  display: flex; flex-direction: column; gap: 8px;
}
.toast {
  padding: 12px 18px; border-radius: var(--radius-sm); font-size: 13px; font-weight: 500;
  display: flex; align-items: center; gap: 10px; min-width: 280px; max-width: 420px;
  animation: toastIn .4s cubic-bezier(.16,1,.3,1); box-shadow: var(--shadow-lg);
  border: 1px solid;
}
.toast-success { background: #f0fdf4; border-color: #dcfce7; color: #15803d; }
.toast-error { background: #fef2f2; border-color: #fecaca; color: #b91c1c; }
.toast-info { background: #eff6ff; border-color: #dbeafe; color: #1d4ed8; }

@keyframes toastIn { from { opacity:0; transform:translateX(30px); } to { opacity:1; transform:translateX(0); } }
@keyframes toastOut { to { opacity:0; transform:translateX(30px); } }
```

## Loading & Empty States

```css
.loading-state {
  display: flex; flex-direction: column; align-items: center;
  justify-content: center; gap: 16px; padding: 60px; color: var(--text-muted);
}
.loader {
  width: 32px; height: 32px; border: 2.5px solid var(--border);
  border-top-color: var(--accent); border-radius: 50%;
  animation: spin .7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

.empty-state { text-align: center; padding: 60px 20px; color: var(--text-muted); }
.empty-state .icon { font-size: 48px; margin-bottom: 16px; opacity: .4; }
.empty-state p { font-size: 14px; max-width: 400px; margin: 0 auto; line-height: 1.6; }
```

## File Upload

```css
.file-zone {
  border: 2px dashed var(--border); border-radius: var(--radius-sm);
  padding: 16px; text-align: center; cursor: pointer;
  transition: all .2s; background: #fafbfc;
}
.file-zone:hover { border-color: var(--accent); background: var(--accent-bg); }
.file-zone.has-file { border-color: var(--success); border-style: solid; }
.file-zone input[type=file] { position: absolute; inset: 0; opacity: 0; cursor: pointer; }

.file-item {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 6px 12px; background: #f8fafc; border: 1px solid var(--border);
  border-radius: var(--radius-sm); font-size: 12px; margin: 3px 2px;
}
```

## Tab Bar

```css
.tab-bar { display: flex; gap: 2px; margin-bottom: 16px; border-bottom: 1px solid var(--border); }
.tab-btn {
  padding: 8px 16px; border: none; border-bottom: 2px solid transparent;
  background: transparent; color: var(--text-muted); cursor: pointer;
  font-size: 13px; font-weight: 500; font-family: inherit;
}
.tab-btn.active { color: var(--accent); border-bottom-color: var(--accent); }
.tab-btn:hover { color: var(--text-primary); }
```

## Catalogue Accordion

Used for hierarchical data (categories > subcategories > items).

```css
.catalogue-accordion {
  border: 1px solid var(--border); border-radius: var(--radius-sm);
  background: var(--bg-secondary); margin-bottom: 10px; overflow: hidden;
}
.catalogue-accordion.sub-group {
  border: none; border-left: 3px solid var(--accent);
  border-radius: 0; margin: 0 0 0 4px; background: transparent;
}
.catalogue-accordion-header {
  display: flex; align-items: center; gap: 10px; padding: 12px 16px;
  cursor: pointer; user-select: none; font-weight: 600; font-size: 14px;
}
.catalogue-accordion-header:hover { background: var(--bg-card-hover); }
.accordion-chevron {
  width: 16px; height: 16px; flex-shrink: 0;
  transition: transform .2s ease; color: var(--text-muted);
}
.catalogue-accordion.open > .catalogue-accordion-header .accordion-chevron {
  transform: rotate(90deg);
}
.catalogue-accordion-body { display: none; padding: 4px 12px 12px; }
.catalogue-accordion.open > .catalogue-accordion-body { display: block; }

/* Cards inside accordion */
.catalogue-card {
  display: flex; align-items: center; gap: 12px; padding: 10px 14px;
  border: 1px solid var(--border); border-radius: var(--radius-xs);
  background: var(--bg-secondary); margin-bottom: 6px; cursor: pointer;
}
.catalogue-card:hover {
  border-color: var(--accent); box-shadow: 0 0 0 2px var(--accent-glow);
}

/* Badges on cards */
.catalogue-badge {
  font-size: 11px; font-weight: 600; padding: 3px 8px; border-radius: 10px;
}
.catalogue-badge.green { background: #dcfce7; color: #15803d; }
.catalogue-badge.orange { background: #fff7ed; color: #c2410c; }
.catalogue-badge.red { background: #fef2f2; color: #dc2626; }
.catalogue-badge.blue { background: #eff6ff; color: #2563eb; }
```

## Profile Header

```css
.profile-header {
  display: flex; align-items: flex-start; gap: 20px; padding: 24px;
  background: #fff; border: 1px solid var(--border); border-radius: var(--radius);
}
.profile-avatar {
  width: 56px; height: 56px; border-radius: var(--radius); flex-shrink: 0;
  background: var(--accent); display: flex; align-items: center; justify-content: center;
  font-size: 20px; font-weight: 700; color: #fff; letter-spacing: 1px;
}
.profile-info { flex: 1; min-width: 0; }
.profile-info h2 { font-size: 18px; font-weight: 600; display: flex; align-items: center; gap: 10px; }
.profile-meta {
  display: flex; flex-wrap: wrap; gap: 12px; margin-top: 6px;
  font-size: 13px; color: var(--text-secondary);
}
.profile-actions { display: flex; flex-direction: column; gap: 6px; flex-shrink: 0; }
```

## Progress Bars

```css
.progress-bar {
  width: 100%; height: 6px; background: #e2e8f0;
  border-radius: 3px; overflow: hidden;
}
.progress-fill {
  height: 100%; background: var(--accent);
  border-radius: 3px; transition: width .5s ease;
}
```

## Responsive

```css
@media (max-width: 768px) {
  .sidebar {
    position: fixed; left: 0; top: 0; bottom: 0;
    transform: translateX(-100%); z-index: 100; box-shadow: var(--shadow-lg);
  }
  .sidebar.open { transform: translateX(0); }
  .mobile-menu { display: block; }
  .sidebar-toggle { display: block; }
  .topbar-actions { display: none; }
  .form-grid { grid-template-columns: 1fr; }
  .content-area { padding: 16px; }
  .stats-grid { gap: 10px; }
  .stat-card { min-width: 100px; padding: 14px 16px; }
  .stat-card .val { font-size: 22px; }
  .profile-header { flex-direction: column; }
  .tab-bar { overflow-x: auto; white-space: nowrap; }
}

/* Mobile sidebar overlay */
.sidebar-overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,.4); z-index: 99; }
.sidebar-overlay.show { display: block; }
```

## Alerts

```css
.alert {
  padding: 12px 16px; border-radius: var(--radius-sm); font-size: 13px;
  margin-bottom: 10px; display: flex; align-items: center; gap: 10px;
}
.alert-info { background: #eff6ff; border: 1px solid #dbeafe; color: #1d4ed8; }
.alert-warn { background: #fffbeb; border: 1px solid #fef3c7; color: #92400e; }
.alert-err { background: #fef2f2; border: 1px solid #fecaca; color: #b91c1c; }
.alert-ok { background: #f0fdf4; border: 1px solid #dcfce7; color: #15803d; }
```

## Checkbox Grid

```css
.checkbox-grid {
  display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 4px 12px; max-height: 220px; overflow-y: auto;
  border: 1px solid var(--border); border-radius: var(--radius-sm);
  padding: 8px; background: #fafbfc;
}
.cb-item {
  display: flex; align-items: center; gap: 6px;
  padding: 4px 6px; border-radius: var(--radius-xs);
  cursor: pointer; font-size: 13px; color: var(--text-secondary);
}
.cb-item:hover { background: #f1f5f9; }
.cb-item input[type=checkbox] { accent-color: var(--accent); width: 16px; height: 16px; }
.cb-item input:checked+span { color: var(--accent); font-weight: 500; }
```
