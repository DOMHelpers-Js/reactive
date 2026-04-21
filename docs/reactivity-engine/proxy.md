[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# Proxy (The Interceptor)

## The Magic Trick Revealed

Remember when we said this was "magic"?

```javascript
const state = state({ count: 0 });

effect(() => {
  console.log(state.count);
});

state.count = 5; // Effect automatically runs! ✨
```

**How does the computer know to run the effect when `state.count` changes?**

The secret is... **Proxy**! 🎩✨

 

## What is a Proxy? (No Jargon)

### Simple Definition

A **Proxy** is like a **middleman** that sits between you and an object.

Instead of talking to the object directly, you talk to the Proxy, and the Proxy talks to the object for you.

**But here's the magic:** The Proxy can **do extra stuff** before passing your message along.

### Super Simple Analogy

Think of a Proxy like a **secretary at a company**:

```
You → Secretary → Boss
```

- **You** want to talk to the **Boss** (the object)
- But you go through the **Secretary** (the Proxy) first
- The Secretary can:
  - ✅ Log your visit: "Someone asked for the boss at 2pm"
  - ✅ Check if you're allowed to talk to the boss
  - ✅ Notify others: "Hey everyone, someone's talking to the boss!"
  - ✅ Then pass your message to the boss

**The Proxy is that secretary!**

 

## Real-World Analogy

### The Hotel Receptionist

Imagine you're staying at a hotel:

```
┌─────────────────────────────────────┐
│  You want to access your room       │
└───────────────┬─────────────────────┘
                │
                ↓
┌─────────────────────────────────────┐
│  Receptionist (Proxy)               │
│  - Checks your keycard              │
│  - Logs your entry time             │
│  - Notifies cleaning staff          │
│  - THEN unlocks your door           │
└───────────────┬─────────────────────┘
                │
                ↓
┌─────────────────────────────────────┐
│  Your Room (The Object)             │
└─────────────────────────────────────┘
```

**The receptionist doesn't change your room**, but they **know every time you access it** and can **do things before you get in**.

That's exactly what a Proxy does with objects!

 

## How Regular Objects Work

### Without Proxy (Direct Access)

```javascript
// Regular object
const user = {
  name: 'Alice',
  age: 30
};

// You access it directly
console.log(user.name); // Just gets the value
user.age = 31;          // Just sets the value

// The computer has NO IDEA you did this!
// No logging, no notifications, nothing! 🤷
```

**Flow:**

```
You
 ↓
user.name
 ↓
Get value directly
```

**Problem:** You can't detect when someone reads or changes the object.

 

## How Proxy Objects Work

### With Proxy (Intercepted Access)

```javascript
// Create a Proxy
const userProxy = new Proxy(user, {
  get(target, property) {
    console.log(`Someone is reading ${property}`);
    return target[property];
  },
  
  set(target, property, value) {
    console.log(`Someone is changing ${property} to ${value}`);
    target[property] = value;
    return true;
  }
});

// Now use the proxy
console.log(userProxy.name);
// Logs: "Someone is reading name"
// Returns: "Alice"

userProxy.age = 31;
// Logs: "Someone is changing age to 31"
// Sets age to 31
```

**Flow:**

```
You
 ↓
userProxy.name
 ↓
Proxy intercepts (runs your code)
 ↓
Get value from original object
 ↓
Return value to you
```

**Magic:** You can now **detect and react** to every access! 🎉

 

## Step-by-Step: Building Your First Proxy

### Step 1: Start with a Regular Object

```javascript
const person = {
  name: 'Bob',
  age: 25
};

console.log(person.name); // Bob
person.age = 26;
console.log(person.age); // 26
```

Nothing special here. Just a normal object.

 

### Step 2: Create a Proxy Wrapper

```javascript
const personProxy = new Proxy(person, {
  // This object contains "traps"
  // Traps are functions that intercept operations
});
```

**Think of it like this:**

```
personProxy = Wrapper around 'person'
```

 

### Step 3: Add a "get" Trap (Intercept Reads)

```javascript
const personProxy = new Proxy(person, {
  get(target, property) {
    // target = the original object (person)
    // property = the property name being accessed
    
    console.log(`📖 Reading: ${property}`);
    
    // Return the actual value
    return target[property];
  }
});

// Try it!
console.log(personProxy.name);
// Logs: "📖 Reading: name"
// Returns: "Bob"
```

**What's happening:**

1. You ask for `personProxy.name`
2. JavaScript calls your `get` function
3. Your function logs a message
4. Your function returns `target[property]` (which is `person.name`)
5. You get `"Bob"` back

 

### Step 4: Add a "set" Trap (Intercept Writes)

```javascript
const personProxy = new Proxy(person, {
  get(target, property) {
    console.log(`📖 Reading: ${property}`);
    return target[property];
  },
  
  set(target, property, value) {
    // target = the original object (person)
    // property = the property name being set
    // value = the new value
    
    console.log(`✏️ Writing: ${property} = ${value}`);
    
    // Actually set the value
    target[property] = value;
    
    // Return true to indicate success
    return true;
  }
});

// Try it!
personProxy.age = 26;
// Logs: "✏️ Writing: age = 26"

console.log(personProxy.age);
// Logs: "📖 Reading: age"
// Returns: 26
```

 

### Step 5: Do Something Cool (Track Changes)

```javascript
const changes = [];

const personProxy = new Proxy(person, {
  get(target, property) {
    return target[property];
  },
  
  set(target, property, value) {
    // 🎯 Track every change!
    changes.push({
      property: property,
      oldValue: target[property],
      newValue: value,
      timestamp: new Date()
    });
    
    target[property] = value;
    return true;
  }
});

// Make some changes
personProxy.age = 26;
personProxy.name = 'Bobby';
personProxy.age = 27;

// Check the log
console.log(changes);
/*
[
  { property: 'age', oldValue: 25, newValue: 26, timestamp: ... },
  { property: 'name', oldValue: 'Bob', newValue: 'Bobby', timestamp: ... },
  { property: 'age', oldValue: 26, newValue: 27, timestamp: ... }
]
*/
```

**You just built a change tracker!** 🎉

 

## The Two Main "Traps"

Think of "traps" as **interception points**. Like speed traps on a highway that catch you doing something.

### 1. The `get` Trap (Reading)

**When it runs:** Every time you **read** a property.

```javascript
get(target, property) {
  // target = the original object
  // property = which property is being read
  
  // Do your custom stuff here
  console.log(`Reading ${property}`);
  
  // Return the value
  return target[property];
}
```

**Examples of when it runs:**

```javascript
const value = obj.name;        // Triggers get trap
console.log(obj.age);          // Triggers get trap
const result = obj.city;       // Triggers get trap
if (obj.active) { }            // Triggers get trap
```

 

### 2. The `set` Trap (Writing)

**When it runs:** Every time you **write** a property.

```javascript
set(target, property, value) {
  // target = the original object
  // property = which property is being set
  // value = the new value
  
  // Do your custom stuff here
  console.log(`Setting ${property} to ${value}`);
  
  // Actually set the value
  target[property] = value;
  
  // Must return true
  return true;
}
```

**Examples of when it runs:**

```javascript
obj.name = 'Alice';            // Triggers set trap
obj.age = 30;                  // Triggers set trap
obj.city = 'Paris';            // Triggers set trap
obj.active = true;             // Triggers set trap
```

 

## Why This Is Magic for Reactivity

### The Problem We're Solving

Remember this?

```javascript
// We want this to work automatically
const state = state({ count: 0 });

effect(() => {
  document.getElementById('count').textContent = state.count;
});

state.count = 5; // How does the effect know to re-run? 🤔
```

### The Solution: Proxy!

Here's how DOM Helpers Reactive uses Proxy:

```javascript
function state(data) {
  // Create a Proxy wrapper
  const proxy = new Proxy(data, {
    get(target, property) {
      // 🎯 TRACK: "Someone is reading this property"
      if (currentEffect) {
        // Remember: "currentEffect depends on this property"
        trackDependency(property, currentEffect);
      }
      
      return target[property];
    },
    
    set(target, property, value) {
      // Set the new value
      target[property] = value;
      
      // 🎯 TRIGGER: "This property changed, run dependent effects"
      triggerEffects(property);
      
      return true;
    }
  });
  
  return proxy;
}
```

### Step-by-Step Flow

**1. Create reactive state:**

```javascript
const state = state({ count: 0 });
// Returns a Proxy wrapper around { count: 0 }
```

**2. Create effect:**

```javascript
let currentEffect = null;

effect(() => {
  console.log(state.count);
});

// Under the hood:
currentEffect = effectFunction;
effectFunction(); // Runs the effect
```

**3. Effect reads `state.count`:**

```javascript
// When state.count is accessed:
// → Proxy's get trap fires
// → Sees currentEffect is set
// → Records: "effectFunction depends on 'count'"
```

**4. Change `state.count`:**

```javascript
state.count = 5;

// Proxy's set trap fires:
// 1. Sets count to 5
// 2. Looks up effects that depend on 'count'
// 3. Finds effectFunction
// 4. Runs effectFunction
// 5. Console logs: 5
```

### Visual Flow

```
┌─────────────────────────────────────┐
│ state.count = 5                     │
└────────────┬────────────────────────┘
             │
             ↓
┌─────────────────────────────────────┐
│ Proxy's SET trap intercepts         │
│ "Someone is setting count to 5"     │
└────────────┬────────────────────────┘
             │
             ↓
┌─────────────────────────────────────┐
│ Look up: Who depends on 'count'?    │
│ → effectFunction                    │
└────────────┬────────────────────────┘
             │
             ↓
┌─────────────────────────────────────┐
│ Run effectFunction                  │
│ → Updates DOM automatically ✨      │
└─────────────────────────────────────┘
```

 

## Common Questions

### Q: "Is the Proxy a copy of the object?"

**No!** The Proxy is a **wrapper** around the original object.

```javascript
const original = { count: 0 };
const proxy = new Proxy(original, {});

proxy.count = 5;
console.log(original.count); // 5 (same object!)
```

Think of it like this:

```
Original Object: The actual book
Proxy: A protective cover around the book
```

When you write in the book through the cover, you're still writing in the same book.

 

### Q: "Do I always use the Proxy, never the original?"

**Yes!** Always use the Proxy if you want the interception to work.

```javascript
const original = { count: 0 };
const proxy = new Proxy(original, {
  set(target, prop, value) {
    console.log('Changed!');
    target[prop] = value;
    return true;
  }
});

// ✅ Use the proxy
proxy.count = 5; // Logs: "Changed!"

// ❌ Use the original directly
original.count = 10; // No log! Proxy doesn't know!
```

 

### Q: "Can I have multiple Proxies for one object?"

**Yes!** Each Proxy can have different behavior.

```javascript
const user = { name: 'Alice' };

const logger = new Proxy(user, {
  set(target, prop, value) {
    console.log(`Logging: ${prop} = ${value}`);
    target[prop] = value;
    return true;
  }
});

const validator = new Proxy(user, {
  set(target, prop, value) {
    if (typeof value !== 'string') {
      throw new Error('Name must be a string!');
    }
    target[prop] = value;
    return true;
  }
});

logger.name = 'Bob';      // Logs change
validator.name = 'Charlie'; // Validates type
```

 

### Q: "What if I want the original object back?"

**Use a Symbol!**

```javascript
const RAW = Symbol('raw');

const proxy = new Proxy(original, {
  get(target, prop) {
    if (prop === RAW) {
      return target; // Return original!
    }
    return target[prop];
  }
});

// Get the original
const orig = proxy[RAW];
```

This is exactly what DOM Helpers does!

 

## Practice Examples

### Example 1: Build a Read Counter

**Goal:** Count how many times properties are read.

```javascript
const readCounts = {};

const data = { name: 'Alice', age: 30 };

const counter = new Proxy(data, {
  get(target, property) {
    // Count the read
    if (!readCounts[property]) {
      readCounts[property] = 0;
    }
    readCounts[property]++;
    
    return target[property];
  }
});

// Use it
console.log(counter.name); // Alice
console.log(counter.name); // Alice
console.log(counter.age);  // 30

console.log(readCounts);
// { name: 2, age: 1 }
```

 

### Example 2: Build a Validator

**Goal:** Prevent invalid values.

```javascript
const user = { age: 25 };

const validated = new Proxy(user, {
  set(target, property, value) {
    if (property === 'age') {
      if (typeof value !== 'number') {
        throw new Error('Age must be a number!');
      }
      if (value < 0 || value > 150) {
        throw new Error('Invalid age!');
      }
    }
    
    target[property] = value;
    return true;
  }
});

validated.age = 30;      // ✅ Works
validated.age = -5;      // ❌ Error: Invalid age!
validated.age = 'thirty'; // ❌ Error: Age must be a number!
```

 

### Example 3: Build a Simple Reactive System

**Goal:** Automatically run functions when data changes.

```javascript
let currentWatcher = null;
const watchers = {};

function watch(fn) {
  currentWatcher = fn;
  fn(); // Run once to track dependencies
  currentWatcher = null;
}

function reactive(obj) {
  return new Proxy(obj, {
    get(target, property) {
      // Track dependency
      if (currentWatcher) {
        if (!watchers[property]) {
          watchers[property] = [];
        }
        watchers[property].push(currentWatcher);
      }
      
      return target[property];
    },
    
    set(target, property, value) {
      target[property] = value;
      
      // Trigger watchers
      if (watchers[property]) {
        watchers[property].forEach(fn => fn());
      }
      
      return true;
    }
  });
}

// Use it!
const state = reactive({ count: 0 });

watch(() => {
  console.log('Count is:', state.count);
});
// Logs: "Count is: 0"

state.count = 5;
// Logs: "Count is: 5"

state.count = 10;
// Logs: "Count is: 10"
```

**You just built a mini reactivity system!** 🎉

 

## Summary

### What is a Proxy?

A **middleman** that intercepts operations on objects.

```
You → Proxy (intercepts) → Object
```

### The Two Main Traps

1. **`get` trap** - Runs when reading properties
2. **`set` trap** - Runs when writing properties

### Basic Syntax

```javascript
const proxy = new Proxy(originalObject, {
  get(target, property) {
    // Intercept reads
    return target[property];
  },
  
  set(target, property, value) {
    // Intercept writes
    target[property] = value;
    return true;
  }
});
```

### Why This Matters for Reactivity

**Proxy lets us:**
- ✅ Detect when data is read (track dependencies)
- ✅ Detect when data changes (trigger updates)
- ✅ Build automatic, reactive systems

**Without Proxy:**
```javascript
let count = 0;
// No way to know when count is read or changed 😢
```

**With Proxy:**
```javascript
const state = new Proxy({ count: 0 }, {
  get(t, k) { console.log('Read!'); return t[k]; },
  set(t, k, v) { console.log('Changed!'); t[k] = v; return true; }
});
// We know everything! 🎉
```

 

**The Proxy is the foundation that makes all the reactivity magic possible!** ✨