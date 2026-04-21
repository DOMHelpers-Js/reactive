[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `DevTools` - Development Debugging System

## Quick Start (30 seconds)

```javascript
// 1. Enable DevTools
DevTools.enable();
// Console: "[DevTools] Enabled - inspect with window.__REACTIVE_DEVTOOLS__"

// 2. Create and track state
const todos = state({
  items: [],
  filter: 'all'
});

DevTools.trackState(todos, 'TodoList');

// 3. Make changes
todos.items.push({ id: 1, text: 'Buy milk', done: false });
todos.filter = 'active';

// 4. Inspect what happened
console.log('Status:', DevTools.enabled);        // true
console.log('States:', DevTools.states.size);    // 1
console.log('History:', DevTools.history.length); // 2

const changes = DevTools.getHistory();
console.log(changes);
// [
//   { stateName: 'TodoList', key: 'items', oldValue: [], newValue: [...] },
//   { stateName: 'TodoList', key: 'filter', oldValue: 'all', newValue: 'active' }
// ]

// 5. Access from browser console
window.__REACTIVE_DEVTOOLS__.getHistory();
// Full access to debugging tools! ✨
```

**What just happened?** You activated a complete debugging system that tracks every state change with full history!

 

## What is DevTools?

`DevTools` is a **comprehensive development-only debugging toolkit** designed for reactive state management systems.

Simply put: it's like Chrome DevTools but specifically for your reactive state - showing you exactly what's changing, when, and why.

Think of it as **a flight data recorder, security camera, and diagnostic tool** all combined into one system for debugging your application's state.

### Core Capabilities

```
DevTools
├── Lifecycle Management
│   ├── enable()  - Turn on debugging
│   └── disable() - Turn off debugging
│
├── Tracking System
│   ├── trackState()  - Monitor state objects
│   └── trackEffect() - Monitor effects
│
├── Inspection System
│   ├── getStates()    - View tracked states
│   ├── getHistory()   - View change log
│   └── clearHistory() - Clear change log
│
└── Data Storage
    ├── enabled      - Status flag
    ├── states       - State registry
    ├── effects      - Effect registry
    ├── history      - Change log
    └── maxHistory   - Size limit
```

 

## Complete API Overview

### Methods (7)

| Method | Purpose | When to Use |
|--------|---------|-------------|
| `enable()` | Activate DevTools | App start (development) |
| `disable()` | Deactivate DevTools | Done debugging |
| `trackState(state, name)` | Register state for tracking | For each important state |
| `trackEffect(effect, name)` | Register effect for tracking | For complex effects |
| `getStates()` | Get all tracked states | Inspect state list |
| `getHistory()` | Get change history | Review what changed |
| `clearHistory()` | Clear change log | Free memory / fresh start |

### Properties (5)

| Property | Type | Purpose |
|----------|------|---------|
| `enabled` | Boolean | Check if DevTools is active |
| `states` | Map | Registry of tracked states |
| `effects` | Map | Registry of tracked effects |
| `history` | Array | Log of all changes |
| `maxHistory` | Number | Maximum history size (default: 50) |

### Relationships

```
enable() → trackState() → changes → getHistory()
   ↓           ↓                         ↓
enabled=true  states Map            history Array
                                         ↓
                                   clearHistory()
```

 

## Why Does This Exist?

### The Challenge: Invisible State Changes

Reactive systems are powerful but create debugging challenges:

```javascript
// Without DevTools - Mystery debugging
const state = ReactiveUtils.state({ count: 0, user: null });

effect(() => {
  updateUI(state.count);
});

// Somewhere in your code...
state.count = 5;

// Questions you can't answer:
// ❌ What changed count?
// ❌ When did it change?
// ❌ What was the previous value?
// ❌ How many times has it changed?
// ❌ What effects were triggered?
// ❌ What's the complete timeline?
```

**The Problem:**

```
State Changes
     ↓
No visibility
     ↓
No history
     ↓
No tracking
     ↓
Debugging is guesswork ❌
```

### The Solution: DevTools

```javascript
// With DevTools - Complete visibility
DevTools.enable();

const state = ReactiveUtils.state({ count: 0, user: null });
DevTools.trackState(state, 'AppState');

effect(() => {
  updateUI(state.count);
});

state.count = 5;

// Now you can answer everything:
const history = DevTools.getHistory();
// ✅ See what changed: 'count'
// ✅ See old value: 0
// ✅ See new value: 5
// ✅ See timestamp: 1704123456789
// ✅ See which state: 'AppState'

const states = DevTools.getStates();
// ✅ See all tracked states
// ✅ See their metadata
// ✅ See update counts

// From browser console:
window.__REACTIVE_DEVTOOLS__.getHistory();
// ✅ Inspect anytime during debugging
```

**The Solution:**

```
State Changes
     ↓
DevTools tracks
     ↓
History recorded
     ↓
Metadata captured
     ↓
Full visibility ✅
```

 

## Mental Model

Think of DevTools as **a comprehensive building security system**:

### Application Without DevTools
```
Your Building
┌─────────────────────────────┐
│  People entering/leaving    │
│  Doors opening/closing      │
│  Activities happening       │
│                             │
│  No cameras                 │
│  No access logs             │
│  No visitor registry        │
│                             │
│  = Can't review events ❌   │
└─────────────────────────────┘
```

### Application With DevTools
```
Your Building
┌─────────────────────────────┐
│  People entering/leaving 🎥 │ ← Tracked
│  Doors opening/closing  🎥  │ ← Tracked
│  Activities happening   🎥  │ ← Tracked
│                             │
│  Cameras recording          │
│  Access logs maintained     │
│  Visitor registry updated   │
│                             │
│  = Complete audit trail ✅  │
└─────────────────────────────┘
```

**Key Insight:** DevTools observes without interfering - your app runs normally while everything is recorded for debugging.

 

## Properties Reference

### 1. `DevTools.enabled`

**Type:** `Boolean`

**Purpose:** Indicates whether DevTools is currently active

**Read-Only:** Yes (use `enable()`/`disable()` to change)

**Example:**
```javascript
console.log(DevTools.enabled);  // false

DevTools.enable();
console.log(DevTools.enabled);  // true

DevTools.disable();
console.log(DevTools.enabled);  // false
```

**Common Use:**
```javascript
// Conditional tracking
if (DevTools.enabled) {
  DevTools.trackState(state, 'MyState');
}

// Feature detection
function debugLog(msg) {
  if (DevTools.enabled) {
    console.log('[DEBUG]', msg);
  }
}
```

 

### 2. `DevTools.states`

**Type:** `Map<State, Metadata>`

**Purpose:** Registry of all tracked state objects with their metadata

**Structure:**
```javascript
Map {
  <StateObject1> => {
    id: 1,
    name: 'TodoList',
    created: 1704123456789,
    updates: [...]
  },
  <StateObject2> => {
    id: 2,
    name: 'UserProfile',
    created: 1704123456790,
    updates: [...]
  }
}
```

**Example:**
```javascript
DevTools.enable();
DevTools.trackState(todos, 'TodoList');
DevTools.trackState(user, 'User');

console.log(DevTools.states.size);  // 2

// Iterate through tracked states
DevTools.states.forEach((metadata, state) => {
  console.log(`${metadata.name}: ${metadata.updates.length} updates`);
});

// Check if state is tracked
const isTracked = DevTools.states.has(myState);
```

**Common Use:**
```javascript
// Get all state names
const stateNames = Array.from(DevTools.states.values())
  .map(meta => meta.name);

// Find most active state
let maxUpdates = 0;
let mostActive = null;

DevTools.states.forEach((meta) => {
  if (meta.updates.length > maxUpdates) {
    maxUpdates = meta.updates.length;
    mostActive = meta.name;
  }
});

console.log(`Most active: ${mostActive} (${maxUpdates} updates)`);
```

 

### 3. `DevTools.effects`

**Type:** `Map<Effect, Metadata>`

**Purpose:** Registry of all tracked effect functions with their metadata

**Structure:**
```javascript
Map {
  <EffectFunction1> => {
    id: 1,
    name: 'CountLogger',
    created: 1704123456789,
    runs: 5
  },
  <EffectFunction2> => {
    id: 2,
    name: 'UIUpdater',
    created: 1704123456790,
    runs: 3
  }
}
```

**Example:**
```javascript
DevTools.enable();

const cleanup = effect(() => {
  console.log('Count:', state.count);
});

DevTools.trackEffect(cleanup, 'CountLogger');

state.count++;
state.count++;

console.log(DevTools.effects.size);  // 1

// Get effect info
DevTools.effects.forEach((metadata, effect) => {
  console.log(`${metadata.name} ran ${metadata.runs} times`);
});
// Output: "CountLogger ran 3 times" (initial + 2 changes)
```

**Common Use:**
```javascript
// Find most-run effect
let maxRuns = 0;
let mostActive = null;

DevTools.effects.forEach((meta) => {
  if (meta.runs > maxRuns) {
    maxRuns = meta.runs;
    mostActive = meta.name;
  }
});

console.log(`Most active effect: ${mostActive} (${maxRuns} runs)`);
```

 

### 4. `DevTools.history`

**Type:** `Array<ChangeRecord>`

**Purpose:** Complete log of all state changes (last 50 by default)

**Structure:**
```javascript
[
  {
    stateId: 1,
    stateName: 'TodoList',
    key: 'items',
    oldValue: [],
    newValue: [{ id: 1, text: 'Buy milk' }],
    timestamp: 1704123456789
  },
  {
    stateId: 1,
    stateName: 'TodoList',
    key: 'filter',
    oldValue: 'all',
    newValue: 'active',
    timestamp: 1704123456790
  }
]
```

**Example:**
```javascript
DevTools.enable();
DevTools.trackState(state, 'MyState');

state.count = 1;
state.count = 2;
state.count = 3;

console.log(DevTools.history.length);  // 3

// Access changes directly
DevTools.history.forEach(change => {
  console.log(
    `${change.stateName}.${change.key}: ` +
    `${change.oldValue} → ${change.newValue}`
  );
});
// Output:
// "MyState.count: 0 → 1"
// "MyState.count: 1 → 2"
// "MyState.count: 2 → 3"
```

**Common Use:**
```javascript
// Get last N changes
const last10 = DevTools.history.slice(-10);

// Filter by state
const todoChanges = DevTools.history
  .filter(c => c.stateName === 'TodoList');

// Filter by property
const countChanges = DevTools.history
  .filter(c => c.key === 'count');

// Get changes in last 5 minutes
const fiveMinAgo = Date.now() - (5 * 60 * 1000);
const recent = DevTools.history
  .filter(c => c.timestamp > fiveMinAgo);
```

 

### 5. `DevTools.maxHistory`

**Type:** `Number`

**Purpose:** Maximum number of history entries to keep

**Default:** `50`

**Writable:** Yes (can be changed)

**Example:**
```javascript
console.log(DevTools.maxHistory);  // 50

// Increase limit
DevTools.maxHistory = 100;

// Decrease limit
DevTools.maxHistory = 25;

// Unlimited (not recommended!)
DevTools.maxHistory = Infinity;
```

**How It Works:**
```javascript
// When history exceeds maxHistory:
if (DevTools.history.length > DevTools.maxHistory) {
  DevTools.history.shift();  // Remove oldest
}
```

**Common Use:**
```javascript
// Large app - more history
DevTools.maxHistory = 200;

// Small app - less history
DevTools.maxHistory = 20;

// Testing - unlimited
DevTools.maxHistory = Infinity;

// Memory-constrained
DevTools.maxHistory = 10;
```

 

## Basic Workflows

### Workflow 1: Simple Debugging Session

```javascript
// 1. Enable
DevTools.enable();

// 2. Track state
const state = ReactiveUtils.state({ count: 0 });
DevTools.trackState(state, 'Counter');

// 3. Make changes
state.count = 1;
state.count = 2;
state.count = 3;

// 4. Review history
const history = DevTools.getHistory();
console.log('Changes:', history);

// 5. Disable when done
DevTools.disable();
```

 

### Workflow 2: Multi-State Tracking

```javascript
// Enable once
DevTools.enable();

// Track multiple states
DevTools.trackState(userState, 'User');
DevTools.trackState(cartState, 'Cart');
DevTools.trackState(settingsState, 'Settings');

// App runs normally...
userState.name = 'Alice';
cartState.items.push(product);
settingsState.theme = 'dark';

// Review all changes
const allChanges = DevTools.getHistory();
console.log(`Total changes: ${allChanges.length}`);

// Review specific state
const userChanges = allChanges
  .filter(c => c.stateName === 'User');
```

 

### Workflow 3: Effect Monitoring

```javascript
DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
DevTools.trackState(state, 'Counter');

// Track effects
const cleanup1 = effect(() => {
  console.log('Count:', state.count);
});
DevTools.trackEffect(cleanup1, 'Logger');

const cleanup2 = effect(() => {
  document.title = `Count: ${state.count}`;
});
DevTools.trackEffect(cleanup2, 'TitleUpdater');

// Trigger changes
state.count++;

// Check effect runs
DevTools.effects.forEach((meta) => {
  console.log(`${meta.name}: ${meta.runs} runs`);
});
```

 

### Workflow 4: Console Inspection

```javascript
// In your app code
DevTools.enable();
DevTools.trackState(state, 'AppState');

// ... app runs ...

// Later, in browser DevTools console:
const dt = window.__REACTIVE_DEVTOOLS__;

dt.getStates();     // See all states
dt.getHistory();    // See all changes
dt.clearHistory();  // Clear log

// Create shortcuts
window.dumpStates = () => console.table(dt.getStates());
window.dumpHistory = () => console.table(dt.getHistory());
```

 

## When to Use DevTools

### ✅ Use DevTools For:

1. **Development Debugging**
   ```javascript
   if (import.meta.env.DEV) {
     DevTools.enable();
   }
   ```

2. **Understanding Data Flow**
   ```javascript
   // Track related states
   DevTools.trackState(sourceState, 'Source');
   DevTools.trackState(derivedState, 'Derived');
   // See how changes flow
   ```

3. **Performance Monitoring**
   ```javascript
   DevTools.trackEffect(expensiveEffect, 'ExpensiveOperation');
   // Count how many times it runs
   ```

4. **Bug Investigation**
   ```javascript
   // Enable when bug occurs
   DevTools.enable();
   // Reproduce issue
   // Review history to find cause
   ```

5. **Learning Reactive Behavior**
   ```javascript
   // See exactly when/how reactive updates work
   DevTools.enable();
   DevTools.trackState(state, 'Learning');
   ```

### ❌ Don't Use For:

1. **Production** - Performance overhead
2. **User-Facing Features** - Development only
3. **Critical Path Code** - Adds minimal overhead

 

## Important Concepts

### 1. Auto-Enable on Localhost

```javascript
// From 06_dh-reactive-enhancements.js
// DevTools automatically enables on localhost!
if (window.location.hostname === 'localhost' || 
    window.location.hostname === '127.0.0.1') {
  // Already enabled
  console.log(DevTools.enabled);  // May be true
}
```

 

### 2. Non-Intrusive Design

DevTools **observes without interfering**:
- ✅ No performance impact on state operations
- ✅ No change to app behavior
- ✅ Can enable/disable anytime
- ✅ ~1-2% overhead when tracking

 

### 3. Memory Management

History is automatically limited:
```javascript
// Oldest entries automatically removed
// when exceeding maxHistory (default: 50)

// For long-running apps:
setInterval(() => {
  if (DevTools.history.length > 100) {
    DevTools.clearHistory();
  }
}, 60000);
```

 

### 4. WeakMaps for Efficiency

```javascript
// States and effects use WeakMaps
// Automatically garbage collected
// No memory leaks
```

 

### 5. Global Console Access

```javascript
// When enabled, available as:
window.__REACTIVE_DEVTOOLS__

// Create shortcuts:
window.dt = window.__REACTIVE_DEVTOOLS__;

// Quick access:
dt.getHistory()
dt.getStates()
```

 

## Summary

**What is DevTools?**  
A comprehensive development-only debugging system for reactive state that provides complete visibility into state changes, effect runs, and data flow.

**Core Components:**
- ✅ **Lifecycle**: `enable()` / `disable()`
- ✅ **Tracking**: `trackState()` / `trackEffect()`
- ✅ **Inspection**: `getStates()` / `getHistory()` / `clearHistory()`
- ✅ **Data**: `enabled`, `states`, `effects`, `history`, `maxHistory`

**Key Benefits:**
- ✅ Complete visibility into state changes
- ✅ Full change history with timestamps
- ✅ Effect execution monitoring
- ✅ Console access for live debugging
- ✅ Non-intrusive observation

**Key Takeaway:**

```
Without DevTools          With DevTools
       |                       |
Blind debugging          Full visibility
       |                       |
No history              Complete timeline
       |                       |
Guesswork               Data-driven debugging ✅
```

**One-Line Rule:** Use DevTools in development to see, understand, and debug your reactive state system.

**Remember:** DevTools is your debugging command center - enable it, track what matters, inspect anytime! 🎉