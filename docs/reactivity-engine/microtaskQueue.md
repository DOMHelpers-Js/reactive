[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# MicrotaskQueue (The Smart Batcher)
## The Update Chaos Problem

Imagine you change a reactive state multiple times in quick succession:

```javascript
const state = state({ count: 0 });

effect(() => {
  console.log('Effect ran:', state.count);
  document.getElementById('display').textContent = state.count;
});

// Change count 5 times rapidly
state.count = 1; // Effect runs
state.count = 2; // Effect runs
state.count = 3; // Effect runs
state.count = 4; // Effect runs
state.count = 5; // Effect runs

// Console output:
// Effect ran: 1
// Effect ran: 2
// Effect ran: 3
// Effect ran: 4
// Effect ran: 5

// The effect ran 5 times! 😱
// The DOM was updated 5 times! 😱
```

**The Problem:**
- ❌ Effect runs **5 times** (wasteful!)
- ❌ DOM updates **5 times** (slow!)
- ❌ User might see flickering (bad UX!)
- ❌ Only the **final value (5)** matters!

**What we want:**
- ✅ Effect runs **once** with the final value
- ✅ DOM updates **once** efficiently
- ✅ No wasted work

**The Solution:** MicrotaskQueue! 🎯

 

## What is the MicrotaskQueue? (No Jargon)

### Simple Definition

The **MicrotaskQueue** is like a **smart to-do list** that JavaScript checks **right before it's about to take a break**.

Instead of doing work immediately, you can say "add this to my to-do list" and JavaScript will:
1. Finish what it's currently doing
2. Check the to-do list
3. Do all the tasks on the list **in one batch**
4. Then take a break

### The Key Point

**queueMicrotask** lets you say: "Don't do this right now, do it as soon as the current code finishes running - but before anything else!"

```javascript
console.log('1: Start');

queueMicrotask(() => {
  console.log('3: Microtask runs');
});

console.log('2: End');

// Output:
// 1: Start
// 2: End
// 3: Microtask runs ← Runs after current code, but before anything else!
```

 

## Real-World Analogy

### The Email Inbox Strategy

Imagine you're a busy manager:

#### **Without Batching (Chaos)** ❌

```
📧 Email arrives: "Change design to blue"
↓
Stop everything! Open design tool. Change to blue.
↓
📧 Email arrives: "Actually, change to red"
↓
Stop everything! Open design tool. Change to red.
↓
📧 Email arrives: "No wait, make it green"
↓
Stop everything! Open design tool. Change to green.
↓
📧 Email arrives: "Final decision: purple"
↓
Stop everything! Open design tool. Change to purple.

Result: You changed the design 4 times! 😱
```

**Problems:**
- ❌ Constantly interrupted
- ❌ Wasted effort (changed 4 times)
- ❌ Inefficient (only final color matters)

 

#### **With Batching (Smart)** ✅

```
📧 Email arrives: "Change design to blue"
↓
Add to to-do list: "Design change"
↓
📧 Email arrives: "Actually, change to red"
↓
Update to-do list: "Design change (red)"
↓
📧 Email arrives: "No wait, make it green"
↓
Update to-do list: "Design change (green)"
↓
📧 Email arrives: "Final decision: purple"
↓
Update to-do list: "Design change (purple)"
↓
Finish current work
↓
Check to-do list
↓
Open design tool ONCE. Change to purple.

Result: You changed the design 1 time! ✨
```

**Benefits:**
- ✅ Work uninterrupted
- ✅ Only do the work once
- ✅ Use the final value
- ✅ Much more efficient!

**That's exactly what queueMicrotask does!**

 

## How JavaScript Executes Code

### The Event Loop

JavaScript runs code in a loop with different queues:

```
┌─────────────────────────────────────────┐
│         JavaScript Event Loop           │
└─────────────────────────────────────────┘
              ↓
    ┌─────────────────────┐
    │   1. Run Script     │ ← Your code runs
    └──────────┬──────────┘
               ↓
    ┌─────────────────────┐
    │ 2. Check Microtask  │ ← queueMicrotask runs here
    │    Queue            │
    └──────────┬──────────┘
               ↓
    ┌─────────────────────┐
    │ 3. Update Screen    │ ← DOM renders
    └──────────┬──────────┘
               ↓
    ┌─────────────────────┐
    │ 4. Check Task Queue │ ← setTimeout runs here
    └──────────┬──────────┘
               ↓
         (Repeat)
```

### The Flow

```
1. Run your code
   ↓
2. Code finishes
   ↓
3. Check: Any microtasks?
   ↓
   YES → Run ALL microtasks
   ↓
4. Render screen
   ↓
5. Check: Any tasks (setTimeout, etc)?
   ↓
6. Repeat
```

 

## Tasks vs Microtasks

### Regular Tasks (setTimeout)

Tasks run **after the current work AND after rendering**.

```javascript
console.log('1: Start');

setTimeout(() => {
  console.log('4: Task runs');
}, 0);

console.log('2: End');

// The screen might update here! 🖼️

// Output:
// 1: Start
// 2: End
// (Screen updates)
// 4: Task runs
```

**Timeline:**

```
Run script → Update screen → Run setTimeout
```

 

### Microtasks (queueMicrotask)

Microtasks run **after the current work BUT before rendering**.

```javascript
console.log('1: Start');

queueMicrotask(() => {
  console.log('3: Microtask runs');
});

console.log('2: End');

// Microtask runs here! (before screen update)

// Output:
// 1: Start
// 2: End
// 3: Microtask runs
// (Then screen updates)
```

**Timeline:**

```
Run script → Run microtasks → Update screen
```

 

### Side-by-Side Comparison

```javascript
console.log('1');

setTimeout(() => console.log('5: Task'), 0);

queueMicrotask(() => console.log('3: Microtask'));

console.log('2');

Promise.resolve().then(() => console.log('4: Promise (also microtask)'));

// Output:
// 1
// 2
// 3: Microtask
// 4: Promise (also microtask)
// 5: Task
```

**Order of execution:**

```
1. Synchronous code (console.log)
   ↓
2. Microtasks (queueMicrotask, Promise)
   ↓
3. Render screen
   ↓
4. Tasks (setTimeout)
```

 

## How queueMicrotask Works

### Basic Syntax

```javascript
queueMicrotask(callback);
```

That's it! Just one function that takes a callback.

### Example 1: Basic Usage

```javascript
console.log('Start');

queueMicrotask(() => {
  console.log('Microtask 1');
});

queueMicrotask(() => {
  console.log('Microtask 2');
});

console.log('End');

// Output:
// Start
// End
// Microtask 1
// Microtask 2
```

**What's happening:**

```
1. console.log('Start')        → Runs immediately
   ↓
2. queueMicrotask(...)          → Adds to queue (doesn't run yet)
   ↓
3. queueMicrotask(...)          → Adds to queue (doesn't run yet)
   ↓
4. console.log('End')           → Runs immediately
   ↓
5. Script finishes
   ↓
6. Check microtask queue
   ↓
7. Run microtask 1
   ↓
8. Run microtask 2
```

 

### Example 2: Multiple Microtasks

```javascript
function addMicrotask(id) {
  queueMicrotask(() => {
    console.log(`Microtask ${id} executed`);
  });
}

console.log('Adding microtasks...');

addMicrotask(1);
addMicrotask(2);
addMicrotask(3);

console.log('Done adding');

// Output:
// Adding microtasks...
// Done adding
// Microtask 1 executed
// Microtask 2 executed
// Microtask 3 executed
```

**All microtasks run together, in order!** ✨

 

## Step-by-Step: Building Your First Microtask

### Example: Batching State Updates

Let's build a simple system that batches multiple state changes.

 

### Step 1: The Problem (Without Batching)

```javascript
let count = 0;
let listeners = [];

function onChange(callback) {
  listeners.push(callback);
}

function setCount(value) {
  count = value;
  // Notify all listeners immediately
  listeners.forEach(fn => fn(count));
}

// Add a listener
onChange((value) => {
  console.log('Count changed to:', value);
});

// Change count 3 times
setCount(1); // Logs: Count changed to: 1
setCount(2); // Logs: Count changed to: 2
setCount(3); // Logs: Count changed to: 3

// Problem: Listener ran 3 times! 😱
```

 

### Step 2: Add Batching with Microtask

```javascript
let count = 0;
let listeners = [];
let isUpdateScheduled = false; // Track if update is queued

function onChange(callback) {
  listeners.push(callback);
}

function notifyListeners() {
  listeners.forEach(fn => fn(count));
  isUpdateScheduled = false; // Reset flag
}

function setCount(value) {
  count = value;
  
  // Only schedule ONE microtask
  if (!isUpdateScheduled) {
    isUpdateScheduled = true;
    queueMicrotask(() => {
      notifyListeners();
    });
  }
}

// Add a listener
onChange((value) => {
  console.log('Count changed to:', value);
});

// Change count 3 times
setCount(1); // Schedules microtask
setCount(2); // Already scheduled, does nothing
setCount(3); // Already scheduled, does nothing

console.log('Done updating');

// Output:
// Done updating
// Count changed to: 3 ← Only runs once with final value! ✨
```

**What's happening:**

```
setCount(1)
↓
count = 1
↓
Schedule microtask? Yes! (not scheduled yet)
↓
isUpdateScheduled = true

setCount(2)
↓
count = 2
↓
Schedule microtask? No! (already scheduled)

setCount(3)
↓
count = 3
↓
Schedule microtask? No! (already scheduled)

Synchronous code finishes
↓
Run queued microtask
↓
notifyListeners() with count = 3
↓
Listener runs ONCE with final value! ✨
```

 

### Step 3: Track Multiple Properties

```javascript
const state = { count: 0, name: 'Alice' };
const listeners = [];
const pendingUpdates = new Set(); // Track which properties changed
let isUpdateScheduled = false;

function onChange(callback) {
  listeners.push(callback);
}

function notifyListeners() {
  const changedProps = Array.from(pendingUpdates);
  pendingUpdates.clear();
  listeners.forEach(fn => fn(changedProps));
  isUpdateScheduled = false;
}

function setState(property, value) {
  state[property] = value;
  pendingUpdates.add(property); // Track this property
  
  if (!isUpdateScheduled) {
    isUpdateScheduled = true;
    queueMicrotask(notifyListeners);
  }
}

onChange((props) => {
  console.log('Properties changed:', props);
  console.log('State:', state);
});

// Make multiple changes
setState('count', 1);
setState('count', 2);
setState('count', 3);
setState('name', 'Bob');
setState('name', 'Charlie');

console.log('Updates queued');

// Output:
// Updates queued
// Properties changed: ['count', 'name']
// State: { count: 3, name: 'Charlie' }
```

**Benefits:**
- ✅ Listener runs once
- ✅ Knows which properties changed
- ✅ Gets final values
- ✅ Efficient batching! 🎉

 

## Why This Is Perfect for Batching Updates

### The Reactivity Problem

In reactive systems, effects might trigger multiple times in a row:

```javascript
const state = state({ x: 0, y: 0 });

effect(() => {
  console.log('Position:', state.x, state.y);
  // Update DOM (expensive!)
  updatePosition(state.x, state.y);
});

// User drags element
state.x = 10; // Effect runs
state.y = 20; // Effect runs
state.x = 15; // Effect runs
state.y = 25; // Effect runs
state.x = 20; // Effect runs
state.y = 30; // Effect runs

// Effect ran 6 times! 😱
// DOM updated 6 times! 😱
```

 

### **Without Batching (Inefficient)** ❌

```javascript
function trigger(property) {
  const effects = deps.get(property);
  if (effects) {
    // Run effects immediately
    effects.forEach(effect => effect());
  }
}

state.x = 10;
trigger('x'); // Effect runs

state.y = 20;
trigger('y'); // Effect runs

state.x = 15;
trigger('x'); // Effect runs

// Effect runs many times! 😱
```

**Problems:**
- ❌ Excessive re-runs
- ❌ DOM thrashing
- ❌ Wasted CPU
- ❌ Poor performance

 

### **With Microtask Batching (Efficient)** ✅

```javascript
const pendingEffects = new Set();
let isFlushPending = false;

function queueEffect(effect) {
  pendingEffects.add(effect); // Add to set (duplicates ignored)
  
  if (!isFlushPending) {
    isFlushPending = true;
    queueMicrotask(flushEffects);
  }
}

function flushEffects() {
  const effects = Array.from(pendingEffects);
  pendingEffects.clear();
  isFlushPending = false;
  
  effects.forEach(effect => effect());
}

function trigger(property) {
  const effects = deps.get(property);
  if (effects) {
    // Queue effects instead of running immediately
    effects.forEach(effect => queueEffect(effect));
  }
}

// Usage
state.x = 10; // Queues effect
state.y = 20; // Effect already queued (Set ignores duplicate)
state.x = 15; // Effect already queued
state.y = 25; // Effect already queued
state.x = 20; // Effect already queued
state.y = 30; // Effect already queued

console.log('Updates queued');

// Microtask runs after synchronous code
// Effect runs ONCE with final values! ✨

// Output:
// Updates queued
// Position: 20 30
```

**Flow:**

```
state.x = 10
↓
trigger('x')
↓
Queue effect (not run yet)
↓
state.y = 20
↓
trigger('y')
↓
Try to queue effect (already in Set, ignored)
↓
(more updates...)
↓
Synchronous code finishes
↓
Microtask runs
↓
Effect executes ONCE with final values
```

**Benefits:**
- ✅ Effect runs once
- ✅ DOM updates once
- ✅ Uses final values
- ✅ Huge performance gain! 🚀

 

### Real Usage in DOM Helpers Reactive

```javascript
const updateQueue = new Map();
let isFlushPending = false;

function queueUpdate(fn, priority) {
  if (!updateQueue.has(priority)) {
    updateQueue.set(priority, new Set());
  }
  updateQueue.get(priority).add(fn);
  
  if (!isFlushPending) {
    isFlushPending = true;
    queueMicrotask(flushQueue); // ← Batch updates!
  }
}

function flushQueue() {
  isFlushPending = false;
  
  // Run updates by priority
  const priorities = Array.from(updateQueue.keys()).sort();
  
  for (const priority of priorities) {
    const effects = updateQueue.get(priority);
    updateQueue.delete(priority);
    
    effects.forEach(effect => {
      try {
        effect(); // Run once per unique effect
      } catch (e) {
        console.error('Effect error:', e);
      }
    });
  }
}
```

 

## Common Questions

### Q: "When do microtasks run?"

**Answer:** Right after the current code finishes, but before any other tasks.

```javascript
console.log('1: Start');

setTimeout(() => console.log('5: setTimeout'), 0);

queueMicrotask(() => console.log('3: Microtask'));

Promise.resolve().then(() => console.log('4: Promise'));

console.log('2: End');

// Output:
// 1: Start
// 2: End
// 3: Microtask
// 4: Promise (also a microtask)
// 5: setTimeout (runs later)
```

**Timeline:**

```
Synchronous code → Microtasks → Render → Tasks
     (1, 2)         (3, 4)        🖼️      (5)
```

 

### Q: "Can I queue too many microtasks?"

**Yes!** Be careful of infinite loops.

```javascript
// ❌ BAD: Infinite loop!
function badIdea() {
  queueMicrotask(() => {
    console.log('Running...');
    badIdea(); // Queues another microtask
  });
}

badIdea(); // Never stops! Browser freezes! 😱
```

**The problem:** Each microtask queues another microtask, so the microtask queue never empties!

**Safe pattern:**

```javascript
// ✅ GOOD: Has an exit condition
function safeBatching(count = 0) {
  if (count >= 10) return; // Stop after 10
  
  queueMicrotask(() => {
    console.log('Batch', count);
    safeBatching(count + 1);
  });
}

safeBatching(); // Runs 10 times, then stops ✅
```

 

### Q: "What's the difference from setTimeout(fn, 0)?"

**Answer:** Timing and priority!

```javascript
console.log('1');

setTimeout(() => console.log('5: setTimeout'), 0);

queueMicrotask(() => console.log('3: Microtask'));

console.log('2');

// Output:
// 1
// 2
// 3: Microtask ← Runs first
// 5: setTimeout ← Runs after screen update
```

**Key differences:**

| Feature | queueMicrotask | setTimeout |
|   |     -|    |
| When | Before render | After render |
| Priority | High | Low |
| Delay | None | Minimum ~4ms |
| Order | Guaranteed | Not guaranteed |

**Use queueMicrotask for:**
- ✅ Batching updates
- ✅ Must run before render
- ✅ Order matters

**Use setTimeout for:**
- ✅ Actual delays
- ✅ Don't block rendering
- ✅ Background work

 

### Q: "Are Promises microtasks?"

**Yes!** Promise `.then()` uses the microtask queue.

```javascript
console.log('1');

Promise.resolve().then(() => console.log('3: Promise'));

queueMicrotask(() => console.log('4: queueMicrotask'));

console.log('2');

// Output:
// 1
// 2
// 3: Promise
// 4: queueMicrotask
```

**Both are microtasks, so they run in order!**

 

### Q: "Can microtasks block rendering?"

**Yes!** Too many microtasks can delay screen updates.

```javascript
// ❌ BAD: 10,000 microtasks
for (let i = 0; i < 10000; i++) {
  queueMicrotask(() => {
    // Do heavy work
    expensiveCalculation();
  });
}

// Screen won't update until ALL microtasks finish! 😱
```

**Solution: Batch intelligently**

```javascript
// ✅ GOOD: Process in chunks
function processBatch(items, batchSize = 100) {
  if (items.length === 0) return;
  
  const batch = items.splice(0, batchSize);
  
  queueMicrotask(() => {
    batch.forEach(item => process(item));
    
    // Use setTimeout for next batch (allows render)
    if (items.length > 0) {
      setTimeout(() => processBatch(items, batchSize), 0);
    }
  });
}
```

 

## Practice Examples

### Example 1: Debounce with Microtask

Batch rapid function calls.

```javascript
function createBatcher() {
  let pending = [];
  let scheduled = false;
  
  return function batch(fn) {
    pending.push(fn);
    
    if (!scheduled) {
      scheduled = true;
      queueMicrotask(() => {
        const functions = pending.slice();
        pending = [];
        scheduled = false;
        
        functions.forEach(f => f());
      });
    }
  };
}

const batch = createBatcher();

console.log('Queueing functions...');

batch(() => console.log('Function 1'));
batch(() => console.log('Function 2'));
batch(() => console.log('Function 3'));

console.log('Done queueing');

// Output:
// Queueing functions...
// Done queueing
// Function 1
// Function 2
// Function 3
```

 

### Example 2: Change Detection

Track and batch property changes.

```javascript
class Observable {
  constructor(data) {
    this.data = data;
    this.listeners = [];
    this.changes = new Set();
    this.scheduled = false;
  }
  
  onChange(callback) {
    this.listeners.push(callback);
  }
  
  set(property, value) {
    if (this.data[property] !== value) {
      this.data[property] = value;
      this.changes.add(property);
      this.scheduleNotification();
    }
  }
  
  scheduleNotification() {
    if (!this.scheduled) {
      this.scheduled = true;
      queueMicrotask(() => {
        const changed = Array.from(this.changes);
        this.changes.clear();
        this.scheduled = false;
        
        this.listeners.forEach(fn => fn(changed, this.data));
      });
    }
  }
}

const state = new Observable({ x: 0, y: 0 });

state.onChange((props, data) => {
  console.log('Changed:', props);
  console.log('Values:', data);
});

state.set('x', 10);
state.set('y', 20);
state.set('x', 15); // x changed again
state.set('y', 20); // Same value, ignored

console.log('Updates queued');

// Output:
// Updates queued
// Changed: ['x', 'y']
// Values: { x: 15, y: 20 }
```

 

### Example 3: DOM Update Batcher

Batch multiple DOM updates into one.

```javascript
class DOMUpdater {
  constructor() {
    this.updates = new Map();
    this.scheduled = false;
  }
  
  update(element, property, value) {
    if (!this.updates.has(element)) {
      this.updates.set(element, {});
    }
    this.updates.get(element)[property] = value;
    this.schedule();
  }
  
  schedule() {
    if (!this.scheduled) {
      this.scheduled = true;
      queueMicrotask(() => this.flush());
    }
  }
  
  flush() {
    this.updates.forEach((props, element) => {
      Object.assign(element, props);
    });
    
    this.updates.clear();
    this.scheduled = false;
  }
}

const updater = new DOMUpdater();
const div = document.getElementById('myDiv');

// Queue multiple updates
updater.update(div, 'textContent', 'Loading...');
updater.update(div, 'className', 'loading');
updater.update(div, 'textContent', 'Processing...');
updater.update(div, 'textContent', 'Done!');

console.log('Updates queued');

// DOM only updates ONCE with final values! ✨
// textContent: 'Done!'
// className: 'loading'
```

 

### Example 4: Priority Queue

Run effects in order of priority.

```javascript
const PRIORITY = {
  HIGH: 1,
  MEDIUM: 2,
  LOW: 3
};

class PriorityQueue {
  constructor() {
    this.queue = new Map();
    this.scheduled = false;
  }
  
  add(fn, priority = PRIORITY.MEDIUM) {
    if (!this.queue.has(priority)) {
      this.queue.set(priority, new Set());
    }
    this.queue.get(priority).add(fn);
    this.schedule();
  }
  
  schedule() {
    if (!this.scheduled) {
      this.scheduled = true;
      queueMicrotask(() => this.flush());
    }
  }
  
  flush() {
    const priorities = Array.from(this.queue.keys()).sort();
    
    for (const priority of priorities) {
      const functions = this.queue.get(priority);
      this.queue.delete(priority);
      
      functions.forEach(fn => fn());
    }
    
    this.scheduled = false;
  }
}

const queue = new PriorityQueue();

queue.add(() => console.log('Low priority'), PRIORITY.LOW);
queue.add(() => console.log('High priority'), PRIORITY.HIGH);
queue.add(() => console.log('Medium priority'), PRIORITY.MEDIUM);
queue.add(() => console.log('Another high'), PRIORITY.HIGH);

console.log('Queued');

// Output:
// Queued
// High priority
// Another high
// Medium priority
// Low priority
```

 

## Summary

### What is the MicrotaskQueue?

A **special queue** for tasks that run **right after the current code finishes**, but **before rendering**.

```javascript
console.log('1: Sync');
queueMicrotask(() => console.log('2: Microtask'));
console.log('3: Sync');

// Output:
// 1: Sync
// 3: Sync
// 2: Microtask ← Runs after sync, before render
```

### The Execution Order

```
1. Run synchronous code
   ↓
2. Run ALL microtasks
   ↓
3. Update screen
   ↓
4. Run tasks (setTimeout, etc.)
```

### When to Use queueMicrotask

✅ **Use queueMicrotask when:**
- Batching rapid updates
- Must run before render
- Order matters
- Performance critical

❌ **Don't use when:**
- Need actual delays → use setTimeout
- Want to allow rendering → use setTimeout
- Processing large data → chunk with setTimeout

### Why It's Perfect for Reactivity

**Batches multiple state changes into one update:**

```javascript
// Without batching:
state.x = 1; // Effect runs
state.y = 2; // Effect runs
state.z = 3; // Effect runs
// Effect ran 3 times! ❌

// With queueMicrotask:
state.x = 1; // Queue update
state.y = 2; // Already queued
state.z = 3; // Already queued
// (code finishes)
// Effect runs ONCE with all changes! ✅
```

### Real Usage in DOM Helpers Reactive

```javascript
function queueUpdate(fn) {
  pendingEffects.add(fn);
  
  if (!isFlushPending) {
    isFlushPending = true;
    queueMicrotask(flushEffects); // ← Smart batching!
  }
}

function flushEffects() {
  pendingEffects.forEach(effect => effect());
  pendingEffects.clear();
  isFlushPending = false;
}
```

### Key Takeaway

**queueMicrotask is the secret sauce that makes reactive systems efficient by batching updates!** ⚡

```
Multiple changes → queueMicrotask → One efficient update ✨
```

 

**The MicrotaskQueue is JavaScript's built-in smart batcher that makes reactivity blazingly fast!** 🚀