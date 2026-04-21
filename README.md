# DOM Helpers Reactive — Documentation

The official documentation for **DOM Helpers Reactive** — a fine-grained reactive state management library built entirely on plain JavaScript. No framework. No compiler. No virtual DOM. No special syntax to learn.

**Live documentation:** https://giovanni1707.github.io/DOMHelpers-Reactive/

---

## Why DOM Helpers?

The JavaScript ecosystem is full of powerful tools — React, Vue, Svelte, Solid, and many more. Each of them is genuinely good at what it does. But they all share one trait: they ask you to step outside of JavaScript to use them.

You learn JSX, or template syntax, or a compiler-driven reactivity model, or a component format that only works inside their build pipeline. The knowledge you gain is real, but it is specific to that tool. Change tools, and you start over.

DOM Helpers takes a different position.

> **You already know the language. The library works with it.**

Everything in DOM Helpers is plain JavaScript. There is no template language to parse, no JSX transformer to configure, no build step required at all. You write JavaScript, and the library responds to it. The same skills you use to write a regular script are the same skills you use to build a fully reactive application.

---

## JavaScript Is More Powerful Than People Give It Credit For

JavaScript is often treated as the beginner's language — the thing you use before you "move on" to a real framework. DOM Helpers is built on the belief that this is exactly backwards.

JavaScript — the actual language, without abstractions on top — is one of the most flexible and expressive languages available in any runtime. Proxies, closures, WeakMaps, the microtask queue, the prototype chain: these are not obscure internals to avoid. They are the foundation of every modern reactive system, including the ones powering the frameworks people reach for when they think JavaScript "isn't enough."

DOM Helpers uses all of these directly. It is not a framework that hides JavaScript from you. It is a library that amplifies what JavaScript already does.

When you learn DOM Helpers, you are also learning JavaScript more deeply. And when you understand JavaScript more deeply, DOM Helpers becomes even more intuitive. The two reinforce each other in a way that framework-specific knowledge rarely does.

---

## The Bridge Between Imperative and Declarative

Most developers learn JavaScript imperatively — you write each step: find this element, set this text, attach this listener. It is direct, readable, and easy to follow. But it does not scale. As your application grows, keeping the DOM in sync with your data becomes a manual, error-prone process.

Declarative frameworks solve this by replacing imperative code with a new abstraction — templates, components, reactive directives. You no longer write steps; you describe what the UI *should look like*. But the cost is that you leave plain JavaScript behind. You cannot just write `element.textContent = value` anymore. You work inside the framework's model.

DOM Helpers sits in the middle. It brings declarative reactivity to plain JavaScript without replacing it.

You still write JavaScript. You still think in JavaScript. But your state becomes reactive, your effects run automatically, and your DOM stays in sync — all without touching a compiler, a template engine, or a virtual DOM.

```javascript
// This is the whole reactive loop — nothing hidden, nothing compiled
const user = state({ name: 'Alice', isOnline: true });

effect(() => {
  Elements.update({
    username:      { textContent: user.name },
    'status-text': { textContent: user.isOnline ? 'Online' : 'Offline' },
    'status-dot':  { className: user.isOnline ? 'dot dot--green' : 'dot dot--grey' }
  });
});

// Later — the effect re-runs automatically, the DOM updates instantly
user.isOnline = false;
```

This is declarative. You described what the UI should look like, not the steps to get there. But it is also just JavaScript. No compilation happened. No special syntax was parsed. You can read every line and know exactly what it does.

---

## What This Looks Like in Practice

### Reactivity without a framework

```javascript
const cart = state({ items: [], total: 0 });

computed(cart, {
  itemCount() { return this.items.length; },
  isEmpty()   { return this.items.length === 0; }
});

effect(() => {
  Elements.update({
    'cart-count':   { textContent: cart.itemCount },
    'cart-total':   { textContent: `$${cart.total.toFixed(2)}` },
    'empty-notice': { hidden: !cart.isEmpty },
    'checkout-btn': { disabled: cart.isEmpty }
  });
});
```

State changes. Computed values recalculate. The effect re-runs. The DOM updates. This is the entire reactive loop — no virtual DOM diffing, no reconciler, no scheduler to configure.

---

### A complete reactive form — no form library needed

```javascript
const form = state({ email: '', password: '' });

computed(form, {
  emailValid()    { return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email); },
  passwordValid() { return this.password.length >= 8; },
  canSubmit()     { return this.emailValid && this.passwordValid; }
});

effect(() => {
  Elements.update({
    'email-error':    { hidden: !form.email || form.emailValid,
                        textContent: 'Enter a valid email address' },
    'password-error': { hidden: !form.password || form.passwordValid,
                        textContent: 'Minimum 8 characters' },
    'submit-btn':     { disabled: !form.canSubmit,
                        textContent: form.canSubmit ? 'Sign In' : 'Complete the form' }
  });
});

Elements.emailInput.addEventListener('input', () => {
  form.email = Elements.emailInput.value;
});

Elements.passwordInput.addEventListener('input', () => {
  form.password = Elements.passwordInput.value;
});
```

Live validation, computed state, automatic DOM updates. The entire form is reactive with zero dependencies beyond the library itself.

---

### A full reactive list — with a rich collection API

```javascript
const todos = createCollection([]);

effect(() => {
  Elements.update({
    'todo-count':     { textContent: `${todos.length} task${todos.length === 1 ? '' : 's'}` },
    'empty-state':    { hidden: !todos.isEmpty() },
    'clear-btn':      { hidden: todos.isEmpty() }
  });
});

// The collection API reads like plain English
todos.add({ text: 'Learn DOM Helpers', done: false });
todos.toggle(t => t.text === 'Learn DOM Helpers', 'done');
todos.removeWhere(t => t.done);
```

---

### Async state — no external library needed

```javascript
const users = asyncState(async () => {
  const res = await fetch('/api/users');
  return res.json();
});

effect(() => {
  Elements.update({
    spinner:       { hidden: !users.loading },
    'user-list':   { hidden: users.loading || users.error },
    'error-msg':   { hidden: !users.error, textContent: users.error?.message ?? '' }
  });
});

users.execute();
```

Loading, error, success, and abort — all tracked automatically in reactive state.

---

### Persistent state — one line

```javascript
const settings = state({ theme: 'light', language: 'en' });

autoSave(settings, 'user-settings', { storage: 'localStorage', debounce: 300 });
```

The state now survives page reloads. No manual `localStorage` calls.

---

## No Build Step Required

Every example above runs directly in a browser. Add one script tag and you have the entire library:

```html
<script src="https://cdn.jsdelivr.net/npm/dom-helpers-js@2.10.0/dist/dom-helpers.full-spa.min.js"></script>
```

Or load only what you need, module by module. See the [CDN page](https://giovanni1707.github.io/DOMHelpers-Reactive/cdn/ALL-CDN-LINKS) for all options including ESM named imports and the automatic module loader.

There is no Webpack config to write. No Babel pipeline to set up. No `node_modules` to install if you do not want them. The library runs wherever JavaScript runs.

---

## What You Can Build

DOM Helpers is not a toy. It is a complete toolkit for building reactive web applications:

| Capability | What it includes |
|---|---|
| **Reactive state** | `state`, `ref`, `store`, `component`, `reactive` |
| **Reactivity** | `effect`, `computed`, `watch`, `batch`, `bindings` |
| **Collections** | 40+ array methods — `add`, `remove`, `filter`, `sort`, `map`, `find`, and more |
| **Forms** | Full form state, validation, touched tracking, field binding, submission |
| **Async** | `asyncState` with loading/error/success, `execute`, `abort`, `refetch` |
| **Storage** | `autoSave`, `reactiveStorage`, `watchStorage`, cross-tab sync |
| **DOM Integration** | `Elements`, `ClassName`, `TagName`, `query`, `queryAll` — declarative DOM updates |
| **Cleanup** | `collector`, `scope` — automatic memory management and lifecycle control |
| **Error handling** | `ErrorBoundary`, `wrap` — runtime error recovery |
| **Builder pattern** | Chainable API for composing reactive components |
| **DevTools** | State inspection, lifecycle tracking, effect tracing |
| **SPA Router** | Hash and history routing with guards and transitions |

Full documentation for every feature lives at **https://giovanni1707.github.io/DOMHelpers-Reactive/**.

---

## Who This Is For

**If you are learning JavaScript** — DOM Helpers teaches you reactivity in the same language you are already learning. You will not need to unlearn anything when you come back to plain JavaScript. The concepts transfer directly.

**If you are experienced with JavaScript** — DOM Helpers gives you the power of a reactive system without the overhead of a framework's mental model, build pipeline, or opinion about how your application should be structured.

**If you are building something that needs to ship fast** — one script tag, no configuration, full reactivity. You can have a working reactive application in minutes.

**If you care about bundle size** — the full bundle is ~49.7 KB gzipped. Load individual modules and pay only for what you use. The core module that powers the entire reactive system is ~9.6 KB gzipped.

---

## What the Documentation Covers

The documentation is structured to take any reader — from complete beginner to experienced developer — through the full library:

- **Getting Started** — what reactivity is, what the library does, and how to install it
- **Fundamentals** — the complete reactive API: state, effects, computed, watch, batch, store, builder
- **Features** — collections, forms, async state, and storage
- **DOM Integration** — how reactive state connects to the real DOM through `Elements`, `ClassName`, `TagName`, `query`, and `queryAll`
- **CDN** — all loading options, module dependencies, and the automatic module loader
- **More** — cleanup system, error handling, the reactivity engine internals, and DevTools

---

## Contributing

Found a mistake, a broken example, or something that could be explained better? Contributions are welcome.

See the [Contributing page](https://giovanni1707.github.io/DOMHelpers-Reactive/creator/contributing) for guidelines.

---

## Support the Project

If DOM Helpers has been useful to you, consider sponsoring:

- [GitHub Sponsors](https://github.com/sponsors/giovanni1707)
- [PayPal](https://paypal.me/GiovanniSylvain)

---

## License

MIT
