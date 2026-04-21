[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Understanding `untrack()` - A Beginner's Guide

## Quick Start (30 seconds)

Need to read reactive state without creating dependencies? Here's how:

```js
const counter = state({ count: 0, debug: false });

// Effect that depends on count but not debug
effect(() => {
  console.log('Count:', counter.count);

  // Read debug without tracking it
  const shouldLog = untrack(() => counter.debug);

  if (shouldLog) {
    console.log('Debug mode active');
  }
});
// Logs: "Count: 0"

counter.debug = true;   // Effect doesn't re-run (not tracked)
counter.count = 5;      // Effect re-runs (tracked)
// Logs: "Count: 5"
// Logs: "Debug mode active"
```

**That's it!** The `untrack()` function lets you read reactive state without creating dependencies!

 

## What is `untrack()`?

`untrack()` is a **dependency control function** that allows you to read reactive state without tracking those reads as dependencies. Code inside `untrack()` can access reactive properties without causing the current effect to depend on them.

**Untracked reads:**
- Access reactive state without creating dependencies
- Effects don't re-run when untracked properties change
- Useful for conditional logic and metadata
- Prevents unnecessary effect re-execution

Think of it as **reading without subscribing** - you can check the value, but you won't be notified when it changes.

 

## Syntax

```js
// Using the shortcut
untrack(fn)

// Using the full namespace
ReactiveUtils.untrack(fn)
```

**Both styles are valid!** Choose whichever you prefer:
- **Shortcut style** (`untrack()`) - Clean and concise
- **Namespace style** (`ReactiveUtils.untrack()`) - Explicit and clear

**Parameters:**
- `fn` - A function that reads reactive state (required)

**Returns:**
- The return value of the function `fn`

 

## Why Does This Exist?

### The Problem with Unwanted Dependencies

Let's say you have an effect that needs to read some reactive values but shouldn't depend on all of them:

```javascript
const app = state({
  count: 0,
  debugMode: false,
  lastUpdate: Date.now()
});

// Effect that logs count
effect(() => {
  console.log('Count:', app.count);

  // We want to check debugMode, but NOT depend on it
  if (app.debugMode) {
    console.log('Debug: Last update at', app.lastUpdate);
  }
});
// Logs: "Count: 0"

// Problem: Effect re-runs when debugMode changes!
app.debugMode = true;
// Logs: "Count: 0"
// Logs: "Debug: Last update at ..."

// We only wanted it to re-run when count changes!
```

This works, but creates **unwanted dependencies**. The effect re-runs when `debugMode` or `lastUpdate` change, even though we only care about `count`.

**What's the Real Issue?**

```
Normal Effect Tracking:
┌──────────────────┐
│  effect(() => {  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Read count       │ ← Tracked
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Read debugMode   │ ← Tracked (unwanted!)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Read lastUpdate  │ ← Tracked (unwanted!)
└────────┬─────────┘
         │
         ▼
  Effect depends on
  ALL 3 properties!
         │
         ▼
  ANY change triggers
  effect re-run!
```

**Problems:**
❌ Effect depends on properties it shouldn't
❌ Effect re-runs when debug metadata changes
❌ Unnecessary re-executions waste performance
❌ Can't read reactive values without creating dependencies
❌ No control over which reads are tracked
❌ Conditional logic creates unwanted dependencies

**Why This Becomes a Problem:**

Sometimes you need to:
- Read configuration values that shouldn't trigger re-runs
- Check debug flags without depending on them
- Access metadata for logging without tracking
- Conditionally use values without creating dependencies
- Optimize by reducing effect re-executions

### The Solution with `untrack()`

When you use `untrack()`, you can read values without tracking them:

```javascript
const app = state({
  count: 0,
  debugMode: false,
  lastUpdate: Date.now()
});

// Effect that only depends on count
effect(() => {
  console.log('Count:', app.count);  // Tracked

  // Read without tracking
  const debug = untrack(() => app.debugMode);
  const lastUpdate = untrack(() => app.lastUpdate);

  if (debug) {
    console.log('Debug: Last update at', lastUpdate);
  }
});
// Logs: "Count: 0"

// Effect doesn't re-run (not tracked)
app.debugMode = true;  // No log
app.lastUpdate = Date.now();  // No log

// Effect re-runs (tracked)
app.count = 5;
// Logs: "Count: 5"
// Logs: "Debug: Last update at ..."
```

**What Just Happened?**

```
Untracked Effect:
┌──────────────────┐
│  effect(() => {  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Read count       │ ← Tracked ✅
└────────┬─────────┘
         │
         ▼
┌──────────────────────┐
│ untrack(() => {      │
│   Read debugMode     │ ← NOT tracked ❌
│   Read lastUpdate    │ ← NOT tracked ❌
│ })                   │
└──────────┬───────────┘
           │
           ▼
  Effect only depends
  on count!
           │
           ▼
  Only count changes
  trigger re-run!
```

With `untrack()`:
- Read any reactive value without creating dependency
- Effect only re-runs for tracked values
- Full control over dependencies
- Better performance
- Cleaner dependency graph

**Benefits:**
✅ Read reactive state without creating dependencies
✅ Effects only re-run when truly needed
✅ Better performance (fewer re-executions)
✅ Fine-grained dependency control
✅ Can use reactive values conditionally
✅ Cleaner, more intentional reactivity

 

## Mental Model

Think of `untrack()` like **window shopping**:

```
Normal Reading (Subscribing):
┌──────────────────┐
│ Read value       │
│ (Subscribe to    │
│  notifications)  │
└────────┬─────────┘
         │
         ▼
   Value changes
         │
         ▼
   🔔 Notification!
         │
         ▼
   You're informed

Untracked Reading (Window Shopping):
┌──────────────────┐
│ untrack(() =>    │
│   Read value     │
│ )                │
│ (Just look,      │
│  don't subscribe)│
└────────┬─────────┘
         │
         ▼
   Value changes
         │
         ▼
   🔕 No notification
         │
         ▼
   You're not informed
```

**Key Insight:** Just like window shopping lets you look at items without buying them (and getting marketing emails), `untrack()` lets you read reactive values without subscribing to their updates.

 

## How Does It Work?

### The Magic: Temporarily Disable Tracking

When you call `untrack()`, here's what happens behind the scenes:

```javascript
// What you write:
effect(() => {
  const count = app.count;  // Tracked
  const debug = untrack(() => app.debugMode);  // Not tracked
});

// What actually happens (simplified):
// In the effect
currentEffect = effectObject;  // Set current effect

// Read count - tracking is ON
app.count;  // currentEffect is set, so this read is tracked

// untrack() call
const previousEffect = currentEffect;
currentEffect = null;  // Disable tracking
try {
  const debug = app.debugMode;  // currentEffect is null, not tracked
  return debug;
} finally {
  currentEffect = previousEffect;  // Restore tracking
}
```

**In other words:** `untrack()`:
1. Saves the current effect context
2. Temporarily sets current effect to `null`
3. Runs your function (reads aren't tracked)
4. Restores the current effect context
5. Returns the function result

### Under the Hood

```
untrack(fn) implementation:
untrack(fn) {
  const prev = currentEffect;  // Save current
  currentEffect = null;        // Disable tracking
  try {
    return fn();               // Run function
  } finally {
    currentEffect = prev;      // Restore
  }
}
```

**What happens:**

1️⃣ **Saves** current effect context
2️⃣ **Disables** tracking (currentEffect = null)
3️⃣ **Runs** your function
4️⃣ **Restores** effect context
5️⃣ **Returns** function result

 

## Basic Usage

### Reading Without Dependencies

The simplest way to use `untrack()`:

```js
const config = state({
  theme: 'dark',
  debugMode: false,
  count: 0
});

effect(() => {
  // Only track count
  console.log('Count:', config.count);

  // Read theme without tracking
  const theme = untrack(() => config.theme);
  console.log('Theme:', theme);
});

config.theme = 'light';  // Effect doesn't re-run
config.count = 5;        // Effect re-runs
```

### Returning Values from untrack()

`untrack()` returns whatever your function returns:

```js
effect(() => {
  const user = app.currentUser;  // Tracked

  // Read multiple values without tracking
  const settings = untrack(() => {
    return {
      debug: app.debugMode,
      theme: app.theme,
      locale: app.locale
    };
  });

  console.log(user, settings);
});
```

### Conditional Untracked Reads

```js
effect(() => {
  const showDetails = app.showDetails;  // Tracked

  if (showDetails) {
    // Only read when needed, and don't track
    const metadata = untrack(() => app.metadata);
    console.log('Metadata:', metadata);
  }
});
```

 

## untrack() vs batch()

Both `untrack()` and `batch()` control reactivity, but in different ways:

### untrack() - Prevents Dependency Tracking

Use `untrack()` to **read without creating dependencies**:

```js
effect(() => {
  const count = app.count;  // Tracked
  const debug = untrack(() => app.debugMode);  // Not tracked

  if (debug) {
    console.log('Count:', count);
  }
});

app.debugMode = true;  // Effect doesn't re-run
app.count = 5;         // Effect re-runs
```

**Purpose:** Control which reads create dependencies

### batch() - Defers Effect Execution

Use `batch()` to **defer effect execution**:

```js
batch(() => {
  app.count = 1;   // Tracked, but effect deferred
  app.count = 2;   // Tracked, but effect deferred
  app.count = 3;   // Tracked, but effect deferred
});  // Effect runs once here
```

**Purpose:** Group multiple updates to run effects once

### Quick Comparison

```javascript
// untrack() - Don't create dependencies
effect(() => {
  const a = state.a;  // Tracked
  const b = untrack(() => state.b);  // NOT tracked
});

state.b = 10;  // Effect doesn't re-run (not tracked)
state.a = 5;   // Effect re-runs (tracked)

// batch() - Defer effect execution
batch(() => {
  state.a = 1;  // Tracked, effect queued
  state.a = 2;  // Tracked, effect queued
  state.a = 3;  // Tracked, effect queued
});  // Effect runs once with final value
```

**Different purposes:**
- `untrack()` controls **dependency creation**
- `batch()` controls **effect execution timing**

 

## When to Use untrack()

### ✅ Good Use Cases

**1. Debug and Logging Metadata**

```js
effect(() => {
  console.log('Data:', app.data);

  // Log debug info without tracking it
  const timestamp = untrack(() => app.lastUpdate);
  const debugMode = untrack(() => app.debugMode);

  if (debugMode) {
    console.log('Last update:', new Date(timestamp));
  }
});
```

**2. Conditional Feature Flags**

```js
effect(() => {
  renderUI(app.currentPage);

  // Check feature flag without depending on it
  const experimentEnabled = untrack(() => app.experiments.newUI);

  if (experimentEnabled) {
    renderExperimentalFeature();
  }
});
```

**3. Configuration and Settings**

```js
effect(() => {
  const items = app.items;  // Tracked

  // Read config without tracking
  const pageSize = untrack(() => app.config.pageSize);

  displayItems(items.slice(0, pageSize));
});
```

**4. Optimization - Reduce Re-renders**

```js
effect(() => {
  // Only depend on the actual data
  const users = app.users;

  // Don't depend on metadata
  const sortOrder = untrack(() => app.sortOrder);
  const filterMode = untrack(() => app.filterMode);

  renderUsers(users, sortOrder, filterMode);
});
```

### ❌ Not Needed

**1. When You Want Tracking**

```js
// Don't use untrack if you want to depend on the value
❌ effect(() => {
     const count = untrack(() => app.count);
     console.log(count);
   });

// Just read normally
✅ effect(() => {
     console.log(app.count);
   });
```

**2. Outside Effects**

```js
// untrack() only matters inside effects
❌ const value = untrack(() => app.value);  // Pointless outside effect

// Just read directly
✅ const value = app.value;
```

 

## Real-World Examples

### Example 1: Analytics with Debug Mode

```js
const app = state({
  pageViews: 0,
  clickCount: 0,
  debugMode: false,
  sessionId: null
});

effect(() => {
  // Track user interactions
  const views = app.pageViews;
  const clicks = app.clickCount;

  // Send analytics
  analytics.track({
    pageViews: views,
    clicks: clicks
  });

  // Log debug info without tracking it
  const debug = untrack(() => app.debugMode);
  const sessionId = untrack(() => app.sessionId);

  if (debug) {
    console.log(`[Session ${sessionId}] Views: ${views}, Clicks: ${clicks}`);
  }
});

app.debugMode = true;   // Effect doesn't re-run
app.sessionId = 'abc';  // Effect doesn't re-run
app.pageViews++;        // Effect re-runs (sends analytics)
```

### Example 2: Conditional Rendering with Feature Flags

```js
const ui = state({
  user: null,
  posts: [],
  experiments: {
    newLayout: false,
    darkMode: false,
    betaFeatures: false
  }
});

effect(() => {
  const user = ui.user;
  const posts = ui.posts;

  // Check feature flags without tracking them
  const newLayout = untrack(() => ui.experiments.newLayout);
  const darkMode = untrack(() => ui.experiments.darkMode);
  const beta = untrack(() => ui.experiments.betaFeatures);

  if (newLayout) {
    renderNewLayout(user, posts, darkMode, beta);
  } else {
    renderOldLayout(user, posts);
  }
});

// Toggling experiments doesn't re-render
ui.experiments.newLayout = true;  // No re-render
ui.experiments.darkMode = true;   // No re-render

// Data changes trigger re-render
ui.posts.push(newPost);  // Re-renders
```

### Example 3: Paginated List with Config

```js
const dataTable = state({
  items: [],
  currentPage: 1,
  config: {
    itemsPerPage: 10,
    sortBy: 'name',
    sortOrder: 'asc'
  }
});

effect(() => {
  const items = dataTable.items;
  const page = dataTable.currentPage;

  // Read config without tracking
  const itemsPerPage = untrack(() => dataTable.config.itemsPerPage);
  const sortBy = untrack(() => dataTable.config.sortBy);
  const sortOrder = untrack(() => dataTable.config.sortOrder);

  // Calculate display
  const sorted = sortItems(items, sortBy, sortOrder);
  const start = (page - 1) * itemsPerPage;
  const pageItems = sorted.slice(start, start + itemsPerPage);

  renderTable(pageItems);
});

// Changing config doesn't re-render (until data/page changes)
dataTable.config.itemsPerPage = 20;  // No re-render
dataTable.config.sortBy = 'date';    // No re-render

// These trigger re-render
dataTable.currentPage = 2;      // Re-renders
dataTable.items.push(newItem);  // Re-renders
```

### Example 4: Form Validation with Settings

```js
const form = state({
  email: '',
  password: '',
  validationSettings: {
    strictMode: false,
    showWarnings: true,
    realTimeValidation: true
  }
});

effect(() => {
  const email = form.email;
  const password = form.password;

  // Read settings without tracking
  const strict = untrack(() => form.validationSettings.strictMode);
  const showWarnings = untrack(() => form.validationSettings.showWarnings);

  // Validate with current settings
  const errors = validate(
    { email, password },
    { strict, showWarnings }
  );

  displayErrors(errors);
});

// Changing settings doesn't re-validate
form.validationSettings.strictMode = true;  // No re-validation

// Changing form fields triggers validation
form.email = 'user@example.com';  // Validates
form.password = 'secure123';      // Validates
```

 

## Common Patterns

### Pattern: Read Multiple Values Without Tracking

```js
effect(() => {
  const data = app.data;  // Tracked

  // Read multiple config values
  const config = untrack(() => ({
    theme: app.theme,
    locale: app.locale,
    debug: app.debugMode,
    version: app.version
  }));

  render(data, config);
});
```

### Pattern: Conditional Untracked Access

```js
effect(() => {
  const showAdvanced = app.showAdvanced;  // Tracked

  if (showAdvanced) {
    // Only read when needed
    const advanced = untrack(() => app.advancedSettings);
    renderAdvanced(advanced);
  }
});
```

### Pattern: Logging Without Dependencies

```js
effect(() => {
  const result = expensiveComputation(app.input);

  // Log without creating dependencies
  untrack(() => {
    console.log('[Debug] Computed:', result);
    console.log('[Debug] Timestamp:', app.timestamp);
    console.log('[Debug] Session:', app.sessionId);
  });

  return result;
});
```

### Pattern: Metadata for Side Effects

```js
effect(() => {
  const todos = app.todos;  // Tracked

  // Save to API
  await api.save(todos);

  // Update lastSaved without tracking it
  untrack(() => {
    app.metadata.lastSaved = Date.now();
    app.metadata.saveCount++;
  });
});
```

 

## Common Pitfalls

### Pitfall #1: Untracking Values You Need to Track

❌ **Wrong:**
```js
effect(() => {
  // Untracking a value you actually need to react to
  const count = untrack(() => app.count);
  console.log('Count:', count);
});

app.count = 5;  // Effect doesn't re-run!
```

✅ **Correct:**
```js
effect(() => {
  // Track values you need to react to
  console.log('Count:', app.count);
});

app.count = 5;  // Effect re-runs
```

 

### Pitfall #2: Using untrack() Outside Effects

❌ **Wrong:**
```js
// Pointless - not inside an effect
const value = untrack(() => app.value);
```

✅ **Correct:**
```js
// untrack() only matters inside effects
effect(() => {
  const tracked = app.tracked;
  const untracked = untrack(() => app.untracked);
});
```

**Why?** `untrack()` only prevents dependency creation inside effects. Outside effects, there's no tracking happening anyway.

 

### Pitfall #3: Expecting untrack() to Prevent Updates

❌ **Wrong:**
```js
effect(() => {
  // Thinking untrack prevents the change
  untrack(() => {
    app.count = 10;  // This still changes the value!
  });
});
```

✅ **Correct:**
```js
// untrack() only prevents reads from being tracked
effect(() => {
  const value = untrack(() => app.value);  // Read without tracking
  // To prevent updates, use batch() or pause()
});
```

**Why?** `untrack()` prevents reads from creating dependencies. It doesn't prevent writes from triggering effects.

 

### Pitfall #4: Over-Using untrack()

❌ **Wrong:**
```js
// Untracking everything defeats the purpose
effect(() => {
  const a = untrack(() => app.a);
  const b = untrack(() => app.b);
  const c = untrack(() => app.c);
  // Effect never re-runs!
});
```

✅ **Correct:**
```js
// Track what you need, untrack what you don't
effect(() => {
  const data = app.data;  // Tracked (main dependency)

  // Untrack metadata
  const debug = untrack(() => app.debugMode);
  const theme = untrack(() => app.theme);
});
```

 

## Summary

**What is `untrack()`?**

`untrack()` is a **dependency control function** that allows you to read reactive state without creating dependencies, preventing effects from re-running when those values change.

 

**Why use `untrack()`?**

- Read reactive values without creating dependencies
- Effects only re-run when truly needed
- Better performance (fewer re-executions)
- Fine-grained control over reactivity
- Perfect for config, metadata, and debug values
- Prevents unwanted effect triggers

 

**Key Points to Remember:**

1️⃣ **Reads without tracking** - Access state without dependencies
2️⃣ **Only in effects** - Only useful inside effects/computed/watchers
3️⃣ **Returns values** - Can return values from the untrack function
4️⃣ **Doesn't prevent writes** - Only affects read tracking, not writes
5️⃣ **Use strategically** - Track main data, untrack metadata

 

**Mental Model:** Think of `untrack()` as **window shopping** - you can look at the value without subscribing to notifications about it.

 

**Quick Reference:**

```js
// Basic usage
effect(() => {
  const data = app.data;  // Tracked
  const config = untrack(() => app.config);  // Not tracked
});

app.config = newConfig;  // Effect doesn't re-run
app.data = newData;      // Effect re-runs

// Read multiple values
effect(() => {
  const user = app.user;  // Tracked

  const metadata = untrack(() => ({
    theme: app.theme,
    debug: app.debugMode,
    version: app.version
  }));  // None tracked
});

// Conditional untracked reads
effect(() => {
  const showDebug = app.showDebug;  // Tracked

  if (showDebug) {
    const info = untrack(() => app.debugInfo);  // Not tracked
    console.log(info);
  }
});
```

 

**Remember:** `untrack()` gives you precise control over reactivity dependencies. Use it to read configuration, metadata, and debug values without causing unnecessary effect re-executions!
