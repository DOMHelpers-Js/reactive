[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Reactivity with `ClassName`, `TagName`, and `Name`

## Quick Start (30 seconds)

```javascript
const app = state({ isLoading: false, hasError: false });

effect(() => {
  ClassName.card.update({ hidden: app.isLoading });
  ClassName.skeleton.update({ hidden: !app.isLoading });
  ClassName.error.update({ hidden: !app.hasError });
});

app.isLoading = true; // ✨ Every .card, .skeleton, .error updates
```

One effect. Every element with a matching class updated together.

---

## What are `ClassName`, `TagName`, and `Name`?

DOM Helpers provides three global shortcuts for working with groups of elements:

| Global | Selects by | Example |
|--------|-----------|---------|
| `ClassName` | CSS class name | `ClassName.btn` — all elements with `class="btn"` |
| `TagName` | HTML tag | `TagName.button` — all `<button>` elements |
| `Name` | `name` attribute | `Name.email` — all `<input name="email">` |

These return **collections** — live groups of matching elements. Every method on a collection applies to **all matching elements at once**.

---

## Accessing Collections

```javascript
// All elements with class "card"
ClassName.card

// All elements with class "card item" (multi-class selector)
ClassName['card item']

// All <input> elements
TagName.input

// All elements with name="username"
Name.username
```

No `document.getElementsByClassName()`, no loops, no manual iteration.

---

## `.update({})` — Update All Matching Elements

The most important method. Applies a property object to every element in the collection.

### Syntax

```javascript
ClassName.className.update({ property: value, ... });
```

### What you can update

```javascript
ClassName.card.update({
  // Content
  textContent: 'Updated',
  innerHTML:   '<strong>Bold</strong>',

  // Visibility
  hidden: true,

  // Styling
  className: 'card active',
  classList: { add: 'active', remove: 'inactive' },
  style: { opacity: '0.5', pointerEvents: 'none' },

  // Attributes
  setAttribute: { 'aria-disabled': 'true' },
  removeAttribute: ['disabled'],

  // Form properties
  disabled: true,
  checked:  false
});
```

### Inside `effect()`

```javascript
const ui = state({ theme: 'light', isDisabled: false });

effect(() => {
  ClassName.card.update({ className: `card card--${ui.theme}` });
  ClassName.btn.update({ disabled: ui.isDisabled });
});
```

---

## Index Access — Target a Specific Element

Access a single element from a collection by its position:

```javascript
ClassName.tab[0]      // First tab
ClassName.tab[1]      // Second tab
ClassName.tab[-1]     // Last tab
```

Each indexed element has `.update()`:

```javascript
ClassName.tab[0].update({ className: 'tab active' });
ClassName.tab[-1].update({ hidden: true });
```

### Inside `effect()` with index

```javascript
const tabs = state({ activeIndex: 0 });

effect(() => {
  // Reset all tabs first, then activate the current one
  ClassName.tab.update({ className: 'tab' });
  ClassName.panel.update({ hidden: true });

  ClassName.tab[tabs.activeIndex].update({ className: 'tab active' });
  ClassName.panel[tabs.activeIndex].update({ hidden: false });
});
```

---

## `.forEach()` — Iterate Over a Collection

When you need different logic per element, use `forEach`:

```javascript
ClassName.card.forEach((element, index) => {
  element.update({ dataset: { index: String(index) } });
});
```

### Inside `effect()` with dynamic values

```javascript
const list = state({ items: ['Alpha', 'Beta', 'Gamma'] });

effect(() => {
  ClassName.list-item.forEach((el, i) => {
    el.update({ textContent: list.items[i] ?? '' });
  });
});
```

---

## `.on()` — Attach Event Listeners to a Collection

Add the same event listener to every element in a collection:

```javascript
ClassName.btn.on('click', (event) => {
  console.log('Button clicked:', event.target.textContent);
});
```

### Delegated selection within an effect

```javascript
const app = state({ selectedTab: 'home' });

// Attach listeners once — outside the effect
ClassName.tab.on('click', (e) => {
  app.selectedTab = e.target.dataset.tab;
});

// Effect handles display
effect(() => {
  ClassName.tab.update({ className: 'tab' });
  ClassName.panel.update({ hidden: true });

  ClassName[`tab--${app.selectedTab}`].update({ className: 'tab active' });
  ClassName[`panel--${app.selectedTab}`].update({ hidden: false });
});
```

---

## Common Patterns

### Toggle all elements in a group

```javascript
const app = state({ isLoggedIn: false });

effect(() => {
  ClassName['auth-only'].update({ hidden: !app.isLoggedIn });
  ClassName['guest-only'].update({ hidden: app.isLoggedIn });
});
```

### Disable all form fields while loading

```javascript
const form = state({ isSubmitting: false });

effect(() => {
  TagName.input.update({ disabled: form.isSubmitting });
  TagName.button.update({ disabled: form.isSubmitting });
  ClassName['submit-btn'].update({
    textContent: form.isSubmitting ? 'Submitting...' : 'Submit',
    className:   form.isSubmitting ? 'btn btn-disabled' : 'btn btn-primary'
  });
});
```

### Highlight active item in a list

```javascript
const nav = state({ active: 'home' });

// Reset all, then activate one
effect(() => {
  ClassName['nav-link'].update({ className: 'nav-link' });
  ClassName[`nav-link--${nav.active}`].update({ className: 'nav-link active' });
});
```

### Notification badges across multiple icons

```javascript
const notifications = state({ count: 0 });

effect(() => {
  ClassName['notification-badge'].update({
    textContent: notifications.count,
    hidden:      notifications.count === 0
  });
});
```

---

## Real-World Example: Tab Navigation

```html
<nav>
  <button class="tab" data-tab="0">Overview</button>
  <button class="tab" data-tab="1">Details</button>
  <button class="tab" data-tab="2">Reviews</button>
</nav>
<div class="panel">Overview content</div>
<div class="panel">Details content</div>
<div class="panel">Reviews content</div>
```

```javascript
const tabs = state({ active: 0 });

// Attach listeners once
ClassName.tab.forEach((tab, index) => {
  tab.addEventListener('click', () => { tabs.active = index; });
});

// Effect drives all visual state
effect(() => {
  ClassName.tab.update({ className: 'tab' });
  ClassName.panel.update({ hidden: true });

  ClassName.tab[tabs.active].update({ className: 'tab active' });
  ClassName.panel[tabs.active].update({ hidden: false });
});
```

---

## Real-World Example: Product Cards with Filter

```html
<div class="card product" data-category="electronics">Laptop</div>
<div class="card product" data-category="electronics">Phone</div>
<div class="card product" data-category="clothing">Shirt</div>
<div class="card product" data-category="clothing">Jacket</div>
```

```javascript
const filter = state({ category: 'all' });

effect(() => {
  if (filter.category === 'all') {
    ClassName.product.update({ hidden: false });
    return;
  }

  ClassName.product.forEach((card) => {
    const matches = card.dataset.category === filter.category;
    card.update({ hidden: !matches });
  });
});

// Attach filter buttons
ClassName['filter-btn'].forEach((btn) => {
  btn.addEventListener('click', () => {
    filter.category = btn.dataset.category;
  });
});
```

---

## Real-World Example: Form Field State

```javascript
const form = state({ errors: {}, isSubmitting: false });

computed(form, {
  hasErrors() { return Object.keys(this.errors).length > 0; }
});

effect(() => {
  Name.email.update({
    classList: {
      add:    form.errors.email    ? 'field-error' : '',
      remove: form.errors.email    ? '' : 'field-error'
    }
  });
  Name.password.update({
    classList: {
      add:    form.errors.password ? 'field-error' : '',
      remove: form.errors.password ? '' : 'field-error'
    }
  });
  TagName.input.update({ disabled: form.isSubmitting });
});
```

---

## `ClassName` vs `Elements` vs `Id()`

| Tool | Best for |
|------|---------|
| `Elements.id.update({})` | One specific element by `id` |
| `Id('my-id').update({})` | One specific element, explicit syntax |
| `ClassName.name.update({})` | All elements sharing a class |
| `TagName.tag.update({})` | All elements of a tag type |
| `Name.name.update({})` | All elements sharing a `name` attribute |

Use `ClassName`/`TagName`/`Name` when multiple elements need the same update. Use `Elements`/`Id()` when targeting a unique element.

---

## Key Takeaways

- `ClassName.name.update({})` — update every element with a class in one call
- `ClassName.name[0]` — access a specific element by index (supports negative indices)
- `ClassName.name.forEach(...)` — iterate when logic differs per element
- `ClassName.name.on('event', handler)` — attach listeners to every element at once
- Use `ClassName` inside `effect()` for reactive group updates
- Attach event listeners **outside** effects — only DOM state should live inside effects
