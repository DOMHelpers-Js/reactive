[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)
[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# What Is DOM Integration?

## Start Here — Before You Touch Any Namespace

You have learned how to create reactive state, write effects, use computed values, and watch for changes. That is the reactive engine — the brain of your application. It tracks data and knows when something changes.

But knowing that something changed is only half the job.

**The other half is making the page actually reflect that change.**

That is what DOM Integration is. It is the bridge between your reactive state and what the user sees on the screen.

---

## The Two Sides of a Reactive Application

Think of every reactive application as having two sides:

```
┌─────────────────────────────┐        ┌─────────────────────────────┐
│         YOUR STATE          │        │          THE DOM            │
│                             │        │                             │
│  const user = state({       │        │  <span id="username">       │
│    name: 'Alice',           │ ──────▶│    Alice                    │
│    isOnline: true           │        │  </span>                    │
│  });                        │        │  <span id="status">         │
│                             │        │    Online                   │
│                             │        │  </span>                    │
└─────────────────────────────┘        └─────────────────────────────┘
         The data                              What the user sees
```

The left side is your JavaScript — your state objects, your computed values, your business logic. The right side is the HTML — the elements your users actually read, click, and interact with.

**DOM Integration is everything that happens in the arrow between them.**

---

## The Problem With Plain JavaScript

In plain JavaScript, keeping these two sides in sync is entirely your responsibility. Every time something changes, you have to manually find the right element and update it yourself:

```javascript
// The user logs in
function onLogin(user) {
  document.getElementById('username').textContent = user.name;
  document.getElementById('status').textContent = 'Online';
  document.getElementById('status').className = 'status status--online';
  document.getElementById('login-btn').hidden = true;
  document.getElementById('logout-btn').hidden = false;
  document.getElementById('avatar').src = user.avatarUrl;
  document.getElementById('avatar').alt = user.name + "'s avatar";
  // ... and so on
}
```

This works, but it has serious problems:

- **It is fragile.** If you add a new element to the HTML but forget to update this function, it stays stale.
- **It is repetitive.** `document.getElementById` appears over and over.
- **It is scattered.** The code that *defines* what the UI should look like is spread across dozens of event handlers and functions.
- **It does not scale.** As your app grows, keeping everything in sync becomes a full-time job.

---

## How DOM Helpers Solves This

DOM Helpers gives you a set of **namespaces** — purpose-built tools for talking to the DOM in a clean, structured way. When you combine them with `effect()`, the synchronisation between your state and your UI becomes automatic.

Here is the same login scenario, rewritten with DOM Helpers:

```javascript
const user = state({
  name: '',
  isOnline: false,
  avatarUrl: '',
  isLoggedIn: false
});

effect(() => {
  Elements.update({
    username:     { textContent: user.name },
    status:       { textContent: user.isOnline ? 'Online' : 'Offline',
                    className: user.isOnline ? 'status status--online' : 'status status--offline' },
    'login-btn':  { hidden: user.isLoggedIn },
    'logout-btn': { hidden: !user.isLoggedIn },
    avatar:       { setAttribute: { src: user.avatarUrl, alt: `${user.name}'s avatar` } }
  });
});

// Now login is just a state update — the DOM takes care of itself
function onLogin(data) {
  user.name      = data.name;
  user.avatarUrl = data.avatarUrl;
  user.isOnline  = true;
  user.isLoggedIn = true;
}
```

The `effect()` runs once at first to set the initial UI. Then, any time any piece of `user` state changes, the effect re-runs and the DOM updates automatically. You never call `document.getElementById` again.

---

## The Three Namespaces — A Map

DOM Helpers provides three distinct namespaces for DOM integration. Each one is designed for a different kind of targeting:

### `Elements` — Target by `id`

Use `Elements` when you need to reach **one specific element** that has a unique `id` attribute.

```html
<h1 id="page-title">Hello</h1>
```

```javascript
Elements['page-title'].update({ textContent: 'Welcome, Alice!' });

// Or update many id-based elements at once:
Elements.update({
  'page-title': { textContent: 'Welcome, Alice!' },
  'page-subtitle': { hidden: false }
});
```

`Elements` is your most common tool. Most meaningful elements on a page have an `id`.

---

### `ClassName`, `TagName`, `Name` — Target Groups

Use these when you need to update **multiple elements at once** based on something they share:

```javascript
// All elements with class "card"
ClassName.card.update({ hidden: false });

// All <button> elements
TagName.button.update({ disabled: true });

// All inputs with name="email"
Name.email.update({ value: '' });
```

These are invaluable when a state change should affect several elements at once — toggling visibility, disabling a form, updating a theme — without writing any loops.

---

### `query` and `queryAll` — Target by CSS Selector

Use these when `Elements` and `ClassName` are not expressive enough. They accept **any valid CSS selector**, giving you the full power of selectors like attribute matchers, pseudo-classes, and compound selectors:

```javascript
// A compound selector — Elements can't do this
query('.modal > .modal-title').update({ textContent: 'Confirm Action' });

// An attribute selector — ClassName can't filter for this
queryAll('input[required]').update({ setAttribute: { 'aria-required': 'true' } });

// A data-attribute-based selector
queryAll('[data-status="pending"]').update({ className: 'item item--pending' });
```

Think of `query` and `queryAll` as the escape hatch — reach for them when you need full CSS selector expressiveness.

---

## How They All Fit Together

Here is a visual map of the whole system — when to use each tool:

```
You want to update...
│
├── One specific element (has a unique id)?
│     └── Elements.myId.update({})
│         Elements.update({ id1: {...}, id2: {...} })
│
├── A group of elements that share a characteristic?
│   │
│   ├── They share a CSS class?     → ClassName.name.update({})
│   ├── They share an HTML tag?     → TagName.tag.update({})
│   └── They share a name attr?     → Name.field.update({})
│
└── A complex CSS selector (attribute, pseudo, compound)?
      ├── One element?  → query('.my-selector').update({})
      └── Many?         → queryAll('.my-selector').update({})
                            queryAll('.my-selector', context) ← scoped to a container
```

---

## The Golden Rule of DOM Integration

There is one rule that, if you follow it, will make everything click into place:

> **State describes your data. Effects describe your UI. DOM Integration applies the result.**

Put another way:

- **State** holds the truth: `user.isLoggedIn = true`
- **Effect** describes what the UI should look like given that truth
- **Elements / ClassName / query** apply it to the actual DOM

You should **never** update the DOM directly in response to an event. Instead, update state — and let the effect handle the DOM:

```javascript
// ❌ Updating the DOM directly in an event handler
loginBtn.addEventListener('click', () => {
  document.getElementById('greeting').textContent = 'Welcome!';
  document.getElementById('login-btn').hidden = true;
});

// ✅ Update state — the effect handles the DOM automatically
loginBtn.addEventListener('click', () => {
  auth.isLoggedIn = true;
  auth.userName = 'Alice';
});

effect(() => {
  Elements.update({
    greeting:    { textContent: auth.isLoggedIn ? `Welcome, ${auth.userName}!` : 'Hello' },
    'login-btn': { hidden: auth.isLoggedIn }
  });
});
```

The second approach scales. The DOM always matches the state, guaranteed.

---

## The Flow of a DOM-Integrated Application

Every interaction in a well-structured reactive app follows this cycle:

```
User does something
       │
       ▼
Event handler runs
       │
       ▼
State is updated  (user.isLoggedIn = true)
       │
       ▼
effect() detects the change and re-runs
       │
       ▼
Elements / ClassName / query apply changes to the DOM
       │
       ▼
User sees the updated page
```

Your job as the developer is to:
1. Define your **state** (what data matters)
2. Write **effects** that describe what the UI should look like
3. Use **DOM Integration** tools inside those effects to apply the changes
4. Wire up **event handlers** that update state — nothing else

The reactive system handles the rest.

---

## What You Will Learn Next

Now that you understand the role of DOM Integration and the tools available, you are ready to dive into each namespace in depth:

- **[Elements Namespace](./elements-namespace)** — Target individual elements by `id`, update single or multiple elements at once, and read values from inputs. Start here — it is the most commonly used tool.

- **[Collections Namespace](./collections-namespace)** — Target groups of elements by class, tag, or name attribute. Learn how to update dozens of elements in one call, access specific elements by index, and iterate with `forEach`.

- **[Selector Namespace](./selector-namespace)** — Use full CSS selector power with `query` and `queryAll`. Learn scoped queries, compound selectors, and attribute-based targeting for cases the other namespaces cannot handle.

Each page follows the same structure: what the tool is, how to use it, common patterns, and real-world examples — built up step by step.
