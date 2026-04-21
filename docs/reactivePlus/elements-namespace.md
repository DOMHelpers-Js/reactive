[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Reactivity with `Elements`

## Quick Start (30 seconds)

```javascript
const counter = state({ count: 0 });

effect(() => {
  Elements.update({
    count:   { textContent: counter.count },
    doubled: { textContent: counter.count * 2 },
    status:  { hidden: counter.count === 0 }
  });
});

counter.count = 5; // ✨ All three elements update in one call
```

One effect. One `Elements.update()` call. Every element in sync.

---

## What is `Elements`?

`Elements` is a DOM utility namespace from **DOM Helpers Core**. It provides clean, chainable methods for reading and updating DOM elements by their `id`.

When combined with the reactive system, `Elements` becomes your primary tool for updating the DOM inside `effect()`, `watch()`, and other reactive contexts.

**The key methods used in reactive code:**

| Method | What it does |
|--------|-------------|
| `Elements.elementId.update({})` | Update a single element by id |
| `Elements.update({})` | Update multiple elements at once |
| `Elements.elementId.value` | Read an input's current value |
| `Elements.elementId.addEventListener(...)` | Attach an event listener |

---

## Accessing Elements by ID

`Elements` uses property access to reference elements by their HTML `id`. No `#` prefix, no `document.getElementById`:

```javascript
// HTML: <span id="user-name">Alice</span>

// ❌ Plain JavaScript
document.getElementById('user-name').textContent = 'Bob';

// ✅ DOM Helpers
Elements['user-name'].update({ textContent: 'Bob' });

// For camelCase ids:
// HTML: <span id="userName">Alice</span>
Elements.userName.update({ textContent: 'Bob' });
```

---

## `.update()` — Update a Single Element

### Syntax

```javascript
Elements.elementId.update({ property: value, ... });
```

### What you can update

```javascript
Elements.myElement.update({
  // Content
  textContent: 'Hello world',
  innerHTML:   '<strong>Hello</strong>',

  // Visibility
  hidden: true,

  // Styling
  className: 'active highlighted',
  classList: { add: 'active', remove: 'inactive' },
  style: { color: 'red', fontSize: '16px' },

  // Attributes
  setAttribute: { 'aria-label': 'Close', 'data-id': '42' },
  removeAttribute: ['disabled'],

  // Form properties
  disabled: true,
  checked:  false,
  value:    'default',

  // Dataset
  dataset: { userId: '42', role: 'admin' }
});
```

### Inside `effect()`

```javascript
const user = state({ name: 'Alice', isOnline: true });

effect(() => {
  Elements.userName.update({ textContent: user.name });
  Elements.statusDot.update({ className: user.isOnline ? 'online' : 'offline' });
});
```

---

## `Elements.update({})` — Update Multiple Elements at Once

This is the **recommended pattern inside effects** — one declarative object, all elements updated together:

### Syntax

```javascript
Elements.update({
  'element-id': { property: value },
  'other-id':   { property: value }
});
```

### Example

```javascript
const dashboard = state({ users: 0, revenue: 0, orders: 0 });

effect(() => {
  Elements.update({
    'user-count':    { textContent: dashboard.users.toLocaleString() },
    'revenue-total': { textContent: `$${dashboard.revenue.toLocaleString()}` },
    'order-count':   { textContent: dashboard.orders }
  });
});

dashboard.users   = 1250; // All three update in one pass
dashboard.revenue = 45000;
```

### Why bulk updates matter

```
❌ Three separate update calls:
effect(() => {
  Elements.userCount.update({ textContent: app.users });      // call 1
  Elements.revenueTotal.update({ textContent: app.revenue }); // call 2
  Elements.orderCount.update({ textContent: app.orders });    // call 3
});

✅ One bulk update call:
effect(() => {
  Elements.update({
    userCount:    { textContent: app.users },
    revenueTotal: { textContent: app.revenue },
    orderCount:   { textContent: app.orders }
  });
});
```

Both produce the same result. The bulk form is more readable and makes the intent clearer — this effect owns exactly these elements.

---

## Common Patterns

### Show / Hide with `hidden`

Prefer `hidden` over `style.display` — it's cleaner and semantic:

```javascript
const app = state({ isLoading: true, hasError: false });

effect(() => {
  Elements.update({
    spinner:      { hidden: !app.isLoading },
    content:      { hidden: app.isLoading || app.hasError },
    errorMessage: { hidden: !app.hasError }
  });
});
```

### Conditional class names

```javascript
const form = state({ isValid: false, isDirty: false });

effect(() => {
  Elements.submitBtn.update({
    disabled:  !form.isValid,
    className: form.isValid ? 'btn btn-primary' : 'btn btn-disabled'
  });
});
```

### Ternary for text content

```javascript
const auth = state({ isLoggedIn: false, userName: '' });

effect(() => {
  Elements.update({
    'nav-greeting': {
      textContent: auth.isLoggedIn ? `Hi, ${auth.userName}` : 'Welcome'
    },
    'login-btn':  { hidden: auth.isLoggedIn },
    'logout-btn': { hidden: !auth.isLoggedIn }
  });
});
```

### Dynamic class based on value

```javascript
const score = state({ value: 0 });

effect(() => {
  const tier = score.value >= 80 ? 'gold'
             : score.value >= 50 ? 'silver'
             : 'bronze';

  Elements['score-badge'].update({
    textContent: `${score.value} pts`,
    className:   `badge badge-${tier}`
  });
});
```

---

## Reading Values from Elements

Read input values imperatively (inside event handlers, not effects):

```javascript
const search = state({ query: '' });

Elements.searchInput.addEventListener('input', () => {
  search.query = Elements.searchInput.value;
});

// Or using a form submit:
Elements.searchForm.addEventListener('submit', (e) => {
  e.preventDefault();
  search.query = Elements.searchInput.value;
});
```

---

## Real-World Example: User Profile Card

```html
<div id="profile-card">
  <img id="avatar" src="" alt="" />
  <h2 id="full-name"></h2>
  <span id="role-badge"></span>
  <span id="online-status"></span>
  <button id="follow-btn">Follow</button>
</div>
```

```javascript
const profile = state({
  name:     'Alice Johnson',
  role:     'admin',
  isOnline: true,
  avatar:   '/avatars/alice.jpg',
  followers: 142
});

effect(() => {
  Elements.update({
    'full-name': {
      textContent: profile.name
    },
    'role-badge': {
      textContent: profile.role,
      className:   `badge badge-${profile.role}`
    },
    'online-status': {
      textContent: profile.isOnline ? 'Online' : 'Offline',
      className:   profile.isOnline ? 'status online' : 'status offline'
    },
    avatar: {
      src: profile.avatar,
      alt: `${profile.name}'s avatar`
    },
    'follow-btn': {
      textContent: `Follow (${profile.followers})`
    }
  });
});

profile.isOnline = false; // All elements reflecting online status update
```

---

## Real-World Example: Form Validation

```javascript
const form = state({ email: '', password: '' });

computed(form, {
  emailValid()    { return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email); },
  passwordValid() { return this.password.length >= 8; },
  canSubmit()     { return this.emailValid && this.passwordValid; }
});

effect(() => {
  Elements.update({
    'email-error': {
      textContent: form.email && !form.emailValid ? 'Enter a valid email' : '',
      hidden:      !form.email || form.emailValid
    },
    'password-error': {
      textContent: form.password && !form.passwordValid ? 'Min 8 characters' : '',
      hidden:      !form.password || form.passwordValid
    },
    'submit-btn': {
      disabled:  !form.canSubmit,
      textContent: form.canSubmit ? 'Sign In' : 'Fill in all fields'
    }
  });
});

Elements.emailInput.addEventListener('input', () => {
  form.email = Elements.emailInput.value;
});
Elements.passwordInput.addEventListener('input', () => {
  form.password = Elements.passwordInput.value;
});
```

---

## `Elements` vs `Id()`

Both are equivalent — use whichever reads better in context:

```javascript
// Elements — good for multiple elements and bulk updates
Elements.myElement.update({ textContent: value });
Elements.update({ a: {...}, b: {...} });

// Id() — good for single targeted updates
Id('my-element').update({ textContent: value });
```

For **single** element updates, `Id()` is slightly more explicit. For **multiple** elements in one effect, `Elements.update({})` is preferred.

---

## Key Takeaways

- `Elements.elementId.update({})` — update a single element by id
- `Elements.update({})` — update multiple elements in one declarative call (preferred inside effects)
- Use `hidden: boolean` instead of `style: { display: ... }`
- Use ternaries for conditional text/class — keeps effects flat and readable
- Read input values with `Elements.inputId.value` inside event handlers
- Combine with `computed()` to keep effects clean — effects display, computed calculates
