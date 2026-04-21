[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)


# `watchStorage`

## Quick Start (30 seconds)

```javascript
// Watch a specific storage key
const cleanup = ReactiveUtils.watchStorage('theme', (newValue, oldValue) => {
  console.log(`Theme changed from ${oldValue} to ${newValue}`);
  document.body.className = newValue;
});

// Update storage from anywhere
localStorage.setItem('theme', 'dark');
// → Callback runs: "Theme changed from null to dark"

localStorage.setItem('theme', 'light');
// → Callback runs: "Theme changed from dark to light"

// Stop watching
cleanup();
```

**What just happened?**
- ✅ Set up a watcher for specific storage key
- ✅ Callback runs automatically when key changes
- ✅ Works with both manual and reactive storage changes
- ✅ Returns cleanup function to stop watching

 

## What is watchStorage?

**Simply put:** `watchStorage()` watches a specific browser storage key and runs a callback whenever that key changes.

Think of it as an **event listener for storage**:
- 👀 Watches one specific key
- 🔔 Notifies you when it changes
- 📊 Provides old and new values
- 🧹 Easy to clean up

**What it does:**
- Monitors a specific storage key
- Calls your callback on changes
- Works across tabs (for localStorage)
- Handles JSON deserialization automatically
- Returns cleanup function

**Comparison:**

| Method | What It Does |
|  --|    -|
| `autoSave()` | Connects reactive state to storage |
| `reactiveStorage()` | Makes entire storage reactive |
| **`watchStorage()`** | Watches **one specific key** |

 

## Why Does This Exist?

### The Problem: Manual Storage Monitoring is Tedious

Without `watchStorage()`, monitoring storage requires manual event listeners:

```javascript
// Manual storage monitoring
function watchTheme() {
  let oldValue = localStorage.getItem('theme');
  
  // Parse JSON manually
  try {
    oldValue = JSON.parse(oldValue);
  } catch (e) {
    // Handle parse errors
  }
  
  window.addEventListener('storage', (event) => {
    // Check if it's the right key
    if (event.key !== 'theme') return;
    
    // Parse new value manually
    let newValue = event.newValue;
    try {
      newValue = JSON.parse(newValue);
    } catch (e) {
      // Handle parse errors
    }
    
    // Update UI
    document.body.className = newValue;
    
    // Update old value
    oldValue = newValue;
  });
}
```

**What's tedious:**

```
Storage changes
   ↓
[Check if it's the right key]
   ↓
[Manually parse JSON]
   ↓
[Handle parse errors]
   ↓
[Track old value yourself]
   ↓
[Update UI]
```

**Problems:**
❌ Must manually check which key changed  
❌ Must manually parse JSON  
❌ Must handle parsing errors  
❌ Must track old values yourself  
❌ Storage event doesn't fire in same tab  
❌ Must poll or use complex workarounds  

### The Solution: watchStorage() Handles Everything

```javascript
// With watchStorage
ReactiveUtils.watchStorage('theme', (newValue, oldValue) => {
  document.body.className = newValue;
});
```

**What's happening:**

```
Storage changes
   ↓
[watchStorage detects change automatically]
   ↓
[Parses JSON automatically]
   ↓
[Tracks old value automatically]
   ↓
[Calls your callback]
   ↓
Your code runs
```

**Benefits:**
✅ One line to watch a key  
✅ Automatic JSON parsing  
✅ Old/new values provided  
✅ Works in same tab  
✅ Works across tabs  
✅ Easy cleanup  

 

## Mental Model

Think of `watchStorage()` as a **security camera** pointing at one specific location:

### Without watchStorage (Manual Monitoring)
```
Your Code
├─ Set up storage event listener
├─ Check every storage event
├─ Filter for your key
├─ Parse JSON yourself
└─ Track old values yourself

❌ Lots of boilerplate
❌ Easy to get wrong
```

### With watchStorage (Automatic Monitoring)
```
Your Code                     watchStorage
├─ Specify key           →   [Watches that key]
├─ Provide callback      →   [Calls on changes]
└─ Get old/new values    ←   [Provided automatically]

✅ Clean & simple
✅ Just works
```

**Key Insight:**  
`watchStorage()` is like setting up a specific alert: "Tell me whenever THIS specific thing changes."

 

## How Does It Work?

### Internal Architecture

```
Your Code               watchStorage              Browser Storage
──────────────────────────────────────────────────────────────────
watchStorage('theme')  [Creates watcher]     →  [Monitors 'theme']
        ↓                      ↓
[Callback registered]    [Tracks old value]
        ↓                      ↓
localStorage.theme=...  →  [Detects change]  →  [Compares values]
        ↓                      ↓
[Callback called]      ←  [Calls with old/new]
```

### Tracking Mechanism

```javascript
// Behind the scenes
{
  key: 'theme',
  oldValue: 'dark',
  callback: (newValue, oldValue) => {...},
  reactiveStorage: ReactiveStorageProxy,
  effectCleanup: Function
}
```

**How it detects changes:**

1. Creates reactive storage internally
2. Sets up effect that reads your key
3. Effect runs when key changes
4. Compares old/new values
5. Calls your callback if different

 

## Basic Usage

### Example 1: Theme Watcher

```javascript
// Watch theme changes
ReactiveUtils.watchStorage('theme', (newValue, oldValue) => {
  console.log(`Theme changed: ${oldValue} → ${newValue}`);
  
  // Update UI
  document.body.className = newValue;
  
  // Update button text
  const btn = document.getElementById('theme-toggle');
  btn.textContent = newValue === 'dark' ? '☀️' : '🌙';
});

// Change theme
localStorage.setItem('theme', 'dark');
// → Console: "Theme changed: null → dark"
// → Body class updated
// → Button text updated
```

 

### Example 2: User Session Watcher

```javascript
// Watch for login/logout
ReactiveUtils.watchStorage('user', (newUser, oldUser) => {
  if (newUser && !oldUser) {
    // User logged in
    console.log('Welcome,', newUser.name);
    showDashboard();
  } else if (!newUser && oldUser) {
    // User logged out
    console.log('Goodbye,', oldUser.name);
    showLoginPage();
  } else if (newUser && oldUser) {
    // User changed
    console.log('Switched user:', newUser.name);
    reloadData();
  }
});

// Login
localStorage.setItem('user', JSON.stringify({ name: 'Alice' }));
// → "Welcome, Alice"

// Logout
localStorage.removeItem('user');
// → "Goodbye, Alice"
```

 

### Example 3: Counter Synchronization

```javascript
// Watch counter in all tabs
ReactiveUtils.watchStorage('count', (newValue, oldValue) => {
  const display = document.getElementById('counter');
  display.textContent = newValue || 0;
  
  // Animate change
  display.classList.add('updated');
  setTimeout(() => display.classList.remove('updated'), 300);
});

// Update counter
function increment() {
  const current = parseInt(localStorage.getItem('count') || '0');
  localStorage.setItem('count', current + 1);
}

// All tabs show updated counter automatically!
```

 

### Example 4: Feature Flag Watcher

```javascript
// Watch feature flags
ReactiveUtils.watchStorage('features', (newFlags, oldFlags) => {
  console.log('Feature flags updated:', newFlags);
  
  // Toggle features
  Object.entries(newFlags).forEach(([feature, enabled]) => {
    const element = document.querySelector(`[data-feature="${feature}"]`);
    if (element) {
      element.style.display = enabled ? 'block' : 'none';
    }
  });
});

// Enable feature
const features = JSON.parse(localStorage.getItem('features') || '{}');
features.darkMode = true;
localStorage.setItem('features', JSON.stringify(features));
// → Feature flags updated, UI updated
```

 

### Example 5: Cart Watcher

```javascript
// Watch shopping cart
ReactiveUtils.watchStorage('cart', (newCart, oldCart) => {
  const newCount = newCart ? newCart.length : 0;
  const oldCount = oldCart ? oldCart.length : 0;
  
  console.log(`Cart items: ${oldCount} → ${newCount}`);
  
  // Update badge
  document.getElementById('cart-count').textContent = newCount;
  
  // Show notification
  if (newCount > oldCount) {
    showNotification('Item added to cart');
  } else if (newCount < oldCount) {
    showNotification('Item removed from cart');
  }
});
```

 

## Options Reference

### Full Syntax

```javascript
ReactiveUtils.watchStorage(key, callback, {
  storage: 'localStorage',    // 'localStorage' or 'sessionStorage'
  namespace: '',              // Namespace prefix
  immediate: false            // Run callback immediately with current value
});
```

### Option: storage

**Default:** `'localStorage'`  
**Options:** `'localStorage'` | `'sessionStorage'`

Specify which storage to watch:

```javascript
// Watch localStorage (default)
ReactiveUtils.watchStorage('theme', callback);

// Watch sessionStorage
ReactiveUtils.watchStorage('sessionData', callback, {
  storage: 'sessionStorage'
});
```

**Note:** sessionStorage watchers only work within the same tab.

 

### Option: namespace

**Default:** `''` (empty string)

Watch keys with a namespace prefix:

```javascript
// Watch 'myapp:theme' key
ReactiveUtils.watchStorage('theme', callback, {
  namespace: 'myapp'
});

// Triggered by:
localStorage.setItem('myapp:theme', 'dark');

// Not triggered by:
localStorage.setItem('theme', 'dark'); // Different key
```

**Use case:**
```javascript
// Organize by feature
ReactiveUtils.watchStorage('profile', callback, {
  namespace: 'user'
});

ReactiveUtils.watchStorage('items', callback, {
  namespace: 'cart'
});

// Keys: 'user:profile', 'cart:items'
```

 

### Option: immediate

**Default:** `false`

Whether to call the callback immediately with the current value:

```javascript
// immediate: false (default)
ReactiveUtils.watchStorage('theme', (newValue, oldValue) => {
  console.log(newValue);
});
// Callback not called yet

localStorage.setItem('theme', 'dark');
// → Console: "dark"

// immediate: true
ReactiveUtils.watchStorage('theme', (newValue, oldValue) => {
  console.log(newValue);
}, { immediate: true });
// → Console: "dark" (called immediately with current value)
```

**When to use `immediate: true`:**

```javascript
// Initialize UI with current value
ReactiveUtils.watchStorage('theme', (theme) => {
  document.body.className = theme;
}, { immediate: true });
// UI updated immediately with saved theme

// Without immediate, need manual initialization:
const theme = localStorage.getItem('theme');
document.body.className = theme;

ReactiveUtils.watchStorage('theme', (theme) => {
  document.body.className = theme;
});
```

 

## Advanced Patterns

### Pattern 1: Multiple Watchers on Same Key

```javascript
// Multiple callbacks for same key
ReactiveUtils.watchStorage('user', (newUser) => {
  updateHeader(newUser);
});

ReactiveUtils.watchStorage('user', (newUser) => {
  updateSidebar(newUser);
});

ReactiveUtils.watchStorage('user', (newUser) => {
  trackAnalytics(newUser);
});

// Update user - all three callbacks run
localStorage.setItem('user', JSON.stringify({ name: 'Alice' }));
```

 

### Pattern 2: Conditional Actions

```javascript
ReactiveUtils.watchStorage('config', (newConfig, oldConfig) => {
  // Only reload if critical settings changed
  const criticalKeys = ['apiUrl', 'apiKey', 'environment'];
  
  const needsReload = criticalKeys.some(key => 
    newConfig[key] !== oldConfig[key]
  );
  
  if (needsReload) {
    console.warn('Critical config changed, reloading...');
    location.reload();
  } else {
    console.log('Config updated, no reload needed');
  }
});
```

 

### Pattern 3: Debounced Watcher

```javascript
// Debounce rapid changes
function createDebouncedWatcher(key, callback, delay) {
  let timeout;
  
  return ReactiveUtils.watchStorage(key, (newValue, oldValue) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      callback(newValue, oldValue);
    }, delay);
  });
}

// Usage
const cleanup = createDebouncedWatcher('search', (query) => {
  console.log('Searching for:', query);
  performSearch(query);
}, 500);
```

 

### Pattern 4: Watcher with Validation

```javascript
ReactiveUtils.watchStorage('settings', (newSettings, oldSettings) => {
  // Validate new settings
  if (!newSettings.theme || !['dark', 'light'].includes(newSettings.theme)) {
    console.error('Invalid theme, reverting');
    localStorage.setItem('settings', JSON.stringify(oldSettings));
    return;
  }
  
  if (newSettings.fontSize < 10 || newSettings.fontSize > 24) {
    console.error('Invalid font size, reverting');
    localStorage.setItem('settings', JSON.stringify(oldSettings));
    return;
  }
  
  // Valid settings, apply them
  applySettings(newSettings);
});
```

 

### Pattern 5: History Tracking

```javascript
const history = [];

ReactiveUtils.watchStorage('value', (newValue, oldValue) => {
  history.push({
    timestamp: Date.now(),
    oldValue,
    newValue
  });
  
  // Keep last 100 changes
  if (history.length > 100) {
    history.shift();
  }
  
  console.log(`History: ${history.length} changes`);
});

// View history
function showHistory() {
  history.forEach(entry => {
    const time = new Date(entry.timestamp).toLocaleTimeString();
    console.log(`[${time}] ${entry.oldValue} → ${entry.newValue}`);
  });
}
```

 

### Pattern 6: Cross-Tab Communication

```javascript
// Tab 1: Send messages
function sendMessage(message) {
  const messages = JSON.parse(localStorage.getItem('messages') || '[]');
  messages.push({
    id: Date.now(),
    text: message,
    sender: 'Tab 1'
  });
  localStorage.setItem('messages', JSON.stringify(messages));
}

// Tab 2: Receive messages
ReactiveUtils.watchStorage('messages', (newMessages, oldMessages) => {
  const oldCount = oldMessages ? oldMessages.length : 0;
  const newCount = newMessages ? newMessages.length : 0;
  
  if (newCount > oldCount) {
    const newMessage = newMessages[newMessages.length - 1];
    console.log('New message from', newMessage.sender + ':', newMessage.text);
    showNotification(newMessage.text);
  }
});

// Tab 1: sendMessage('Hello from Tab 1!');
// Tab 2: → Notification: "Hello from Tab 1!"
```

 

### Pattern 7: Synchronized State Machine

```javascript
const states = ['idle', 'loading', 'success', 'error'];

ReactiveUtils.watchStorage('appState', (newState, oldState) => {
  console.log(`State: ${oldState} → ${newState}`);
  
  // Update UI based on state
  document.querySelectorAll('[data-state]').forEach(el => {
    el.style.display = el.dataset.state === newState ? 'block' : 'none';
  });
  
  // State-specific actions
  switch (newState) {
    case 'loading':
      showSpinner();
      break;
    case 'success':
      hideSpinner();
      break;
    case 'error':
      hideSpinner();
      showErrorMessage();
      break;
  }
});

// Transition states
function setState(state) {
  if (!states.includes(state)) {
    console.error('Invalid state:', state);
    return;
  }
  localStorage.setItem('appState', state);
}

// All tabs transition together
setState('loading');
```

 

## Comparison with Other Methods

### watchStorage vs. autoSave

```javascript
// autoSave: Connects reactive state to storage
const user = state({ name: 'Alice' });
ReactiveUtils.autoSave(user, 'user');
user.name = 'Bob'; // Automatically saves

// watchStorage: Watches storage key
ReactiveUtils.watchStorage('user', (newUser) => {
  console.log('User changed:', newUser);
});
```

**When to use each:**
- Use `autoSave()` when you have reactive state that should persist
- Use `watchStorage()` when you want to react to storage changes from any source

 

### watchStorage vs. reactiveStorage

```javascript
// reactiveStorage: Makes storage reactive, tracks ANY key accessed
const storage = ReactiveUtils.reactiveStorage();

effect(() => {
  const theme = storage.get('theme'); // Tracks 'theme'
  const lang = storage.get('lang');   // Tracks 'lang'
  console.log(theme, lang);
});

// watchStorage: Watches ONE specific key
ReactiveUtils.watchStorage('theme', (theme) => {
  console.log('Theme:', theme);
});
// Only tracks 'theme', not 'lang'
```

**When to use each:**
- Use `reactiveStorage()` when you need to track multiple dynamic keys in effects
- Use `watchStorage()` when you want to watch ONE specific key

 

### watchStorage vs. effect

```javascript
// With effect + reactiveStorage
const storage = ReactiveUtils.reactiveStorage();

const cleanup = effect(() => {
  const theme = storage.get('theme');
  console.log('Theme:', theme);
});

// With watchStorage
const cleanup = ReactiveUtils.watchStorage('theme', (theme) => {
  console.log('Theme:', theme);
});
```

**They're similar, but watchStorage:**
- ✅ Simpler syntax for single keys
- ✅ Provides old/new values
- ✅ Works with namespace option
- ✅ Easier to understand intent

 

## Common Pitfalls

### Pitfall 1: Forgetting to Clean Up

```javascript
// ❌ Watcher keeps running forever
function setupThemeWatcher() {
  ReactiveUtils.watchStorage('theme', (theme) => {
    console.log('Theme:', theme);
  });
  // Watcher never stops!
}

// ✅ Store and call cleanup
function setupThemeWatcher() {
  const cleanup = ReactiveUtils.watchStorage('theme', (theme) => {
    console.log('Theme:', theme);
  });
  
  return cleanup;
}

// Later
const cleanup = setupThemeWatcher();
// When done:
cleanup();
```

 

### Pitfall 2: Modifying Storage in Callback

```javascript
// ❌ Creates infinite loop
ReactiveUtils.watchStorage('count', (count) => {
  // This triggers the watcher again!
  localStorage.setItem('count', count + 1);
});

// ✅ Only read, don't modify
ReactiveUtils.watchStorage('count', (count) => {
  console.log('Count:', count);
  updateUI(count);
});
```

 

### Pitfall 3: Not Handling Null Values

```javascript
// ❌ Crashes if key doesn't exist
ReactiveUtils.watchStorage('user', (user) => {
  console.log(user.name); // Error if user is null!
});

// ✅ Handle null
ReactiveUtils.watchStorage('user', (user) => {
  if (user) {
    console.log(user.name);
  } else {
    console.log('No user');
  }
});

// ✅ Or provide default
ReactiveUtils.watchStorage('user', (user) => {
  const name = user ? user.name : 'Guest';
  console.log(name);
});
```

 

### Pitfall 4: Expecting sessionStorage to Sync

```javascript
// ❌ sessionStorage doesn't sync across tabs
ReactiveUtils.watchStorage('data', (data) => {
  console.log('Data:', data);
}, { storage: 'sessionStorage' });

// Tab 1: localStorage.setItem('data', 'value');
// Tab 2: Callback doesn't run (different session!)

// ✅ Use localStorage for cross-tab sync
ReactiveUtils.watchStorage('data', (data) => {
  console.log('Data:', data);
}, { storage: 'localStorage' });
```

 

### Pitfall 5: Setting Up Multiple Identical Watchers

```javascript
// ❌ Creates duplicate watchers
function init() {
  ReactiveUtils.watchStorage('theme', callback);
}

init(); // Watcher 1
init(); // Watcher 2 (duplicate!)
init(); // Watcher 3 (duplicate!)

// Callback runs 3 times per change!

// ✅ Track and clean up
let themeWatcher = null;

function init() {
  if (themeWatcher) {
    themeWatcher(); // Clean up old
  }
  themeWatcher = ReactiveUtils.watchStorage('theme', callback);
}
```

 

## Summary

**watchStorage() watches a specific storage key and runs a callback when it changes.**

**Key Takeaways:**

1. **Basic Usage:**
   ```javascript
   const cleanup = ReactiveUtils.watchStorage(key, callback, options);
   ```

2. **Callback Signature:**
   ```javascript
   (newValue, oldValue) => {
     // newValue: Current value
     // oldValue: Previous value
   }
   ```

3. **Options:**
   - `storage` - 'localStorage' or 'sessionStorage'
   - `namespace` - Key prefix
   - `immediate` - Run callback immediately

4. **Returns Cleanup Function:**
   ```javascript
   const cleanup = ReactiveUtils.watchStorage('key', callback);
   cleanup(); // Stop watching
   ```

5. **Works Across Tabs:**
   - localStorage watchers sync across tabs
   - sessionStorage watchers work only in same tab

**Quick Mental Model:**  
watchStorage() = addEventListener() for storage. Simple, focused, and predictable.

**When to Use:**
- ✅ Need to watch ONE specific key
- ✅ Want old/new values in callback
- ✅ Don't need full reactive integration
- ✅ Simple storage monitoring

**When Not to Use:**
- ❌ Need to track multiple dynamic keys → Use `reactiveStorage()`
- ❌ Have reactive state to persist → Use `autoSave()`
- ❌ Need complex reactive patterns → Use `effect()` + `reactiveStorage()`

**Best Practices:**
- ✅ Always store and call cleanup function
- ✅ Handle null values in callback
- ✅ Don't modify watched key in callback
- ✅ Use localStorage for cross-tab sync
- ✅ Use `immediate: true` for initial UI setup

**Common Pattern:**
```javascript
const cleanup = ReactiveUtils.watchStorage('theme', (theme, prevTheme) => {
  console.log(`Theme: ${prevTheme} → ${theme}`);
  document.body.className = theme;
}, {
  storage: 'localStorage',
  immediate: true
});

// Remember to cleanup
window.addEventListener('beforeunload', cleanup);
```

watchStorage() makes storage monitoring simple and reliable! 🎉
```

 
