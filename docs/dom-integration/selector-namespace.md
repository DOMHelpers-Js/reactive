[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)
[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# The Selector Namespace — `query` and `queryAll`

## Before You Start — The Problem They Solve

`Elements` targets elements by `id`. `ClassName`, `TagName`, and `Name` target elements by shared characteristics. Both approaches cover the vast majority of real-world cases — but occasionally you need more power.

What if you need to target:
- All inputs that are **required** — `input[required]`
- The first paragraph **inside** a specific container — `.modal p:first-child`
- Every element that has a **data attribute** set to a specific value — `[data-status="pending"]`

These are complex CSS selectors. `Elements` and `ClassName` can't express them. This is where `query` and `queryAll` come in.

They give you the **full power of CSS selectors** — any selector that works in `document.querySelector()` works here — while keeping the same clean `.update()` API you already know.

---

## What Are `query` and `queryAll`?

They are two global functions that let you find DOM elements using any valid CSS selector:

| Function | What it returns | Plain JS equivalent |
|---|---|---|
| `query(selector)` | A **single** element with `.update()` | `document.querySelector(selector)` |
| `queryAll(selector)` | A **collection** of elements with `.update()`, `.forEach()`, etc. | `document.querySelectorAll(selector)` |

The key difference from plain JavaScript: the elements they return already have `.update()` and all the other DOM Helpers methods on them. You don't need to manually set properties — just call `.update({})` the same way you do everywhere else.

---

## `query()` — Find One Element With a CSS Selector

Use `query()` when you need **exactly one element** and you need a selector more precise than just an `id`.

### Syntax

```javascript
query(selector)
query(selector, context)  // Search only within a specific container element
```

### Basic examples

```javascript
// Find by attribute value
query('[data-role="primary"]').update({ className: 'btn btn--primary' });

// Find by pseudo-class
query('input:first-of-type').update({ value: '' });

// Find by compound selector (element inside a specific parent)
query('.form-group > input.required').update({
  setAttribute: { 'aria-required': 'true' }
});
```

### Inside `effect()` — reacting to state

```javascript
const form = state({ activeField: 'email' });

effect(() => {
  // Deactivate all fields first
  queryAll('.field').update({ className: 'field' });

  // Then activate only the currently focused field
  query(`.field[data-name="${form.activeField}"]`).update({
    className: 'field field--active'
  });
});
```

When `form.activeField` changes, the effect re-runs: all fields reset, then the matching one activates.

---

## `queryAll()` — Find Many Elements With a CSS Selector

Use `queryAll()` when you need to target **a group of elements** using a CSS selector that `ClassName` or `TagName` can't express.

### Syntax

```javascript
queryAll(selector)
queryAll(selector, context)  // Search only within a specific container element
```

### Basic examples

```javascript
// All elements with the "card" class (equivalent to ClassName.card, but selector-based)
queryAll('.card').update({ hidden: false });

// All required inputs — something TagName.input can't filter for
queryAll('input[required]').update({
  setAttribute: { 'aria-required': 'true' }
});

// All elements in a specific state (tracked via data attributes)
queryAll('[data-status="pending"]').update({
  className: 'item item--pending'
});
```

---

## Methods Available on `queryAll()` Results

The result of `queryAll()` behaves exactly like a `ClassName` or `TagName` collection. All the same methods are available:

| Method | What it does |
|---|---|
| `.update({})` | Apply property updates to every matched element |
| `.forEach(fn)` | Iterate with a callback — each element gets its own logic |
| `.on(event, fn)` | Attach an event listener to every matched element |
| `[index]` | Access a single element by its position |
| `.first()` | Get the first matched element |
| `.last()` | Get the last matched element |
| `.length` | The number of elements matched |
| `.isEmpty()` | `true` if no elements matched |
| `.filter(fn)` | Filter the collection down based on a condition |
| `.map(fn)` | Map over matched elements and return new values |

---

## Scoped Queries — Searching Inside a Container

Both `query()` and `queryAll()` accept an optional second argument: a **context**. This limits the search to elements *inside* that container, instead of searching the whole page.

This is useful when you have repeated patterns on the same page — like multiple modal dialogs or multiple card components — and you only want to update elements inside a specific one.

```javascript
// Find elements only inside #sidebar
const sidebar = query('#sidebar');
queryAll('.nav-link', sidebar).update({ className: 'nav-link' });

// Or pass a selector string as context
queryAll('.nav-link', '#sidebar').update({ className: 'nav-link' });
```

There are also dedicated scoped functions for convenience:

```javascript
// queryWithin and queryAllWithin are explicit about the scoping intent
queryWithin('#sidebar', '.nav-link')       // → single element inside #sidebar
queryAllWithin('#sidebar', '.nav-link')    // → collection inside #sidebar
```

### Inside `effect()` — scoped and reactive

```javascript
const sidebar = state({ activeSection: 'overview' });

effect(() => {
  // Reset all links in the sidebar
  queryAllWithin('#sidebar', '.nav-link').update({ className: 'nav-link' });

  // Activate the one matching the current section
  queryWithin('#sidebar', `[data-section="${sidebar.activeSection}"]`)
    .update({ className: 'nav-link nav-link--active' });
});
```

---

## Common Patterns You Will Use Every Day

### Show and hide sections by data attribute

This is a clean pattern for simple routing — each page section has a `data-page` attribute, and only the active one is shown:

```javascript
const router = state({ page: 'home' });

effect(() => {
  // Hide all pages first
  queryAll('[data-page]').update({ hidden: true });

  // Show only the current one
  query(`[data-page="${router.page}"]`).update({ hidden: false });
});
```

To navigate, just update the state:

```javascript
router.page = 'about';   // Instantly shows the about section, hides the rest
router.page = 'contact'; // Shows contact, hides everything else
```

---

### Set all form fields to read-only mode

```javascript
const form = state({ isReadonly: false });

effect(() => {
  // Target inputs, textareas, and selects all at once with a compound selector
  queryAll('input, textarea, select').update({ disabled: form.isReadonly });

  query('.form-title').update({
    textContent: form.isReadonly ? 'Viewing (read-only)' : 'Editing'
  });
});
```

A single state change puts the entire form into read-only mode — or brings it back to edit mode.

---

### Update elements inside a specific container only

Useful when you have multiple instances of the same component on the page:

```javascript
const modal = state({ isOpen: false, title: '' });

effect(() => {
  Elements.modalOverlay.update({ hidden: !modal.isOpen });

  if (modal.isOpen) {
    // Only update elements *inside* the modal dialog
    query('#modal-dialog .modal-title').update({ textContent: modal.title });
    queryAll('#modal-dialog .close-btn').on('click', () => {
      modal.isOpen = false;
    });
  }
});
```

---

### Highlight search results

Filter a list by whether each item contains the search text:

```javascript
const search = state({ query: '' });

effect(() => {
  if (!search.query) {
    // No search active — show everything in its default state
    queryAll('[data-searchable]').update({ className: 'result', hidden: false });
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

As the user types into a search field and updates `search.query`, the list instantly filters.

---

## Real-World Example: Form Validation Display

Highlight invalid fields and show their error messages — driven entirely by state:

```html
<div class="field-wrapper" data-field="email">
  <input name="email" type="email" />
  <p class="field-error" data-field="email" hidden></p>
</div>
<div class="field-wrapper" data-field="password">
  <input name="password" type="password" />
  <p class="field-error" data-field="password" hidden></p>
</div>
```

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
  // Step 1: Reset all field wrappers and error messages
  queryAll('.field-wrapper').update({ className: 'field-wrapper' });
  queryAll('.field-error').update({ hidden: true, textContent: '' });

  // Step 2: Apply error styling to only the invalid fields
  form.invalidFields.forEach((fieldName) => {
    query(`.field-wrapper[data-field="${fieldName}"]`)
      .update({ className: 'field-wrapper field-wrapper--error' });

    query(`.field-error[data-field="${fieldName}"]`)
      .update({
        hidden:      false,
        textContent: form.fields[fieldName].message
      });
  });
});
```

---

## Real-World Example: Permission-Based UI

Show or hide UI sections based on the user's role and permissions — a pattern common in dashboards and admin panels:

```html
<button data-requires-role="admin">Delete User</button>
<section data-requires-permission="analytics">Analytics Dashboard</section>
<section data-requires-role="any">Welcome Message</section>
```

```javascript
const user = state({ role: 'guest', permissions: [] });

effect(() => {
  // Start by hiding everything that is role or permission gated
  queryAll('[data-requires-role]').update({ hidden: true });
  queryAll('[data-requires-permission]').update({ hidden: true });

  // Show what the user's role allows
  queryAll(`[data-requires-role="${user.role}"], [data-requires-role="any"]`)
    .update({ hidden: false });

  // Show what the user's individual permissions allow
  user.permissions.forEach((permission) => {
    queryAll(`[data-requires-permission="${permission}"]`).update({ hidden: false });
  });
});

// When the user's role or permissions change, the entire UI re-evaluates:
user.role        = 'admin';
user.permissions = ['analytics', 'exports'];
```

---

## Real-World Example: Accordion

A fully reactive accordion where clicking a section expands it and collapses any previously open section:

```html
<div class="accordion-item" data-index="0">
  <button class="accordion-trigger">Section 1</button>
  <div class="accordion-panel">Content for section 1</div>
</div>
<div class="accordion-item" data-index="1">
  <button class="accordion-trigger">Section 2</button>
  <div class="accordion-panel">Content for section 2</div>
</div>
<div class="accordion-item" data-index="2">
  <button class="accordion-trigger">Section 3</button>
  <div class="accordion-panel">Content for section 3</div>
</div>
```

```javascript
const accordion = state({ openIndex: null });

// Attach click listeners to every trigger — done once, outside the effect
queryAll('.accordion-trigger').forEach((trigger, index) => {
  trigger.addEventListener('click', () => {
    // Toggle: clicking the open section closes it, clicking another opens it
    accordion.openIndex = accordion.openIndex === index ? null : index;
  });
});

// The effect drives all visual state
effect(() => {
  // Step 1: Collapse everything
  queryAll('.accordion-panel').update({ hidden: true });
  queryAll('.accordion-trigger').update({
    setAttribute: { 'aria-expanded': 'false' }
  });

  // Step 2: Expand the open one (if any)
  if (accordion.openIndex !== null) {
    query(`.accordion-item[data-index="${accordion.openIndex}"] .accordion-panel`)
      .update({ hidden: false });

    query(`.accordion-item[data-index="${accordion.openIndex}"] .accordion-trigger`)
      .update({ setAttribute: { 'aria-expanded': 'true' } });
  }
});
```

Click a trigger → `accordion.openIndex` updates → effect re-runs → only the right panel opens.

---

## When to Use `query` / `queryAll` vs. the Others

Start with `Elements` and `ClassName` for the most common cases. Reach for `query` and `queryAll` when you need selectors they can't express.

| You need... | Use |
|---|---|
| One element by `id` | `Elements.myId.update({})` |
| All elements with a class | `ClassName.name.update({})` |
| All elements of a tag | `TagName.tag.update({})` |
| A compound selector, attribute selector, or pseudo-class | `query()` / `queryAll()` |
| Elements inside a specific container only | `queryWithin()` / `queryAllWithin()` |

A practical decision guide:
- **Does the element have a unique `id`?** → `Elements`
- **Do multiple elements share a class?** → `ClassName`
- **Do you need a CSS selector?** → `query` / `queryAll`
- **Do you need it scoped to a container?** → `queryWithin` / `queryAllWithin`

---

## Key Takeaways

| What you want to do | How to do it |
|---|---|
| Find one element by any CSS selector | `query('.my-selector').update({ ... })` |
| Find many elements by any CSS selector | `queryAll('.my-selector').update({ ... })` |
| Search only inside a container | `queryAll('.item', context)` or `queryAllWithin('#container', '.item')` |
| Iterate with different logic per element | `queryAll('.item').forEach((el, i) => ...)` |
| Attach listeners to all matched elements | `queryAll('.btn').on('click', handler)` |

- `query` and `queryAll` are the escape hatch — use them when `Elements` and `ClassName` are not flexible enough.
- They return the same `.update()` API you already know, so there is nothing new to learn.
- Put `.update()` calls inside `effect()` for automatic reactivity.
- Attach event listeners **outside** of effects — they only need to run once.
- For simple cases, always prefer `Elements` or `ClassName` — they are more readable and show intent more clearly.
