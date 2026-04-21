[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Understanding `isReactive()` - A Beginner's Guide

## Quick Start (30 seconds)

Need to check if a value is reactive? Here's how:

```js
// Create reactive and non-reactive values
const reactiveObj = state({ count: 0 });
const plainObj = { count: 0 };

// Check if reactive
console.log(isReactive(reactiveObj));  // true
console.log(isReactive(plainObj));     // false

// Use in conditional logic
if (isReactive(someValue)) {
  console.log('This value is reactive!');
} else {
  console.log('This is a plain value');
}
```

**That's it!** The `isReactive()` function checks if a value is a reactive proxy!

 

## What is `isReactive()`?

`isReactive()` is a **type-checking utility function** that determines whether a value is a reactive proxy created by the reactivity system. It returns `true` for reactive objects and `false` for everything else.

**Checking reactivity:**
- Returns `true` for reactive proxies
- Returns `false` for plain objects, primitives, and null/undefined
- Useful for conditional logic
- Helps prevent double-wrapping reactive values

Think of it as **asking for ID** - it checks if the value has the special "reactive" identification badge.

 

## Syntax

```js
// Using the shortcut
isReactive(value)

// Using the full namespace
ReactiveUtils.isReactive(value)
```

**Both styles are valid!** Choose whichever you prefer:
- **Shortcut style** (`isReactive()`) - Clean and concise
- **Namespace style** (`ReactiveUtils.isReactive()`) - Explicit and clear

**Parameters:**
- `value` - Any value to check (required)

**Returns:**
- `true` if the value is a reactive proxy
- `false` if the value is not reactive

 

## Why Does This Exist?

### The Problem with Type Confusion

Let's say you're building a utility function that works with state:

```javascript
function logValue(value) {
  // Is this value already reactive?
  // Or do I need to make it reactive first?
  // How can I tell?

  // ❌ Without isReactive(), you might double-wrap
  const reactiveValue = state(value);  // Problem if value is already reactive!

  console.log(reactiveValue.count);
}

// What if value is already reactive?
const myState = state({ count: 5 });
logValue(myState);  // Wraps a reactive in another reactive! (Bad!)
```

This creates problems. You can't tell if a value is already reactive or not!

**What's the Real Issue?**

```
Without isReactive():
┌──────────────────┐
│ Unknown value    │
│ type            │
└────────┬─────────┘
         │
         ▼
    Is it reactive?
    ¯\_(ツ)_/¯
         │
         ▼
┌──────────────────┐
│ Make it reactive │
│ state(value)     │
└────────┬─────────┘
         │
         ▼
  If already reactive:
  Double-wrapped!
  Broken reactivity!
```

**Problems:**
❌ Can't distinguish reactive from non-reactive values
❌ Risk of double-wrapping reactive objects
❌ No way to write defensive code
❌ Difficult to debug reactivity issues
❌ Can't conditionally apply reactivity
❌ Type confusion in utility functions

**Why This Becomes a Problem:**

When building utilities or frameworks, you need to:
- Check if a value is already reactive before wrapping it
- Write conditional logic based on reactivity
- Debug reactivity issues
- Prevent double-wrapping bugs
- Provide better error messages

### The Solution with `isReactive()`

When you use `isReactive()`, you can check before acting:

```javascript
function makeReactive(value) {
  // Check first!
  if (isReactive(value)) {
    console.log('Already reactive, returning as-is');
    return value;
  }

  console.log('Not reactive, wrapping it');
  return state(value);
}

// Safe to call with any value
const plain = { count: 0 };
const reactive1 = makeReactive(plain);       // Wraps it
const reactive2 = makeReactive(reactive1);   // Returns as-is
```

**What Just Happened?**

```
With isReactive():
┌──────────────────┐
│ Unknown value    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ isReactive()?    │
└────────┬─────────┘
         │
    Yes  │  No
    ─────┼─────
         │
    ✅   │   ❌
         │
         ▼
  Return as-is
         │
         ▼
  Make reactive
```

With `isReactive()`:
- Know exactly what type of value you have
- Prevent double-wrapping bugs
- Write defensive, safe code
- Better debugging and error messages
- Conditional reactivity logic

**Benefits:**
✅ Identify reactive values reliably
✅ Prevent double-wrapping bugs
✅ Write safe utility functions
✅ Better debugging capabilities
✅ Conditional reactivity handling
✅ Clear type checking

 

## Mental Model

Think of `isReactive()` like **checking for a special badge**:

```
Plain Object (No Badge):
┌──────────────┐
│   { ... }    │
│              │
│   No badge   │
└──────────────┘
       │
       ▼
isReactive(obj)
       │
       ▼
    false


Reactive Object (Has Badge):
┌──────────────┐
│   { ... }    │
│              │
│   🎫 Reactive│
│      Badge   │
└──────────────┘
       │
       ▼
isReactive(obj)
       │
       ▼
    true
```

**Key Insight:** Just like checking if someone has a VIP badge to know if they have special access, `isReactive()` checks if a value has the special "reactive" marker to know if it has reactivity powers.

 

## How Does It Work?

### The Magic: Symbol-Based Type Checking

When you call `isReactive()`, here's what happens behind the scenes:

```javascript
// What you write:
const isRx = isReactive(someValue);

// What actually happens (simplified):
// Every reactive proxy has a special hidden symbol
const IS_REACTIVE = Symbol('reactive');

function isReactive(value) {
  // Check if value exists and has the reactive symbol
  return !!(value && value[IS_REACTIVE]);
}

// When creating reactive proxies:
const proxy = new Proxy(target, {
  get(obj, key) {
    if (key === IS_REACTIVE) return true;  // Mark as reactive
    // ... rest of proxy logic
  }
});
```

**In other words:** `isReactive()`:
1. Checks if value exists
2. Checks if value has the special `IS_REACTIVE` symbol
3. Returns `true` if both conditions are met
4. Returns `false` otherwise

### Under the Hood

```
isReactive() implementation:
isReactive(value) {
  return !!(value && value[IS_REACTIVE]);
}

Breaking it down:
- value           → Check if value exists
- value[IS_REACTIVE]  → Check if has reactive symbol
- !!              → Convert to boolean
```

**What happens:**

1️⃣ **Checks** if value is truthy (not null/undefined)
2️⃣ **Accesses** the special IS_REACTIVE symbol
3️⃣ **Converts** to boolean with `!!`
4️⃣ **Returns** true or false

 

## Basic Usage

### Checking Single Values

The simplest way to use `isReactive()`:

```js
// Create values
const reactive = state({ count: 0 });
const plain = { count: 0 };

// Check reactivity
console.log(isReactive(reactive));  // true
console.log(isReactive(plain));     // false
console.log(isReactive(null));      // false
console.log(isReactive(undefined)); // false
console.log(isReactive(42));        // false
console.log(isReactive('hello'));   // false
```

### Conditional Logic

Use in if statements:

```js
function processValue(value) {
  if (isReactive(value)) {
    console.log('Working with reactive value');
    value.count++;  // Will trigger effects
  } else {
    console.log('Working with plain value');
    value.count++;  // Won't trigger effects
  }
}
```

### Preventing Double-Wrapping

Safe wrapper function:

```js
function ensureReactive(value) {
  if (isReactive(value)) {
    return value;  // Already reactive
  }
  return state(value);  // Make it reactive
}

const obj = { count: 0 };
const r1 = ensureReactive(obj);      // Wraps it
const r2 = ensureReactive(r1);       // Returns as-is
console.log(r1 === r2);              // true (same object)
```

 

## When to Use isReactive()

### ✅ Good Use Cases

**1. Utility Functions**

```js
function getValue(obj, key) {
  if (isReactive(obj)) {
    // Use reactive API
    return obj[key];
  } else {
    // Use plain object access
    return obj[key];
  }
}
```

**2. Preventing Double-Wrapping**

```js
function createReactiveIfNeeded(data) {
  if (isReactive(data)) {
    console.warn('Data is already reactive');
    return data;
  }
  return state(data);
}
```

**3. Debugging**

```js
function debugValue(value, label) {
  console.log(`${label}:`, {
    value: value,
    isReactive: isReactive(value),
    type: typeof value
  });
}
```

**4. Conditional Reactivity**

```js
function smartUpdate(target, updates) {
  if (isReactive(target)) {
    // Use batch for reactive objects
    batch(() => {
      Object.assign(target, updates);
    });
  } else {
    // Direct assignment for plain objects
    Object.assign(target, updates);
  }
}
```

### ❌ Not Needed

**1. When You Know the Type**

```js
// Don't check if you already know
❌ const myState = state({ count: 0 });
   if (isReactive(myState)) { ... }  // Pointless, you just created it

// Just use it
✅ const myState = state({ count: 0 });
   myState.count++;
```

**2. In Normal Application Code**

```js
// Don't overuse in regular code
❌ if (isReactive(app.user)) {
     app.user.name = 'John';
   }

// Just update it
✅ app.user.name = 'John';
```

 

## Real-World Examples

### Example 1: Safe State Wrapper

```js
function safeState(data) {
  // Prevent double-wrapping
  if (isReactive(data)) {
    console.warn(
      'Data is already reactive. Returning as-is to prevent double-wrapping.'
    );
    return data;
  }

  // Also handle primitives
  if (typeof data !== 'object' || data === null) {
    console.warn('Cannot make primitives reactive. Use ref() instead.');
    return data;
  }

  return state(data);
}

// Usage
const plain = { count: 0 };
const s1 = safeState(plain);           // Creates reactive
const s2 = safeState(s1);              // Returns same object
console.log(s1 === s2);                // true

const primitive = 42;
const s3 = safeState(primitive);       // Returns 42 with warning
```

### Example 2: Plugin System

```js
class Plugin {
  install(app) {
    if (!isReactive(app)) {
      throw new Error(
        'Plugin requires a reactive app instance. ' +
        'Create with: const app = state({ ... })'
      );
    }

    // Safe to add reactive properties
    app.pluginData = {
      version: '1.0.0',
      enabled: true
    };
  }
}

// Usage
const app = state({ name: 'MyApp' });
const plugin = new Plugin();
plugin.install(app);  // Works

const plainApp = { name: 'MyApp' };
plugin.install(plainApp);  // Throws error
```

### Example 3: Deep Clone with Reactivity Preservation

```js
function deepClone(obj) {
  // Check if reactive
  const wasReactive = isReactive(obj);

  // Get raw object if reactive
  const raw = wasReactive ? toRaw(obj) : obj;

  // Clone the raw object
  const cloned = JSON.parse(JSON.stringify(raw));

  // Re-apply reactivity if original was reactive
  return wasReactive ? state(cloned) : cloned;
}

// Usage
const reactive = state({ user: { name: 'John' } });
const cloned = deepClone(reactive);

console.log(isReactive(reactive));  // true
console.log(isReactive(cloned));    // true (preserved)
```

### Example 4: Framework Integration

```js
class DataStore {
  constructor(data) {
    this.data = isReactive(data) ? data : state(data);
    this.history = [];
  }

  get(key) {
    return this.data[key];
  }

  set(key, value) {
    // Check if value is reactive
    if (isReactive(value)) {
      console.warn(
        `Setting reactive value for key "${key}". ` +
        `This may cause nested reactivity issues.`
      );
    }

    const oldValue = this.data[key];
    this.data[key] = value;

    this.history.push({
      key,
      oldValue,
      newValue: value,
      timestamp: Date.now()
    });
  }

  isReactive() {
    return isReactive(this.data);
  }
}

// Usage
const store1 = new DataStore({ count: 0 });
console.log(store1.isReactive());  // true

const reactiveData = state({ count: 0 });
const store2 = new DataStore(reactiveData);
console.log(store2.isReactive());  // true
```

### Example 5: Type-Safe Utilities

```js
function merge(target, source) {
  // Validate inputs
  if (isReactive(source)) {
    throw new Error(
      'Cannot merge from reactive source. ' +
      'Use toRaw() first to get plain object.'
    );
  }

  const targetIsReactive = isReactive(target);

  if (targetIsReactive) {
    // Batch updates for reactive targets
    batch(() => {
      Object.assign(target, source);
    });
  } else {
    // Direct assignment for plain objects
    Object.assign(target, source);
  }

  return target;
}

// Usage
const reactiveTarget = state({ a: 1 });
const plainSource = { b: 2 };

merge(reactiveTarget, plainSource);
console.log(reactiveTarget);  // { a: 1, b: 2 }

// Error case
const reactiveSource = state({ c: 3 });
merge(reactiveTarget, reactiveSource);  // Throws error
```

 

## Common Patterns

### Pattern: Safe Wrapper Function

```js
function wrapIfNeeded(value) {
  if (value === null || value === undefined) {
    return state({});
  }

  if (isReactive(value)) {
    return value;
  }

  if (typeof value === 'object') {
    return state(value);
  }

  return ref(value);  // For primitives
}
```

### Pattern: Type Validation

```js
function requireReactive(value, name = 'value') {
  if (!isReactive(value)) {
    throw new TypeError(
      `Expected ${name} to be reactive, but got ${typeof value}`
    );
  }
  return value;
}

// Usage
function processState(state) {
  requireReactive(state, 'state');
  // Safe to use as reactive
  state.count++;
}
```

### Pattern: Conditional Processing

```js
function processData(data) {
  const isRx = isReactive(data);

  console.log(`Processing ${isRx ? 'reactive' : 'plain'} data`);

  if (isRx) {
    // Use reactive-specific optimizations
    batch(() => {
      data.processed = true;
      data.timestamp = Date.now();
    });
  } else {
    data.processed = true;
    data.timestamp = Date.now();
  }
}
```

### Pattern: Debug Helper

```js
function inspectValue(value, label = 'Value') {
  const info = {
    label: label,
    isReactive: isReactive(value),
    type: typeof value,
    constructor: value?.constructor?.name,
    value: isReactive(value) ? toRaw(value) : value
  };

  console.table(info);
  return info;
}

// Usage
const reactive = state({ count: 0 });
inspectValue(reactive, 'MyState');
// Logs table with all info
```

 

## Common Pitfalls

### Pitfall #1: Checking Nested Properties

❌ **Wrong:**
```js
const state = { nested: state({ count: 0 }) };

// Only checks outer object
console.log(isReactive(state));         // false
console.log(isReactive(state.nested));  // true
```

✅ **Correct:**
```js
// Make the entire structure reactive
const state = ReactiveUtils.state({
  nested: { count: 0 }  // Deep reactivity
});

console.log(isReactive(state));         // true
console.log(isReactive(state.nested));  // true (deep)
```

**Why?** `isReactive()` only checks the immediate value, not nested properties.

 

### Pitfall #2: Using with Primitives

❌ **Wrong:**
```js
const num = 42;
if (isReactive(num)) {
  // This never runs (primitives can't be reactive)
}
```

✅ **Correct:**
```js
// Use ref for primitives
const num = ref(42);
console.log(isReactive(num));  // true
```

**Why?** Only objects can be reactive proxies. Primitives need `ref()`.

 

### Pitfall #3: Assuming false Means Plain Object

❌ **Wrong:**
```js
if (!isReactive(value)) {
  // Assuming it's a plain object
  Object.assign(value, updates);  // Might fail if value is primitive
}
```

✅ **Correct:**
```js
if (!isReactive(value)) {
  // Check if it's actually an object
  if (typeof value === 'object' && value !== null) {
    Object.assign(value, updates);
  }
}
```

**Why?** `isReactive(value) === false` could mean it's a primitive, null, or plain object.

 

### Pitfall #4: Over-Using in Application Code

❌ **Wrong:**
```js
// Checking everywhere is unnecessary
function incrementCount(state) {
  if (isReactive(state)) {
    state.count++;
  }
}
```

✅ **Correct:**
```js
// Just use it if you know it's reactive
function incrementCount(state) {
  state.count++;
}
```

**Why?** If you know the type, checking is wasteful. Use `isReactive()` for utilities and edge cases, not normal app code.

 

## Summary

**What is `isReactive()`?**

`isReactive()` is a **type-checking utility** that determines whether a value is a reactive proxy, returning `true` for reactive values and `false` for everything else.

 

**Why use `isReactive()`?**

- Check if values are reactive before processing
- Prevent double-wrapping bugs
- Write safe utility functions
- Better debugging and error messages
- Conditional reactivity logic
- Type validation

 

**Key Points to Remember:**

1️⃣ **Type check** - Returns boolean indicating reactivity
2️⃣ **Prevents bugs** - Stops double-wrapping issues
3️⃣ **Utility use** - Best for framework/utility code, not normal app code
4️⃣ **Only immediate** - Doesn't check nested properties
5️⃣ **Objects only** - Primitives need `ref()` to be reactive

 

**Mental Model:** Think of `isReactive()` as **checking for a VIP badge** - it identifies values that have special reactive powers.

 

**Quick Reference:**

```js
// Basic usage
const reactive = state({ count: 0 });
const plain = { count: 0 };

console.log(isReactive(reactive));  // true
console.log(isReactive(plain));     // false

// Safe wrapper
function ensureReactive(value) {
  if (isReactive(value)) {
    return value;
  }
  return state(value);
}

// Validation
function requireReactive(value) {
  if (!isReactive(value)) {
    throw new Error('Value must be reactive');
  }
  return value;
}

// Conditional logic
if (isReactive(data)) {
  batch(() => {
    Object.assign(data, updates);
  });
} else {
  Object.assign(data, updates);
}
```

 

**Remember:** `isReactive()` is your type-checking tool for reactive values. Use it in utilities and defensive code to prevent bugs and write safer applications!
