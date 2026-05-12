# JS Frontend Patterns â KKRI Style

Architecture and patterns for the vanilla JS single-page application.

## Table of Contents
1. [Global State](#global-state)
2. [API Wrapper](#api-wrapper)
3. [Toast Notifications](#toasts)
4. [Authentication Flow](#auth)
5. [Navigation & Panel System](#navigation)
6. [Entity Data Loading](#data-loading)
7. [Rendering Patterns](#rendering)
8. [Form Generation](#forms)
9. [Display Helpers](#display-helpers)
10. [Catalogue Accordion](#catalogue)
11. [Attribution Modals](#attribution)
12. [File Management](#files)
13. [Icons](#icons)

---

## Global State

```javascript
const API = 'api/';
let csrfToken = '';
let currentEntity = null;
let entityData = {};   // Cache: { entityKey: [records] }
let entityMeta = {};   // Cache: { entityKey: { fields, types, required, fk } }
```

Keep it simple â no state management library. Just module-level variables.
`entityData` and `entityMeta` are populated once at startup and updated after mutations.

## API Wrapper

Every API call goes through this wrapper. It handles CSRF injection and 401 redirects.

```javascript
async function api(endpoint, opts = {}) {
  const headers = { 'Content-Type': 'application/json' };
  if (csrfToken) headers['X-CSRF-Token'] = csrfToken;
  try {
    const res = await fetch(API + endpoint, {
      credentials: 'same-origin', headers, ...opts
    });
    const data = await res.json();
    if (res.status === 401) {
      window.location.href = 'index.html';
      return null;
    }
    return data;
  } catch (e) {
    toast('Server connection error', 'error');
    return null;
  }
}
```

**Usage:**
```javascript
// GET list
const data = await api('crud.php?entity=project');

// POST create
await api('crud.php?entity=project', {
  method: 'POST',
  body: JSON.stringify({ name: 'New Project', status: 'active' })
});

// PUT update
await api('crud.php?entity=project&id=5', {
  method: 'PUT',
  body: JSON.stringify({ name: 'Updated Name' })
});

// DELETE
await api('crud.php?entity=project&id=5', { method: 'DELETE' });
```

## Toasts

Notification system in the bottom-right corner.

```javascript
function toast(msg, type = 'info') {
  const c = document.getElementById('toast-container');
  const t = document.createElement('div');
  t.className = `toast toast-${type}`;
  const icon = type === 'success' ? 'check-circle'
             : type === 'error' ? 'alert-circle' : 'info';
  t.innerHTML = `<i data-lucide="${icon}" style="width:16px;height:16px;flex-shrink:0"></i>
                 <span>${msg}</span>`;
  c.appendChild(t);
  refreshIcons();
  setTimeout(() => {
    t.style.animation = 'toastOut .3s forwards';
    setTimeout(() => t.remove(), 300);
  }, 4000);
}
```

Types: `'success'` (green), `'error'` (red), `'info'` (blue).

## Authentication Flow

On page load, check auth and initialize:

```javascript
async function checkAuth() {
  const data = await api('auth.php?action=check');
  if (!data || !data.authenticated) {
    window.location.href = 'index.html';
    return;
  }
  csrfToken = data.csrf_token;
  await loadAllCounts();
  showPanel('dashboard');
}

async function handleLogout() {
  await api('auth.php?action=logout', { method: 'POST' });
  window.location.href = 'index.html';
}

// Initialize on page load
checkAuth();
```

## Navigation

Sidebar-driven navigation. Each nav item maps to a panel (entity or special view).

```javascript
function showPanel(entity) {
  currentEntity = entity;

  // Update active state in sidebar
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  const navEl = document.querySelector(`[data-panel="${entity}"]`);
  if (navEl) navEl.classList.add('active');

  // Update topbar title
  document.getElementById('panel-title').textContent = panelTitles[entity] || entity;

  // Close mobile sidebar
  document.getElementById('sidebar').classList.remove('open');
  const ov = document.querySelector('.sidebar-overlay');
  if (ov) ov.classList.remove('show');

  // Render the appropriate view
  const content = document.getElementById('content');
  switch (entity) {
    case 'dashboard': renderDashboard(); break;
    default: renderEntityTable(entity); break;
  }
}

// Sidebar toggle for mobile
function toggleSidebar() {
  document.getElementById('sidebar').classList.toggle('open');
  let ov = document.querySelector('.sidebar-overlay');
  if (!ov) {
    ov = document.createElement('div');
    ov.className = 'sidebar-overlay';
    ov.onclick = toggleSidebar;
    document.body.appendChild(ov);
  }
  ov.classList.toggle('show');
}
```

**HTML sidebar structure:**
```html
<div class="nav-item" data-panel="project" onclick="showPanel('project')">
  <i data-lucide="folder"></i> Projects
  <span class="count" id="cnt-project">0</span>
</div>
```

## Data Loading

Load all entity data at startup for fast navigation:

```javascript
async function loadAllCounts() {
  const entities = ['organisation', 'project', 'person', ...];
  let total = 0;
  for (const e of entities) {
    const data = await api(`crud.php?entity=${e}`);
    entityData[e] = data?.data || [];
    if (data?.meta) entityMeta[e] = data.meta;
    const el = document.getElementById(`cnt-${e}`);
    if (el) el.textContent = data?.count || 0;
    total += data?.count || 0;
  }
  document.getElementById('cnt-total').textContent = total;
}

// Load single entity (used after mutations)
async function loadEntity(entityKey) {
  const data = await api(`crud.php?entity=${entityKey}`);
  if (data) {
    entityData[entityKey] = data.data || [];
    if (data.meta) entityMeta[entityKey] = data.meta;
  }
  return entityData[entityKey];
}
```

## Rendering Patterns

All rendering uses template literals. Write HTML as strings and inject into the DOM.

### Stats Row
```javascript
function renderStats(stats) {
  return `<div class="stats-grid">
    ${stats.map(s => `
      <div class="stat-card" ${s.onclick ? `onclick="${s.onclick}"` : ''}>
        <div class="stat-icon"><i data-lucide="${s.icon}"></i></div>
        <div class="val">${s.value}</div>
        <div class="lbl">${s.label}</div>
      </div>
    `).join('')}
  </div>`;
}
```

### Data Table
```javascript
function renderTable(records, columns, meta, entityKey) {
  if (!records.length) {
    return `<div class="empty-state">
      <div class="icon"><i data-lucide="inbox"></i></div>
      <p>No records yet</p>
    </div>`;
  }

  const headers = columns.map(c => `<th>${meta.fields ? c : c}</th>`).join('');
  const rows = records.map(r => `
    <tr onclick="showDetail('${entityKey}', ${r.id})" style="cursor:pointer">
      ${columns.map(c => `<td>${formatCell(r, c, meta)}</td>`).join('')}
      <td>
        <button class="btn-icon-danger" onclick="event.stopPropagation();confirmDelete('${entityKey}',${r.id})">
          <i data-lucide="trash-2" style="width:14px;height:14px"></i>
        </button>
      </td>
    </tr>
  `).join('');

  return `
    <table>
      <thead><tr>${headers}<th></th></tr></thead>
      <tbody>${rows}</tbody>
    </table>`;
}
```

### Entity Panel (full rendering)
```javascript
function renderEntityTable(entityKey) {
  const records = entityData[entityKey] || [];
  const meta = entityMeta[entityKey] || {};
  const columns = meta.columns || meta.fields?.slice(0, 5) || [];

  document.getElementById('content').innerHTML = `
    <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:16px">
      <h2 style="font-size:18px;font-weight:600">${panelTitles[entityKey] || entityKey}</h2>
      <button class="btn btn-primary" onclick="openCreateForm('${entityKey}')">
        <i data-lucide="plus"></i> Add
      </button>
    </div>
    ${renderTable(records, columns, meta, entityKey)}
  `;
  refreshIcons();
}
```

## Form Generation

Generate forms dynamically from entity meta:

```javascript
function renderForm(entityKey, record = null) {
  const meta = entityMeta[entityKey] || {};
  const fields = meta.fields || [];
  const types = meta.types || {};
  const required = meta.required || [];
  const fk = meta.fk || {};

  const formFields = fields.map(field => {
    const isRequired = required.includes(field);
    const type = types[field] || 'string';
    const value = record ? (record[field] || '') : '';
    const req = isRequired ? '<span class="req">*</span>' : '';

    // FK field â select dropdown
    if (fk[field]) {
      const options = (entityData[fk[field]] || []).map(r =>
        `<option value="${r.id}" ${r.id == value ? 'selected' : ''}>${fkDisplay(fk[field], r.id)}</option>`
      ).join('');
      return `<div class="form-group">
        <label>${field} ${req}</label>
        <select id="field-${field}"><option value="">â Select â</option>${options}</select>
      </div>`;
    }

    // Type-based input
    let inputType = 'text';
    if (type === 'date') inputType = 'date';
    else if (type === 'int' || type === 'decimal') inputType = 'number';

    if (type === 'json' || field.includes('description') || field.includes('contenu')) {
      return `<div class="form-group" style="grid-column:1/-1">
        <label>${field} ${req}</label>
        <textarea id="field-${field}" rows="3">${value}</textarea>
      </div>`;
    }

    return `<div class="form-group">
      <label>${field} ${req}</label>
      <input type="${inputType}" id="field-${field}" value="${value}"
             ${isRequired ? 'required' : ''}>
    </div>`;
  }).join('');

  return `<div class="form-grid">${formFields}</div>`;
}
```

## Display Helpers

```javascript
// Resolve FK display name from cached data
function fkDisplay(entityKey, id) {
  if (!id) return 'â';
  const rec = (entityData[entityKey] || []).find(r => r.id == id);
  if (!rec) return `#${id}`;
  if (entityKey === 'person') return `${rec.first_name || ''} ${rec.last_name || ''}`.trim();
  return rec.name || rec.title || `#${id}`;
}

// Status tag with color coding
function statusTag(val) {
  if (!val) return '';
  const colors = {
    active: 'green', valid: 'green', done: 'green', approved: 'green',
    pending: 'orange', in_progress: 'orange', review: 'orange',
    expired: 'red', cancelled: 'red', rejected: 'red', overdue: 'red',
    draft: 'blue', new: 'blue', planned: 'purple',
  };
  return `<span class="tag tag-${colors[val] || 'blue'}">${val}</span>`;
}

// Date formatting
function formatDate(val) {
  if (!val) return '';
  const m = String(val).match(/^(\d{4})-(\d{2})-(\d{2})/);
  return m ? `${m[3]}/${m[2]}/${m[1]}` : val;
}
```

## Catalogue Accordion

For rendering hierarchical data (groups > subgroups > items):

```javascript
function renderCatalogue(groups, items, options = {}) {
  // Build tree: group items by their group_id
  const grouped = {};
  items.forEach(item => {
    const gid = item.groupe_id || 'ungrouped';
    (grouped[gid] = grouped[gid] || []).push(item);
  });

  // Render groups recursively
  function renderGroup(group, depth = 0) {
    const children = groups.filter(g => g.parent_id == group.id);
    const groupItems = grouped[group.id] || [];
    const isSubGroup = depth > 0;

    return `
      <div class="catalogue-accordion ${isSubGroup ? 'sub-group' : ''} open">
        <div class="catalogue-accordion-header" onclick="this.parentElement.classList.toggle('open')">
          <i data-lucide="chevron-right" class="accordion-chevron"></i>
          <span>${group.nom}</span>
          <span class="group-count">${groupItems.length} items</span>
        </div>
        <div class="catalogue-accordion-body">
          ${children.map(c => renderGroup(c, depth + 1)).join('')}
          ${groupItems.map(item => renderCatalogueCard(item, options)).join('')}
        </div>
      </div>`;
  }

  // Top-level groups (parent_id is null)
  const topGroups = groups.filter(g => !g.parent_id);
  const ungrouped = grouped['ungrouped'] || [];

  return `
    ${topGroups.map(g => renderGroup(g)).join('')}
    ${ungrouped.length ? `
      <div class="catalogue-ungrouped-title">Ungrouped</div>
      ${ungrouped.map(item => renderCatalogueCard(item, options)).join('')}
    ` : ''}`;
}

function renderCatalogueCard(item, options = {}) {
  return `
    <div class="catalogue-card" onclick="${options.onclick}(${item.id})">
      <div class="catalogue-card-icon"><i data-lucide="${options.icon || 'file'}"></i></div>
      <div class="catalogue-card-main">
        <div class="catalogue-card-title">${item.nom || item.name}</div>
        <div class="catalogue-card-meta">
          ${options.metaFields ? options.metaFields(item) : ''}
        </div>
      </div>
      <div class="catalogue-card-stats">
        ${item.nb_attributions ? `<span class="catalogue-badge blue">${item.nb_attributions}</span>` : ''}
        ${item.nb_expired ? `<span class="catalogue-badge red">${item.nb_expired}</span>` : ''}
      </div>
    </div>`;
}
```

## Attribution Modals

Pattern for assigning items to users (many-to-many via junction table):

```javascript
// Open modal with user list
async function openAttribution(type, itemId, extraData = {}) {
  // Load active users
  const users = (entityData['person'] || []).filter(u => u.actif !== 'false');

  // Populate user checkboxes
  const container = document.getElementById(`attrib-${type}-users`);
  container.innerHTML = users.map(u => `
    <div class="attribution-user-row">
      <label>
        <input type="checkbox" value="${u.id}">
        <span>${u.first_name} ${u.last_name}</span>
      </label>
    </div>
  `).join('');

  // Pre-fill date
  document.getElementById(`attrib-${type}-date`).value =
    new Date().toISOString().slice(0, 10);

  // Show modal
  document.getElementById(`modal-attrib-${type}`).style.display = 'flex';
}

// Filter users by search input
function filterAttribUsers(type) {
  const q = document.getElementById(`attrib-${type}-search`).value.toLowerCase();
  document.querySelectorAll(`#attrib-${type}-users .attribution-user-row`).forEach(row => {
    row.style.display = row.textContent.toLowerCase().includes(q) ? '' : 'none';
  });
}

// Submit attribution
async function submitAttribution(type) {
  const checked = [...document.querySelectorAll(`#attrib-${type}-users input:checked`)];
  if (!checked.length) { toast('Select at least one user', 'error'); return; }

  const date = document.getElementById(`attrib-${type}-date`).value;
  if (!date) { toast('Date is required', 'error'); return; }

  for (const cb of checked) {
    await api('crud.php?entity=item_person', {
      method: 'POST',
      body: JSON.stringify({
        item_id: currentItemId,
        person_id: parseInt(cb.value),
        date_assigned: date,
        // ... other fields from the modal
      })
    });
  }

  toast('Attribution saved', 'success');
  closeAttribModal(type);
  // Refresh the detail view
}

function closeAttribModal(type) {
  document.getElementById(`modal-attrib-${type}`).style.display = 'none';
}
```

## File Management

Upload and display files attached to any entity:

```javascript
async function uploadFile(entityType, entityId, file) {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('entity_type', entityType);
  formData.append('entity_id', entityId);

  const res = await fetch(API + 'upload.php', {
    method: 'POST',
    credentials: 'same-origin',
    headers: { 'X-CSRF-Token': csrfToken },
    body: formData
  });
  return res.json();
}

function renderFiles(files) {
  if (!files || !files.length) return '';
  return `
    <div class="files-section">
      <div class="section-title">Attached Files (${files.length})</div>
      ${files.map(f => `
        <div class="file-item">
          <i data-lucide="file" style="width:14px;height:14px"></i>
          <a href="uploads/${f.stored_name}" target="_blank">${f.original_name}</a>
          <span class="file-meta">${formatFileSize(f.file_size)}</span>
          <span class="file-del" onclick="deleteFile(${f.id})">&times;</span>
        </div>
      `).join('')}
    </div>`;
}
```

## Icons

Uses Lucide Icons via CDN. Call `refreshIcons()` after any DOM update that includes icon markup.

```html
<!-- In app.html head -->
<script src="https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"></script>

<!-- After initial render -->
<script>lucide.createIcons();</script>
```

```javascript
// Call after any innerHTML update
function refreshIcons() {
  if (typeof lucide !== 'undefined') lucide.createIcons();
}
```

**Usage in HTML strings:**
```javascript
`<i data-lucide="users"></i>`     // Users icon
`<i data-lucide="plus"></i>`      // Plus icon
`<i data-lucide="trash-2"></i>`   // Delete icon
`<i data-lucide="edit-3"></i>`    // Edit icon
`<i data-lucide="download"></i>`  // Download icon
```

Browse available icons at https://lucide.dev/icons
