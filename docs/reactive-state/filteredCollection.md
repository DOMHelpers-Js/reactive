[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `filteredCollection(collection, predicate)` - Create Filtered View

## Quick Start (30 seconds)

```javascript
// Source collection
const allTodos = createCollection([
  { id: 1, text: 'Buy milk', done: false, priority: 'high' },
  { id: 2, text: 'Walk dog', done: true, priority: 'low' },
  { id: 3, text: 'Clean room', done: false, priority: 'high' },
  { id: 4, text: 'Cook dinner', done: true, priority: 'medium' }
]);

// Create filtered view - only incomplete todos
const activeTodos = filteredCollection(
  allTodos,
  todo => !todo.done
);

console.log(activeTodos.length);  // 2
console.log(activeTodos.items);
// [{ id: 1, text: 'Buy milk', ... }, { id: 3, text: 'Clean room', ... }]

// Create another view - only high priority
const highPriority = filteredCollection(
  allTodos,
  todo => todo.priority === 'high'
);

console.log(highPriority.length);  // 2

// Change source - filtered views auto-update!
allTodos.update(t => t.id === 1, { done: true });

console.log(activeTodos.length);  // 1 (automatically updated!)
console.log(highPriority.length); // 2 (still shows high priority) ✨
```

**What just happened?** You created live filtered views that automatically stay in sync with the source collection!

 

## What is `filteredCollection(collection, predicate)`?

`filteredCollection()` creates a **reactive filtered view** of an existing collection that automatically updates when the source changes.

Simply put: it's a live window into your data that only shows items matching your criteria, and the view stays current as data changes.

Think of it as **a smart camera with auto-tracking** - point it at your data with a filter (predicate), and it automatically shows you only matching items, updating in real-time as things change.

 

## Syntax

```javascript
filteredCollection(collection, predicate)
```

**Available as:**
```javascript
// Global function (if available)
const filtered = filteredCollection(source, predicate);

// Collections namespace
const filtered = Collections.createFiltered(source, predicate);
```

**Parameters:**
- `collection` (Collection) - Source collection to filter
- `predicate` (Function) - Filter function that returns `true` for items to include

**Predicate Signature:**
```javascript
function predicate(item, index) {
  return boolean;  // true = include, false = exclude
}
```

**Returns:** 
- New filtered collection that auto-syncs with source

 


## Why Does This Exist?

### Two Approaches to Filtered Data Views

The Reactive library offers flexible ways to work with filtered subsets of collections, each suited to different use cases.

### Function-Based Filtering

When you need **on-demand filtering** and want explicit control over when filtering occurs:

```javascript
// Create a collection
const allTodos = createCollection([...]);

// Define filter functions
function getActiveTodos() {
  return allTodos.filter(t => !t.done);
}

function getCompletedTodos() {
  return allTodos.filter(t => t.done);
}

function getHighPriorityTodos() {
  return allTodos.filter(t => t.priority === 'high');
}

// Call functions when needed
const active = getActiveTodos();
console.log(active.length);

// Add todo
allTodos.add({ text: 'New task', done: false });

// Call again to get updated view
const updatedActive = getActiveTodos();
console.log(updatedActive.length);

// Use in effects
effect(() => {
  const active = getActiveTodos();
  const completed = getCompletedTodos();
  render(active, completed);
});
```

**This approach is great when you need:**
✅ Explicit control over when filtering happens
✅ One-time snapshots of filtered data
✅ Performance optimization for infrequent updates
✅ Standard JavaScript filtering patterns

### When Auto-Synchronized Filtered Views Fit Your Workflow

In scenarios where you want **filtered views that stay synchronized automatically** with the source collection, `filteredCollection()` provides a more direct approach:

```javascript
// Create source collection
const allTodos = createCollection([...]);

// Create filtered views once
const activeTodos = filteredCollection(
  allTodos,
  t => !t.done
);

const completedTodos = filteredCollection(
  allTodos,
  t => t.done
);

const highPriority = filteredCollection(
  allTodos,
  t => t.priority === 'high'
);

// Views are always current
console.log(activeTodos.length);

// Add todo - all views auto-update
allTodos.add({ text: 'New task', done: false, priority: 'high' });

console.log(activeTodos.length);    // Auto-updated!
console.log(highPriority.length);   // Auto-updated!

// In effects, use views directly
effect(() => {
  render(activeTodos.items, completedTodos.items);
  // Always current, reactively updates
});
```

**This method is especially useful when:**

```
filteredCollection Flow:
┌──────────────────────┐
│ Define filter once   │
└──────────┬───────────┘
           │
           ▼
   Source changes
           │
           ▼
   Filtered view
   auto-updates
           │
           ▼
  ✅ Always synchronized
```

**Where filteredCollection() shines:**
✅ **Automatic synchronization** - Filtered views stay current with source changes
✅ **Reactive by default** - Works seamlessly with effects and watchers
✅ **Multiple views** - Create many filtered perspectives of the same data
✅ **Collection methods** - Filtered views have full collection API (add, remove, etc.)
✅ **Efficient updates** - Only recalculates when source changes

**The Choice is Yours:**
- Use function-based filtering when you need explicit control over when filtering occurs
- Use `filteredCollection()` when you want auto-synchronized filtered views
- Both approaches work with reactive collections

**Benefits of the filteredCollection approach:**
✅ **Define once, use everywhere** - Filter logic in one place
✅ **Automatic updates** - Filtered views sync when source changes
✅ **Multiple perspectives** - Create as many filtered views as needed
✅ **Full collection API** - Filtered views have all collection methods
✅ **Reactive integration** - Works seamlessly with effects and watchers
✅ **Consistent state** - Filtered views never out of sync with source

## Mental Model

Think of filtered collections as **live security camera views with smart filters**:

### Without filteredCollection (Static Photos)
```
Main Collection          Filtered "Snapshots"
┌──────────────┐        ┌──────────────┐
│ All Todos    │   ───→ │ Active ❌    │
│ [10 items]   │        │ (snapshot)   │
└──────────────┘        └──────────────┘
      ↓                        ↓
  Add new todo            Still shows old
      ↓                        ↓
  Photos outdated         Must retake photo
```

### With filteredCollection (Live Camera Feed)
```
Main Collection          Filtered Views
┌──────────────┐        ┌──────────────┐
│ All Todos    │   ───→ │ Active ✅    │
│ [10 items]   │        │ (live feed)  │
└──────────────┘        └──────────────┘
      ↓                        ↓
  Add new todo            View updates instantly
      ↓                        ↓
  Always in sync          Always current
```

**Key Insight:** Filtered collections are live views that automatically track their source, not static snapshots.

 

## How Does It Work?

### Creation Process

```
1️⃣ Create new collection
   filteredCollection = createCollection([])
        ↓
2️⃣ Set up synchronization
   effect(() => {
     const matching = source.items.filter(predicate);
     filteredCollection.reset(matching);
   })
        ↓
3️⃣ Return filtered collection
   Automatically updates when source changes
```

### Sync Mechanism

```
Source collection changes
        ↓
Reactive effect detects change
        ↓
Reapply filter to source.items
        ↓
Update filtered collection.items
        ↓
Filtered collection reactive updates fire
        ↓
Your UI updates ✨
```

### Data Flow Diagram

```
Source Collection
    ┌─────────────────┐
    │ Item 1 ✓ match  │──┐
    │ Item 2 ✗ no     │  │
    │ Item 3 ✓ match  │──┤   Predicate Filter
    │ Item 4 ✗ no     │  │         ↓
    │ Item 5 ✓ match  │──┘   Filtered View
    └─────────────────┘      ┌─────────────┐
                             │ Item 1 ✓    │
           Auto-sync         │ Item 3 ✓    │
              ↕              │ Item 5 ✓    │
           Live view         └─────────────┘
```

 

## Basic Usage

### Example 1: Simple Filtering

```javascript
const numbers = createCollection([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

// Even numbers only
const evenNumbers = filteredCollection(
  numbers,
  n => n % 2 === 0
);

console.log(evenNumbers.items);  // [2, 4, 6, 8, 10]

// Add number to source
numbers.add(12);

console.log(evenNumbers.items);  // [2, 4, 6, 8, 10, 12] (auto-updated!)

// Add odd number
numbers.add(13);

console.log(evenNumbers.items);  // [2, 4, 6, 8, 10, 12] (no change)
```

**What's happening?**
- Filtered view shows only matching items
- Updates automatically when source changes
- Non-matching items ignored

 

### Example 2: Multiple Views

```javascript
const todos = createCollection([
  { text: 'Task 1', done: false },
  { text: 'Task 2', done: true },
  { text: 'Task 3', done: false }
]);

// Create multiple views
const active = filteredCollection(todos, t => !t.done);
const completed = filteredCollection(todos, t => t.done);

console.log(active.length);      // 2
console.log(completed.length);   // 1

// Update source
todos.update(t => t.text === 'Task 1', { done: true });

console.log(active.length);      // 1 (updated!)
console.log(completed.length);   // 2 (updated!)
```

 

### Example 3: Complex Filtering

```javascript
const products = createCollection([
  { name: 'Widget', price: 10, category: 'tools', inStock: true },
  { name: 'Gadget', price: 25, category: 'electronics', inStock: false },
  { name: 'Tool', price: 15, category: 'tools', inStock: true }
]);

// Affordable tools in stock
const affordableTools = filteredCollection(
  products,
  p => p.category === 'tools' && p.price < 20 && p.inStock
);

console.log(affordableTools.length);  // 2

// Update price
products.update(p => p.name === 'Widget', { price: 25 });

console.log(affordableTools.length);  // 1 (Widget no longer affordable)
```

 

## Real-World Examples

### Example 1: Todo App with Multiple Views

```javascript
const allTodos = createCollection([]);

// Create filtered views
const activeTodos = filteredCollection(
  allTodos,
  todo => !todo.done
);

const completedTodos = filteredCollection(
  allTodos,
  todo => todo.done
);

const todayTodos = filteredCollection(
  allTodos,
  todo => {
    const today = new Date().toDateString();
    return new Date(todo.dueDate).toDateString() === today;
  }
);

const highPriorityTodos = filteredCollection(
  allTodos,
  todo => todo.priority === 'high' && !todo.done
);

// UI automatically updates for all views
effect(() => {
  document.getElementById('active-count').textContent = 
    activeTodos.length;
  document.getElementById('completed-count').textContent = 
    completedTodos.length;
  document.getElementById('today-count').textContent = 
    todayTodos.length;
  document.getElementById('high-priority-count').textContent = 
    highPriorityTodos.length;
});

// Add todo - all relevant views update
function addTodo(text, dueDate, priority) {
  allTodos.add({
    id: Date.now(),
    text,
    done: false,
    dueDate,
    priority,
    createdAt: new Date()
  });
}

// Complete todo - views auto-update
function completeTodo(id) {
  allTodos.update(t => t.id === id, { done: true });
}
```

 

### Example 2: E-commerce Product Filters

```javascript
const allProducts = createCollection([]);

// Category views
const electronics = filteredCollection(
  allProducts,
  p => p.category === 'electronics'
);

const clothing = filteredCollection(
  allProducts,
  p => p.category === 'clothing'
);

// Price range views
const budget = filteredCollection(
  allProducts,
  p => p.price < 50
);

const premium = filteredCollection(
  allProducts,
  p => p.price >= 100
);

// Availability views
const inStock = filteredCollection(
  allProducts,
  p => p.stock > 0
);

const onSale = filteredCollection(
  allProducts,
  p => p.salePrice && p.salePrice < p.price
);

// Combined filters
const budgetElectronicsInStock = filteredCollection(
  allProducts,
  p => p.category === 'electronics' && 
       p.price < 50 && 
       p.stock > 0
);

// Display product counts
effect(() => {
  document.getElementById('electronics-count').textContent = 
    electronics.length;
  document.getElementById('instock-count').textContent = 
    inStock.length;
  document.getElementById('sale-count').textContent = 
    onSale.length;
});

// Update product - all relevant views update
function updateProduct(id, updates) {
  allProducts.update(p => p.id === id, updates);
}
```

 

### Example 3: User Management Dashboard

```javascript
const allUsers = createCollection([]);

// Status views
const activeUsers = filteredCollection(
  allUsers,
  u => u.status === 'active'
);

const inactiveUsers = filteredCollection(
  allUsers,
  u => u.status === 'inactive'
);

const suspendedUsers = filteredCollection(
  allUsers,
  u => u.status === 'suspended'
);

// Role views
const admins = filteredCollection(
  allUsers,
  u => u.role === 'admin'
);

const moderators = filteredCollection(
  allUsers,
  u => u.role === 'moderator'
);

// Special views
const newUsers = filteredCollection(
  allUsers,
  u => {
    const weekAgo = Date.now() - (7 * 24 * 60 * 60 * 1000);
    return u.createdAt > weekAgo;
  }
);

const unverifiedEmails = filteredCollection(
  allUsers,
  u => !u.emailVerified
);

// Dashboard stats
effect(() => {
  const stats = document.getElementById('user-stats');
  stats.innerHTML = `
    <div class="stat">
      <h3>Active Users</h3>
      <p>${activeUsers.length}</p>
    </div>
    <div class="stat">
      <h3>New This Week</h3>
      <p>${newUsers.length}</p>
    </div>
    <div class="stat">
      <h3>Unverified</h3>
      <p>${unverifiedEmails.length}</p>
    </div>
  `;
});
```

 

### Example 4: Task Management with Priority Views

```javascript
const allTasks = createCollection([]);

// Priority views
const urgentTasks = filteredCollection(
  allTasks,
  t => t.priority === 'urgent' && !t.done
);

const highTasks = filteredCollection(
  allTasks,
  t => t.priority === 'high' && !t.done
);

const normalTasks = filteredCollection(
  allTasks,
  t => t.priority === 'normal' && !t.done
);

// Time-based views
const overdueTasks = filteredCollection(
  allTasks,
  t => !t.done && t.dueDate && new Date(t.dueDate) < new Date()
);

const dueTodayTasks = filteredCollection(
  allTasks,
  t => {
    if (t.done || !t.dueDate) return false;
    const today = new Date().toDateString();
    return new Date(t.dueDate).toDateString() === today;
  }
);

// Assignment views
const myTasks = filteredCollection(
  allTasks,
  t => t.assignedTo === currentUserId && !t.done
);

const unassignedTasks = filteredCollection(
  allTasks,
  t => !t.assignedTo && !t.done
);

// Dashboard
effect(() => {
  // Urgent notification
  if (urgentTasks.length > 0) {
    showNotification(`You have ${urgentTasks.length} urgent tasks!`);
  }
  
  // Update task lists
  renderTaskList('urgent-list', urgentTasks.items);
  renderTaskList('overdue-list', overdueTasks.items);
  renderTaskList('today-list', dueTodayTasks.items);
  renderTaskList('my-tasks-list', myTasks.items);
});
```

 

### Example 5: Analytics Dashboard

```javascript
const allEvents = createCollection([]);

// Event type views
const pageViews = filteredCollection(
  allEvents,
  e => e.type === 'pageview'
);

const clicks = filteredCollection(
  allEvents,
  e => e.type === 'click'
);

const conversions = filteredCollection(
  allEvents,
  e => e.type === 'conversion'
);

// Time-based views
const todayEvents = filteredCollection(
  allEvents,
  e => {
    const today = new Date().toDateString();
    return new Date(e.timestamp).toDateString() === today;
  }
);

const lastHourEvents = filteredCollection(
  allEvents,
  e => {
    const hourAgo = Date.now() - (60 * 60 * 1000);
    return e.timestamp > hourAgo;
  }
);

// User segment views
const mobileUsers = filteredCollection(
  allEvents,
  e => e.device === 'mobile'
);

const returningUsers = filteredCollection(
  allEvents,
  e => e.returning === true
);

// Live analytics
effect(() => {
  const analytics = document.getElementById('analytics');
  analytics.innerHTML = `
    <div class="metric">
      <h3>Page Views Today</h3>
      <p>${todayEvents.items.filter(e => e.type === 'pageview').length}</p>
    </div>
    <div class="metric">
      <h3>Conversions</h3>
      <p>${conversions.length}</p>
    </div>
    <div class="metric">
      <h3>Activity (Last Hour)</h3>
      <p>${lastHourEvents.length} events</p>
    </div>
  `;
});
```

 

## Common Patterns

### Pattern 1: Nested Filters

```javascript
const active = filteredCollection(all, item => item.active);
const highPriority = filteredCollection(active, item => item.priority === 'high');
// Shows active AND high priority items
```

 

### Pattern 2: Dynamic Filters

```javascript
let currentCategory = 'all';

function updateCategoryFilter(category) {
  currentCategory = category;
  // Recreate filtered view with new predicate
  return filteredCollection(
    allProducts,
    p => currentCategory === 'all' || p.category === currentCategory
  );
}
```

 

### Pattern 3: Search Filter

```javascript
let searchQuery = '';

const searchResults = filteredCollection(
  allItems,
  item => {
    if (!searchQuery) return true;
    return item.name.toLowerCase().includes(searchQuery.toLowerCase());
  }
);

function search(query) {
  searchQuery = query;
  // Filter updates automatically
}
```

 

## Important Notes

### 1. Filtered View is Read-Only for Source

```javascript
const filtered = filteredCollection(source, predicate);

// ❌ Don't modify filtered view directly
filtered.add(item);  // This won't affect source!

// ✅ Modify source instead
source.add(item);  // Filtered view updates automatically
```

 

### 2. Predicate Runs on Source Changes

```javascript
let count = 0;
const filtered = filteredCollection(source, item => {
  count++;  // Runs every time source changes
  return item.active;
});

console.log(count);  // Initial run

source.add(item);
console.log(count);  // Ran again
```

 

### 3. Multiple Filters are Independent

```javascript
const view1 = filteredCollection(source, p1);
const view2 = filteredCollection(source, p2);

// Both update independently when source changes
// Each has its own items array
```

 

### 4. Filters Create New Collections

```javascript
const filtered = filteredCollection(source, pred);

// filtered is a full collection with all methods
filtered.find(item => ...)
filtered.filter(item => ...)
filtered.length
filtered.first
// etc.
```

 

## Summary

**What is `filteredCollection()`?**  
Creates a reactive filtered view that automatically syncs with a source collection.

**Why use it?**
- ✅ Auto-sync with source
- ✅ Always up-to-date
- ✅ Multiple views possible
- ✅ Efficient updates

**Key Takeaway:**

```
filter() Method          filteredCollection()
      |                         |
Static snapshot          Live view
      |                         |
Manual updates          Auto-sync
      |                         |
Gets stale              Always current ✅
```

**One-Line Rule:** Use `filteredCollection()` when you need a live view of data that automatically updates as the source changes.

**Remember:** Filtered collections are live camera feeds, not static snapshots - they track their source automatically! 🎉