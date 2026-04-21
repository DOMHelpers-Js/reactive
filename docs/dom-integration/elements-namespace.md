[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)
[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# The `Elements` Namespace

## Before You Start — The Problem It Solves

If you've ever written plain JavaScript to update the DOM, you've written code like this:

```javascript
document.getElementById('username').textContent = 'Alice';
document.getElementById('status').className = 'online';
document.getElementById('avatar').src = '/images/alice.jpg';
```

That's three separate calls, three times writing `document.getElementById`, and three different places where your UI can fall out of sync with your data. Now multiply that across a whole application — it becomes hard to read, hard to maintain, and easy to miss an element.

`Elements` is the answer to this problem. It gives you a clean, readable way to talk to your DOM elements — and when you combine it with `effect()`, your UI stays automatically in sync with your state. No manual updates. No forgetting to refresh something. It just works.

---

## What Is `Elements`?

`Elements` is a **namespace** — think of it as a smart object that knows about every element on your page that has an `id`. Instead of calling `document.getElementById('myElement')` every time, you simply write `Elements.myElement`.

It comes from **DOM Helpers Core** and is designed to work hand-in-hand with the reactive system. When you use `Elements` inside an `effect()`, the DOM updates automatically every time your state changes.

**The mental model is simple:**

> Your state holds the data. `effect()` watches for changes. `Elements` applies those changes to the DOM.

```
state changes → effect() runs → Elements updates the DOM
```

You never have to manually touch the DOM again.

---

## How to Access an Element

Every HTML element that has an `id` attribute is accessible through `Elements`. You just use the `id` as a property name — no quotes, no `#`, no `document` calls.

```html
<!-- Your HTML -->
<h1 id="title">Hello</h1>
<span id="username">Guest</span>
<button id="loginBtn">Log In</button>
```

```javascript
// Plain JavaScript (the old way)
document.getElementById('title').textContent = 'Welcome';
document.getElementById('username').textContent = 'Alice';

// With Elements (the clean way)
Elements.title.update({ textContent: 'Welcome' });
Elements.username.update({ textContent: 'Alice' });
```

If your `id` contains hyphens (like `my-element`), use bracket notation — the same way you access a property with special characters in plain JavaScript:

```javascript
// HTML: <span id="user-name">Guest</span>
Elements['user-name'].update({ textContent: 'Alice' });
```

If it's camelCase, dot notation works fine:

```javascript
// HTML: <span id="userName">Guest</span>
Elements.userName.update({ textContent: 'Alice' });
```

---

## `.update()` — Changing What an Element Shows

The `.update()` method is how you change anything about an element. You pass it an object where each key is a property you want to change and each value is what you want it to be.

Think of it like filling out a form: "for this element, set *these* properties to *these* values."

### Syntax

```javascript
Elements.myElement.update({
  propertyToChange: newValue,
  anotherProperty: anotherValue
});
```

### Everything You Can Change

Here is a complete reference of what `.update()` accepts:

```javascript
Elements.myElement.update({

  // --- Text and HTML Content ---
  textContent: 'Plain text here',        // Sets visible text (safe — no HTML parsed)
  innerHTML: '<strong>Bold text</strong>', // Sets HTML content (use with trusted data only)

  // --- Visibility ---
  hidden: true,   // Hides the element (like display: none)
  hidden: false,  // Shows the element

  // --- CSS Classes ---
  className: 'card active highlighted',   // Replaces the entire class list
  classList: {
    add: 'active',       // Adds a class without removing others
    remove: 'inactive',  // Removes a specific class
  },

  // --- Inline Styles ---
  style: {
    color: 'red',
    fontSize: '18px',
    backgroundColor: '#f0f0f0'
  },

  // --- HTML Attributes ---
  setAttribute: {
    'aria-label': 'Close dialog',
    'data-user-id': '42'
  },
  removeAttribute: ['disabled', 'readonly'],

  // --- Form-Specific Properties ---
  disabled: true,     // Disables a button or input
  checked: false,     // Sets a checkbox state
  value: 'default',   // Sets an input's value

  // --- Data Attributes ---
  dataset: {
    userId: '42',     // Sets data-user-id="42"
    role: 'admin'     // Sets data-role="admin"
  }

});
```

You don't need to use all of these at once — just include the ones that apply to what you're doing.

### A Practical Example

Imagine you have a profile card and the user's status changes. Here is how you'd update it:

```html
<div id="profile-card">
  <span id="status-text">Offline</span>
  <span id="status-dot"></span>
</div>
```

```javascript
const user = state({ isOnline: false });

effect(() => {
  Elements['status-text'].update({
    textContent: user.isOnline ? 'Online' : 'Offline'
  });

  Elements['status-dot'].update({
    className: user.isOnline ? 'dot dot--green' : 'dot dot--grey'
  });
});

// Later, when the user comes online:
user.isOnline = true;
// → status-text instantly shows "Online"
// → status-dot instantly gets class "dot dot--green"
```

No manual DOM calls needed. The `effect()` detects that `user.isOnline` changed and re-runs, and `Elements` applies the new values.

---

## `Elements.update({})` — Updating Multiple Elements at Once

When your state changes often affect more than one element on the page (which is most of the time), there is a better form of `.update()` — the **bulk update**. Instead of calling `.update()` separately for each element, you call `Elements.update()` once and pass *all* your elements together.

### Syntax

```javascript
Elements.update({
  'element-id': { propertyToChange: value },
  'other-id':   { propertyToChange: value }
});
```

### Why This Is Better

Compare these two approaches — they produce the same result, but one is clearly easier to read and reason about:

```javascript
// ❌ Three separate calls — harder to see what this effect controls
effect(() => {
  Elements.userCount.update({ textContent: dashboard.users });
  Elements.revenueTotal.update({ textContent: `$${dashboard.revenue}` });
  Elements.orderCount.update({ textContent: dashboard.orders });
});

// ✅ One bulk call — you can see at a glance exactly what this effect manages
effect(() => {
  Elements.update({
    userCount:    { textContent: dashboard.users },
    revenueTotal: { textContent: `$${dashboard.revenue}` },
    orderCount:   { textContent: dashboard.orders }
  });
});
```

The bulk form makes your effects read like a **declaration**: "this effect is responsible for these elements." That clarity pays off as your codebase grows.

### A Full Dashboard Example

```html
<div id="user-count">0</div>
<div id="revenue-total">$0</div>
<div id="order-count">0</div>
```

```javascript
const dashboard = state({ users: 0, revenue: 0, orders: 0 });

effect(() => {
  Elements.update({
    'user-count':    { textContent: dashboard.users.toLocaleString() },
    'revenue-total': { textContent: `$${dashboard.revenue.toLocaleString()}` },
    'order-count':   { textContent: dashboard.orders }
  });
});

// Updating any of these triggers the effect and refreshes all three elements
dashboard.users   = 1250;
dashboard.revenue = 45000;
dashboard.orders  = 312;
```

---

## Common Patterns You Will Use Every Day

### Show and Hide Elements

The cleanest way to show or hide something is with the `hidden` property. It maps directly to the HTML `hidden` attribute and is more readable than setting `style.display`.

```javascript
const app = state({ isLoading: true, hasError: false });

effect(() => {
  Elements.update({
    spinner:       { hidden: !app.isLoading },   // Show spinner while loading
    mainContent:   { hidden: app.isLoading },    // Hide content while loading
    errorMessage:  { hidden: !app.hasError }     // Show error only when there is one
  });
});
```

**Reading it:** "spinner is hidden when we are NOT loading. Content is hidden when we ARE loading. Error is hidden when there is NO error." The logic maps directly to your intent.

---

### Conditional Text

Use a ternary expression (`condition ? valueIfTrue : valueIfFalse`) to switch between two pieces of text based on state:

```javascript
const auth = state({ isLoggedIn: false, userName: '' });

effect(() => {
  Elements.update({
    greeting: {
      textContent: auth.isLoggedIn ? `Welcome back, ${auth.userName}!` : 'Hello, Guest'
    },
    'login-btn':  { hidden: auth.isLoggedIn },   // Hide login when logged in
    'logout-btn': { hidden: !auth.isLoggedIn }   // Hide logout when logged out
  });
});
```

---

### Conditional CSS Classes

Switch between two class names based on a condition — perfect for active states, themes, or validation:

```javascript
const form = state({ isValid: false });

effect(() => {
  Elements.submitBtn.update({
    disabled:  !form.isValid,
    className: form.isValid ? 'btn btn--primary' : 'btn btn--disabled'
  });
});
```

When `form.isValid` becomes `true`, the button is immediately enabled and switches to the primary style. When it becomes `false`, it disables and switches back. No extra logic needed.

---

### Classes Based on a Value (More Than Two Options)

When you have more than two possible states, compute the value before calling `.update()`:

```javascript
const score = state({ value: 0 });

effect(() => {
  // Calculate the tier first — keeps the update call clean
  const tier = score.value >= 80 ? 'gold'
             : score.value >= 50 ? 'silver'
             : 'bronze';

  Elements['score-badge'].update({
    textContent: `${score.value} pts`,
    className:   `badge badge--${tier}`
  });
});
```

---

## Reading Values From the DOM

`Elements` is not just for writing to the DOM — you can also read from it. This is most useful when you need to get the current value of an input field.

**Important rule:** Read DOM values inside **event handlers**, not inside `effect()`. Effects are for writing to the DOM. Event handlers are for reading from it and updating state.

```javascript
const search = state({ query: '' });

// ✅ Read inside an event handler — correct
Elements.searchInput.addEventListener('input', () => {
  search.query = Elements.searchInput.value;
});

// The effect reacts to the state change and updates the UI
effect(() => {
  Elements.resultsCount.update({
    textContent: `Results for: "${search.query}"`
  });
});
```

The flow is: **user types → event handler reads the value → state updates → effect runs → DOM updates**. Each piece has one clear job.

---

## Real-World Example: User Profile Card

This example ties everything together — a profile card that updates automatically when the underlying data changes.

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
  name:      'Alice Johnson',
  role:      'admin',
  isOnline:  true,
  avatar:    '/avatars/alice.jpg',
  followers: 142
});

effect(() => {
  Elements.update({
    'full-name': {
      textContent: profile.name
    },
    'role-badge': {
      textContent: profile.role,
      className:   `badge badge--${profile.role}`  // e.g. "badge badge--admin"
    },
    'online-status': {
      textContent: profile.isOnline ? 'Online' : 'Offline',
      className:   profile.isOnline ? 'status status--online' : 'status status--offline'
    },
    avatar: {
      setAttribute: {
        src: profile.avatar,
        alt: `${profile.name}'s avatar`
      }
    },
    'follow-btn': {
      textContent: `Follow (${profile.followers})`
    }
  });
});

// Now if anything changes, the card updates automatically:
profile.isOnline  = false; // Status dot and text update instantly
profile.followers = 143;   // Follow button count updates instantly
```

---

## Real-World Example: Login Form With Validation

This shows how `Elements` and the reactive system work together for a complete interactive form — reading input, tracking state, and showing live feedback.

```html
<input id="emailInput" type="email" placeholder="Email" />
<p id="email-error" hidden></p>

<input id="passwordInput" type="password" placeholder="Password" />
<p id="password-error" hidden></p>

<button id="submit-btn" disabled>Sign In</button>
```

```javascript
const form = state({ email: '', password: '' });

// Computed properties derive values from state automatically
computed(form, {
  emailValid()    { return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email); },
  passwordValid() { return this.password.length >= 8; },
  canSubmit()     { return this.emailValid && this.passwordValid; }
});

// The effect reads from state and computed, then drives the UI
effect(() => {
  Elements.update({
    'email-error': {
      textContent: form.email && !form.emailValid ? 'Please enter a valid email address.' : '',
      hidden:      !form.email || form.emailValid   // Only show the error if they've typed something invalid
    },
    'password-error': {
      textContent: form.password && !form.passwordValid ? 'Password must be at least 8 characters.' : '',
      hidden:      !form.password || form.passwordValid
    },
    'submit-btn': {
      disabled:    !form.canSubmit,
      textContent: form.canSubmit ? 'Sign In' : 'Complete the form to continue'
    }
  });
});

// Event handlers read from the DOM and write to state
Elements.emailInput.addEventListener('input', () => {
  form.email = Elements.emailInput.value;
});

Elements.passwordInput.addEventListener('input', () => {
  form.password = Elements.passwordInput.value;
});
```

As the user types, `state` updates, `computed` recalculates, and the `effect` refreshes the error messages and button — all automatically.

---

## `Elements` vs `Id()`

Both `Elements` and `Id()` let you target a single element by its `id`. They are functionally equivalent — choose whichever feels more readable in context:

```javascript
// Elements — feels natural when updating several elements in the same block
Elements.myTitle.update({ textContent: 'Hello' });

// Id() — slightly more explicit when you want to be very clear about a single target
Id('my-title').update({ textContent: 'Hello' });
```

When you need to update **multiple elements** in one call, always use `Elements.update({})` — `Id()` does not have a bulk form.

---

## Key Takeaways

| What you want to do | How to do it |
|---|---|
| Update one element by id | `Elements.myId.update({ ... })` |
| Update many elements at once | `Elements.update({ id1: {...}, id2: {...} })` |
| Show or hide an element | `{ hidden: true }` or `{ hidden: false }` |
| Switch CSS classes | `{ className: condition ? 'a' : 'b' }` |
| Read an input's value | `Elements.myInput.value` (inside event handlers) |
| Listen for events | `Elements.myBtn.addEventListener('click', fn)` |

- Keep your `effect()` focused on **updating the DOM** — don't put business logic inside it.
- Use `computed()` to derive values, then display them inside `effect()`.
- Attach event listeners **outside** effects — they only need to be attached once.
