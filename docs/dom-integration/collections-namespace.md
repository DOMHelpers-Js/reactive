[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)
[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# The Collections Namespace — `ClassName`, `TagName`, and `Name`

## Before You Start — The Problem They Solve

The `Elements` namespace is great when you need to target **one specific element** by its `id`. But what about when you need to update *many* elements that share something in common — like all buttons, all cards, or all navigation links?

In plain JavaScript, you'd write something like this:

```javascript
// Update every element with class "card"
const cards = document.querySelectorAll('.card');
cards.forEach(card => {
  card.classList.add('active');
  card.style.opacity = '1';
});
```

That's a `querySelectorAll`, a manual loop, and individual property changes — and if you want this to react to state, you have to manage all of that yourself.

The collections namespace makes this effortless. Instead of writing a loop every time, you get a clean API that works on **entire groups of elements at once**, and reacts to your state automatically inside `effect()`.

---

## What Are `ClassName`, `TagName`, and `Name`?

These are three global **collection selectors** — each one gives you access to a group of DOM elements based on a shared characteristic:

| Namespace | Selects elements by... | Example |
|---|---|---|
| `ClassName` | CSS class name | `ClassName.card` — every element with `class="card"` |
| `TagName` | HTML tag type | `TagName.button` — every `<button>` on the page |
| `Name` | `name` attribute | `Name.email` — every `<input name="email">` |

What you get back is not a single element — it's a **collection**: a live group of all matching elements. Every method you call on the collection applies to **every element inside it**, all at once, with no loop required.

---

## How to Access a Collection

Access is straightforward — use the selector as a property name, the same way you use `Elements`:

```javascript
// Every element with class "card"
ClassName.card

// Every <input> element on the page
TagName.input

// Every element with name="username"
Name.username

// Multi-word class names use bracket notation
ClassName['nav-link']
ClassName['card item']
```

Once you have the collection, everything else flows from calling methods on it.

---

## `.update({})` — Change All Matching Elements at Once

This is the most important method on any collection. It works exactly like `Elements.myId.update({})`, except it applies to **every element in the collection simultaneously**.

### Syntax

```javascript
ClassName.myClass.update({ property: value, ... });
```

### What you can update

You can change anything about the elements — content, visibility, classes, styles, attributes, and more:

```javascript
ClassName.card.update({

  // Text and content
  textContent: 'Updated',
  innerHTML:   '<strong>New content</strong>',

  // Visibility
  hidden: true,

  // CSS classes
  className: 'card card--active',
  classList: {
    add:    'highlighted',
    remove: 'dimmed'
  },

  // Inline styles
  style: {
    opacity: '0.5',
    pointerEvents: 'none'
  },

  // HTML attributes
  setAttribute: { 'aria-disabled': 'true', 'data-state': 'inactive' },
  removeAttribute: ['disabled'],

  // Form properties
  disabled: true,
  checked:  false

});
```

### Inside `effect()` — where collections shine

This is where things get powerful. Put a collection update inside an `effect()` and every matching element reacts to your state automatically:

```javascript
const ui = state({ isDisabled: false, theme: 'light' });

effect(() => {
  // Every .card gets the current theme class
  ClassName.card.update({
    className: `card card--${ui.theme}`
  });

  // Every .btn gets disabled or enabled based on state
  ClassName.btn.update({
    disabled: ui.isDisabled
  });
});

// Now when state changes, all cards and all buttons update instantly:
ui.theme      = 'dark';   // Every card switches to dark theme
ui.isDisabled = true;     // Every button becomes disabled
```

No loops. No manual DOM queries. The effect handles it.

---

## Targeting a Specific Element by Index

Sometimes you don't want to update the entire collection — you want one specific element from the group. You can access elements by their **position** using bracket notation, just like an array:

```javascript
ClassName.tab[0]    // The first tab
ClassName.tab[1]    // The second tab
ClassName.tab[-1]   // The last tab (negative index counts from the end)
```

Each indexed element has `.update()`, so you can target it precisely:

```javascript
ClassName.tab[0].update({ className: 'tab tab--active' });
ClassName.tab[-1].update({ hidden: true });
```

### Practical use — a tab switcher

Here is a common pattern: reset all tabs to their default state, then highlight just the active one:

```javascript
const tabs = state({ activeIndex: 0 });

effect(() => {
  // Step 1: Reset everything to its default state
  ClassName.tab.update({ className: 'tab' });
  ClassName.panel.update({ hidden: true });

  // Step 2: Activate just the current tab and panel
  ClassName.tab[tabs.activeIndex].update({ className: 'tab tab--active' });
  ClassName.panel[tabs.activeIndex].update({ hidden: false });
});
```

When `tabs.activeIndex` changes, the effect re-runs: all tabs reset, then the right one activates. Clean and predictable.

---

## `.forEach()` — When Each Element Needs Different Logic

`.update({})` is perfect when all elements get the same value. But sometimes each element needs something different based on its position or its own data. That's what `.forEach()` is for.

```javascript
// Give each list item its index as a data attribute
ClassName['list-item'].forEach((element, index) => {
  element.update({ dataset: { index: String(index) } });
});
```

The callback receives two arguments:
- `element` — a single element with `.update()` available on it
- `index` — its position in the collection (starting from 0)

### Inside `effect()` with dynamic data

```javascript
const list = state({ items: ['Alpha', 'Beta', 'Gamma'] });

effect(() => {
  ClassName['list-item'].forEach((el, i) => {
    el.update({ textContent: list.items[i] ?? '' });
  });
});
```

When `list.items` changes, the effect re-runs and every list item gets its correct text.

---

## `.on()` — Attach Event Listeners to Every Element at Once

Instead of looping through elements to attach event listeners, `.on()` does it for the whole collection in one call:

```javascript
// Attach the same click listener to every button with class "btn"
ClassName.btn.on('click', (event) => {
  console.log('Clicked:', event.target.textContent);
});
```

**Important:** Attach event listeners **outside** of `effect()`. Listeners only need to be set up once. Effects are for reacting to state — not for attaching handlers:

```javascript
const app = state({ selectedTab: 'home' });

// ✅ Listeners go outside the effect — they only need to be attached once
ClassName.tab.on('click', (e) => {
  app.selectedTab = e.target.dataset.tab;
});

// ✅ The effect handles what changes visually
effect(() => {
  ClassName.tab.update({ className: 'tab' });
  ClassName.panel.update({ hidden: true });

  ClassName[`tab--${app.selectedTab}`].update({ className: 'tab tab--active' });
  ClassName[`panel--${app.selectedTab}`].update({ hidden: false });
});
```

---

## Common Patterns You Will Use Every Day

### Show guest vs. authenticated content

```javascript
const auth = state({ isLoggedIn: false });

effect(() => {
  ClassName['auth-only'].update({ hidden: !auth.isLoggedIn });
  ClassName['guest-only'].update({ hidden: auth.isLoggedIn });
});
```

When `auth.isLoggedIn` becomes `true`, every element marked `auth-only` appears and every element marked `guest-only` disappears. Perfect for nav menus, dashboards, and gated sections.

---

### Disable all form inputs while submitting

```javascript
const form = state({ isSubmitting: false });

effect(() => {
  TagName.input.update({ disabled: form.isSubmitting });
  TagName.button.update({ disabled: form.isSubmitting });

  ClassName['submit-btn'].update({
    textContent: form.isSubmitting ? 'Submitting...' : 'Submit',
    className:   form.isSubmitting ? 'btn btn--loading' : 'btn btn--primary'
  });
});
```

Every input and button on the page disables the moment `form.isSubmitting` is set to `true`. No need to find them individually.

---

### Highlight the active nav link

```javascript
const nav = state({ active: 'home' });

effect(() => {
  // First reset all links to their default state
  ClassName['nav-link'].update({ className: 'nav-link' });

  // Then activate only the current one
  ClassName[`nav-link--${nav.active}`].update({ className: 'nav-link nav-link--active' });
});
```

---

### Notification badge across multiple icons

```javascript
const notifications = state({ count: 0 });

effect(() => {
  ClassName['notification-badge'].update({
    textContent: notifications.count,
    hidden:      notifications.count === 0
  });
});
```

Every badge on the page — in the header, sidebar, mobile menu — all stay in sync from one state value.

---

## Real-World Example: Tab Navigation

A complete tab system — HTML, state, listeners, and effect.

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

// Attach click listeners to every tab — done once, outside the effect
ClassName.tab.forEach((tab, index) => {
  tab.addEventListener('click', () => {
    tabs.active = index;
  });
});

// The effect drives all the visual state
effect(() => {
  // Reset all to default
  ClassName.tab.update({ className: 'tab' });
  ClassName.panel.update({ hidden: true });

  // Activate the current tab and show its panel
  ClassName.tab[tabs.active].update({ className: 'tab tab--active' });
  ClassName.panel[tabs.active].update({ hidden: false });
});
```

Click a tab → `tabs.active` changes → effect re-runs → DOM updates. That's the whole system.

---

## Real-World Example: Product Filter

Show only the products that match the selected category, hide the rest.

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
    // Show everything when no filter is active
    ClassName.product.update({ hidden: false });
    return;
  }

  // Otherwise check each card's category and show or hide accordingly
  ClassName.product.forEach((card) => {
    const matches = card.dataset.category === filter.category;
    card.update({ hidden: !matches });
  });
});

// Attach filter button listeners once
ClassName['filter-btn'].forEach((btn) => {
  btn.addEventListener('click', () => {
    filter.category = btn.dataset.category;
  });
});
```

---

## `ClassName` vs `Elements` vs `Id()` — When to Use Which

| You want to target... | Use |
|---|---|
| One specific element by `id` | `Elements.myId.update({})` or `Id('my-id').update({})` |
| All elements with a particular class | `ClassName.myClass.update({})` |
| All elements of a specific HTML tag | `TagName.input.update({})` |
| All elements with a specific `name` attribute | `Name.fieldName.update({})` |

A good rule of thumb: if the element has a unique `id`, use `Elements`. If multiple elements share a characteristic, use `ClassName`, `TagName`, or `Name`.

---

## Key Takeaways

| What you want to do | How to do it |
|---|---|
| Update all elements with a class | `ClassName.name.update({ ... })` |
| Update all elements by tag | `TagName.tag.update({ ... })` |
| Update all elements by name attribute | `Name.field.update({ ... })` |
| Target one element in a collection | `ClassName.name[0]` or `ClassName.name[-1]` |
| Run different logic per element | `ClassName.name.forEach((el, i) => ...)` |
| Attach a listener to all elements | `ClassName.name.on('event', handler)` |

- `ClassName`, `TagName`, and `Name` are for groups — `Elements` is for individuals.
- Put `.update()` calls inside `effect()` for automatic reactivity.
- Attach event listeners **outside** of effects — they only need to run once.
- Use `.forEach()` when elements need different values based on their position or data.
