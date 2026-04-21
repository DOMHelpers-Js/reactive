[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Understanding `effect()` - A Beginner's Guide

## Quick Start (30 seconds)

Need code that automatically runs when reactive data changes? Here's how:

```js
const counter = state({ count: 0 });

// Effect automatically runs whenever counter.count changes
effect(() => {
  Id('display').update({ textContent: counter.count });
  console.log(`Count is now: ${counter.count}`);
});
// Runs immediately: displays 0 and logs "Count is now: 0"

counter.count = 5;
// Displays 5 and logs "Count is now: 5"

counter.count++;
// Displays 6 and logs "Count is now: 6"
```

**That's it!** The `effect()` function creates reactive side effects that automatically re-run when the data they depend on changes.

 

## What is `effect()`?

`effect()` is a function that **creates reactive side effects** — code that automatically re-runs whenever the reactive data it reads changes. It's the foundation of automatic UI updates and reactive behavior.

**An effect:**
- Automatically tracks which reactive data it reads
- Re-runs whenever any of that data changes
- Runs immediately when created
- Can perform any side effects (DOM updates, logging, API calls, etc.)
- Returns a cleanup function to stop the effect

Think of it as **connecting your code to reactive data** — whenever the data changes, your code automatically runs again to stay synchronized.

 

## Syntax

```js
effect(fn)
```

**Parameters:**
- `fn` — A function that will run immediately and re-run when reactive data it reads changes

**Returns:**
- A cleanup function that stops the effect when called

**Example:**
```js
const stopEffect = effect(() => {
  console.log(`Count: ${counter.count}`);
});

// Later, stop the effect
stopEffect();
```

 

## Why Does This Exist?

### The Challenge with Plain JavaScript

In vanilla JavaScript, when data changes, you must manually update everything that depends on it:

```javascript
// ❌ Plain JavaScript approach
let count = 0;

function updateUI() {
  document.getElementById('display').textContent = count;
  document.getElementById('doubled').textContent = count * 2;

  if (count > 10) {
    document.getElementById('status').textContent = 'High';
  } else {
    document.getElementById('status').textContent = 'Low';
  }
}

count = 5;
updateUI(); // ❌ Easy to forget

count++;
updateUI(); // ❌ Repetitive and error-prone
```

**Problems with this approach:**
❌ Must manually call update functions after every change
❌ Easy to forget, causing UI to get out of sync
❌ Adding new UI elements means updating the function and all call sites
❌ Code becomes scattered and hard to maintain

### How Does `effect()` Help?

With `effect()`, you declare what should happen, and it automatically happens whenever relevant data changes:

```javascript
const counter = state({ count: 0 });

effect(() => {
  Elements.update({
    display: { textContent: counter.count },
    doubled: { textContent: counter.count * 2 },
    status:  { textContent: counter.count > 10 ? 'High' : 'Low' }
  });
});

effect(() => {
  console.log(`Count: ${counter.count}`);
});

// Just change the data — everything updates automatically ✨
counter.count = 5;
counter.count++;
counter.count *= 2;
```

**Benefits:**
✅ Automatic synchronization with reactive data
✅ No manual update calls needed
✅ Impossible to forget updates
✅ Dependencies tracked automatically
✅ Clean, declarative code

### When Does `effect()` Shine?

- You need automatic UI updates
- Code should run whenever data changes
- You want automatic dependency tracking
- You need side effects that stay synchronized with data

 

## Mental Model

Think of `effect()` like a **smart assistant that watches and responds**:

```
Regular Code (Manual Work):
┌─────────────────┐
│  count = 5      │ You change data
└─────────────────┘
         │
         ▼
[Your responsibility]
         │
         ▼
Call updateUI()     ← ❌ You must remember
Call logChange()    ← ❌ You must remember
Call saveData()     ← ❌ You must remember

Reactive Effect (Automatic Assistant):
┌─────────────────────┐
│  counter.count = 5  │ You change data
└─────────────────────┘
         │
         ▼
   ┌──────────┐
   │  Effect  │ ← Watches automatically
   │ Function │
   └─────┬────┘
         │
         ▼
Automatically executes:
  ✓ Updates UI
  ✓ Logs changes
  ✓ Saves data
  ✓ Everything in sync!
```

 

## How Does It Work?

### Automatic Dependency Tracking

When an effect runs, the reactive system tracks which reactive properties it accesses:

```
1️⃣ Create effect
   ↓
effect(() => {
  console.log(counter.count);  ← Reads counter.count
});
   ↓
Effect runs immediately
Tracks: "This effect depends on counter.count"
Logs: Count is now: 0

2️⃣ Data changes
   ↓
counter.count = 5;
   ↓
Reactive system checks: "Which effects depend on count?"
Finds: The effect we created
   ↓
Effect re-runs automatically!
Logs: Count is now: 5
```

### Under the Hood

```
effect(() => { ... })
      │
      ▼
┌─────────────────────┐
│  Run Function       │
│  Track Which Props  │
│  Are Read           │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Register Effect    │
│  as Dependent On    │
│  Those Props        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  When Any Prop      │
│  Changes, Re-run    │
│  Effect             │
└─────────────────────┘
```

 

## Basic Usage

### Creating a Simple Effect

```js
const message = state({ text: 'Hello' });

effect(() => {
  console.log(message.text);
});
// Immediately logs: "Hello"

message.text = 'Hello World';
// Automatically logs: "Hello World"
```

### Updating the DOM

Effects keep the DOM synchronized with state. Use `Elements.update()` for bulk updates:

```js
const user = state({ name: 'John', age: 25 });

effect(() => {
  Elements.update({
    userName: { textContent: user.name },
    userAge:  { textContent: user.age }
  });
});

// Both DOM elements update automatically
user.name = 'Jane';
user.age = 26;
```

For a single element, use `Id()`:

```js
const app = state({ title: 'Welcome' });

effect(() => {
  Id('page-title').update({ textContent: app.title });
});

app.title = 'Dashboard'; // DOM updates automatically
```

### Conditional Logic in Effects

Effects can contain any logic, including conditionals. Prefer declarative ternaries:

```js
const account = state({ balance: 100 });

effect(() => {
  const tier = account.balance > 1000 ? 'premium'
             : account.balance > 100  ? 'standard'
             : 'basic';

  Id('status').update({
    textContent: tier.charAt(0).toUpperCase() + tier.slice(1) + ' Account',
    className:   `status-${tier}`
  });
});

account.balance = 1500;
// Status automatically updates to "Premium Account"
```

 

## Automatic Dependency Tracking

### Effects Track What They Read

```js
const data = state({ a: 1, b: 2, c: 3 });

effect(() => {
  // This effect only reads 'a' and 'b'
  console.log(`Sum: ${data.a + data.b}`);
});
// Dependencies: a, b

data.a = 10; // Effect re-runs (depends on 'a')
data.b = 20; // Effect re-runs (depends on 'b')
data.c = 30; // Effect does NOT re-run (doesn't depend on 'c')
```

### Dynamic Dependencies

Dependencies can change based on conditions:

```js
const app = state({
  showDetails: false,
  name: 'John',
  details: 'Developer'
});

effect(() => {
  console.log(app.name);

  if (app.showDetails) {
    console.log(app.details); // Only read when showDetails is true
  }
});

// Initially depends on: name
// When showDetails is true, depends on: name, details

app.showDetails = true; // Effect re-runs, now tracks 'details' too
app.details = 'Manager'; // Effect re-runs (now a dependency)
```

 

## Effects Run Immediately

Unlike watchers, effects run immediately when created:

```js
const counter = state({ count: 0 });

console.log('Before effect');

effect(() => {
  console.log(`Count: ${counter.count}`);
});
// Immediately logs: "Count: 0"

console.log('After effect');

// Output:
// Before effect
// Count: 0
// After effect
```

This is useful for initializing UI without a separate setup call:

```js
const user = state({ name: 'John', isOnline: false });

effect(() => {
  Elements.update({
    userName:    { textContent: user.name },
    statusDot:   { className: user.isOnline ? 'online' : 'offline' }
  });
});
// UI initialized immediately — no separate init call needed
```

 

## Multiple Dependencies

### Depending on Multiple Properties

```js
const rect = state({ width: 10, height: 20 });

effect(() => {
  const area = rect.width * rect.height;
  console.log(`Area: ${area}`);
});

rect.width = 15;  // Logs: "Area: 300"
rect.height = 30; // Logs: "Area: 450"
```

### Depending on Multiple Objects

```js
const cart     = state({ items: [{ price: 10, quantity: 2 }] });
const settings = state({ taxRate: 0.1 });

effect(() => {
  const subtotal = cart.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  const tax      = subtotal * settings.taxRate;
  const total    = subtotal + tax;

  console.log(`Total: $${total.toFixed(2)}`);
});
// Depends on both cart.items AND settings.taxRate
```

### Depending on Computed Properties

```js
const user = state({ firstName: 'John', lastName: 'Doe' });

computed(user, {
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
});

effect(() => {
  document.title = user.fullName;
});

user.firstName = 'Jane'; // document.title updates automatically
```

 

## Effects with Side Effects

### Logging and Debugging

```js
const app = state({ currentPage: '/home', user: null });

effect(() => {
  console.log('=== State Update ===');
  console.log('Page:', app.currentPage);
  console.log('User:', app.user);
});
```

### API Calls

```js
const search = state({ query: '', results: [] });

effect(() => {
  if (search.query) {
    fetch(`/api/search?q=${search.query}`)
      .then(res => res.json())
      .then(data => { search.results = data; });
  }
});
```

### Local Storage Synchronization

```js
const preferences = state({ theme: 'light', fontSize: 14, language: 'en' });

effect(() => {
  localStorage.setItem('preferences', JSON.stringify({
    theme:    preferences.theme,
    fontSize: preferences.fontSize,
    language: preferences.language
  }));
});
```

 

## Cleanup and Disposal

### Stopping Effects

```js
const counter = state({ count: 0 });

const stopEffect = effect(() => {
  console.log(`Count: ${counter.count}`);
});

counter.count = 1; // Logs: "Count: 1"

stopEffect();

counter.count = 2; // Nothing logged — effect is stopped
```

### Cleanup in Component Lifecycle

```js
function createTimer() {
  const timer = state({ seconds: 0 });

  const interval = setInterval(() => { timer.seconds++; }, 1000);

  const stopEffect = effect(() => {
    Id('timer').update({ textContent: timer.seconds });
  });

  return () => {
    clearInterval(interval);
    stopEffect();
  };
}

const cleanup = createTimer();

// Later, when removing the timer
cleanup();
```

### Multiple Effects Cleanup

```js
const app = state({ user: null, products: [], cart: [] });

const cleanups = [
  effect(() => {
    if (app.user) {
      Id('user-name').update({ textContent: app.user.name });
    }
  }),
  effect(() => {
    Elements.update({
      'product-count': { textContent: app.products.length },
      'cart-count':    { textContent: app.cart.length }
    });
  })
];

function cleanup() {
  cleanups.forEach(fn => fn());
}
```

 

## Real-World Examples

### Example 1: Live Counter

```js
const counter = state({ count: 0, step: 1 });

effect(() => {
  Elements.update({
    count:   { textContent: counter.count },
    step:    { textContent: counter.step },
    doubled: { textContent: counter.count * 2 }
  });
});

Elements.increment.addEventListener('click', () => { counter.count += counter.step; });
Elements.decrement.addEventListener('click', () => { counter.count -= counter.step; });
Elements.changeStep.addEventListener('click', () => {
  counter.step = counter.step === 1 ? 10 : 1;
});
```

### Example 2: Todo List

```js
const todos = state({
  items: [
    { id: 1, text: 'Learn Reactive', done: false },
    { id: 2, text: 'Build App', done: false }
  ]
});

effect(() => {
  Id('todo-list').update({
    innerHTML: todos.items.map(todo => `
      <div class="todo ${todo.done ? 'done' : ''}">
        <input type="checkbox" ${todo.done ? 'checked' : ''} data-id="${todo.id}" />
        <span>${todo.text}</span>
      </div>
    `).join('')
  });
});

effect(() => {
  const total     = todos.items.length;
  const done      = todos.items.filter(t => t.done).length;
  const remaining = total - done;

  Id('stats').update({ textContent: `${remaining} remaining / ${total} total` });
});

function addTodo(text) {
  todos.items.push({ id: Date.now(), text, done: false });
}
```

### Example 3: Form Validation Display

```js
const form = state({ email: '', password: '' });

const validation = state({ emailError: '', passwordError: '' });

effect(() => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  validation.emailError = form.email && !emailRegex.test(form.email)
    ? 'Please enter a valid email'
    : '';
});

effect(() => {
  validation.passwordError = form.password && form.password.length < 8
    ? 'Password must be at least 8 characters'
    : '';
});

effect(() => {
  const isValid = !validation.emailError && !validation.passwordError
               && form.email && form.password;
  Elements.update({
    'email-error':    { textContent: validation.emailError, hidden: !validation.emailError },
    'password-error': { textContent: validation.passwordError, hidden: !validation.passwordError },
    submit:           { disabled: !isValid }
  });
});
```

### Example 4: Dark Mode Toggle

```js
const settings = state({ darkMode: false });

effect(() => {
  document.body.className = settings.darkMode ? 'dark-theme' : 'light-theme';
});

effect(() => {
  Id('theme-toggle').update({
    textContent: settings.darkMode ? '☀️ Light Mode' : '🌙 Dark Mode'
  });
});

effect(() => {
  localStorage.setItem('darkMode', settings.darkMode);
});

function toggleTheme() {
  settings.darkMode = !settings.darkMode;
}

settings.darkMode = localStorage.getItem('darkMode') === 'true';
```

### Example 5: Real-Time Dashboard

```js
const dashboard = state({ sales: 0, visitors: 0, conversionRate: 0 });

effect(() => {
  const rate  = dashboard.conversionRate;
  const trend = dashboard.sales / (dashboard.visitors || 1);

  Elements.update({
    sales:      { textContent: `$${dashboard.sales.toLocaleString()}` },
    visitors:   { textContent: dashboard.visitors.toLocaleString() },
    conversion: {
      textContent: `${rate.toFixed(2)}%`,
      className: rate > 5 ? 'metric excellent' : rate > 2 ? 'metric good' : 'metric poor'
    },
    trend: {
      textContent: trend > 50 ? '📈 Trending Up' : trend > 20 ? '➡️ Steady' : '📉 Needs Attention'
    }
  });
});

setInterval(() => {
  dashboard.sales      += Math.floor(Math.random() * 1000);
  dashboard.visitors   += Math.floor(Math.random() * 50);
  dashboard.conversionRate = (dashboard.sales / dashboard.visitors) * 0.1;
}, 2000);
```

### Example 6: Auto-Save Document

```js
const doc = state({ title: '', content: '', lastSaved: null, isDirty: false });

let saveTimeout;

effect(() => {
  void (doc.title + doc.content); // track both
  doc.isDirty = true;
});

effect(() => {
  if (doc.isDirty) {
    clearTimeout(saveTimeout);
    saveTimeout = setTimeout(async () => {
      await saveToServer({ title: doc.title, content: doc.content });
      doc.lastSaved = new Date();
      doc.isDirty   = false;
    }, 2000);
  }
});

effect(() => {
  Id('save-status').update({
    textContent: doc.isDirty
      ? '● Unsaved changes'
      : doc.lastSaved
        ? `✓ Saved at ${doc.lastSaved.toLocaleTimeString()}`
        : '',
    className: doc.isDirty ? 'unsaved' : 'saved'
  });
});
```

 

## Common Pitfalls

### Pitfall #1: Infinite Loops

❌ **Wrong:**
```js
const counter = state({ count: 0 });

effect(() => {
  counter.count++; // ❌ Reads AND writes count — infinite loop!
});
```

✅ **Correct:**
```js
const counter = state({ count: 0 });

effect(() => {
  console.log(counter.count); // ✅ Read only
});

counter.count++; // Modify outside the effect
```

 

### Pitfall #2: Not Understanding Immediate Execution

```js
console.log('Before');

effect(() => {
  console.log('Effect runs immediately');
});
// ← Effect has already run by this point

console.log('After');

// Output: Before → Effect runs immediately → After
```

 

### Pitfall #3: Forgetting Dependencies

❌ **Wrong:**
```js
const data = state({ value: 10 });
let multiplier = 2; // Not reactive

effect(() => {
  console.log(data.value * multiplier); // multiplier changes are invisible
});
```

✅ **Correct:**
```js
const data = state({ value: 10, multiplier: 2 }); // Make it reactive

effect(() => {
  console.log(data.value * data.multiplier); // Both tracked
});
```

 

### Pitfall #4: Heavy Computations in Effects

❌ **Wrong:**
```js
effect(() => {
  const sorted = [...data.numbers].sort((a, b) => b - a); // Expensive every change
  console.log('Largest:', sorted[0]);
});
```

✅ **Correct:**
```js
computed(data, {
  largestNumber() { return Math.max(...this.numbers); } // Cached
});

effect(() => {
  console.log('Largest:', data.largestNumber); // Uses cache
});
```

 

### Pitfall #5: Not Cleaning Up Effects

```js
// ✅ Correct — save and call cleanup
function createWidget(data) {
  const stopEffect = effect(() => {
    Id('widget').update({ textContent: data.value });
  });

  return { destroy: stopEffect };
}

const widget = createWidget(myData);
// Later:
widget.destroy();
```

 

## Summary

**What is `effect()`?**
Creates reactive side effects that automatically run when the reactive data they depend on changes.

**Key Points to Remember:**

1️⃣ **Effects run immediately** — They execute when created, not just on changes

2️⃣ **Automatic dependency tracking** — Effects track what they read automatically

3️⃣ **Returns cleanup function** — Always save and call it when done

4️⃣ **Avoid infinite loops** — Don't modify state you're reading in the effect

5️⃣ **Declarative DOM updates** — Use `Elements.update({})` for multiple elements, `Id().update({})` for single

6️⃣ **Keep effects focused** — Each effect should have a clear, single purpose

**Quick Reference:**

```js
const counter = state({ count: 0 });

const stopEffect = effect(() => {
  Elements.update({
    display: { textContent: counter.count },
    doubled: { textContent: counter.count * 2 }
  });
});

counter.count = 5;  // Both elements update automatically
counter.count++;    // Both elements update automatically

stopEffect(); // Stop when done
```

**Remember:** `effect()` is the foundation of reactive programming. Set it up once, and it keeps your UI synchronized automatically. ✨
