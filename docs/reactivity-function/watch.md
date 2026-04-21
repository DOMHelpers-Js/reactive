[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Understanding `watch()` - A Beginner's Guide

## Quick Start (30 seconds)

Need to run code when specific state properties change? Here's how:

```js
const user = state({ name: 'John', age: 25 });

watch(user, {
  name(newValue, oldValue) {
    console.log(`Name changed from "${oldValue}" to "${newValue}"`);
  },
  age(newValue, oldValue) {
    console.log(`Age changed from ${oldValue} to ${newValue}`);
  }
});

user.name = 'Jane'; // Logs: Name changed from "John" to "Jane"
user.age = 26;      // Logs: Age changed from 25 to 26
```

**That's it!** The `watch()` function lets you respond to specific property changes with custom callbacks.

 

## What is `watch()`?

`watch()` is a function that **monitors specific properties in reactive state** and executes callback functions when those properties change. It's designed for responding to changes with side effects or custom logic.

**A watcher:**
- Monitors specific properties you choose
- Executes your callback when the property changes
- Provides both old and new values
- Can track multiple properties at once
- Returns a cleanup function to stop watching

Think of it as **setting up custom notifications** for state changes — whenever specific data changes, your code automatically runs.

 

## Syntax

```js
watch(state, definitions)
```

**Parameters:**
- `state` — The reactive state object to watch
- `definitions` — An object where each key is a property name to watch, and each value is a callback function

**Returns:** A cleanup function that stops all watchers when called

**Example:**
```js
const stopWatching = watch(user, {
  name(newValue, oldValue) {
    console.log(`Name: ${oldValue} → ${newValue}`);
  }
});

// Later, stop watching
stopWatching();
```

 

## Why Does This Exist?

### The Challenge with Plain JavaScript

In vanilla JavaScript, there's no built-in way to detect when a variable changes:

```javascript
// ❌ Plain JavaScript approach
let userName = 'John';

function updateUserName(newName) {
  const oldName = userName;
  userName = newName;
  console.log(`Name changed: ${oldName} → ${newName}`);
  saveToServer(newName); // ❌ Must remember to call manually
}

userName = 'Bob'; // ❌ Silent — no logging, no saving!
```

**Problems with this approach:**
❌ Must manually wrap every change in a function
❌ Direct assignments are silent and miss side effects
❌ Cannot watch changes made by other code
❌ Code becomes scattered and hard to follow

### How Does `watch()` Help?

With `watch()`, you declare what should happen when properties change:

```javascript
const user = state({ name: 'John', email: 'john@example.com' });

watch(user, {
  name(newValue, oldValue) {
    console.log(`Name changed: ${oldValue} → ${newValue}`);
    logChange('name', oldValue, newValue);
  },
  email(newValue) {
    validateEmail(newValue);
    saveToServer({ email: newValue });
  }
});

// Just change the values — watchers execute automatically ✨
user.name = 'Jane';
user.email = 'jane@example.com';
```

**Benefits:**
✅ Changes are automatically detected
✅ Side effects execute automatically
✅ No manual function wrappers needed
✅ Centralized change handling
✅ Clean separation of concerns

### When Does `watch()` Shine?

- You need to respond to specific property changes with custom logic
- Side effects should run on change (logging, API calls, storage sync)
- You want to track old and new values
- You're building auto-save, validation, or audit-trail features

 

## Mental Model

Think of `watch()` like **motion-activated security cameras**:

```
Regular Variables (No Surveillance):
┌─────────────────┐
│  name: "John"   │ ← Changes happen silently
└─────────────────┘
   No detection
   No alerts
   No recording

Reactive State with Watchers (Monitored Area):
┌─────────────────────────┐
│  name: "John"           │ ←─── Watcher installed
└─────────────────────────┘
         │
         ▼
   ┌─────────────┐
   │  Watcher    │
   │  Callback   │
   └──────┬──────┘
          │
          ▼
When name changes to "Jane":
  ✓ Old value: "John"
  ✓ New value: "Jane"
  ✓ Callback executes
  ✓ Side effects run
```

 

## How Does It Work?

```
1️⃣ Set up watcher
   ↓
watch(user, { name(newVal, oldVal) { ... } })
   ↓
System registers: "Watch user.name"
Stores current value: "John"

2️⃣ Property changes
   ↓
user.name = 'Jane'
   ↓
Proxy detects change to 'name'
Old: "John" / New: "Jane"
   ↓
Callback executes automatically!

3️⃣ Subsequent changes
   ↓
user.name = 'Bob'
   ↓
Old: "Jane" (updated) / New: "Bob"
   ↓
Callback executes again!
```

 

## Basic Usage

### Watching a Single Property

```js
const counter = state({ count: 0 });

watch(counter, {
  count(newValue, oldValue) {
    console.log(`Count changed from ${oldValue} to ${newValue}`);
  }
});

counter.count = 5;  // Logs: Count changed from 0 to 5
counter.count = 10; // Logs: Count changed from 5 to 10
```

### Understanding Callback Parameters

```js
const user = state({ name: 'John' });

watch(user, {
  name(newValue, oldValue) {
    // newValue: the current (new) value
    // oldValue: the previous value
    console.log(`Old: ${oldValue}, New: ${newValue}`);
  }
});

user.name = 'Jane'; // Logs: Old: John, New: Jane
user.name = 'Bob';  // Logs: Old: Jane, New: Bob
```

### Watchers Execute on Change, Not on Setup

Watchers only execute when the property *changes*, not when defined:

```js
const user = state({ name: 'John' });

watch(user, {
  name(newValue) {
    console.log('Name changed!');
  }
});

// Nothing logged yet — watcher is just set up

user.name = 'Jane'; // NOW it logs: "Name changed!"
```

 

## Watching Multiple Properties

```js
const form = state({ username: '', email: '', password: '' });

watch(form, {
  username(newValue, oldValue) {
    console.log(`Username: ${oldValue} → ${newValue}`);
  },
  email(newValue, oldValue) {
    console.log(`Email: ${oldValue} → ${newValue}`);
  },
  password() {
    console.log('Password changed');
  }
});
```

Theme and settings — each watcher fires only for its property:

```js
const settings = state({ theme: 'light', fontSize: 14, notifications: true });

watch(settings, {
  theme(newValue) {
    document.body.className = `theme-${newValue}`; // body is a special global
  },
  fontSize(newValue) {
    document.body.style.fontSize = `${newValue}px`;
  },
  notifications(newValue) {
    console.log(`Notifications: ${newValue ? 'enabled' : 'disabled'}`);
  }
});

settings.theme    = 'dark'; // Only theme watcher fires
settings.fontSize = 16;    // Only fontSize watcher fires
```

 

## Accessing Old and New Values

### Comparing Old and New

```js
const inventory = state({ stockLevel: 100 });

watch(inventory, {
  stockLevel(newValue, oldValue) {
    const change = newValue - oldValue;

    if (change > 0) {
      console.log(`Stock increased by ${change} units`);
    } else if (change < 0) {
      console.log(`Stock decreased by ${Math.abs(change)} units`);
    }

    if (newValue < 10 && oldValue >= 10) {
      console.log('⚠️ LOW STOCK ALERT!');
    }
  }
});

inventory.stockLevel = 120; // Increased by 20
inventory.stockLevel = 5;   // Decreased by 115 + LOW STOCK ALERT
```

### Tracking Change History

```js
const doc = state({ content: '' });
const changeHistory = [];

watch(doc, {
  content(newValue, oldValue) {
    changeHistory.push({
      timestamp:  new Date(),
      old:        oldValue,
      new:        newValue,
      changeSize: newValue.length - oldValue.length
    });
  }
});
```

 

## Watching Computed Properties

```js
const cart = state({ items: [{ price: 10, quantity: 2 }, { price: 25, quantity: 1 }] });

computed(cart, {
  total() {
    return this.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
});

watch(cart, {
  total(newValue, oldValue) {
    console.log(`Total changed: $${oldValue} → $${newValue}`);
    if (newValue > 100) console.log('🎉 You qualify for free shipping!');
  }
});

cart.items.push({ price: 50, quantity: 1 }); // Logs total changed
```

 

## Cleanup and Disposal

### Stopping Watchers

```js
const user = state({ name: 'John' });

const stopWatching = watch(user, {
  name(newValue, oldValue) {
    console.log(`Name: ${oldValue} → ${newValue}`);
  }
});

user.name = 'Jane'; // Logs

stopWatching();

user.name = 'Bob'; // Nothing logged — watcher stopped
```

### Cleanup in Component Lifecycle

```js
function createUserComponent(userData) {
  const user = state(userData);

  const cleanup = watch(user, {
    name(newValue) {
      Id('user-name').update({ textContent: newValue });
    },
    email(newValue) {
      Id('user-email').update({ textContent: newValue });
    }
  });

  return {
    state: user,
    destroy() { cleanup(); }
  };
}

const component = createUserComponent({ name: 'John', email: 'john@example.com' });
// Later:
component.destroy();
```

 

## Real-World Examples

### Example 1: Auto-Save to Server

```js
const doc = state({ title: '', content: '', lastSaved: null });

let saveTimeout;

watch(doc, {
  title()   { debouncedSave(); },
  content() { debouncedSave(); }
});

function debouncedSave() {
  clearTimeout(saveTimeout);
  saveTimeout = setTimeout(async () => {
    await saveToServer({ title: doc.title, content: doc.content });
    doc.lastSaved = new Date();
    Id('save-status').update({ textContent: `Saved at ${doc.lastSaved.toLocaleTimeString()}` });
  }, 1000);
}
```

### Example 2: Form Validation

```js
const form   = state({ email: '', password: '', confirmPassword: '' });
const errors = state({ email: '', password: '', confirmPassword: '' });

watch(form, {
  email(newValue) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    errors.email = newValue && !emailRegex.test(newValue)
      ? 'Please enter a valid email address'
      : '';
  },
  password(newValue) {
    errors.password = newValue && newValue.length < 8
      ? 'Password must be at least 8 characters'
      : '';
    if (form.confirmPassword) validateConfirm();
  },
  confirmPassword() { validateConfirm(); }
});

function validateConfirm() {
  errors.confirmPassword = form.confirmPassword && form.password !== form.confirmPassword
    ? 'Passwords do not match'
    : '';
}

// Display errors reactively
effect(() => {
  Elements.update({
    'email-error':   { textContent: errors.email,           hidden: !errors.email },
    'password-error':{ textContent: errors.password,        hidden: !errors.password },
    'confirm-error': { textContent: errors.confirmPassword, hidden: !errors.confirmPassword }
  });
});
```

### Example 3: Analytics and Tracking

```js
const analytics = state({ currentPage: '/home', searchQuery: '' });
const eventLog  = [];

watch(analytics, {
  currentPage(newValue, oldValue) {
    const event = { type: 'page_view', from: oldValue, to: newValue, timestamp: new Date() };
    eventLog.push(event);
    sendToAnalytics(event);
  },
  searchQuery(newValue) {
    if (newValue) sendToAnalytics({ type: 'search', query: newValue });
  }
});
```

### Example 4: State Synchronization

```js
const settings = state({ theme: 'light', fontSize: 14, language: 'en' });

watch(settings, {
  theme(newValue)    { localStorage.setItem('theme', newValue); },
  fontSize(newValue) { localStorage.setItem('fontSize', String(newValue)); },
  language(newValue) { localStorage.setItem('language', newValue); }
});

// Load from storage on startup
settings.theme    = localStorage.getItem('theme')    || 'light';
settings.fontSize = parseInt(localStorage.getItem('fontSize')) || 14;
settings.language = localStorage.getItem('language') || 'en';
```

### Example 5: Notification Badge

```js
const notifications = state({ unreadCount: 0, lastNotification: null });

watch(notifications, {
  unreadCount(newValue, oldValue) {
    Id('notification-badge').update({
      textContent: newValue,
      hidden:      newValue === 0
    });

    document.title = newValue > 0 ? `(${newValue}) Messages - MyApp` : 'MyApp';

    if (newValue > oldValue) playNotificationSound();
  },
  lastNotification(newValue) {
    if (newValue) {
      showToast(newValue.message);
      notifications.unreadCount++;
    }
  }
});
```

### Example 6: Debug Logger

```js
const appState = state({ user: null, session: null, ui: { loading: false, error: null } });

if (process.env.NODE_ENV === 'development') {
  watch(appState, {
    user(newValue, oldValue)    { console.log('👤 User changed:', { old: oldValue, new: newValue }); },
    session(newValue, oldValue) { console.log('🔐 Session changed:', { old: oldValue, new: newValue }); },
    ui(newValue, oldValue)      { console.log('🎨 UI changed:', { old: oldValue, new: newValue }); }
  });
}
```

 

## Common Pitfalls

### Pitfall #1: Infinite Loops

❌ **Wrong:**
```js
watch(counter, {
  count(newValue) {
    counter.count = newValue + 1; // ❌ Modifying what we're watching!
  }
});
```

✅ **Correct:**
```js
watch(counter, {
  count(newValue) {
    derived.doubled = newValue * 2; // ✅ Modify a different property
  }
});
```

 

### Pitfall #2: Watchers Don't Run on Setup

```js
// ✅ If you need immediate initialization, call manually:
const user = state({ name: 'John' });

watch(user, { name(newValue) { Id('user-name').update({ textContent: newValue }); } });

// Initial render — watcher hasn't fired yet
Id('user-name').update({ textContent: user.name });

// Or use effect() instead — it runs immediately:
effect(() => { Id('user-name').update({ textContent: user.name }); });
```

 

### Pitfall #3: Not Cleaning Up Watchers

```js
// ✅ Always save and call the cleanup function
function createComponent() {
  const data = state({ value: 0 });

  const cleanup = watch(data, {
    value(newValue) { Id('widget').update({ textContent: newValue }); }
  });

  return { data, destroy: cleanup };
}

const widget = createComponent();
// Later:
widget.destroy(); // Stops the watcher, prevents memory leak
```

 

### Pitfall #4: Watching Non-Existent Properties

```js
// ✅ Always initialize the properties you plan to watch
const user = state({
  name:  'John',
  email: ''     // Initialize even if empty
});

watch(user, {
  email(newValue) { console.log('Email:', newValue); }
});
```

 

### Pitfall #5: Complex Logic in Watchers

```js
// ✅ Keep watchers simple — delegate to clear functions
watch(app, {
  user(newValue) {
    if (newValue) onUserLogin(newValue);
    else          onUserLogout();
  }
});

function onUserLogin(user) {
  fetchPreferences(user.id);
  loadOrders(user.id);
  // ...
}
```

 

## Summary

**What is `watch()`?**
Monitors specific properties in reactive state and executes callbacks when those properties change, providing both old and new values.

**Key Points to Remember:**

1️⃣ **Runs on change, not setup** — Executes when values change, not when defined

2️⃣ **Returns cleanup function** — Always save and call it when done

3️⃣ **Provides old and new values** — Both are passed to your callback

4️⃣ **Avoid infinite loops** — Don't modify the watched property inside its watcher

5️⃣ **Initialize properties** — Define properties before watching them

6️⃣ **Keep callbacks simple** — Delegate complex logic to separate functions

**`watch()` vs `effect()`:**

| Need | Use |
|------|-----|
| Run immediately on setup | `effect()` |
| Run only when something changes | `watch()` |
| Access old and new values | `watch()` |
| React to any dependency change | `effect()` |

**Quick Reference:**

```js
const user = state({ name: 'John', email: 'john@example.com' });

const stopWatching = watch(user, {
  name(newValue, oldValue) {
    console.log(`Name: ${oldValue} → ${newValue}`);
    Id('user-name').update({ textContent: newValue });
  },
  email(newValue) {
    saveToServer({ email: newValue });
  }
});

user.name  = 'Jane'; // Watcher fires automatically
user.email = 'jane@example.com'; // Watcher fires automatically

stopWatching(); // Stop when done
```

**Remember:** `watch()` is your tool for responding to specific property changes — set it up once and it monitors continuously. ✨
