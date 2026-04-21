[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `watchStorage() options.immediate` - Immediate Callback Configuration

## Quick Start (30 seconds)

```javascript
// Without immediate - callback only on changes
watchStorage('theme', (value) => {
  console.log('Theme changed:', value);
});
// Callback fires ONLY when theme changes in storage

// With immediate - callback fires immediately
watchStorage('theme', (value) => {
  console.log('Current theme:', value);
}, {
  immediate: true
});
// Callback fires NOW with current value, then on changes ✨
```

**What just happened?** You made the callback fire immediately with the current storage value instead of waiting for changes!

  

## What is `options.immediate`?

`options.immediate` is a configuration option for `watchStorage()` that **causes the callback to execute immediately with the current storage value**.

Simply put: instead of waiting for the value to change, the callback runs right away with whatever is currently stored.

Think of it as **checking the mailbox immediately** instead of waiting for new mail.

  

## Syntax

```javascript
watchStorage(key, callback, {
  immediate: boolean
})
```

**Value:**
- `true` - Call callback immediately with current value
- `false` - Only call callback on changes

**Default:** `false` (only fire on changes)

  

## Why Does This Exist?

### The Problem: Need Current Value at Startup

Without immediate mode, you miss the current value:

```javascript
// Theme is already set in storage
localStorage.setItem('theme', 'dark');

// Setup watcher
watchStorage('theme', (theme) => {
  applyTheme(theme);
});

// Problem:
// - Callback never fires ❌
// - Current theme not applied ❌
// - Theme only updates on next change ❌
// - Need to manually load initial value ❌
```

**What's the Real Issue?**

```
Current value in storage
        |
        v
Setup watcher
        |
        v
Callback waits for changes
        |
        v
Current value ignored ❌
```

**Problems:**
❌ **Missing initial value** - Current state not loaded  
❌ **Manual initialization** - Must separately load value  
❌ **Code duplication** - Init logic + change logic  
❌ **Inconsistent state** - UI doesn't match storage  

### The Solution with `options.immediate`

```javascript
// Theme already in storage
localStorage.setItem('theme', 'dark');

// Setup watcher with immediate
watchStorage('theme', (theme) => {
  applyTheme(theme);
}, {
  immediate: true
});

// Benefits:
// - Callback fires immediately ✅
// - Current theme applied ✅
// - No manual loading needed ✅
// - UI matches storage ✅
```

**What Just Happened?**

```
Current value in storage
        |
        v
Setup watcher with immediate: true
        |
        v
Callback fires NOW with 'dark'
        |
        v
Then watches for future changes ✅
```

**Benefits:**
✅ **Automatic initialization** - Current value loaded  
✅ **No duplication** - Same callback for init + changes  
✅ **Consistent state** - UI always matches storage  
✅ **Cleaner code** - One function handles everything  

  

## Mental Model

Think of watching without immediate as **waiting for mail**:

```
immediate: false (Wait for Mail)
┌─────────────────────┐
│  Mailbox has letter │
│                     │
│  Setup watcher      │
│                     │
│  Ignores current    │
│  letter ❌          │
│                     │
│  Only watches for   │
│  NEW letters        │
└─────────────────────┘
```

Think of immediate as **checking mailbox first**:

```
immediate: true (Check Then Wait)
┌─────────────────────┐
│  Mailbox has letter │
│                     │
│  Setup watcher      │
│       ↓             │
│  Check mailbox      │
│  Read current ✅    │
│       ↓             │
│  Then watch for     │
│  new letters        │
└─────────────────────┘
```

**Key Insight:** immediate mode handles both current value and future changes.

  

## How Does It Work?

The immediate option calls the callback right away:

### Execution Flow

```
watchStorage(key, callback, { immediate: true })
        |
        v
Read current value from storage
        |
        v
Call callback(currentValue, null) ✨
        |
        v
Setup watcher for future changes
        |
        v
Wait for storage events
```

### Implementation

```javascript
function watchStorage(key, callback, options = {}) {
  const { immediate = false, storage = 'localStorage' } = options;
  
  // Call immediately if requested
  if (immediate) {
    const currentValue = window[storage].getItem(key);
    callback(currentValue, null);
  }
  
  // Watch for future changes
  window.addEventListener('storage', (event) => {
    if (event.key === key) {
      callback(event.newValue, event.oldValue);
    }
  });
}
```

  

## Basic Usage

### Example 1: Without Immediate (Default)

```javascript
// Only fires on changes
watchStorage('theme', (theme) => {
  console.log('Theme changed:', theme);
});

// If theme is already 'dark', nothing happens
// Callback only fires when theme changes to something else
```

  

### Example 2: With Immediate

```javascript
// Fires immediately + on changes
watchStorage('theme', (theme) => {
  console.log('Theme:', theme);
  applyTheme(theme);
}, {
  immediate: true
});

// Immediately logs current theme
// Then continues to watch for changes
```

  

### Example 3: Initialization Pattern

```javascript
// Before: Manual initialization
const currentTheme = localStorage.getItem('theme');
if (currentTheme) {
  applyTheme(currentTheme);
}
watchStorage('theme', applyTheme);

// After: Automatic initialization
watchStorage('theme', applyTheme, {
  immediate: true
});
// Much cleaner! ✅
```

  

## Real-World Examples

### Example 1: Theme Application

```javascript
// Apply theme immediately and watch for changes
watchStorage('theme', (theme) => {
  if (theme) {
    document.body.className = theme;
  } else {
    document.body.className = 'light'; // Default
  }
}, {
  immediate: true
});

// Page loads with correct theme immediately ✨
// Updates when changed in another tab ✨
```

  

### Example 2: User Preferences Sync

```javascript
// Load and apply all preferences immediately
const preferences = [
  { key: 'fontSize', apply: (val) => document.body.style.fontSize = val + 'px' },
  { key: 'language', apply: (val) => setLanguage(val) },
  { key: 'notifications', apply: (val) => toggleNotifications(val === 'true') }
];

preferences.forEach(({ key, apply }) => {
  watchStorage(key, (value) => {
    if (value) apply(value);
  }, {
    immediate: true
  });
});

// All preferences applied on page load ✨
```

  

### Example 3: Authentication State

```javascript
// Check auth status immediately
watchStorage('authToken', (token) => {
  if (token) {
    // User is logged in
    showDashboard();
    loadUserData();
  } else {
    // User is logged out
    showLoginScreen();
  }
}, {
  immediate: true,
  storage: 'sessionStorage'
});

// Correct screen shown immediately ✨
// Updates if user logs out in another tab ✨
```

  

### Example 4: Feature Flags

```javascript
// Apply feature flags immediately
watchStorage('featureFlags', (flagsJson) => {
  const flags = flagsJson ? JSON.parse(flagsJson) : {};
  
  // Apply each flag
  Object.entries(flags).forEach(([flag, enabled]) => {
    if (enabled) {
      enableFeature(flag);
    } else {
      disableFeature(flag);
    }
  });
}, {
  immediate: true
});

// Features enabled correctly on page load ✨
```

  

### Example 5: Shopping Cart Restoration

```javascript
// Restore cart immediately
watchStorage('shoppingCart', (cartJson) => {
  const cart = cartJson ? JSON.parse(cartJson) : { items: [] };
  
  // Update UI
  document.getElementById('cart-count').textContent = cart.items.length;
  
  // Render cart items
  renderCartItems(cart.items);
  
  // Calculate total
  const total = cart.items.reduce((sum, item) => sum + item.price, 0);
  document.getElementById('cart-total').textContent = `$${total}`;
}, {
  immediate: true
});

// Cart restored on page load ✨
// Updates when modified in another tab ✨
```

  

## Common Patterns

### Pattern 1: Initialize and Watch

```javascript
// Clean pattern for init + watch
function setupTheme() {
  watchStorage('theme', (theme) => {
    applyTheme(theme || 'light');
  }, {
    immediate: true
  });
}

// One function, two behaviors ✅
```

  

### Pattern 2: Conditional Immediate

```javascript
function watchWithOptionalImmediate(key, callback, loadImmediately = true) {
  return watchStorage(key, callback, {
    immediate: loadImmediately
  });
}

// Usage
watchWithOptionalImmediate('theme', applyTheme, true);  // Load now
watchWithOptionalImmediate('temp', handleTemp, false);  // Wait for changes
```

  

### Pattern 3: Default Value Handling

```javascript
watchStorage('language', (lang) => {
  const language = lang || 'en'; // Default to English
  setLanguage(language);
}, {
  immediate: true
});

// Handles both missing and present values ✅
```

  

### Pattern 4: Multiple Immediate Watchers

```javascript
const watchers = [
  { key: 'theme', handler: applyTheme },
  { key: 'language', handler: setLanguage },
  { key: 'fontSize', handler: setFontSize }
];

watchers.forEach(({ key, handler }) => {
  watchStorage(key, handler, {
    immediate: true
  });
});

// All initialized immediately ✅
```

  

### Pattern 5: Async Initialization

```javascript
watchStorage('config', async (configJson) => {
  if (!configJson) return;
  
  const config = JSON.parse(configJson);
  
  // Async initialization
  await loadDependencies(config);
  await applyConfiguration(config);
  
  console.log('Config applied');
}, {
  immediate: true
});
```

  

### Pattern 6: State Restoration

```javascript
// Restore complete app state
watchStorage('appState', (stateJson) => {
  if (!stateJson) {
    // No saved state - use defaults
    loadDefaultState();
    return;
  }
  
  const savedState = JSON.parse(stateJson);
  
  // Restore each piece
  restoreUI(savedState.ui);
  restoreData(savedState.data);
  restorePreferences(savedState.prefs);
  
  console.log('State restored');
}, {
  immediate: true
});

// App state restored on load ✨
```

  

## When to Use Immediate

### Use `immediate: true` For:

✅ **Theme/appearance** - Apply on page load  
✅ **User preferences** - Initialize UI state  
✅ **Authentication** - Check login status immediately  
✅ **Feature flags** - Enable features on startup  
✅ **Saved data** - Restore previous state  
✅ **Configuration** - Load settings immediately  

### Use `immediate: false` For:

✅ **Real-time updates** - Only care about changes  
✅ **Notifications** - Only show new notifications  
✅ **Activity tracking** - Only track new activity  
✅ **Event logging** - Only log new events  

  

## Important Notes

### 1. Callback Signature with Immediate

```javascript
watchStorage('key', (newValue, oldValue) => {
  console.log('New:', newValue);
  console.log('Old:', oldValue);
}, {
  immediate: true
});

// On immediate call:
// newValue = current storage value
// oldValue = null (no previous value)

// On subsequent changes:
// newValue = new value
// oldValue = previous value
```

  

### 2. Null Handling

```javascript
watchStorage('settings', (value) => {
  if (value === null) {
    // Key doesn't exist or was removed
    loadDefaults();
  } else {
    // Key has a value
    applySettings(JSON.parse(value));
  }
}, {
  immediate: true
});
```

  

### 3. Does Not Fire in Same Tab

```javascript
// Important: storage event doesn't fire in same tab
watchStorage('theme', (theme) => {
  console.log('Theme changed:', theme);
}, {
  immediate: true
});

// immediate: true fires on setup ✅
// But local changes won't trigger callback ⚠️
// (Only changes from other tabs trigger callback)
```

  

## Summary

**What is `watchStorage() options.immediate`?**  
A configuration option that makes the callback execute immediately with the current storage value.

**Why use it?**
- ✅ Initialize state on page load
- ✅ No manual value loading needed
- ✅ Single callback for init + changes
- ✅ Cleaner, less duplicated code
- ✅ Consistent state from the start

**Key Takeaway:**

```
immediate: false         immediate: true
      |                       |
Wait for changes         Call immediately
      |                       |
Current value ignored    Current value used
      |                       |
Need manual load ❌     Automatic init ✅
```

**One-Line Rule:** Use `immediate: true` to initialize state from storage on page load.

**Best Practices:**
- Use `immediate: true` for UI initialization
- Handle null values (key doesn't exist)
- Provide default values when null
- Use for theme, preferences, auth state
- Skip immediate for notification-only watchers

**Remember:** immediate mode loads current state automatically! 🎉