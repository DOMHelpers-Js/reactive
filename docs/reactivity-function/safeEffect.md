[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `safeEffect()` - Error-Safe Effects That Never Break Your App

## Quick Start (30 seconds)

```javascript
const state = ReactiveUtils.state({ count: 0, items: [] });

// Regular effect - ONE error crashes EVERYTHING
effect(() => {
  console.log(state.count);
  state.items.forEach(item => item.process()); // 💥 Error here stops ALL effects
});

// Safe effect - errors are contained and handled gracefully
safeEffect(() => {
  console.log(state.count);
  state.items.forEach(item => item.process()); // ✅ Error logged, app keeps running
}, {
  errorBoundary: {
    onError: (error) => console.error('Handled:', error)
  }
});
```

**What just happened?** The error in the safe effect was caught, logged, and didn't crash anything else. Your app keeps running smoothly.

 

## What is safeEffect()?

`safeEffect()` creates a reactive effect that **automatically catches and handles errors** instead of letting them crash your application.

Think of it as putting a **safety net** under your effect. If something goes wrong, the error gets caught, logged, and handled gracefully — your app keeps running.

### Simple Definition

**Regular `effect()`**: If an error occurs, it bubbles up and can crash your app or stop other effects from running.

**`safeEffect()`**: If an error occurs, it's caught, handled according to your rules, and life goes on.

 

## Syntax

### Shorthand (Recommended)
```javascript
safeEffect(fn, options)
```

### Full Namespace Style
```javascript
ReactiveUtils.safeEffect(fn, options)
```

### Parameters

| Parameter | Type | Description |
|   --|  |    -|
| `fn` | Function | The effect function to run reactively |
| `options` | Object | Configuration for error handling (optional) |

### Options Object

```javascript
{
  errorBoundary: {
    onError: (error, context) => { /* Handle error */ },
    fallback: (error, context) => { /* Return fallback value */ },
    retry: true,           // Should retry on error? (default: true)
    maxRetries: 3,         // Maximum retry attempts (default: 3)
    retryDelay: 0          // Delay between retries in ms (default: 0)
  }
}
```

### Returns

- **Cleanup function** - Call this to stop the effect and prevent memory leaks

 

## Why Does This Exist?

### The Problem with Regular Effects

Let's say you're building a dashboard that tracks multiple data sources:

```javascript
const state = ReactiveUtils.state({
  userStats: { visits: 100 },
  salesData: { total: 5000 },
  analytics: null  // ← This might be null sometimes
});

// Regular effect - looks harmless
effect(() => {
  console.log('User visits:', state.userStats.visits);
  console.log('Sales total:', state.salesData.total);
  console.log('Analytics:', state.analytics.pageViews);  // 💥 BOOM!
});
```

At first glance, this looks fine. But when `state.analytics` is `null`, this happens:

```
💥 TypeError: Cannot read property 'pageViews' of null
```

**What's the Real Issue?**

```
Effect Runs
    ↓
Line 1: ✅ Success
    ↓
Line 2: ✅ Success
    ↓
Line 3: 💥 ERROR - Cannot read 'pageViews' of null
    ↓
Effect CRASHES
    ↓
❌ All other effects might stop
❌ UI might freeze
❌ App might become unresponsive
```

**Problems:**

❌ **One error crashes everything** - A single null value breaks the entire effect  
❌ **No recovery mechanism** - The effect is just... dead  
❌ **Silent failures** - You might not even know what went wrong  
❌ **Fragile code** - You need defensive checks everywhere  
❌ **Poor user experience** - One API failing shouldn't break the whole dashboard  

### The Solution with safeEffect()

```javascript
const state = ReactiveUtils.state({
  userStats: { visits: 100 },
  salesData: { total: 5000 },
  analytics: null
});

// Safe effect - errors are handled gracefully
safeEffect(() => {
  console.log('User visits:', state.userStats.visits);
  console.log('Sales total:', state.salesData.total);
  console.log('Analytics:', state.analytics.pageViews);  // ✅ Error caught!
}, {
  errorBoundary: {
    onError: (error, context) => {
      console.error('Dashboard error:', error.message);
      // Maybe show a notification to the user
      // Maybe log to your error tracking service
    }
  }
});

// The effect continues running, other parts still work!
```

**What Just Happened?**

```
Effect Runs
    ↓
Line 1: ✅ Success - displays user visits
    ↓
Line 2: ✅ Success - displays sales total
    ↓
Line 3: 💥 ERROR - Cannot read 'pageViews' of null
    ↓
Error Boundary CATCHES IT
    ↓
onError() handler runs
    ↓
✅ Effect stays alive
✅ Other effects keep running
✅ App continues working
✅ Error is logged for debugging
```

**Benefits:**

✅ **Resilient effects** - Errors don't crash your app  
✅ **Automatic recovery** - Can retry failed operations automatically  
✅ **Visible errors** - You get clear error reports  
✅ **Clean code** - No defensive null checks everywhere  
✅ **Better UX** - Parts of your UI work even when others fail  

 

## Mental Model

Think of `safeEffect()` as the difference between a **regular juggler** and a **safety net juggler**.

### Regular Effect (Regular Juggler)

```
Juggler
  ↓
Catches Ball 1 ✅
  ↓
Catches Ball 2 ✅
  ↓
Drops Ball 3 💥
  ↓
❌ PERFORMANCE ENDS
❌ Everyone goes home
❌ Show is over
```

**One mistake = complete failure.**

### Safe Effect (Safety Net Juggler)

```
Juggler + Safety Net
  ↓
Catches Ball 1 ✅
  ↓
Catches Ball 2 ✅
  ↓
Drops Ball 3 💥
  ↓
✅ Safety net catches it
✅ Error is logged
✅ Juggler continues
✅ Show goes on!
```

**Mistakes are caught and handled gracefully.**

 

## How Does It Work?

Under the hood, `safeEffect()` wraps your effect function in a **try-catch block** with intelligent error handling.

### Step-by-Step Internal Flow

1️⃣ **Effect Creation**
```
safeEffect() called
    ↓
Creates ErrorBoundary instance
    ↓
Wraps your function with error catching
    ↓
Passes wrapped function to regular effect()
```

2️⃣ **Effect Execution**
```
Reactive dependency changes
    ↓
Effect triggers
    ↓
Try Block:
  → Run your function
  → Track dependencies
  → Everything works? Done!
    ↓
Catch Block (if error):
  → Catch the error
  → Run onError callback
  → Maybe retry
  → Maybe return fallback
```

3️⃣ **Error Handling Flow**

```
Error Occurs
    ↓
ErrorBoundary catches it
    ↓
Check: Should retry?
    ├─→ YES: Attempt < maxRetries?
    │         ├─→ YES: Wait retryDelay ms → Retry
    │         └─→ NO: Run fallback or log error
    │
    └─→ NO: Run fallback or log error
    ↓
Effect continues (doesn't crash!)
```

### Visual: Regular Effect vs Safe Effect

**Regular Effect:**
```
Your Code
    ↓
[Effect Runs]
    ↓
💥 Error
    ↓
App Crashes
```

**Safe Effect:**
```
Your Code
    ↓
[ErrorBoundary Wrapper]
    ├─→ Try: [Effect Runs] ✅ Success → Done
    │
    └─→ Catch: 💥 Error
              ↓
          [onError Handler]
              ↓
          [Retry Logic?]
              ↓
          [Fallback Value?]
              ↓
          App Continues ✅
```

 

## Basic Usage

### Example 1: Basic Error Handling

The simplest use case — just catch and log errors:

```javascript
const state = ReactiveUtils.state({ 
  user: { name: 'Alice' } 
});

// Create a safe effect
const cleanup = safeEffect(() => {
  // This might error if user becomes null
  console.log(`Hello, ${state.user.name}!`);
}, {
  errorBoundary: {
    onError: (error) => {
      console.error('Effect error:', error.message);
    }
  }
});

// Later, if state.user becomes null:
state.user = null;  // ✅ Error caught, app keeps running

// Clean up when done
cleanup();
```

**What's happening?**

- Effect runs normally when data is valid
- When `user` becomes `null`, trying to access `.name` throws an error
- The error boundary catches it and calls `onError`
- The effect stays alive and continues to track dependencies

 

### Example 2: Without Error Boundary (Default Logging)

If you don't provide `errorBoundary` options, errors are still caught and logged to console:

```javascript
const state = ReactiveUtils.state({ items: [] });

safeEffect(() => {
  // This will error if items[0] doesn't exist
  console.log('First item:', state.items[0].name);
});

// This triggers the effect
state.items = [];  // ✅ Error logged to console, effect continues
```

**Output:**
```
[Enhancements] Error in effect : Cannot read property 'name' of undefined
```

 

### Example 3: Multiple Safe Effects

Each effect has its own error boundary — they're completely independent:

```javascript
const state = ReactiveUtils.state({ 
  users: [],
  products: []
});

// Effect 1: Handle users
safeEffect(() => {
  state.users.forEach(user => {
    console.log(user.name);  // Might error
  });
}, {
  errorBoundary: {
    onError: () => console.error('User processing failed')
  }
});

// Effect 2: Handle products
safeEffect(() => {
  state.products.forEach(product => {
    console.log(product.price);  // Might error
  });
}, {
  errorBoundary: {
    onError: () => console.error('Product processing failed')
  }
});

// If users data is bad, ONLY Effect 1 fails
// Effect 2 keeps working perfectly ✅
```

 

### Example 4: Error Context Information

The `onError` callback receives both the error and context information:

```javascript
safeEffect(() => {
  // Some complex operation
  state.items.forEach(item => processItem(item));
}, {
  errorBoundary: {
    onError: (error, context) => {
      console.error('Error:', error.message);
      console.error('Context:', context);
      // context = {
      //   type: 'effect',
      //   created: 1704672000000,
      //   attempt: 1,
      //   maxRetries: 3,
      //   willRetry: true
      // }
    }
  }
});
```

**The context object contains:**

- `type` - Always `'effect'` for effects (vs 'watch')
- `created` - Timestamp when the effect was created
- `attempt` - Current retry attempt number
- `maxRetries` - Maximum allowed retries
- `willRetry` - Boolean indicating if a retry will happen

 

## Deep Dive: Error Boundaries

Error boundaries are the core feature that makes safe effects work. Let's explore them in detail.

### What is an Error Boundary?

An error boundary is an **isolated error-catching zone** around your code. Think of it like a **firebreak in a forest** — it stops errors from spreading.

```
Normal Code:
[Code A] → [Code B] → [💥 Error] → [❌ App Crashes]

With Error Boundary:
[Code A] → [Code B] → [💥 Error] → [✅ Caught & Handled] → [Code C continues]
```

### The ErrorBoundary Class

Behind the scenes, `safeEffect()` uses the `ErrorBoundary` class:

```javascript
class ErrorBoundary {
  constructor(options) {
    this.onError = options.onError || defaultErrorHandler;
    this.fallback = options.fallback;
    this.retry = options.retry !== false;  // Default true
    this.maxRetries = options.maxRetries || 3;
    this.retryDelay = options.retryDelay || 0;
  }
  
  wrap(fn, context) {
    // Returns a wrapped version of fn that catches errors
  }
}
```

### Configuring Error Boundaries

#### Option 1: Custom Error Handler

```javascript
safeEffect(() => {
  // Your effect code
}, {
  errorBoundary: {
    onError: (error, context) => {
      // Send to error tracking service
      trackError(error, {
        component: 'dashboard',
        user: getCurrentUser(),
        timestamp: Date.now()
      });
      
      // Show user-friendly notification
      showNotification('Something went wrong', 'error');
      
      // Log for debugging
      console.error('Effect failed:', error);
    }
  }
});
```

#### Option 2: Disable Retries

```javascript
safeEffect(() => {
  // This should NOT retry on failure
  sendAnalytics(state.event);
}, {
  errorBoundary: {
    retry: false,  // Don't retry
    onError: (error) => {
      console.error('Analytics failed:', error);
    }
  }
});
```

#### Option 3: Custom Retry Configuration

```javascript
safeEffect(() => {
  // Might fail due to network issues
  fetchData(state.apiUrl);
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 5,      // Try 5 times
    retryDelay: 1000,   // Wait 1 second between attempts
    onError: (error, context) => {
      if (context.willRetry) {
        console.log(`Retry ${context.attempt}/${context.maxRetries}...`);
      } else {
        console.error('All retries failed:', error);
      }
    }
  }
});
```

 

## Deep Dive: Retry Logic

One of the most powerful features of `safeEffect()` is **automatic retry logic**. Let's understand how it works.

### Why Automatic Retries?

Sometimes errors are **temporary**:

- Network requests fail momentarily
- Race conditions resolve themselves
- External services come back online
- Data loads in the next tick

Instead of failing permanently, we can **try again automatically**.

### How Retry Works

```
Effect Runs
    ↓
💥 Error Occurs
    ↓
Check: attempt < maxRetries?
    ├─→ YES: Wait retryDelay ms
    │         ↓
    │    Increment attempt
    │         ↓
    │    Run Effect Again
    │         ↓
    │    Success? Done! ✅
    │    Error? Loop back ↑
    │
    └─→ NO: Give up
          ↓
     Run fallback (if provided)
          ↓
     Call onError with willRetry: false
```

### Example: Retrying Failed API Calls

```javascript
const state = ReactiveUtils.state({ 
  apiUrl: '/api/data',
  data: null 
});

safeEffect(() => {
  // Simulated fetch that might fail
  const response = fetch(state.apiUrl);
  
  if (!response.ok) {
    throw new Error('API request failed');
  }
  
  state.data = response.json();
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 3,
    retryDelay: 2000,  // Wait 2 seconds between retries
    
    onError: (error, context) => {
      if (context.willRetry) {
        console.log(`Attempt ${context.attempt} failed. Retrying in 2s...`);
      } else {
        console.error('All retries exhausted:', error);
        showErrorMessage('Unable to load data. Please try again later.');
      }
    }
  }
});
```

**Output on failure:**
```
Attempt 1 failed. Retrying in 2s...
Attempt 2 failed. Retrying in 2s...
Attempt 3 failed. Retrying in 2s...
All retries exhausted: Error: API request failed
```

### Retry Without Delay

For fast operations, you might want instant retries:

```javascript
safeEffect(() => {
  // Quick synchronous operation
  processData(state.items);
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 5,
    retryDelay: 0  // No delay, retry immediately
  }
});
```

### Understanding Retry Context

The `context` object in `onError` tells you everything about the retry state:

```javascript
safeEffect(() => {
  riskyOperation();
}, {
  errorBoundary: {
    maxRetries: 3,
    onError: (error, context) => {
      console.log({
        attempt: context.attempt,      // Current attempt (1, 2, 3...)
        maxRetries: context.maxRetries, // Total allowed (3)
        willRetry: context.willRetry    // true if more attempts left
      });
      
      // Example output on attempt 2:
      // {
      //   attempt: 2,
      //   maxRetries: 3,
      //   willRetry: true  ← Will try again!
      // }
      
      // Example output on attempt 3 (last):
      // {
      //   attempt: 3,
      //   maxRetries: 3,
      //   willRetry: false  ← This was the last try
      // }
    }
  }
});
```

 

## Deep Dive: Fallback Values

When an effect fails and retries are exhausted, you can provide a **fallback value** to use instead.

### What is a Fallback?

A fallback is a **safe default value** returned when the effect permanently fails.

```
Effect Runs
    ↓
💥 Error
    ↓
Retries Exhausted
    ↓
fallback() function called
    ↓
Returns safe default value
    ↓
App uses fallback instead
```

### Basic Fallback Example

```javascript
const state = ReactiveUtils.state({ 
  userId: 123,
  userName: null
});

safeEffect(() => {
  // Try to fetch user name
  const user = fetchUser(state.userId);
  state.userName = user.name;  // Might fail
}, {
  errorBoundary: {
    retry: false,
    
    fallback: (error, context) => {
      // Return a safe default
      state.userName = 'Guest';  // ✅ Safe fallback
      console.log('Using fallback name: Guest');
    }
  }
});
```

**What happens:**

1. Effect tries to fetch user
2. Fetch fails (network error, invalid ID, etc.)
3. Fallback function runs
4. `userName` is set to `'Guest'` instead
5. App continues working with fallback data

### Fallback with UI Updates

```javascript
const state = ReactiveUtils.state({ 
  profileImage: null,
  imageLoaded: false
});

safeEffect(() => {
  // Try to load profile image
  const img = new Image();
  img.src = state.profileImage;
  img.onload = () => state.imageLoaded = true;
  img.onerror = () => {
    throw new Error('Image failed to load');
  };
}, {
  errorBoundary: {
    maxRetries: 2,
    
    fallback: (error) => {
      // Use default avatar instead
      state.profileImage = '/images/default-avatar.png';
      console.log('Using default avatar');
    },
    
    onError: (error, context) => {
      if (!context.willRetry) {
        console.log('Image load failed, showing default');
      }
    }
  }
});
```

### Fallback vs onError

**Key Difference:**

- **`onError`**: Called on **every** error (including during retries)
- **`fallback`**: Called **only once** after all retries are exhausted

```javascript
safeEffect(() => {
  dangerousOperation();
}, {
  errorBoundary: {
    maxRetries: 3,
    
    onError: (error, context) => {
      // Called 3 times (once per retry)
      console.log('Error occurred');
    },
    
    fallback: (error, context) => {
      // Called ONCE after all 3 retries fail
      console.log('All attempts failed, using fallback');
      return 'default-value';
    }
  }
});
```

 

## Deep Dive: Integration Patterns

Let's explore how `safeEffect()` works with other reactive features.

### Pattern 1: Safe Effects with Computed Properties

```javascript
const state = ReactiveUtils.state({ 
  items: [],
  total: 0
});

// Add computed property
computed(state, {
  itemCount: function() {
    return this.items.length;
  }
});

// Safe effect that uses computed
safeEffect(() => {
  console.log(`Processing ${state.itemCount} items`);
  
  state.items.forEach(item => {
    // This might fail if items have bad data
    state.total += item.price;
  });
}, {
  errorBoundary: {
    onError: () => {
      console.error('Failed to calculate total');
      state.total = 0;  // Reset to safe value
    }
  }
});
```

### Pattern 2: Safe Effects with Forms

```javascript
const userForm = form({
  email: '',
  password: ''
}, {
  validators: {
    email: validators.email(),
    password: validators.minLength(8)
  }
});

// Safely handle form submission side effects
safeEffect(() => {
  if (userForm.isValid && userForm.submitCount > 0) {
    // This might fail (network error, server error, etc.)
    sendToAnalytics({
      event: 'form_submitted',
      email: userForm.values.email
    });
  }
}, {
  errorBoundary: {
    retry: false,  // Don't retry analytics
    onError: (error) => {
      console.error('Analytics failed:', error);
      // Don't block user - analytics is non-critical
    }
  }
});
```

### Pattern 3: Safe Effects with Collections

```javascript
const todos = collection([
  { id: 1, title: 'Task 1', done: false },
  { id: 2, title: 'Task 2', done: false }
]);

// Safely sync to server
safeEffect(() => {
  // Every time todos change, sync to server
  todos.items.forEach(todo => {
    // This might fail (network, auth, etc.)
    syncToServer(todo);
  });
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 3,
    retryDelay: 1000,
    
    onError: (error, context) => {
      if (!context.willRetry) {
        showNotification('Failed to sync todos', 'error');
      }
    }
  }
});
```

### Pattern 4: Combining Multiple Safe Effects

Create a **coordinated error handling system** across multiple effects:

```javascript
const state = ReactiveUtils.state({
  user: null,
  preferences: null,
  notifications: []
});

// Shared error handler
const handleError = (componentName) => (error, context) => {
  console.error(`[${componentName}] Error:`, error.message);
  
  // Log to error tracking
  errorTracker.log({
    component: componentName,
    error: error,
    timestamp: Date.now()
  });
  
  // Show user notification if serious
  if (!context.willRetry) {
    showNotification(`${componentName} failed to load`, 'warning');
  }
};

// Safe effect 1: Load user
safeEffect(() => {
  if (state.user) {
    console.log('User:', state.user.name);
    updateUserDisplay(state.user);
  }
}, {
  errorBoundary: {
    onError: handleError('UserComponent'),
    fallback: () => {
      state.user = { name: 'Guest', role: 'visitor' };
    }
  }
});

// Safe effect 2: Load preferences
safeEffect(() => {
  if (state.preferences) {
    applyTheme(state.preferences.theme);
  }
}, {
  errorBoundary: {
    onError: handleError('PreferencesComponent'),
    fallback: () => {
      state.preferences = { theme: 'light' };  // Default theme
    }
  }
});

// Safe effect 3: Handle notifications
safeEffect(() => {
  state.notifications.forEach(notification => {
    displayNotification(notification);
  });
}, {
  errorBoundary: {
    onError: handleError('NotificationsComponent'),
    retry: false  // Don't retry notifications
  }
});
```

 

## Common Patterns

### Pattern 1: Try Operation, Fallback on Failure

```javascript
const state = ReactiveUtils.state({ 
  data: null,
  loading: false 
});

safeEffect(() => {
  state.loading = true;
  
  try {
    const result = fetchData();
    state.data = result;
  } catch (error) {
    throw error;  // Let error boundary handle it
  } finally {
    state.loading = false;
  }
}, {
  errorBoundary: {
    maxRetries: 2,
    retryDelay: 1000,
    
    fallback: () => {
      // Use cached data or default
      state.data = getCachedData() || getDefaultData();
    }
  }
});
```

### Pattern 2: Progressive Error Handling

```javascript
const state = ReactiveUtils.state({ 
  criticalData: null,
  optionalData: null 
});

safeEffect(() => {
  // Load critical data - MUST succeed
  state.criticalData = fetchCriticalData();
  
  // Try optional data - okay if it fails
  try {
    state.optionalData = fetchOptionalData();
  } catch (error) {
    console.log('Optional data unavailable');
    state.optionalData = null;
  }
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 5,  // Retry aggressively for critical data
    
    onError: (error) => {
      console.error('Critical data load failed:', error);
      showErrorScreen();
    }
  }
});
```

### Pattern 3: Conditional Error Handling

```javascript
const state = ReactiveUtils.state({ 
  environment: 'production',
  data: null 
});

safeEffect(() => {
  processData(state.data);
}, {
  errorBoundary: {
    onError: (error, context) => {
      // Different handling based on environment
      if (state.environment === 'development') {
        // Show detailed error in dev
        console.error('Detailed error:', error);
        debugger;  // Pause for debugging
      } else {
        // Log silently in production
        errorTracker.log(error);
        showGenericErrorMessage();
      }
    }
  }
});
```

 

## Real-World Examples

### Example 1: Dashboard with Multiple Data Sources

```javascript
const dashboard = state({
  userStats: null,
  salesData: null,
  analytics: null,
  errors: []
});

// Each data source has its own safe effect
// If one fails, others keep working

// Load user stats
safeEffect(() => {
  dashboard.userStats = fetchUserStats();
  updateUserStatsChart(dashboard.userStats);
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 3,
    retryDelay: 2000,
    
    onError: (error) => {
      dashboard.errors.push('User stats unavailable');
    },
    
    fallback: () => {
      dashboard.userStats = { visits: 0, signups: 0 };
    }
  }
});

// Load sales data
safeEffect(() => {
  dashboard.salesData = fetchSalesData();
  updateSalesChart(dashboard.salesData);
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 3,
    retryDelay: 2000,
    
    onError: (error) => {
      dashboard.errors.push('Sales data unavailable');
    },
    
    fallback: () => {
      dashboard.salesData = { total: 0, orders: 0 };
    }
  }
});

// Load analytics
safeEffect(() => {
  dashboard.analytics = fetchAnalytics();
  updateAnalyticsChart(dashboard.analytics);
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 3,
    retryDelay: 2000,
    
    onError: (error) => {
      dashboard.errors.push('Analytics unavailable');
    },
    
    fallback: () => {
      dashboard.analytics = { pageViews: 0, sessions: 0 };
    }
  }
});

// Even if 2 out of 3 data sources fail,
// the dashboard still shows available data! ✅
```

### Example 2: Real-Time Chat Application

```javascript
const chat = state({
  messages: [],
  connected: false,
  typingUsers: []
});

// Safe effect for WebSocket connection
safeEffect(() => {
  if (!chat.connected) return;
  
  // This might fail if network drops
  chat.messages.forEach(message => {
    if (!message.sent) {
      sendToServer(message);
      message.sent = true;
    }
  });
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 10,  // Keep trying for important messages
    retryDelay: 3000,
    
    onError: (error, context) => {
      if (context.attempt === 1) {
        showNotification('Connection issue. Retrying...', 'warning');
      }
      
      if (!context.willRetry) {
        showNotification('Message failed to send', 'error');
        // Mark message as failed
        chat.messages.forEach(msg => {
          if (!msg.sent) msg.failed = true;
        });
      }
    }
  }
});

// Safe effect for typing indicators
safeEffect(() => {
  // Less critical - don't retry
  updateTypingIndicators(chat.typingUsers);
}, {
  errorBoundary: {
    retry: false,
    onError: () => {
      console.log('Typing indicators unavailable');
      // Fail silently - not critical
    }
  }
});
```

### Example 3: E-Commerce Product Page

```javascript
const product = state({
  details: null,
  reviews: null,
  recommendations: null,
  inStock: false
});

// Critical: Product details MUST load
safeEffect(() => {
  const data = fetchProductDetails(productId);
  product.details = data.details;
  product.inStock = data.inStock;
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 5,
    retryDelay: 2000,
    
    onError: (error, context) => {
      if (!context.willRetry) {
        // Show error page if product can't load
        showErrorPage('Product not available');
      }
    }
  }
});

// Nice-to-have: Reviews can fail gracefully
safeEffect(() => {
  product.reviews = fetchProductReviews(productId);
}, {
  errorBoundary: {
    retry: false,
    
    fallback: () => {
      product.reviews = [];
      showMessage('Reviews temporarily unavailable');
    }
  }
});

// Optional: Recommendations are totally optional
safeEffect(() => {
  product.recommendations = fetchRecommendations(productId);
}, {
  errorBoundary: {
    retry: false,
    
    fallback: () => {
      product.recommendations = [];
      // Fail silently - user doesn't need to know
    }
  }
});
```

 

## Summary

### Key Takeaways

✅ **`safeEffect()` wraps effects in an error boundary** - Errors are caught and handled instead of crashing your app

✅ **Automatic retry logic** - Failed operations can retry automatically with configurable delay and max attempts

✅ **Fallback values** - Provide safe defaults when operations permanently fail

✅ **Isolated failures** - One effect failing doesn't affect other effects

✅ **Better user experience** - Apps stay functional even when parts fail

✅ **Clean error handling** - No need for try-catch blocks everywhere

✅ **Development visibility** - Errors are logged clearly with context

### When to Use `safeEffect()`

**Use `safeEffect()` when:**

- Working with external APIs that might fail
- Processing user data that might be invalid
- Handling network operations
- Dealing with third-party libraries
- Building dashboards with multiple data sources
- Creating resilient, production-ready applications

**Use regular `effect()` when:**

- You want errors to propagate (for debugging)
- The operation is simple and unlikely to fail
- You're in development and want to see raw errors
- Performance is absolutely critical (minimal overhead difference though)

### Quick Reference

```javascript
// Basic usage
safeEffect(() => {
  // Your effect code
});

// With error handler
safeEffect(() => {
  // Your effect code
}, {
  errorBoundary: {
    onError: (error, context) => {
      console.error(error);
    }
  }
});

// With retries
safeEffect(() => {
  // Your effect code
}, {
  errorBoundary: {
    retry: true,
    maxRetries: 3,
    retryDelay: 1000
  }
});

// With fallback
safeEffect(() => {
  // Your effect code
}, {
  errorBoundary: {
    fallback: (error) => {
      return 'safe-default-value';
    }
  }
});

// Full configuration
safeEffect(() => {
  // Your effect code
}, {
  errorBoundary: {
    onError: (error, context) => { /* handle */ },
    fallback: (error, context) => { /* return default */ },
    retry: true,
    maxRetries: 3,
    retryDelay: 1000
  }
});
```

 

**That's `safeEffect()`!** Your safety net for building resilient reactive applications. 🎉
