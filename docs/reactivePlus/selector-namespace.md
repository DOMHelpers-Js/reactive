[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Reactivity with `query` and `queryAll`

## Quick Start (30 seconds)

```javascript
const app = state({ isAdmin: false });

effect(() => {
  queryAll('.admin-section').update({ hidden: !app.isAdmin });
  queryAll('.guest-section').update({ hidden: app.isAdmin });
});

app.isAdmin = true; // ✨ Every .admin-section shows, every .guest-section hides
```

Full CSS selector power. Inside a reactive effect.

---

## What are `query` and `queryAll`?

DOM Helpers provides two global functions for CSS selector-based queries:

| Function | Returns | Equivalent |
|----------|---------|------------|
| `query(selector)` | Single element with `.update()` | `document.querySelector()` + `.update()` |
| `queryAll(selector)` | Collection with `.update()`, `.forEach()`, etc. | `document.querySelectorAll()` + library methods |

These are the escape hatch when `Elements` and `ClassName` are not flexible enough — any valid CSS selector works.

---

## `query()` — Single Element

### Syntax

```javascript
query(selector)
query(selector, context)  // Search within a container element
```

### Basic usage

```javascript
// Select by attribute
query('[data-role="primary"]').update({ className: 'btn btn-primary' });

// Select by pseudo-class
query('input:first-of-type').update({ value: '' });

// Select by complex selector
query('.form-group > input.required').update({ setAttribute: { 'aria-required': 'true' } });
```

### Inside `effect()`

```javascript
const form = state({ activeField: 'email' });

effect(() => {
  // Deactivate all, then activate the focused one
  queryAll('.field').update({ className: 'field' });
  query(`.field[data-name="${form.activeField}"]`).update({ className: 'field field--active' });
});
```

---

## `queryAll()` — Multiple Elements

### Syntax

```javascript
queryAll(selector)
queryAll(selector, context)  // Search within a container element
```

### Basic usage

```javascript
queryAll('.card').update({ hidden: false });
queryAll('input[required]').update({ setAttribute: { 'aria-required': 'true' } });
queryAll('[data-status="pending"]').update({ className: 'item item--pending' });
```

---

## Collection Methods on `queryAll()`

The result of `queryAll()` has the same methods as `ClassName` collections:

| Method | What it does |
|--------|-------------|
| `.update({})` | Apply property updates to all matched elements |
| `.forEach(fn)` | Iterate with a callback |
| `.on(event, fn)` | Attach event listeners to all matched |
| `[index]` | Access a single element by position |
| `.at(index)` | Access by position (supports negative index) |
| `.first()` | First matched element |
| `.last()` | Last matched element |
| `.length` | Number of matched elements |
| `.isEmpty()` | True if no elements matched |
| `.filter(fn)` | Filter matched elements |
| `.map(fn)` | Map over matched elements |

---

## Common Patterns

### Show/hide sections by attribute

```javascript
const router = state({ page: 'home' });

effect(() => {
  queryAll('[data-page]').update({ hidden: true });
  query(`[data-page="${router.page}"]`).update({ hidden: false });
});
```

### Bulk attribute update

```javascript
const form = state({ isReadonly: false });

effect(() => {
  queryAll('input, textarea, select').update({ disabled: form.isReadonly });
  query('.form-title').update({
    textContent: form.isReadonly ? 'View Mode' : 'Edit Mode'
  });
});
```

### Scoped updates within a container

```javascript
const modal = state({ isOpen: false, content: '' });

effect(() => {
  Elements.modalOverlay.update({ hidden: !modal.isOpen });

  if (modal.isOpen) {
    // Update elements inside the modal only
    const modalEl = query('#modal-dialog');
    queryAll('p', modalEl).update({ textContent: modal.content });
    queryAll('.close-btn', modalEl).on('click', () => { modal.isOpen = false; });
  }
});
```

### Highlight matching search results

```javascript
const search = state({ query: '' });

effect(() => {
  if (!search.query) {
    queryAll('[data-searchable]').update({ className: 'result' });
    return;
  }

  queryAll('[data-searchable]').forEach((el) => {
    const matches = el.textContent.toLowerCase().includes(search.query.toLowerCase());
    el.update({
      hidden:    !matches,
      className: matches ? 'result result--match' : 'result'
    });
  });
});
```

---

## `queryWithin()` and `queryAllWithin()` — Scoped Queries

When you need to scope a query to a specific container:

```javascript
// Find within a container element or selector string
queryWithin('#sidebar', '.nav-link')          // → single element
queryAllWithin('#sidebar', '.nav-link')       // → collection

// Or pass the element directly
const sidebar = query('#sidebar');
queryAllWithin(sidebar, '.nav-link').update({ className: 'nav-link' });
```

### Inside `effect()`

```javascript
const sidebar = state({ activeSection: 'overview' });

effect(() => {
  queryAllWithin('#sidebar', '.nav-link').update({ className: 'nav-link' });
  queryWithin('#sidebar', `[data-section="${sidebar.activeSection}"]`)
    .update({ className: 'nav-link active' });
});
```

---

## Real-World Example: Dynamic Form Validation Display

```javascript
const form = state({ fields: {} });

computed(form, {
  invalidFields() {
    return Object.entries(this.fields)
      .filter(([, v]) => !v.valid)
      .map(([k]) => k);
  }
});

effect(() => {
  // Reset all fields
  queryAll('.field-wrapper').update({ className: 'field-wrapper' });
  queryAll('.field-error').update({ hidden: true, textContent: '' });

  // Apply error state to invalid fields
  form.invalidFields.forEach((fieldName) => {
    query(`.field-wrapper[data-field="${fieldName}"]`)
      .update({ className: 'field-wrapper field-wrapper--error' });
    query(`.field-error[data-field="${fieldName}"]`)
      .update({ hidden: false, textContent: form.fields[fieldName].message });
  });
});
```

---

## Real-World Example: Permission-Based UI

```javascript
const user = state({ role: 'guest', permissions: [] });

effect(() => {
  // Hide everything role-gated first
  queryAll('[data-requires-role]').update({ hidden: true });
  queryAll('[data-requires-permission]').update({ hidden: true });

  // Show what the current role allows
  queryAll(`[data-requires-role="${user.role}"], [data-requires-role="any"]`)
    .update({ hidden: false });

  // Show what the user's permissions allow
  user.permissions.forEach((perm) => {
    queryAll(`[data-requires-permission="${perm}"]`).update({ hidden: false });
  });
});
```

---

## Real-World Example: Accordion

```html
<div class="accordion-item" data-index="0">
  <button class="accordion-trigger">Section 1</button>
  <div class="accordion-panel">Content 1</div>
</div>
<div class="accordion-item" data-index="1">
  <button class="accordion-trigger">Section 2</button>
  <div class="accordion-panel">Content 2</div>
</div>
```

```javascript
const accordion = state({ openIndex: null });

// Attach triggers
queryAll('.accordion-trigger').forEach((trigger, index) => {
  trigger.addEventListener('click', () => {
    accordion.openIndex = accordion.openIndex === index ? null : index;
  });
});

// Effect drives state
effect(() => {
  queryAll('.accordion-panel').update({ hidden: true });
  queryAll('.accordion-trigger').update({ setAttribute: { 'aria-expanded': 'false' } });

  if (accordion.openIndex !== null) {
    query(`.accordion-item[data-index="${accordion.openIndex}"] .accordion-panel`)
      .update({ hidden: false });
    query(`.accordion-item[data-index="${accordion.openIndex}"] .accordion-trigger`)
      .update({ setAttribute: { 'aria-expanded': 'true' } });
  }
});
```

---

## When to Use What

| Need | Use |
|------|-----|
| One element by `id` | `Elements.id.update({})` or `Id('id').update({})` |
| All elements with a class | `ClassName.name.update({})` |
| All elements with a tag | `TagName.tag.update({})` |
| Complex selector (attribute, pseudo, compound) | `query()` / `queryAll()` |
| Scoped query inside a container | `queryWithin()` / `queryAllWithin()` |

Start with `Elements` and `ClassName` for the common cases. Reach for `query`/`queryAll` when you need the full expressiveness of CSS selectors.

---

## Key Takeaways

- `query(selector)` — select one element with full CSS selector support, returns it with `.update()`
- `queryAll(selector)` — select many elements, returns a collection with `.update()`, `.forEach()`, `.on()`
- `queryWithin(container, selector)` / `queryAllWithin(container, selector)` — scope queries to a container
- Use `queryAll()` inside `effect()` for any selector pattern `ClassName` or `TagName` can't express
- Attach event listeners **outside** effects using `.on()` — only reactive DOM updates go inside effects
