[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# ErrorBoundary

A class for creating error boundaries that isolate and handle errors in reactive effects and watchers.

## Quick Start (30 seconds)

```javascript
// Create error boundary
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error('Caught error:', error.message);
  },
  retry: true,
  maxRetries: 3
});

// Wrap a function that might fail
const safeFetch = boundary.wrap(async () => {
  const response = await fetch('/api/data');
  return response.json();
}, { type: 'fetch' });

// Call it - errors are caught and handled
const result = await safeFetch();
```

**The magic:** `ErrorBoundary` **isolates errors** so one failing function doesn't crash your entire application!

 

## What is ErrorBoundary?

`ErrorBoundary` is a **class** that creates error boundaries for wrapping functions with automatic error handling, retry logic, and fallback values.

Simply put: **It's a safety net that catches errors from risky functions.**

Think of it like this:
- You have functions that might fail (API calls, parsing, etc.)
- Without error boundaries, one error can break everything
- With error boundaries, errors are caught and handled gracefully
- Your app continues running even when things go wrong

 

## Syntax

```javascript
// Access the class
const ErrorBoundary = ReactiveUtils.ErrorBoundary;

// Create instance
const boundary = new ErrorBoundary(options);

// Wrap functions
const safeFunction = boundary.wrap(riskyFunction, context);
```

**Parameters:**
- `options` (optional) - Configuration object for the boundary

**Returns:**
- A new ErrorBoundary instance with `wrap()` method

 

## Why Does This Exist?

### The Challenge Without Error Boundaries

When errors occur in reactive code, they can break the entire system:

```javascript
const state = ReactiveUtils.state({ count: 0, data: null });

// Risky effect - might throw error
effect(() => {
  // What if this fails?
  const parsed = JSON.parse(state.data);
  console.log('Parsed:', parsed);
});
// TypeError: Cannot read property 'parse' of null
// 💥 Effect crashes, stops running forever

// Another effect that's perfectly fine
effect(() => {
  console.log('Count:', state.count);
});
// This effect also stops! 💥

// State updates cause cascading failures
state.count = 5; // Nothing happens anymore
```

At first glance, it might seem like just one error. But the consequences ripple.

**What's the Real Issue?**

```
Effect 1 runs
      ↓
Throws error
      ↓
Effect system crashes
      ↓
All other effects stop
      ↓
Entire reactivity breaks 💥
      ↓
App becomes unresponsive
```

**Problems:**
❌ One error breaks all effects  
❌ No error isolation  
❌ No retry mechanism  
❌ No graceful degradation  
❌ Hard to recover from errors  
❌ User experience suffers  

### The Solution with ErrorBoundary

With ErrorBoundary, errors are isolated and handled:

```javascript
const state = ReactiveUtils.state({ count: 0, data: null });

// Create error boundary
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error('Effect error:', error.message);
  },
  fallback: () => {
    return { default: 'value' };
  }
});

// Wrap risky effect
effect(boundary.wrap(() => {
  const parsed = JSON.parse(state.data);
  console.log('Parsed:', parsed);
}, { type: 'parse' }));
// Error caught! Logs: Effect error: Cannot read property...
// Returns fallback value

// Other effects continue working!
effect(() => {
  console.log('Count:', state.count);
});
// Still works! ✓

state.count = 5;
// Logs: Count: 5 ✓
```

**What just happened?**

```
Effect 1 runs
      ↓
Throws error
      ↓
ErrorBoundary catches it
      ↓
Calls onError handler
      ↓
Returns fallback value
      ↓
Other effects continue normally ✨
      ↓
App stays responsive
```

**Benefits:**
✅ Errors are isolated  
✅ Other effects keep running  
✅ Automatic retry logic available  
✅ Fallback values for graceful degradation  
✅ Detailed error context  
✅ App remains stable  

 

## Mental Model

Think of `ErrorBoundary` like **circuit breakers in a house**:

### Without ErrorBoundary (No Circuit Breakers)
```
House Electrical System
├─ Kitchen appliance shorts out ⚡
      ↓
Entire house loses power 💥
├─ Living room (no power)
├─ Bedroom (no power)
└─ Bathroom (no power)

One problem = everything stops!
```

### With ErrorBoundary (Circuit Breakers)
```
House Electrical System
├─ Kitchen appliance shorts out ⚡
      ↓
Kitchen circuit breaker trips 🔌
      ↓
✓ Kitchen isolated (safe)
├─ ✓ Living room (still powered)
├─ ✓ Bedroom (still powered)
└─ ✓ Bathroom (still powered)

One problem = isolated safely! ✨
```

**Key insight:** Just like circuit breakers prevent one electrical problem from shutting down your entire house, ErrorBoundary prevents one code error from crashing your entire app.

 

## How Does It Work?

### Under the Hood

The ErrorBoundary class wraps functions in try-catch blocks with additional features:

```javascript
// Simplified implementation
class ErrorBoundary {
  constructor(options = {}) {
    this.onError = options.onError || ((error) => {
      console.error('Error:', error);
    });
    this.fallback = options.fallback;
    this.retry = options.retry !== false; // Default true
    this.maxRetries = options.maxRetries || 3;
    this.retryDelay = options.retryDelay || 0;
  }
  
  wrap(fn, context = {}) {
    let retries = 0;
    
    return (...args) => {
      const attempt = () => {
        try {
          return fn(...args);
        } catch (error) {
          retries++;
          
          const shouldRetry = this.retry && retries < this.maxRetries;
          
          // Call error handler
          this.onError(error, {
            ...context,
            attempt: retries,
            maxRetries: this.maxRetries,
            willRetry: shouldRetry
          });
          
          // Retry if configured
          if (shouldRetry) {
            if (this.retryDelay > 0) {
              setTimeout(attempt, this.retryDelay);
            } else {
              return attempt();
            }
          } else if (this.fallback) {
            return this.fallback(error, context);
          }
        }
      };
      
      return attempt();
    };
  }
}
```

**What's happening:**

```
1️⃣ Create ErrorBoundary with options
        ↓
2️⃣ Wrap risky function
        ↓
3️⃣ Call wrapped function
        ↓
4️⃣ Try to execute
        ↓
5️⃣ If error: catch it
        ↓
6️⃣ Call onError handler
        ↓
7️⃣ Retry if configured
        ↓
8️⃣ Or return fallback
        ↓
9️⃣ Function continues safely ✨
```

 

## Basic Usage

### Example 1: Simple Error Boundary

```javascript
// Create error boundary
const boundary = new ErrorBoundary({
  onError: (error) => {
    console.error('Caught error:', error.message);
  }
});

// Wrap a function that might fail
const riskyFunction = boundary.wrap(() => {
  throw new Error('Something went wrong!');
});

// Call it - error is caught
riskyFunction();
// Logs: Caught error: Something went wrong!
// Function returns undefined (no fallback)
```

**What's happening?**
1. Create boundary with error handler
2. Wrap function that throws error
3. Call wrapped function
4. Error is caught and logged
5. Function returns gracefully

 

### Example 2: With Fallback Value

```javascript
const boundary = new ErrorBoundary({
  onError: (error) => {
    console.error('Error:', error.message);
  },
  fallback: () => {
    return { status: 'error', data: null };
  }
});

const fetchData = boundary.wrap(async () => {
  const response = await fetch('/api/nonexistent');
  if (!response.ok) {
    throw new Error('Failed to fetch');
  }
  return response.json();
});

const result = await fetchData();
// Logs: Error: Failed to fetch
console.log(result);
// { status: 'error', data: null }
```

**What's happening?**
- Function throws error
- Error is caught and logged
- Fallback function returns safe default value
- App continues with fallback data

 

### Example 3: With Retry Logic

```javascript
let attemptCount = 0;

const boundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  onError: (error, context) => {
    console.log(`Attempt ${context.attempt}/${context.maxRetries}`);
    console.log('Will retry:', context.willRetry);
  }
});

const flaky = boundary.wrap(() => {
  attemptCount++;
  console.log('Attempt number:', attemptCount);
  
  if (attemptCount < 3) {
    throw new Error('Still failing...');
  }
  
  return 'Success!';
});

const result = flaky();
// Logs:
// Attempt number: 1
// Attempt 1/3
// Will retry: true
// Attempt number: 2
// Attempt 2/3
// Will retry: true
// Attempt number: 3

console.log(result); // Success!
```

**What's happening?**
- First two attempts fail
- ErrorBoundary retries automatically
- Third attempt succeeds
- Result is returned

 

## Constructor Options

### Option: `onError`

Error handler callback:

```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error('Error occurred:', error.message);
    console.log('Context:', context);
    
    // Send to error tracking service
    sendToSentry(error, context);
  }
});
```

**Parameters:**
- `error` - The error object that was thrown
- `context` - Context object with error details

**Context object:**
```javascript
{
  type: 'fetch',           // Custom context from wrap()
  attempt: 2,              // Current attempt number
  maxRetries: 3,           // Maximum retries configured
  willRetry: true          // Whether another retry will happen
}
```

 

### Option: `fallback`

Fallback value function:

```javascript
const boundary = new ErrorBoundary({
  fallback: (error, context) => {
    if (context.type === 'parse') {
      return { default: 'value' };
    }
    
    if (context.type === 'fetch') {
      return { cached: true, data: [] };
    }
    
    return null;
  }
});
```

**Parameters:**
- `error` - The error object
- `context` - Context from wrap()

**Returns:**
- Value to return instead of the error

 

### Option: `retry`

Enable/disable retry logic:

```javascript
// Retry enabled (default)
const boundary1 = new ErrorBoundary({
  retry: true
});

// Retry disabled
const boundary2 = new ErrorBoundary({
  retry: false,
  fallback: () => 'default'
});
```

**Default:** `true`

 

### Option: `maxRetries`

Maximum number of retry attempts:

```javascript
const boundary = new ErrorBoundary({
  retry: true,
  maxRetries: 5,  // Try up to 5 times
  onError: (error, context) => {
    console.log(`Attempt ${context.attempt}/${context.maxRetries}`);
  }
});
```

**Default:** `3`

 

### Option: `retryDelay`

Delay between retry attempts (milliseconds):

```javascript
const boundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  retryDelay: 1000,  // Wait 1 second between retries
  onError: (error, context) => {
    console.log('Waiting before retry...');
  }
});
```

**Default:** `0` (immediate retry)

 

## Instance Methods

### `wrap(fn, context)`

Wraps a function with error handling:

```javascript
const boundary = new ErrorBoundary({ /* options */ });

const safeFunction = boundary.wrap(
  () => {
    // Your code here
  },
  { type: 'custom', operation: 'parse' }
);
```

**Parameters:**
- `fn` - Function to wrap (can be sync or async)
- `context` - Optional context object for error tracking

**Returns:**
- Wrapped function that catches errors

**See detailed documentation:** [wrap() method](./wrap.md)

 

## Common Patterns

### Pattern 1: Shared Boundary for Effects

```javascript
// Create one boundary for all effects
const effectBoundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error(`[Effect ${context.name}] Error:`, error.message);
  },
  fallback: () => undefined
});

const state = ReactiveUtils.state({ count: 0, name: 'Alice', data: null });

// Wrap all effects with same boundary
effect(effectBoundary.wrap(() => {
  console.log('Count:', state.count);
}, { name: 'count-logger' }));

effect(effectBoundary.wrap(() => {
  const parsed = JSON.parse(state.data); // Might fail
  console.log('Parsed:', parsed);
}, { name: 'data-parser' }));

effect(effectBoundary.wrap(() => {
  console.log('Name length:', state.name.length);
}, { name: 'name-length' }));

// If one fails, others continue
state.data = 'invalid json'; // Parse error caught, others work
state.count = 5;              // All effects run (except broken one)
```

 

### Pattern 2: Separate Boundaries for Different Operations

```javascript
// API boundary - with retry
const apiBoundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  retryDelay: 1000,
  onError: (error, context) => {
    console.error('API Error:', error.message);
    trackError('api', error);
  }
});

// Parse boundary - no retry, with fallback
const parseBoundary = new ErrorBoundary({
  retry: false,
  fallback: () => ({ error: 'Invalid data' }),
  onError: (error, context) => {
    console.error('Parse Error:', error.message);
    trackError('parse', error);
  }
});

// Use appropriate boundary for each operation
const fetchUser = apiBoundary.wrap(async () => {
  const response = await fetch('/api/user');
  return response.json();
}, { type: 'fetch' });

const parseConfig = parseBoundary.wrap((configString) => {
  return JSON.parse(configString);
}, { type: 'parse' });
```

 

### Pattern 3: Error Boundary Per Component

```javascript
class TodoList {
  constructor() {
    // Component-specific error boundary
    this.boundary = new ErrorBoundary({
      onError: (error, context) => {
        console.error(`[TodoList] ${context.operation}:`, error.message);
        this.showErrorMessage(error.message);
      },
      fallback: (error, context) => {
        if (context.operation === 'load') {
          return [];
        }
        return null;
      }
    });
    
    this.state = state({ todos: [], loading: false });
  }
  
  async load() {
    this.state.loading = true;
    
    const safeFetch = this.boundary.wrap(async () => {
      const response = await fetch('/api/todos');
      return response.json();
    }, { operation: 'load' });
    
    this.state.todos = await safeFetch();
    this.state.loading = false;
  }
  
  showErrorMessage(message) {
    console.log(`UI: Error - ${message}`);
  }
}
```

 

### Pattern 4: Global Error Tracking

```javascript
// Global error tracker
const errorStats = {
  total: 0,
  byType: {}
};

const trackingBoundary = new ErrorBoundary({
  onError: (error, context) => {
    // Track statistics
    errorStats.total++;
    const type = context.type || 'unknown';
    errorStats.byType[type] = (errorStats.byType[type] || 0) + 1;
    
    // Log to console
    console.error(`[${type}] Error:`, error.message);
    
    // Send to monitoring service
    if (typeof sendToMonitoring !== 'undefined') {
      sendToMonitoring({
        error: error.message,
        stack: error.stack,
        context: context,
        timestamp: Date.now()
      });
    }
  }
});

// Use throughout app
const fetchData = trackingBoundary.wrap(async () => {
  // ...
}, { type: 'api', endpoint: '/data' });

const parseData = trackingBoundary.wrap((data) => {
  // ...
}, { type: 'parse', format: 'json' });

// Check stats
console.log('Error stats:', errorStats);
// { total: 5, byType: { api: 3, parse: 2 } }
```

 

### Pattern 5: Conditional Retry Based on Error

```javascript
const smartBoundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  onError: (error, context) => {
    // Don't retry client errors (4xx)
    if (error.status >= 400 && error.status < 500) {
      context.maxRetries = 0; // Stop retrying
      console.log('Client error - not retrying');
    }
    
    // Do retry server errors (5xx)
    if (error.status >= 500) {
      console.log(`Server error - retry ${context.attempt}/${context.maxRetries}`);
    }
  },
  fallback: (error) => {
    return { error: error.message, data: null };
  }
});
```

 

## Summary

### Key Takeaways

✅ **`ErrorBoundary` is a class** for creating error isolation boundaries  
✅ **Catches and handles errors** preventing cascading failures  
✅ **Supports retry logic** with configurable attempts and delays  
✅ **Fallback values** for graceful degradation  
✅ **Custom error handlers** for logging and tracking  
✅ **Context tracking** provides detailed error information  
✅ **Wrap functions** with `boundary.wrap(fn, context)`  

### Quick Reference

```javascript
// Create error boundary
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error('Error:', error.message, context);
  },
  fallback: (error, context) => {
    return 'safe default value';
  },
  retry: true,
  maxRetries: 3,
  retryDelay: 1000
});

// Wrap functions
const safeFunction = boundary.wrap(
  () => {
    // Risky code here
  },
  { type: 'operation', name: 'my-operation' }
);

// Call wrapped function - errors are caught
const result = safeFunction();
```

### One-Line Rule

> **Use `ErrorBoundary` to isolate errors in reactive code—one failing function won't crash your entire application, and you get automatic retry logic and fallback values.**

 

**Next Steps:**
- Learn about [`wrap()`](./wrap.md) method in detail
- Explore [error handling patterns](./error-patterns.md)
- Read about [safeEffect()](./safeEffect.md) for wrapped effects